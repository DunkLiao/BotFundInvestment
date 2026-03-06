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
[Step 1] fetch_nav.ipynb - 基金淨值抓取
    ├─ 讀取 fund_list.xlsx（基金代號、名稱、查詢傳輸代號）
    ├─ 從 BCD API 抓取歷史淨值數據（近1年）
    ├─ 自動判斷國內/海外基金類型
    ├─ 計算每日漲/跌幅
    └─ 輸出 → fund_nav_output.xlsx

[Step 2] weekly_fund_report_notebook_with_pdf.ipynb - 完整週報
    ├─ 讀取基金淨值數據（長表）並建立日報酬矩陣
    ├─ 基金週報計算：
    │  ├─ 上週/近4週/近12週報酬
    │  ├─ 目前回撤 & 回撤詳情
    │  └─ 高點日期 & 回撤持續天數
    ├─ 投組回測分析：
    │  ├─ 等權投組 + 自訂權重投組
    │  ├─ 再平衡策略（月度/季度/不再平衡）
    │  └─ 績效指標：CAGR、年化波動、Sharpe Ratio、最大回撤
    ├─ 智能告警系統（基金 + 投組）
    │  └─ 三重條件觸發 + 嚴重度評分
    ├─ 相關係數矩陣分析
    ├─ 風險貢獻分解（RC% & AbsRC）
    ├─ 生成 4 張 PNG 圖表
    ├─ 輸出 → weekly_fund_report.xlsx（含嵌入圖表）
    └─ 輸出 → weekly_fund_report.pdf（Summary + 圖表，使用思源黑體）
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

## 📋 Notebook 詳細說明

### 1️⃣ fetch_nav.ipynb - 基金淨值歷史資料抓取器

**目的**：批量下載基金歷史淨值資料

**主要功能**：

- 讀取 `fund_list.xlsx`（基金代號、名稱、查詢傳輸代號）
- **自動判斷基金類型**（國內/海外）並調用相應 BCD API 端點：
  - 先嘗試 `tBCDNavList`（國內基金）
  - 失敗後自動降級至 `BCDNavList`（海外基金）
- 解析 API 回傳的淨值數據（YYYYMMDD 格式轉換為 DateTime）
- 計算每日漲/跌幅（漲/跌點數及百分比）
- **輸出**：`fund_nav_output.xlsx`
  - 工作表 `nav`：淨值明細（基金代號、名稱、日期、淨值、漲/跌幅）
  - 工作表 `fail`：抓取失敗的基金列表（若有）

**使用時機**：定期（建議週一或每日）執行，在 `fetch_nav.ipynb` 完成後執行 `weekly_fund_report_notebook_with_pdf.ipynb`

**關鍵代碼流程**：

1. 匯入依賴（requests, pandas, datetime 等）
2. 參數設定（輸入/輸出檔案、回溯天數、超時設定）
3. 定義 BCD API 調用函數
4. 讀取基金清單，驗證必填欄位
5. 批次抓取淨值並自動檢測基金類型
6. 評估淨值差異，輸出 Excel

---

### 2️⃣ weekly_fund_report_notebook_with_pdf.ipynb - 完整週報系統（含 PDF）

**目的**：產生全面的基金績效追蹤、投組回測、告警及正式報告

**主要功能**：

#### 📊 **1. 數據獲取與處理**

- 讀取並驗證 `fund_list.xlsx`
  - 必填：基金代號、基金名稱、查詢傳輸代號
  - 選填：權重（用於自訂投組）
- 批次下載 1 年淨值（時間範圍可自訂 `DAYS_BACK`）
- 自動判斷國內/海外基金，呼叫相應 API
- 建立日報酬矩陣（交易日×基金）

#### 📈 **2. 週報計算**

基於「週末（W-FRI）」的周期性聚合：

- **上週報酬**：最新周報酬率
- **近4週報酬**：過去4個周末累積報酬 $(1+r_1)(1+r_2)...-1$
- **近12週報酬**：過去12個周末累積報酬
- **回撤詳情**：
  - 目前回撤 = (當前淨值 / 歷史高點 - 1)
  - 高點日期、回撤開始日
  - 距離高點天數、回撤持續天數

