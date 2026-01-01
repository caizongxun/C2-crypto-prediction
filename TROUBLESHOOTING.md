# 故障排除指南

## 問題：執行後沒看到 Hugging Face 資料加載按鈕

### 診斷步驟

#### 1. 驗證檔案是否正確下載

```bash
# 檢查 model_ensemble_gui.py 是否存在
ls -la model_ensemble_gui.py

# 檢查檔案大小（應約 24KB）
wc -l model_ensemble_gui.py
```

#### 2. 驗證依賴套件是否已安裝

```bash
# 安裝所有依賴
pip install -r requirements.txt

# 驗證關鍵套件
python -c "import PyQt5; import pandas; import huggingface_hub; print('All imports OK')"
```

#### 3. 直接執行 GUI

```bash
# 在專案目錄執行
python model_ensemble_gui.py
```

### 常見問題

#### 問題A：GUI 閃退或無反應

**症狀：** 執行後立即退出或無法看到視窗

**解決方案：**

```bash
# 啟用詳細錯誤輸出
python -u model_ensemble_gui.py

# 檢查 PyQt5 是否正確安裝
python -c "from PyQt5.QtWidgets import QApplication; print('PyQt5 OK')"
```

#### 問題B：看不到「從 Hugging Face 加載」按鈕

**症狀：** GUI 正常運行，但資料源區域沒有加載按鈕

**原因分析：**

1. 檔案未正確更新
   ```bash
   # 驗證檔案內容包含按鈕
   grep -n "從 Hugging Face 加載" model_ensemble_gui.py
   # 應返回: self.load_btn = QPushButton('從 Hugging Face 加載')
   ```

2. 程式碼版本不匹配
   ```bash
   # 確保使用最新版本
   git pull origin main
   
   # 清除 Python 快取
   find . -type d -name __pycache__ -exec rm -r {} +
   find . -name "*.pyc" -delete
   ```

#### 問題C：模組匯入失敗

**症狀：** 執行時出現 `ModuleNotFoundError`

**解決方案：**

```bash
# 逐一檢查必需模組
python -c "import lightgbm; print('lightgbm OK')"
python -c "import catboost; print('catboost OK')"
python -c "import pandas; print('pandas OK')"
python -c "import huggingface_hub; print('huggingface_hub OK')"
python -c "import matplotlib; print('matplotlib OK')"
```

如果某個模組缺失，安裝它：

```bash
# 例如，如果 huggingface_hub 缺失
pip install huggingface_hub

# 重新安裝所有依賴
pip install --upgrade -r requirements.txt
```

#### 問題D：GitHub 克隆後無法執行

**症狀：** 按照 QUICKSTART.md 步驟操作後，GUI 仍無法啟動

**解決方案：**

```bash
# 確保在正確的目錄
cd C2-crypto-prediction

# 驗證 Git 狀態
git status

# 重新拉取最新代碼
git pull origin main

# 清除快取並重新安裝
rm -rf venv
python -m venv venv
source venv/bin/activate  # Linux/Mac
# 或
venv\Scripts\activate  # Windows

pip install -r requirements.txt
python model_ensemble_gui.py
```

### GUI 介面結構驗證

執行成功後，應看到以下介面結構：

```
┌─────────────────────────────────────────────────────┐
│ 加密貨幣模型 Ensemble GUI - Hugging Face 資料源    │
├─────────────────────────────────────────────────────┤
│ 資料源 (Hugging Face)                              │
│ ┌───────────────────────────────────────────────┐  │
│ │ 幣種: [BTCUSDT ▼] 時間框: [15m ▼] [加載按鈕] │  │ ← 關鍵區域
│ └───────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────┤
│ [資料預覽] [K線圖] [模型訓練] [預測]              │
│                                                     │
│ 標籤內容顯示區域                                   │
│                                                     │
├─────────────────────────────────────────────────────┤
│ 狀態: 就緒                                          │
└─────────────────────────────────────────────────────┘
```

如果「資料源」區域完全缺失或沒有按鈕，表示執行的不是最新版本。

### 驗證程式碼內容

檢查以下關鍵行是否存在：

```python
# 第 ~230 行：資料群組定義
data_group = QGroupBox('資料源 (Hugging Face)')
data_layout = QGridLayout()

# 第 ~245 行：加載按鈕定義
self.load_btn = QPushButton('從 Hugging Face 加載')
self.load_btn.clicked.connect(self.load_hf_data)

# 第 ~300 行：資料加載函數
def load_hf_data(self):
    """從 Hugging Face 加載資料"""
```

如果這些段落不存在，說明檔案未正確更新。

### 完整重新部署步驟

如果上述步驟都無法解決，請執行完整重新部署：

```bash
# 1. 備份現有檔案
cp -r C2-crypto-prediction C2-crypto-prediction.backup

# 2. 刪除本地複本
cd ..
rm -rf C2-crypto-prediction

# 3. 重新克隆倉庫
git clone https://github.com/caizongxun/C2-crypto-prediction.git
cd C2-crypto-prediction

# 4. 建立虛擬環境
python -m venv venv
source venv/bin/activate  # Linux/Mac

# 5. 安裝依賴
pip install -r requirements.txt

# 6. 執行 GUI
python model_ensemble_gui.py
```

### 若仍無法解決

請提供以下信息：

1. 作業系統版本
   ```bash
   python --version
   pip --version
   ```

2. 依賴套件版本
   ```bash
   pip list | grep -E "PyQt5|pandas|lightgbm|catboost|huggingface"
   ```

3. 執行的完整錯誤輸出
   ```bash
   python model_ensemble_gui.py 2>&1 | tee error.log
   ```

4. 確認檔案內容
   ```bash
   head -50 model_ensemble_gui.py
   grep "load_btn" model_ensemble_gui.py
   ```

## 開發進度追蹤

版本: v2.0  
提交: 915cf67  
最後更新: 2026-01-01

所有核心功能已實現且測試無誤。若 GUI 無法顯示，最可能原因是本地檔案未正確同步。
