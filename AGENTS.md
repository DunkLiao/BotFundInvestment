# Bot Fund Investment - 開發環境設置指南

## 自動環境檢查

本專案包含自動檢查機制，用於確保開發環境正確配置。

### 環境檢查邏輯

**檢查項目**：`.vscode/settings.json` 檔案

**檢查結果處理**：

#### 當 `.vscode/settings.json` 檔案不存在時，請執行以下步驟：

1. **在專案資料夾中創建 `.vscode/settings.json` 檔案**

2. **加入以下內容**：

```json
{
  "python.defaultInterpreterPath": "D:\\software\\WinPython\\python-3.12.3.amd64\\python.exe",
  "jupyter.kernels.filter": [
    {
      "path": "D:\\software\\WinPython\\python-3.12.3.amd64\\python.exe",
      "type": "pythonEnvironment"
    }
  ]
}
```

3. **設定 `.gitignore`**

確保 `.gitignore` 檔案包含以下條目，避免本地設置被上傳到版本控制：

```
.vscode/settings.json
```

這樣可以保護你的本地 Python 環境路徑配置，使其不被提交到遠端 Repository。每個開發者可以根據其本地環境情況自行設置。

### 說明

- `python.defaultInterpreterPath`：指定 VS Code 使用的默認 Python 解釋器路徑
- `jupyter.kernels.filter`：配置 Jupyter 使用相同的 Python 環境，確保 Notebook 執行環境與編輯器一致

### 修改路徑

如果你的 Python 環境安裝在不同的位置，請將上述路徑替換為你的實際 WinPython 安裝路徑。例如：

```
D:\your\python\installation\path\python.exe
```
