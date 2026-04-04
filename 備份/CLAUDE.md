# Project: [你的專案名稱]

## Overview
[簡述你的專案目標，例如：一個經濟數據儀表板，用來追蹤美國主要經濟指標]

## Tech Stack
- Language: [Python / Node.js / etc.]
- Framework: [FastAPI / Express / etc.]
- [其他相關技術]

---

## FRED API Reference

- 官方文件: https://fred.stlouisfed.org/docs/api/fred/
- Base URL: `https://api.stlouisfed.org/fred`
- API Key: 存放在環境變數 `FRED_API_KEY`（不可寫死在程式碼中）
- 預設回傳格式為 XML，我們一律使用 `file_type=json`

### 常用 Endpoints

| Endpoint | 用途 | 關鍵參數 |
|---|---|---|
| `/fred/series` | 取得單一經濟數據序列的 metadata | `series_id` |
| `/fred/series/observations` | 取得序列的實際數據值（最常用） | `series_id`, `observation_start`, `observation_end`, `frequency`, `units` |
| `/fred/series/search` | 用關鍵字搜尋序列 | `search_text`, `limit`, `order_by` |
| `/fred/category/series` | 取得某分類下的所有序列 | `category_id`, `limit` |
| `/fred/releases` | 取得所有資料發布 | `limit`, `offset` |
| `/fred/release/series` | 取得某次發布中的所有序列 | `release_id` |

### 取得數據的標準呼叫模式

```
GET https://api.stlouisfed.org/fred/series/observations
  ?series_id=GDP
  &api_key={FRED_API_KEY}
  &file_type=json
  &observation_start=2020-01-01
  &observation_end=2024-12-31
  &frequency=q
  &units=lin
```

### 回傳格式範例（JSON）

```json
{
  "realtime_start": "2024-01-01",
  "realtime_end": "2024-01-01",
  "observation_start": "2020-01-01",
  "observation_end": "2024-12-31",
  "units": "lin",
  "output_type": 1,
  "file_type": "json",
  "order_by": "observation_date",
  "sort_order": "asc",
  "count": 20,
  "offset": 0,
  "limit": 100000,
  "observations": [
    {
      "realtime_start": "2024-01-01",
      "realtime_end": "2024-01-01",
      "date": "2020-01-01",
      "value": "21538.032"
    }
  ]
}
```

### 頻率參數 (frequency)

- `d` = Daily, `w` = Weekly, `bw` = Biweekly
- `m` = Monthly, `q` = Quarterly, `sa` = Semiannual, `a` = Annual
- 只能往低頻率聚合（例如 日→月 可以，月→日 不行）

### 單位轉換參數 (units)

- `lin` = 原始值（預設）
- `chg` = 與前一期的差值
- `ch1` = 與一年前的差值
- `pch` = 與前一期的百分比變化
- `pc1` = 與一年前的百分比變化
- `pca` = 年化百分比變化
- `log` = 自然對數

### 聚合方法 (aggregation_method)

- `avg` = 平均值（預設）
- `sum` = 加總
- `eop` = 期末值

### 本專案常用的 Series IDs

| Series ID | 名稱 | 頻率 | 說明 |
|---|---|---|---|
| `GDP` | Gross Domestic Product | Quarterly | 美國 GDP |
| `UNRATE` | Unemployment Rate | Monthly | 失業率 |
| `CPIAUCSL` | Consumer Price Index | Monthly | 消費者物價指數 |
| `FEDFUNDS` | Federal Funds Rate | Monthly | 聯邦基金利率 |
| `DGS10` | 10-Year Treasury Rate | Daily | 10年期公債殖利率 |
| [依專案需求自行增減] | | | |

---

## Coding Conventions

- API Key 從 `os.environ["FRED_API_KEY"]` 或 `.env` 讀取，禁止 hardcode
- 所有 API 請求加上 `file_type=json`
- 數據值 `value` 欄位可能為 `"."` 代表缺失值，需處理為 null/NaN
- API 有 rate limit，需實作重試機制與適當的間隔
- 日期格式統一使用 `YYYY-MM-DD`

## Error Handling

FRED API 錯誤回傳格式：
```json
{
  "error_code": 400,
  "error_message": "Bad Request. Variable series_id is not set..."
}
```

常見錯誤碼：400（參數錯誤）、403（API key 無效）、429（請求過多）

## 需要更詳細的 endpoint 文件時

直接讀取線上文件：https://fred.stlouisfed.org/docs/api/fred/{endpoint_name}.html
例如：https://fred.stlouisfed.org/docs/api/fred/series_observations.html