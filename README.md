# BotFundInvestment - 基金投資追蹤與分析系統

這是一個**自動化的基金週報系統**，用於追蹤基金績效、進行投組回測分析，並生成週報告。

---

## 📊 專案概述

**BotFundInvestment** 核心功能：

- ✅ 自動從 BCD API 抓取基金淨值資料
- ✅ 自動判斷基金類型（國內/海外）
- ✅ 計算投組績效指標（CAGR、Sharpe Ratio、最大回撤等）
- ✅ 多維度投組回測（等權/自訂權重 × 再平衡/不再平衡）
- ✅ 智能告警系統（多重條件判定）
- ✅ 風險貢獻分析
- ✅ 生成專業 Excel 報告 + 可選 PDF 輸出

---

## 🔄 專案流程

```
fund_list.xlsx（輸入）
    ↓
[Step 1] fetch_nav.ipynb
    ├─ 從 BCD API 抓取基金淨值數據（近1年）
    ├─ 自動判斷國內/海外基金
    ├─ 計算漲/跌幅
    └─ 輸出 → fund_nav_output.xlsx

[Step 2] weekly_fund_report_notebook.ipynb
    ├─ 讀取基金淨值數據（近2年）
    ├─ 計算績效指標：
    │  ├─ 上週報酬
    │  ├─ 近4週/12週報酬
    │  ├─ 最大回撤（Drawdown）
    │  └─ Sharpe Ratio
    ├─ 生成告警系統（多重條件判定）
    ├─ 投組回測分析：
    │  ├─ 等權配置
    │  ├─ 自訂權重
    │  └─ 再平衡策略（月度/季度）
    ├─ 風險貢獻分析
    ├─ 生成 PNG 圖表
    └─ 輸出 → weekly_fund_report.xlsx（含圖表）

[Step 3] weekly_fund_report_notebook_with_pdf.ipynb（可選）
    └─ 在上述基礎上，額外輸出
       └─ weekly_fund_report.pdf（PDF 報告）
```

---

## 🛠️ 技術棧

| 技術                 | 用途             |
| -------------------- | ---------------- |
| **Python**           | 核心開發語言     |
| **Jupyter Notebook** | 交互式分析與展示 |
| **Pandas**           | 數據處理與操作   |
| **NumPy**            | 數值計算         |
| **Matplotlib**       | 圖表可視化       |
| **OpenPyXL**         | Excel 檔案讀寫   |
| **ReportLab**        | PDF 生成         |
| **Requests**         | HTTP API 調用    |

---

## 📋 三個 Notebook 詳細說明

### 1️⃣ fetch_nav.ipynb - 基金淨值抓取器

**目的**：批量下載基金歷史淨值資料

**主要功能**：

- 讀取 `fund_list.xlsx`（基金代號、名稱、查詢傳輸代號）
- 自動判斷基金類型（國內/海外），分別呼叫不同的 BCD API：
  - 先試 `tBCDNavList`（國內基金）
  - 失敗後試 `BCDNavList`（海外基金）
- 解析 API 回傳的淨值資料（yyyymmdd 格式轉換為 DateTime）
- 計算每日漲跌與漲跌幅
- **輸出**：`fund_nav_output.xlsx`（包含近 1 年的基金淨值明細）

**使用時機**：定期（如週一）執行以更新基金資料庫

**程式單元**：

1. 匯入套件
2. 設定參數（輸入/輸出檔案、時間範圍）
3. BCD API 相關函數定義
4. 讀取基金清單並驗證
5. 批次下載並解析淨值
6. 輸出結果到 Excel

---

### 2️⃣ weekly_fund_report_notebook.ipynb - 完整週報系統（無 PDF）

**目的**：產生詳細的基金及投組週報、告警、圖表

**主要功能**：

#### 📊 數據處理

- 讀取並驗證 `fund_list.xlsx`（必填：基金代號、名稱、查詢傳輸代號；選填：權重）
- 批次下載淨值（2 年歷史資料）
- 建立日報酬/淨值矩陣

#### 📈 投組回測

- **等權投組**：每支基金佔比相等
- **自訂投組**：依 Excel 權重欄位調整
- **再平衡策略**：每月（M）或每季（Q）調整回初始權重
- **Buy & Hold**：不再平衡，放任權重漂移
- 計算績效指標：
  - CAGR（年複合成長率）
  - 年化報酬 & 年化波動
  - Sharpe Ratio（風險調整報酬）
  - 最大回撤 & 回撤恢復期
  - Sortino Ratio（負向波動調整）