#### 💼 **3. 投組回測（4 種情景）**

| 情景             | 配置     | 再平衡 | 用途            |
| ---------------- | -------- | ------ | --------------- |
| 等權（再平衡）   | 每支平均 | 月/季  | 基準對標        |
| 等權（不再平衡） | 每支平均 | 無     | Buy & Hold 進度 |
| 自訂（再平衡）   | 按 Excel | 月/季  | 實際投組追蹤    |
| 自訂（不再平衡） | 按 Excel | 無     | 現況對標        |

**績效指標** $\text{CAGR} = \left(\frac{\text{NT}_{\text{end}}}{\text{NT}_{\text{start}}}\right)^{\frac{365}{\text{days}}} - 1$

- **累積報酬**：起訖淨值變化百分比
- **CAGR**：年複合成長率
- **年化波動**：日報酬標準差 × $\sqrt{252}$
- **Sharpe Ratio（簡化）**：$\frac{\text{報酬} - \text{無風險率}}{\text{波動}}$
- **最大回撤（MDD）**：歷史最深谷幅度
- **Sortino Ratio（選項）**：僅計下行波動

#### 🚨 **4. 智能告警系統**

**觸發邏輯**（AND 三重條件）：

```
告警成立條件：
  [上週報酬 < -1%] AND [近4週報酬 < -2%] AND [回撤 < -5%]
```

**嚴重度評分** $\text{Severity} = (|\text{DD}| \times 0.5) + (|近4週| \times 0.3) + (|上週| \times 0.2)$

- 範圍：0 ~ 1（越高越嚴重）
- 同時涵蓋基金及投組層級的告警

#### 📊 **5. 相關性分析**

- **相關係數矩陣**：基於 60 個交易日計算
- **風險貢獻（RC）**：
  - **RC%**：各基金對投組總風險的百分比貢獻
  - **AbsRC（絕對風險貢獻）**：年化波動角度的風險值
  - 按風險貢獻度排序，識別風險集中度

#### 🎨 **6. 圖表生成（PNG @ 220 DPI）**

1. **p1_01_portfolio_nav.png**：投組淨值走勢
   - 上圖：等權（再平衡） vs 自訂權重（再平衡）
   - 下圖：等權（不再平衡） vs 自訂權重（不再平衡）
2. **p1_02_alerts.png**：告警柱狀圖
   - X軸：告警標的，Y軸：目前回撤
   - 若無告警則顯示「本週無符合條件告警 ✅」
3. **p2_01_risk_return.png**：風險報酬散點圖
   - X軸：年化波動，Y軸：年化報酬
   - 標記基金代號便於識別
4. **p2_02_corr_heatmap.png**：相關係數熱圖
   - 色階：-1（藍，負相關）~ +1（紅，正相關）

#### 📋 **7. Excel 報表結構**

| 工作表                   | 內容                       | 備註              |
| ------------------------ | -------------------------- | ----------------- |
| **charts_p1**            | 投組淨值圖 + 告警圖        | 兩張放大圖        |
| **charts_p2**            | 風險散點圖 + 熱圖          | 兩張放大圖        |
| **summary**              | 綜合指標、投組对比、Top RC | 首頁總結          |
| **nav_long**             | 基金淨值長表               | 原始數據          |
| **fund_weekly**          | 基金週報                   | 上週/近4週/回撤等 |
| **alerts**               | 告警清單                   | 含嚴重度評分      |
| **port_equal_rb_daily**  | 等權（再平衡）日報         | 供進階分析        |
| **port_equal_bh_daily**  | 等權（不再平衡）日報       | 供進階分析        |
| **port_custom_rb_daily** | 自訂（再平衡）日報         | 若有自訂權重      |
| **port_custom_bh_daily** | 自訂（不再平衡）日報       | 若有自訂權重      |
| **corr**                 | 相關係數矩陣               | 60 日計算窗口     |
| **drawdown_fund**        | 基金回撤詳情               | 回撤持續天數等    |
| **drawdown_port**        | 投組回撤詳情               |                   |
| **risk_contrib_equal**   | 等權投組風險貢獻           | RC% 排序          |
| **risk_contrib_custom**  | 自訂投組風險貢獻           | 若有自訂權重      |
| **fail**                 | 抓取失敗列表               | API 異常或無數據  |

