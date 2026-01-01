# 快速開始指南

## 安裝步驟

### 1. 克隆倉庫
```bash
git clone https://github.com/caizongxun/C2-crypto-prediction.git
cd C2-crypto-prediction
```

### 2. 建立虛擬環境 (推薦)
```bash
# Windows
python -m venv venv
venv\Scripts\activate

# macOS/Linux
python3 -m venv venv
source venv/bin/activate
```

### 3. 安裝依賴
```bash
pip install -r requirements.txt
```

或單獨安裝:
```bash
pip install pyqt5 pandas numpy lightgbm catboost scikit-learn matplotlib huggingface-hub
```

## 運行 GUI

### 第一次運行
```bash
python model_ensemble_gui.py
```

窗口將打開，顯示以下界面:
- **數據源**: 選擇幣種和時間框架
- **4 個標籤頁**: 數據預覽、K線圖、模型訓練、預測

## 完整工作流程

### 步驟 1: 選擇數據

1. 打開 GUI
2. 在「幣種」下拉菜單中選擇 (例如: BTCUSDT)
3. 在「時間框」下拉菜單中選擇 (15m 或 1h)
4. **點擊「從 Hugging Face 加載」按鈕**

```
✓ 自動從 Hugging Face 下載
✓ 自動驗證數據完整性
✓ 自動轉換時間戳格式
```

**預期結果:**
```
狀態欄: "已加載 219643 筆數據，處理後 219642 筆數據"
```

### 步驟 2: 查看數據

**標籤頁 1 - 數據預覽:**
- 顯示前 20 筆 K 線數據的表格
- 列: 時間、Open、High、Low、Close、Volume

**標籤頁 2 - K 線圖:**
- 可視化顯示最後 1000 根 K 線
- 綠色蠟燭 = 上漲 (Close > Open)
- 紅色蠟燭 = 下跌 (Close <= Open)

### 步驟 3: 訓練模型

**標籤頁 3 - 模型訓練:**

1. **設置訓練參數** (可選):
   - Lookback 週期: 14 (預設) - 用於特徵工程的回顧期數
   - 測試比例: 20% (預設) - 訓練/測試數據分割

2. **點擊「開始訓練」按鈕**:
   ```
   進度條: 0% → 100%
   實時日誌:
   - [時間] 開始訓練模型...
   - [時間] 訓練 LightGBM 模型...
   - [時間] 訓練 CatBoost 模型...
   - [時間] 訓練完成 - LightGBM: 0.5257, CatBoost: 0.5256, Ensemble: 0.5310
   ```

3. **保存模型**:
   - 訓練完成後點擊「保存訓練的模型"
   - 模型自動保存到 `trained_models/` 目錄
   - 命名格式: `{model}_{symbol}_{timeframe}_{timestamp}.pkl`

### 步驟 4: 執行預測

**標籤頁 4 - 預測:**

1. **加載模型**:
   - 點擊「加載已訓練的模型」
   - 選擇 LightGBM 模型文件
   - 自動查找對應的 CatBoost 和 Scaler 文件

2. **運行預測**:
   - 點擊「執行預測」
   - 對完整數據集進行預測
   - 結果包括:
     - LightGBM 預測
     - CatBoost 預測
     - Ensemble 集成預測
     - 各模型預測概率

3. **導出結果**:
   - 點擊「導出預測結果為 CSV"
   - 選擇保存位置
   - CSV 包含所有預測結果和概率

## 數據流示例

```
入力: 選擇 BTCUSDT, 15m
  |
  v
Hugging Face
klines/BTCUSDT/BTC_15m.parquet
  |
  v
DataFrame (
  timestamp: 219643 筆
  open, high, low, close, volume
)
  |
  v
Feature Engineering
生成 15+ 個技術指標特徵
  |
  v
Train-Test Split (80-20)
  |
  v
LightGBM + CatBoost 訓練
  |
  v
Ensemble 集成 (平均概率)
  |
  v
輸出: prediction_results.csv
列: timestamp, lgb_pred, lgb_prob, 
    cb_pred, cb_prob, ensemble_pred, ensemble_prob
```