#### 🚨 告警系統

同時滿足三個條件時觸發告警：

- 上週報酬 < -1%（`TH_WEEK = -0.01`）
- 近 4 週報酬 < -2%（`TH_4W = -0.02`）
- 目前回撤 < -5%（`TH_DD = -0.05`）
- **嚴重度分數**：綜合三項指標加權
  - 回撤權重：50%（`SEV_W_DD = 0.5`）
  - 近 4 週權重：30%（`SEV_W_4W = 0.3`）
  - 上週權重：20%（`SEV_W_WEEK = 0.2`）

#### 📋 報表輸出

- **summary**：關鍵績效指標、投組比較、風險貢獻 Top5
- **fund_weekly**：基金週報（上週/近 4 週/近 12 週報酬）
- **alerts**：告警清單（基金 & 投組都包含）
- **charts_p1 / charts_p2**：圖表頁面（每頁 2 張放大圖表）
- **correlation_matrix**：基金相關係數熱力圖
- **fund_risk_contrib**：基金風險貢獻分解
- **drawdown_details**：高點/回撤開始日/持續天數

#### 📊 圖表（PNG）

1. 投組淨值走勢（再平衡 vs 不再平衡比較）
2. 告警清單柱狀圖（嚴重度指標）
3. 風險報酬散點圖（年化報酬 vs 年化波動）
4. 相關係數熱力圖（基金關聯度）
5. 風險貢獻餅圖（每支基金佔比）

**輸出**：`weekly_fund_report.xlsx`（含嵌入 PNG 圖表）

---

### 3️⃣ weekly_fund_report_notebook_with_pdf.ipynb - 完整週報系統（含 PDF）

**目的**：功能同上，額外產生專業 PDF 報告

**與第 2 個 Notebook 的主要差異**：

- ✅ 額外產生 `weekly_fund_report.pdf`
- ✅ PDF 首頁是 Summary 文字頁面
  - Key Statistics（CAGR、Sharpe、Max DD）
  - 投組績效對比表
  - Top 10 告警清單
- ✅ 後續兩頁為圖表頁面
- ✅ **中文字型支援**：
  - 優先使用思源黑體（`D:\fonts\NotoSansTC-Regular.ttf`）
  - 若字型檔不存在，自動 fallback 至 ReportLab 內建字型 `MSung-Light`

**輸出**：

- `weekly_fund_report.xlsx`（Excel 報告）
- `weekly_fund_report.pdf`（PDF 報告）

---

## 📁 輸入 & 輸出檔案

### 必需輸入：fund_list.xlsx

| 欄位         | 類型 | 必填 | 說明                      | 例子         |
| ------------ | ---- | ---- | ------------------------- | ------------ |
| 基金代號     | 文字 | ✅   | 台灣上市基金代碼          | `0050`       |
| 基金名稱     | 文字 | ✅   | 基金名稱                  | `元大台灣50` |
| 查詢傳輸代號 | 文字 | ✅   | BCD API 查詢碼            | `ACDS134`    |
| 權重         | 數字 | ⭕   | 投組權重（%），留空時等權 | `30`         |

### 輸出檔案

| 檔案                      | 來源                                       | 說明                      |
| ------------------------- | ------------------------------------------ | ------------------------- |
| `fund_nav_output.xlsx`    | fetch_nav.ipynb                            | 淨值歷史數據              |
| `weekly_fund_report.xlsx` | weekly_fund_report_notebook.ipynb          | 完整週報（含 Excel 圖表） |
| `weekly_fund_report.pdf`  | weekly_fund_report_notebook_with_pdf.ipynb | PDF 正式報告              |
| `charts/`                 | 兩個報告 Notebook                          | 臨時圖表資料夾            |

---

## ⚙️ 關鍵參數設定

### fetch_nav.ipynb 參數

```python
INPUT_EXCEL  = "fund_list.xlsx"
OUTPUT_EXCEL = "fund_nav_output.xlsx"
DAYS_BACK    = 365      # 抓取天數（近一年）
SORT_DESC    = True     # 輸出日期新到舊
TIMEOUT      = 30       # HTTP 逾時秒數
```

### weekly_fund_report_notebook.ipynb / .with_pdf.ipynb 參數

