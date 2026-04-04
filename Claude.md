# FRED API 參考

**官方文件:** https://fred.stlouisfed.org/docs/api/fred/

## 概覽

FRED（Federal Reserve Economic Data）API 是聖路易聯邦儲備銀行提供的 REST web service，可以程式化查詢 FRED 和 ALFRED（Archival FRED）資料庫，取得經濟數據。

- **Base URL:** `https://api.stlouisfed.org/fred/`
- **架構:** RESTful, HTTPS
- **回應格式:** XML（預設）、JSON；部分端點支援 XLSX 和 CSV
- **Content-Type:** `text/xml`（XML）、`application/json`（JSON）

## 認證

API Key 為 32 字元小寫英數字串，於 `fredaccount.stlouisfed.org` 註冊取得。

| 版本 | 方式 |
|------|------|
| v1 (legacy) | Query parameter: `?api_key=<your_key>` |
| v2 | Request header: `Authorization: Bearer <your_key>` |

## 速率限制與錯誤

- **限制:** 約每秒 2 次請求；超過回傳 HTTP `429 Too Many Requests`

| HTTP 代碼 | 意義 |
|-----------|------|
| 400 | Bad Request |
| 404 | Not Found |
| 423 | Locked |
| 429 | Too Many Requests |
| 500 | Internal Server Error |

## 通用參數（大多數端點適用）

| 參數 | 類型 | 預設值 | 說明 |
|------|------|--------|------|
| `api_key` | string | 必填 | 32 字元驗證金鑰 |
| `file_type` | string | `xml` | `xml`、`json`、`xlsx`（僅觀測值）、`csv`（僅觀測值）|
| `realtime_start` | YYYY-MM-DD | 今天 | 實時期間起始 |
| `realtime_end` | YYYY-MM-DD | 今天 | 實時期間結束 |
| `limit` | integer | 1000 | 最大結果數（1–1000；部分端點允許 10000）|
| `offset` | integer | 0 | 分頁偏移量 |
| `order_by` | string | 依端點而異 | 排序屬性 |
| `sort_order` | string | `asc` | `asc` 或 `desc` |

---

## 端點類別

### 1. CATEGORIES（分類）

用於導覽 FRED 系列的階層式分類樹。

| 端點 | 說明 |
|------|------|
| `GET fred/category` | 依 `category_id` 取得分類 |
| `GET fred/category/children` | 取得子分類 |
| `GET fred/category/related` | 取得相關分類（非階層性） |
| `GET fred/category/series` | 取得分類內所有系列 |
| `GET fred/category/tags` | 取得分類內系列的標籤 |
| `GET fred/category/related_tags` | 取得分類內系列的相關標籤 |

---

### 2. SERIES（系列）

存取經濟數據系列元資料和數值的核心端點。

| 端點 | 說明 |
|------|------|
| `GET fred/series` | 取得特定系列的元資料 |
| `GET fred/series/categories` | 取得系列所屬分類 |
| `GET fred/series/observations` | **取得系列的實際數據值** |
| `GET fred/series/release` | 取得系列相關的發布資訊 |
| `GET fred/series/search` | 以文字搜尋系列 |
| `GET fred/series/search/tags` | 取得搜尋結果系列的標籤 |
| `GET fred/series/search/related_tags` | 取得搜尋結果系列的相關標籤 |
| `GET fred/series/tags` | 取得特定系列的標籤 |
| `GET fred/series/updates` | 取得最近更新的系列 |
| `GET fred/series/vintagedates` | 取得系列數值被修訂或發布的日期 |

#### `fred/series/observations` 完整參數

| 參數 | 類型 | 預設值 | 說明 |
|------|------|--------|------|
| `series_id` | string | 必填 | 系列識別碼，如 `GDP`、`UNRATE`、`CPIAUCSL` |
| `observation_start` | YYYY-MM-DD | `1776-07-04` | 資料起始日期 |
| `observation_end` | YYYY-MM-DD | `9999-12-31` | 資料結束日期 |
| `units` | string | `lin` | 資料轉換（見下表）|
| `frequency` | string | 無 | 頻率聚合（見下表）|
| `aggregation_method` | string | `avg` | `avg`、`sum`、`eop`（期末值）|
| `output_type` | integer | `1` | 1=依實時期間, 2=依版本(全部), 3=依版本(新/修訂), 4=僅初始發布 |
| `vintage_dates` | YYYY-MM-DD,… | 無 | 逗號分隔的歷史快照日期 |

**`units` 轉換代碼：**

| 代碼 | 轉換方式 |
|------|---------|
| `lin` | 原始數值（無轉換）|
| `chg` | 變化量：`x_t - x_{t-1}` |
| `ch1` | 與一年前相比的變化量 |
| `pch` | 百分比變化 |
| `pc1` | 與一年前相比的百分比變化 |
| `pca` | 複合年化變化率 |
| `cch` | 連續複合變化率 |
| `cca` | 連續複合年化變化率 |
| `log` | 自然對數 |