## 輸出文件說明

### 1. 訓練的模型
位置: `trained_models/`

```
lightgbm_BTC_15m_20260101_075310.pkl  (LightGBM 模型)
catboost_BTC_15m_20260101_075310.pkl  (CatBoost 模型)
scaler_BTC_15m_20260101_075310.pkl    (特徵縮放器)
```

### 2. 預測結果 CSV

```csv
timestamp,lgb_prediction,lgb_probability,cb_prediction,cb_probability,ensemble_prediction,ensemble_probability
2025-12-30 07:00:00,1,0.627,1,0.614,1,0.620
2025-12-30 07:15:00,0,0.442,0,0.448,0,0.445
2025-12-30 07:30:00,1,0.583,1,0.591,1,0.587
...
```

**列說明:**
- **timestamp**: K 線時間
- **lgb_prediction**: LightGBM 預測 (0=下跌, 1=上漲)
- **lgb_probability**: LightGBM 預測概率 (0-1)
- **cb_prediction**: CatBoost 預測 (0=下跌, 1=上漲)
- **cb_probability**: CatBoost 預測概率 (0-1)
- **ensemble_prediction**: 集成預測 (0=下跌, 1=上漲)
- **ensemble_probability**: 集成預測概率 (兩個模型概率的平均值)

## 常見問題

### Q1: 下載速度慢怎麼辦?
```bash
# Hugging Face 會自動緩存文件到 ~/.cache/huggingface/
# 第二次加載同一對幣種時會直接使用本地文件
```

### Q2: 可以加載不同的幣種嗎?
```
是的! 支持 22 種主流加密貨幣:
AVAX, ADA, ALGO, ARB, ATOM,
AVAX, BCH, BNB, BTC, DOGE,
DOT, ETC, ETH, FIL, LINK,
LTC, MATIC, NEAR, OP, SOL,
UNI, XRP
```

### Q3: 能同時訓練多個幣種的模型嗎?
```
可以! 但需要先關閉 GUI 重新啟動
訓練不同幣種時自動生成新的模型文件
```

### Q4: 預測準確率如何?
```
取決於市場條件和數據周期
典型的 Ensemble 準確率: 51-54%
建議結合其他技術指標和風險管理
```

## 性能參考

### 硬件要求
- CPU: 4 核以上
- RAM: 4GB 最小, 8GB 推薦
- 硬盤: 500MB (含 Hugging Face 緩存)

### 速度預期
- 數據下載: 10-30 秒 (取決於網路)
- 特徵工程: 5-10 秒
- 模型訓練: 20-60 秒
- 預測: 5-10 秒

## 進階配置

### 修改模型參數
在 `model_ensemble_gui.py` 中修改:

```python
# 第 160 行 LightGBM 參數
lgb_model = lgb.LGBMClassifier(
    n_estimators=150,      # 增加樹的數量
    max_depth=10,          # 增加樹的深度
    learning_rate=0.03,    # 降低學習率
)

# 第 174 行 CatBoost 參數
cb_model = CatBoostClassifier(
    iterations=150,
    max_depth=10,
    learning_rate=0.03,
)
```

### 自定義特徵
在 `FeatureEngineer.engineer_features()` 中添加:

```python
# 添加 MACD
df['macd'] = ...

# 添加 Bollinger Bands
df['bb_upper'] = ...
df['bb_lower'] = ...
```

## 疑難排解

### 錯誤: "No module named 'PyQt5'"
```bash
pip install PyQt5 --upgrade
```

### 錯誤: "Connection timeout from Hugging Face"
```bash
# 檢查網路連接
# 或使用代理設置
set HF_DATASETS_OFFLINE=0
```

### 錯誤: "缺少必要欄位"
```
確保選擇的幣種和時間框架在 Hugging Face 中存在
檢查數據集是否已更新
```

## 下一步

1. 實驗不同的參數組合
2. 測試多個幣種尋找最佳表現
3. 添加風險管理邏輯
4. 將預測結果集成到交易系統

---

祝你使用愉快! 有問題歡迎提交 Issue!
