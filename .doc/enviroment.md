# Bot Fund Investment - 開發環境設置指南

## 自動環境檢查

本專案包含自動檢查機制，用於確保開發環境正確配置。

### 環境檢查邏輯

**檢查項目**：\.vscode/settings.json\ 檔案

**檢查結果處理**：

#### 當 \.vscode/settings.json\ 檔案不存在時，請執行以下步驟：

1. **在專案資料夾中創建 \.vscode/settings.json\ 檔案**

2. **加入以下內容**：

\\\json
{
"python.defaultInterpreterPath": "D:\\\\software\\\\WinPython\\\\python-3.12.3.amd64\\\\python.exe",
"jupyter.kernels.filter": [
{
"path": "D:\\\\software\\\\WinPython\\\\python-3.12.3.amd64\\\\python.exe",
"type": "pythonEnvironment"
}
]
}
\\\

3. **註冊 Jupyter Kernel（重要！）**

即使設定了 \settings.json\，仍需註冊 IPython Kernel 才能在 Notebook 中使用：

\\\ash
D:\\software\\WinPython\\python-3.12.3.amd64\\python.exe -m pip install ipykernel
D:\\software\\WinPython\\python-3.12.3.amd64\\python.exe -m ipykernel install --user --name=winpython312 --display-name "Python 3.12.3 (WinPython)"
\\\

驗證 Kernel 是否註冊成功：

\\\ash
D:\\software\\WinPython\\python-3.12.3.amd64\\python.exe -m jupyter kernelspec list
\\\

應該會看到 \winpython312\ 出現在列表中。

4. **在 VS Code 中重新載入並選擇 Kernel**
   - 按 \Ctrl + Shift + P\
   - 執行 \Developer: Reload Window\
   - 開啟 \.ipynb\ 檔案
   - 點擊右上角 **Select Kernel**
   - 選擇 **Jupyter Kernel...** → **Python 3.12.3 (WinPython)**

5. \*\*設定 \.gitignore\*\*

確保 \.gitignore\ 檔案包含以下條目，避免本地設置被上傳到版本控制：

\\\
.vscode/settings.json
\\\

這樣可以保護你的本地 Python 環境路徑配置，使其不被提交到遠端 Repository。每個開發者可以根據其本地環境情況自行設置。

### 說明

- \python.defaultInterpreterPath\：指定 VS Code 使用的默認 Python 解釋器路徑
- \jupyter.kernels.filter\：配置 Jupyter 使用相同的 Python 環境
- **Kernel 註冊**：讓 Jupyter 能夠在 Notebook 中使用指定的 Python 環境

### 修改路徑

如果你的 Python 環境安裝在不同的位置，請將上述路徑替換為你的實際 WinPython 安裝路徑。例如：

\\\
D:\\your\\python\\installation\\path\\python.exe
\\\

### 常見問題

**Q: 為什麼設定了 \settings.json\ 還是無法選擇 Kernel？**

A: 因為 VS Code 的 Python 解釋器設定和 Jupyter Kernel 是兩個獨立的系統。\settings.json\ 只設定編輯器要用哪個 Python，但 Jupyter Notebook 需要透過 \ipykernel install\ 命令額外註冊才能使用。

**Q: 如何驗證 Kernel 是否正確註冊？**

A: 執行以下命令檢查已註冊的 Kernels：
\\\ash
D:\\software\\WinPython\\python-3.12.3.amd64\\python.exe -m jupyter kernelspec list
\\\

應該會看到 \winpython312\ 出現在列表中。