#### 📄 **8. PDF 報告結構**

**Page 1 - Summary 文字頁**

- 報表產出時間、週期結束日、基金數量
- 投組績效對比表（CAGR、年化波動、MDD、Sharpe）
- Top 10 告警清單（含嚴重度評分）

**Page 2 - 圖表頁（投組分析）**

- 投組淨值走勢（再平衡 vs BH）

**Page 3 - 圖表頁（風險分析）**

- 風險報酬散點圖
- 相關係數熱圖

**字型支援**：

- 優先使用思源黑體：`D:\fonts\NotoSansTC-Regular.ttf`
- 若字型檔不存在，自動降級至 ReportLab 內建 `MSung-Light`
- 列印友善，支援 A4 橫向排版

#### 🔧 **9. 輸出文件清單**

- `weekly_fund_report.xlsx`：完整 Excel 報表（含嵌入圖表）
- `weekly_fund_report.pdf`：正式 PDF 報告
- `charts/` 資料夾：臨時存放 4 張 PNG 圖表

**使用時機**：每週一或交易周結束時執行（建議 W-FRI 為報告周期結束點）

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
INPUT_EXCEL  = "fund_list.xlsx"      # 輸入基金清單
OUTPUT_EXCEL = "fund_nav_output.xlsx" # 輸出淨值數據

DAYS_BACK    = 365      # 回溯天數（1年）
SORT_DESC    = True     # 輸出排序：最新到最舊
TIMEOUT      = 30       # HTTP 逾時（秒）
```

### weekly_fund_report_notebook_with_pdf.ipynb 參數

```python
# 檔案路徑
INPUT_EXCEL  = "fund_list.xlsx"
OUTPUT_EXCEL = "weekly_fund_report.xlsx"
OUTPUT_PDF   = "weekly_fund_report.pdf"

# 數據抓取
DAYS_BACK    = 365      # 回溯天數（建議1年+）
TIMEOUT      = 30       # HTTP 逾時（秒）

# 投組設定
REBALANCE    = "M"      # 再平衡頻率：'M'=月、'Q'=季、None=不再平衡
RISK_FREE    = 0.0      # 無風險利率（用於 Sharpe 計算）

# 告警門檻（三條件 AND 邏輯）
TH_WEEK      = -0.01    # 上週報酬臨界值 (-1%)
TH_4W        = -0.02    # 近4週報酬臨界值 (-2%)
TH_DD        = -0.05    # 回撤臨界值 (-5%)

# 嚴重度評分權重
SEV_W_DD     = 0.5      # 回撤權重（50%）
SEV_W_4W     = 0.3      # 近4週權重（30%）
SEV_W_WEEK   = 0.2      # 上週權重（20%）

# 輸出路徑
CHART_DIR    = "charts" # 圖表暫存資料夾

# PDF 字型（中文支援）
FONT_PATH    = r"D:\fonts\NotoSansTC-Regular.ttf"  # 思源黑體路徑
```

**參數調整建議**：
| 場景 | DAYS_BACK | REBALANCE | TH_WEEK | TH_4W | TH_DD |
|------|-----------|-----------|---------|-------|--------|
| 短期追蹤 | 180 | "M" | -0.02 | -0.03 | -0.03 |
| 標準配置 | 365 | "M" | -0.01 | -0.02 | -0.05 |
| 長期分析 | 730 | "Q" | -0.005 | -0.01 | -0.07 |
| 保守告警 | 365 | "M" | -0.005 | -0.01 | -0.03 |

---

## 🚀 使用流程

### 建議執行方式

**每週一上午執行**（報告周期以上週五收盤（W-FRI）為基準）：

```
1. 準備 fund_list.xlsx
   ├─ 基金代號
   ├─ 基金名稱
   ├─ 查詢傳輸代號（BCD API 代碼）
   └─ 權重（可選，用於自訂投組）

2. 執行 fetch_nav.ipynb
   └─ 生成 fund_nav_output.xlsx（淨值基礎數據）