```python
INPUT_EXCEL  = "fund_list.xlsx"
OUTPUT_EXCEL = "weekly_fund_report.xlsx"
OUTPUT_PDF   = "weekly_fund_report.pdf"  # 僅 .with_pdf 版本

DAYS_BACK    = 730      # 抓取天數（約 2 年）
REBALANCE    = "M"      # 再平衡頻率：'M'=月、'Q'=季、None=不再平衡
RISK_FREE    = 0.0      # 無風險利率（Sharpe 簡化用）
TIMEOUT      = 30       # HTTP 逾時秒數

# 告警門檻
TH_WEEK      = -0.01    # 上週報酬 < -1%
TH_4W        = -0.02    # 近4週報酬 < -2%
TH_DD        = -0.05    # 回撤 < -5%

# 嚴重度分數權重
SEV_W_DD     = 0.5      # 回撤權重 50%
SEV_W_4W     = 0.3      # 近4週權重 30%
SEV_W_WEEK   = 0.2      # 上週權重 20%

CHART_DIR    = "charts" # 圖表輸出資料夾
```

---

## 🚀 使用流程

### 建議執行方式

**每週一上午執行**（報告週期以上週五收盤（W-FRI）為基準）：

```
1. 準備 fund_list.xlsx
   ├─ 基金代號
   ├─ 基金名稱
   ├─ 查詢傳輸代號
   └─ 權重（可選）

2. 執行 fetch_nav.ipynb
   └─ 生成 fund_nav_output.xlsx

3. 執行 weekly_fund_report_notebook.ipynb
   └─ 生成 weekly_fund_report.xlsx

或

3. 執行 weekly_fund_report_notebook_with_pdf.ipynb
   ├─ 生成 weekly_fund_report.xlsx
   └─ 生成 weekly_fund_report.pdf
```

### 安裝依賴

```bash
pip install requests pandas openpyxl matplotlib reportlab
```

---

## 📈 核心功能詳解

### 數據來源

**BCD API 自動適配**：

- 國內基金：`https://fund.bot.com.tw/w/bcd/tBCDNavList.djbcd`
- 海外基金：`https://fund.bot.com.tw/w/bcd/BCDNavList.djbcd`
- 自動自適應判斷基金類型

### 績效計算

| 指標             | 計算方法                                | 用途         |
| ---------------- | --------------------------------------- | ------------ |
| **日報酬**       | (淨值[t] - 淨值[t-1]) / 淨值[t-1]       | 基礎數據     |
| **累積報酬**     | (淨值[end] - 淨值[start]) / 淨值[start] | 總體表現     |
| **CAGR**         | (淨值[end]/淨值[start])^(1/年數) - 1    | 年化成長     |
| **年化波動**     | 日報酬 × √252                           | 風險度量     |
| **Sharpe Ratio** | (報酬 - 無風險率) / 波動                | 風險調整報酬 |
| **最大回撤**     | (高點 - 當前) / 高點                    | 下檔風險     |

### 投組回測情景

| 情景 | 配置     | 再平衡    | 用途                   |
| ---- | -------- | --------- | ---------------------- |
| 等權 | 每支平均 | 月度/季度 | 基準比較               |
| 等權 | 每支平均 | 不再平衡  | Buy & Hold 進度追蹤    |
| 自訂 | 依 Excel | 月度/季度 | 實際投組追蹤           |
| 自訂 | 依 Excel | 不再平衡  | 實際投組現況（未調整） |

### 告警邏輯

```
告警觸發條件（AND 邏輯）：
  [上週報酬 < -1%] AND [近4週報酬 < -2%] AND [回撤 < -5%]

嚴重度分數：
  Severity = (|回撤| × 0.5) + (|近4週| × 0.3) + (|上週| × 0.2)
  範圍：0 ~ 1（越高越嚴重）
```

---

## 🎯 適用場景

✅ **基金經理人**：追蹤投資組合績效  
✅ **投資分析師**：批量回測與告警系統  
✅ **財務部門**：定期產出正式報告供管理層查閱  
✅ **量化交易員**：進行相關性與風險分析  
✅ **顧問顧問**：基金績效展示與風險揭露

---

## 📝 重要備註

- **資料來源**：基金大觀園平台（`fund.bot.com.tw`）
- **中文字型**（PDF 報告）：`D:\fonts\NotoSansTC-Regular.ttf`（思源黑體），不存在時自動降級
- **時區**：API 回傳資料為台灣時間
- **更新頻率**：建議每週一上午執行，報告涵蓋上一個完整交易週
- **本報表僅供資料分析與追蹤使用，非投資建議**

---

## 📧 聯絡方式

若遇到問題或需要功能擴展，請參考專案結構進行客製化調整。

---

**最後更新**：2026 年 3 月 4 日