**`frequency` 頻率代碼：**

| 代碼 | 頻率 |
|------|------|
| `d` | 每日 |
| `w` | 每週 |
| `bw` | 每兩週 |
| `m` | 每月 |
| `q` | 每季 |
| `sa` | 每半年 |
| `a` | 每年 |

#### `fred/series/search` 主要參數

- `search_text`（必填）：搜尋關鍵字
- `search_type`：`full_text`（全文搜尋）或 `series_id`（ID 子字串比對）
- `filter_variable`：`frequency`、`units`、`seasonal_adjustment`
- `filter_value`：對應 `filter_variable` 的篩選值
- `tag_names` / `exclude_tag_names`：分號分隔的標籤

---

### 3. RELEASES（發布）

用於處理資料發布（經濟統計數據的預定出版）。

| 端點 | 說明 |
|------|------|
| `GET fred/releases` | 取得所有資料發布 |
| `GET fred/releases/dates` | 取得所有發布的日期 |
| `GET fred/release` | 依 `release_id` 取得特定發布 |
| `GET fred/release/dates` | 取得特定發布的日期 |
| `GET fred/release/series` | 取得發布相關的系列 |
| `GET fred/release/sources` | 取得發布的來源 |
| `GET fred/release/tags` | 取得發布內系列的標籤 |
| `GET fred/release/related_tags` | 取得發布的相關標籤 |

---

### 4. SOURCES（來源）

資料提供者/來源的端點。

| 端點 | 說明 |
|------|------|
| `GET fred/sources` | 取得所有資料來源 |
| `GET fred/source` | 依 `source_id` 取得特定來源 |
| `GET fred/source/releases` | 取得特定來源的所有發布 |

---

### 5. TAGS（標籤）

標籤是分配給系列的元資料屬性，用於篩選和探索。

| 端點 | 說明 |
|------|------|
| `GET fred/tags` | 取得所有 FRED 標籤（可選篩選）|
| `GET fred/related_tags` | 取得與指定標籤共同出現的標籤 |
| `GET fred/tags/series` | 取得符合所有指定標籤的系列 |

**主要參數：**
- `tag_names`：分號分隔的必要標籤（如 `slovenia;food`）
- `exclude_tag_names`：分號分隔的排除標籤（如 `alcohol;quarterly`）
- `tag_group_id`：依標籤群組類別篩選
- `search_text`：匹配標籤名稱的關鍵字

---

### 6. GeoFRED / Maps API

用於地理/區域資料的獨立 API，支援 FRED Maps 功能。

**Base URL:** `https://api.stlouisfed.org/geofred/`

| 端點 | 說明 |
|------|------|
| `GET geofred/series/group` | 取得地理系列群組的元資料 |
| `GET geofred/series/data` | 取得指定日期的區域資料截面 |
| `GET geofred/regional/data` | 使用 FRED 系列群組 ID 取得區域資料 |

**主要參數：**
- `series_id` / `series_group`：系列或群組識別碼
- `region_type`：地理單位（如 `state`、`county`、`msa`）
- `date`：YYYY-MM-DD 資料快照日期
- `season`：`SA`（季節調整）、`NSA`（未季節調整）

---

## 實時期間（ALFRED 功能）

FRED 透過實時期間參數支援時間旅行查詢，可存取任意日期當時存在的歷史資料。

- `realtime_start` / `realtime_end`：定義實時窗口
- 觀測值的版本日期查詢：使用 `output_type` 2–4 和/或 `vintage_dates`
- 大多數端點預設實時期間為今天，但 `fred/series/vintagedates` 預設為完整歷史（`1776-07-04` 至 `9999-12-31`）

---

## 範例請求

```bash
# 取得 GDP 觀測值（JSON 格式）
GET https://api.stlouisfed.org/fred/series/observations?series_id=GDP&api_key=YOUR_KEY&file_type=json

# 搜尋通膨相關系列
GET https://api.stlouisfed.org/fred/series/search?search_text=inflation&api_key=YOUR_KEY&file_type=json

# 取得月度失業率（UNRATE），百分比變化
GET https://api.stlouisfed.org/fred/series/observations?series_id=UNRATE&units=pch&frequency=m&api_key=YOUR_KEY&file_type=json

# 取得所有標記 "usa" 和 "m2" 的系列
GET https://api.stlouisfed.org/fred/tags/series?tag_names=usa;m2&api_key=YOUR_KEY&file_type=json
```

---

## 常用系列 ID

| Series ID | 說明 |
|-----------|------|
| `GDP` | 美國國內生產毛額 |
| `UNRATE` | 失業率 |
| `CPIAUCSL` | 消費者物價指數（CPI）|
| `FEDFUNDS` | 聯邦基金利率 |
| `DGS10` | 10年期國債殖利率 |
| `M2SL` | M2 貨幣供給 |
| `PAYEMS` | 非農就業人數 |
| `INDPRO` | 工業生產指數 |
| `HOUST` | 新屋開工數 |
| `RSAFS` | 零售銷售 |
