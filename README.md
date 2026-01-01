# 加密貨幣模型 Ensemble GUI - Hugging Face 資料源

改進的 K 線資料加載邏輯,支援從 Hugging Face 動態選擇幣種進行預測分析。

## 新增功能

### 1. Hugging Face 資料源整合

- **數據集**: `zongowo111/v2-crypto-ohlcv-data`
- **自動下載**: 從 Hugging Face 直接下載 parquet 格式 K 線資料
- **動態選擇**: 在 GUI 啟動時選擇幣種和時間框架

### 2. 支援的幣種 (22 種)

```
AAVEUSDT, ADAUSDT, ALGOUSDT, ARBUSDT, ATOMUSDT,
AVAXUSDT, BCHUSDT, BNBUSDT, BTCUSDT, DOGEUSDT,
DOTUSDT, ETCUSDT, ETHUSDT, FILUSDT, LINKUSDT,
LTCUSDT, MATICUSDT, NEARUSDT, OPUSDT, SOLUSDT,
UNIUSDT, XRPUSDT
```

### 3. 支援的時間框架

- 15 分鐘 K 線 (15m)
- 1 小時 K 線 (1h)

## 資料結構

```
Hugging Face 數據集結構:
https://huggingface.co/datasets/zongowo111/v2-crypto-ohlcv-data

├── klines/
│   ├── BTCUSDT/
│   │   ├── BTC_15m.parquet  (Bitcoin 15分鐘K線)
│   │   └── BTC_1h.parquet   (Bitcoin 1小時K線)
│   ├── ETHUSDT/
│   │   ├── ETH_15m.parquet
│   │   └── ETH_1h.parquet
│   └── ... (其他幣種)
```

## 安裝依賴

```bash
pip install pyqt5 pandas numpy lightgbm catboost scikit-learn matplotlib huggingface-hub
```

## 使用方式

### 1. 啟動 GUI

```bash
python model_ensemble_gui.py
```

### 2. 加載資料 (新改進流程)

1. **選擇幣種**: 在「幣種」下拉選單中選擇要測試的加密貨幣
   - 預設為 BTCUSDT (比特幣)
   - 可選 22 種主流幣種

2. **選擇時間框架**: 在「時間框」下拉選單中選擇
   - 15m: 15 分鐘 K 線 (適合短期分析)
   - 1h: 1 小時 K 線 (適合中期分析)

3. **按下「從 Hugging Face 加載」按鈕**
   - 自動從 Hugging Face 下載對應的 parquet 文件
   - 自動驗證資料完整性
   - 顯示已加載的 K 線數量

### 3. GUI 功能介紹

#### 標籤頁 1: 資料預覽
- 顯示加載的 K 線資料表格
- 包含時間戳、Open、High、Low、Close、Volume

#### 標籤頁 2: K 線圖
- 視覺化顯示 K 線圖表
- 預設顯示最後 1000 根 K 線
- 綠色 K 線表示上漲,紅色表示下跌

#### 標籤頁 3: 模型訓練
- **Lookback 週期**: 特徵工程使用的回顧期數 (預設 14)
- **測試比例**: 訓練/測試資料分割比例 (預設 20%)
- 支援 LightGBM 和 CatBoost 兩個模型
- 自動計算 Ensemble 集成預測結果
- 訓練完成後可保存模型

#### 標籤頁 4: 預測
- 加載已訓練的模型
- 執行完整資料集預測
- 匯出預測結果為 CSV

## 代碼架構

### 關鍵類別

#### DataLoaderWorker (QThread)
```python
# 負責從 Hugging Face 下載資料的工作線程
- 參數: symbol (幣種), timeframe (時間框)
- 自動構建文件路徑: klines/{SYMBOL}/{SYMBOL_15m|1h}.parquet
- 驗證必要欄位: timestamp, open, high, low, close, volume
```

#### FeatureEngineer
```python
# 特徵工程類
- engineer_features(): 生成 15+ 個技術指標特徵
  * SMA (簡單移動平均): 5, 10, 20 期
  * RSI (相對強度指數): 5, 10, 20 期
  * 波動性 (Volatility)
  * 收益率 (Returns)
  * 高低差占比
  * 開收差占比
```

#### ModelEnsembleGUI (QMainWindow)
```python
# 主 GUI 窗口類
- init_ui(): 初始化所有 UI 元素
- load_hf_data(): 加載 Hugging Face 資料
- start_training(): 訓練 LightGBM 和 CatBoost
- make_predictions(): 執行集成預測
```

## 資料流程圖

```
Hugging Face 資料集
        |
        v
DataLoaderWorker (多線程下載)
        |
        v
DataFrame (timestamp, open, high, low, close, volume)
        |
        v
FeatureEngineer (特徵生成)
        |
        v
X, y (特徵矩陣, 標籤向量)
        |
        v
Train-Test Split
        |
        +---> LightGBM 訓練 --+
        |                     |
        +---> CatBoost 訓練 --+---> Ensemble 預測
        |
        v
預測結果 CSV 匯出
```

## 特徵集

### 基礎特徵 (5 個)
1. **returns**: 收益率 (close 日日百分比變化)
2. **high_low**: (high - low) / close
3. **close_open**: (close - open) / open
4. **volume**: 交易量

### 移動平均線特徵 (6 個)
- sma_5, sma_10, sma_20: 簡單移動平均線

### RSI 特徵 (3 個)
- rsi_5, rsi_10, rsi_20: 相對強度指數

### 波動性特徵 (1 個)
- volatility: 14 期收益率標準差

### 目標變量
- **target**: 二分類標籤
  - 1: 下一根 K 線收盤價 > 當前收盤價 (上漲)
  - 0: 下一根 K 線收盤價 <= 當前收盤價 (下跌)

## 模型參數

### LightGBM
```python
n_estimators=100
max_depth=7
learning_rate=0.05
random_state=42
```

### CatBoost
```python
iterations=100
max_depth=7
learning_rate=0.05
random_state=42
verbose=False
```

## 預測輸出

```csv
timestamp,lgb_prediction,lgb_probability,cb_prediction,cb_probability,ensemble_prediction,ensemble_probability
2025-12-30 10:00:00,1,0.625,1,0.612,1,0.618
2025-12-30 10:15:00,0,0.445,0,0.451,0,0.448
...
```

- **_prediction**: 0 (下跌) 或 1 (上漲)
- **_probability**: 預測概率 (0-1)
- **ensemble_***: 兩個模型結果平均

## 疑難排解

### 1. Hugging Face 連線問題
```
錯誤: Connection timeout
解決: 檢查網路連線,或設置 huggingface_hub 代理
```

### 2. 缺少依賴
```bash
# 完整安裝
pip install --upgrade pyqt5 lightgbm catboost huggingface-hub
```

### 3. 記憶體不足
- 選擇較小的資料集 (例如 15m 而非 1h)
- 減少訓練樣本量

## 改進方向

1. **多時間框架分析**: 同時分析 15m 和 1h 進行確認
2. **動態特徵調整**: 根據市場條件調整特徵權重
3. **實時預測**: 集成 WebSocket 進行實時 K 線推送
4. **風險管理**: 加入止損和持倉管理
5. **性能優化**: 使用 GPU 加速模型訓練

## 許可證

MIT License

## 貢獻

Pull Requests 歡迎!

---

**最後更新**: 2026-01-01  
**作者**: caizongxun  
**版本**: 2.0 (Hugging Face 資料源版本)