3. 執行 weekly_fund_report_notebook_with_pdf.ipynb
   ├─ 生成 weekly_fund_report.xlsx（完整週報）
   └─ 生成 weekly_fund_report.pdf（正式報告）
```

**注意事項**：

- 若 `fetch_nav.ipynb` 已在其他時間更新，可直接執行第 3 步
- PDF 字型依賴於 `D:\fonts\NotoSansTC-Regular.ttf`，不存在時會自動降級
- 首次運行建議檢查參數配置（參見下節）

### 安裝依賴

```bash
pip install requests pandas openpyxl matplotlib reportlab
```

**驗證安裝**：

```python
import requests, pandas, openpyxl, matplotlib, reportlab
print("All packages installed successfully!")
```

---

## 📈 核心功能詳解

### 數據來源與自動判斷

**BCD API 自動判斷流程**：

```
基金查詢代號 (a parameter)
    ↓
[嘗試] tBCDNavList (國內基金端點)
    ├─ 若成功 → 標記「國內」，返回數據
    └─ 若失敗 ↓
[嘗試] BCDNavList (海外基金端點)
    ├─ 若成功 → 標記「海外」，返回數據
    └─ 若失敗 → 標記「異常」，記錄到 fail 工作表
```

**API 參數格式**：

```
base_url: https://fund.bot.com.tw/w/bcd/{tBCDNavList|BCDNavList}.djbcd
params:
  - a: 查詢傳輸代號（fund_list.xlsx 提供）
  - b: 固定為 1
  - c: 開始日期（YYYY-M-D 格式，無零補）
  - d: 結束日期（YYYY-M-D 格式，無零補）

response format:
  <dates_csv: YYYYMMDD,YYYYMMDD,...> <navs_csv: 123.45,124.67,...>
```

**回期率計算**：

- 日報酬：$r_t = \frac{NAV_t - NAV_{t-1}}{NAV_{t-1}}$
- 累積報酬：$R = \frac{NAV_{end} - NAV_{start}}{NAV_{start}}$
- 周報酬：按「周末（W-FRI）」重採樣

### 績效指標計算

| 指標                  | 公式                                               | 解釋           | 適用場景 |
| --------------------- | -------------------------------------------------- | -------------- | -------- |
| **日報酬**            | $(NAV_t - NAV_{t-1}) / NAV_{t-1}$                  | 天級別基礎報酬 | 日常監控 |
| **周報酬**            | 周末重採樣後的百分比變化                           | 週級別聚合報酬 | 週報生成 |
| **累積報酬**          | $(NAV_{end} / NAV_{start}) - 1$                    | 期間總體表現   | 績效概括 |
| **CAGR**              | $(NAV_{end}/NAV_{start})^{365/\text{days}} - 1$    | 年化成長率     | 長期對標 |
| **年化波動**          | $\sigma_{\text{daily}} \times \sqrt{252}$          | 風險量化       | 風險評估 |
| **Sharpe Ratio**      | $\frac{CAGR - r_f}{\sigma_{\text{annual}}}$        | 風險調整報酬   | 基金對標 |
| **最大回撤（MDD）**   | $\min\left(\frac{NAV_t}{NAV_{\max(t)}}\right) - 1$ | 最大跌幅       | 風險警示 |
| **RC%（風險貢獻比）** | $\frac{w_i \times \text{Cov}[\cdot]}{p_\sigma^2}$  | 基金風險佔比   | 風險分解 |

### 投組回測機制

**數據準備**：

1. 構建日報酬矩陣：每行為交易日，每列為基金
2. 權重初始化：等權 = $w_i = 1/N$，自訂權重從 Excel 讀取並正規化
3. 淨值初始化：以 100 為起點

**再平衡邏輯**：

```python
# 每月/季度重新調整至初始權重
if current_period != prev_period:
    current_weights = initial_weights.copy()

# 計算本日報酬
daily_return = sum(asset_return[i] * current_weights[i])

# 更新個別資產權重（權重漂移）
current_weights[i] *= (1 + asset_return[i])
current_weights = normalize(current_weights)
```

**不再平衡（Buy & Hold）**：

- 一次投入按初始權重配置
- 權重隨市場表現自然漂移
- 模擬「放著不動」的實際表現

### 告警系統

**三重條件 AND 邏輯**：

```
Alert Triggered = (Week_Return < -1%) AND (4W_Return < -2%) AND (Drawdown < -5%)
```

**嚴重度評分計算**：

$$\text{Severity} = |DD| \times 0.5 + |R_{4W}| \times 0.3 + |R_{\text{week}}| \times 0.2$$

其中：

- $DD$ = 目前回撤（負值）
- $R_{4W}$ = 近 4 週報酬（負值）
- $R_{\text{week}}$ = 上週報酬（負值）
- 結果範圍：0 ~ 1，值越高越嚴重

**應用場景**：

- $\text{Severity} > 0.7$：立即風險提示
- $0.5 < \text{Severity} \leq 0.7$：中等關注
- $\text{Severity} \leq 0.5$：低風險警示

### 相關性與風險分解

**相關係數計算**：

- 窗口：60 個交易日
- 方法：Pearson 相關係數
- 用途：基金間互動特性識別

**風險貢獻分解**：

$$RC_i = w_i \times \frac{(\Sigma \vec{w})_i}{\sigma_p^2} \times \sigma_p$$

其中：

- $w_i$ = 基金 $i$ 的權重
- $\Sigma$ = 年化協方差矩陣
- $\sigma_p$ = 投組年化波動
- $RC_i\%$ = 基金 $i$ 在總風險中的百分比貢獻

**解讀**：

- 若某基金 RC% 過高（>40%），需關注集中度風險
- RC% 高不一定是壞事（高報酬也可能高風險）
- 結合收益率觀察，評估性價比

---

## 🎯 適用場景

✅ **基金經理人**：追蹤投資組合績效  
✅ **投資分析師**：批量回測與告警系統  
✅ **財務部門**：定期產出正式報告供管理層查閱  
✅ **量化交易員**：進行相關性與風險分析  
✅ **顧問顧問**：基金績效展示與風險揭露

---

## 📝 重要備註

- **資料來源**：基金大觀園平台（`fund.bot.com.tw`）的 BCD 定投服務 API
- **API 政策**：
  - 無認證需求（公開端點）
  - 無官方文檔，端點可能變更
  - 建議添加異常处理及重试机制
- **時區**：API 回傳資料為台灣時間（GMT+8）
- **淨值頻率**：交易日每日更新（非交易日無新數據）
- **中文字型**（PDF 報告）：
  - 優先使用思源黑體（OpenType）：`D:\fonts\NotoSansTC-Regular.ttf`
  - 若字型檔不存在，自動降級至 ReportLab 內建 `MSung-Light`
  - 可自行調整 `FONT_PATH` 指向其他 TTF 字型
- **更新頻率**：建議每週一上午執行，報告涵蓋上一個完整交易週（W-FRI 為周期結束）
- **性能提示**：
  - 基金數量 > 20 時，Notebook 執行時間 > 5 分鐘（取決於網路)
  - 圖表生成使用較高 DPI（220），PDF 文件 > 5 MB
  - 首次執行會在 `charts/` 資料夾生成臨時圖表
- **免責聲明**：本報表僅供數據分析與投資組合監控使用，**非投資建議**。基金投資風險由投資人自行承擔

---

## 🔗 項目依賴流程

```
fetch_nav.ipynb
    ├─ 輸入 ← fund_list.xlsx
    └─ 輸出 → fund_nav_output.xlsx
                    ↓（可選：供參考或外部系統）

weekly_fund_report_notebook_with_pdf.ipynb
    ├─ 輸入 ← fund_list.xlsx + 基金代號/傳輸代號
    ├─ 自動抓取 ← BCD API
    ├─ 生成圖表 → charts/ (p1_01.png, p1_02.png, p2_01.png, p2_02.png)
    ├─ 輸出 → weekly_fund_report.xlsx
    └─ 輸出 → weekly_fund_report.pdf

分發/歸檔
    ├─ weekly_fund_report.xlsx (給分析師/基金經理)
    ├─ weekly_fund_report.pdf  (正式報告)
    └─ charts/                 (臨時文件)
```

---

## 🛠️ 技術棧詳細版本要求

| 套件       | 最低版本 | 用途                  | 安裝命令                      |
| ---------- | -------- | --------------------- | ----------------------------- |
| Python     | 3.8+     | 執行環境              | -                             |
| requests   | 2.25+    | HTTP API 調用         | `pip install requests`        |
| pandas     | 1.3+     | 數據操作 & Excel 讀寫 | `pip install pandas openpyxl` |
| numpy      | 1.20+    | 數值計算              | `pip install numpy`           |
| matplotlib | 3.4+     | 圖表可視化            | `pip install matplotlib`      |
| openpyxl   | 3.7+     | Excel 檔案操作        | `pip install openpyxl`        |
| reportlab  | 3.6+     | PDF 生成              | `pip install reportlab`       |

**安裝全部依賴**：

```bash
pip install requests pandas numpy matplotlib openpyxl reportlab
```

**驗證環境**：

```bash
python -c "import requests, pandas, numpy, matplotlib, openpyxl, reportlab; print('✅ All dependencies OK')"
```

---

## 📧 常見問題排查

### Q1: 運行時出現 "ModuleNotFoundError"

**A**：缺少依賴套件，執行 `pip install requests pandas openpyxl matplotlib reportlab`

### Q2: API 返回異常或無數據

**A**：

- 檢查 `fund_list.xlsx` 中的「查詢傳輸代號」是否正確
- 嘗試在基金大觀園網站確認該代號是否在綫
- 檢查網路連接及防火牆設定
- 若連續失敗，可能是 BCD API 端點變更

### Q3: PDF 産出的中文字型顯示為方塊

**A**：

- 確認思源黑體檔案是否存在於 `D:\fonts\NotoSansTC-Regular.ttf`
- 若無，手動下載思源黑體並調整 `FONT_PATH` 參數
- 或使用提供的系統字型名稱

### Q4: Excel 報表圖表無法顯示

**A**：

- 檢查 `charts/` 資料夾是否包含 4 張 PNG 圖表
- 確認 Notebook 執行完整（無任何 Cell 出錯）
- 嘗試手動在 Excel 中重新插入圖表

### Q5: 速度很慢

**A**：

- 減少 `DAYS_BACK` 天數（如改為 180 天）
- 檢查網路速度（API 調用可能受限）
- 若基金數量 > 50，考慮分批執行

---

## 📄 檔案清單

```
BotFundInvestment/
├── README.md                                (本檔案)
├── fetch_nav.ipynb                         (步驟 1：淨值抓取)
├── weekly_fund_report_notebook_with_pdf.ipynb (步驟 2：完整週報)
├── fund_list.xlsx                          (輸入：基金清單)
├── fund_nav_output.xlsx                    (輸出：淨值數據，由 fetch_nav 生成)
├── weekly_fund_report.xlsx                 (輸出：完整週報，含圖表)
├── weekly_fund_report.pdf                  (輸出：PDF 正式報告)
├── charts/                                 (輸出：臨時 PNG 圖表)
│   ├── p1_01_portfolio_nav.png
│   ├── p1_02_alerts.png
│   ├── p2_01_risk_return.png
│   └── p2_02_corr_heatmap.png
├── .doc/
│   └── enviroment.md                       (開發環境設置指南)
├── .vscode/
│   └── settings.json                       (VS Code 配置)
├── .git/                                   (Git 版本控制)
└── .gitignore                              (IDE/環境變數排除)
```

---

## 🎯 適用場景

✅ **基金經理人**：投資組合週期性績效追蹤與風險監控  
✅ **投資分析師**：批量基金回測、相關性分析、情景模擬  
✅ **財務部門**：定期產出正式報告供公司內部管理層查閱  
✅ **量化團隊**：基礎數據輸入、相關矩陣生成、風險貢獻分解  
✅ **投資顧問**：客戶報告、績效展示、風險揭露

---

**最後更新**：2026 年 3 月 6 日  
**版本**：2.0（整合 PDF 功能，簡化為單一 Notebook）
