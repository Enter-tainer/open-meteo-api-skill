# Air Quality API

## Endpoint

- `https://air-quality-api.open-meteo.com/v1/air-quality`

## 适用场景

- PM2.5 / PM10 / O3 / NO2 / SO2 / CO 等污染物预报
- 欧洲 AQI / 美国 AQI
- 紫外线、沙尘、AOD
- 欧洲花粉季花粉预报
- 当前空气质量条件 + 小时序列

## 顶层能力

- 默认返回 **5 天**，最多 **7 天**
- 同时支持 `hourly` 与 `current`
- `domains=auto` 时自动组合欧洲与全球域
- 花粉变量只在**欧洲花粉季**可用

## 数据域

| 参数值 | 含义 |
|---|---|
| `auto` | 自动组合欧洲与全球域 |
| `cams_europe` | 欧洲域 |
| `cams_global` | 全球域 |

## 数据来源摘要

| 数据集 | 区域 | 分辨率 | 时间分辨率 | 可用期 | 更新 |
|---|---|---|---|---|---|
| CAMS European Air Quality Forecast | 欧洲 | 0.1° (~11 km) | 1h | 2023-10 起 | 每 24 小时，4 天预报 |
| CAMS European Air Quality Reanalysis | 欧洲 | 0.1° (~11 km) | 1h | 2013 起 | - |
| CAMS Global Atmospheric Composition Forecasts | 全球 | 0.4° (~45 km) | 3h | 2022-08 起 | 每 12 小时，5 天预报 |
| CAMS Global Greenhouse Gas Forecast | 全球 | 0.1° (~11 km) | 3h | 2024-11 起 | 每 24 小时，5 天预报 |

## 请求参数

| 参数 | 类型 | 必填 | 默认 | 说明 |
|---|---|---:|---|---|
| `latitude`,`longitude` | float | 是 | - | WGS84 坐标；支持多地点。 |
| `hourly` | string[] | 否 | - | 小时变量列表。 |
| `current` | string[] | 否 | - | 当前条件变量。任何 hourly 变量都可用于 current。 |
| `domains` | string | 否 | `auto` | `auto` / `cams_europe` / `cams_global`。 |
| `timeformat` | string | 否 | `iso8601` | `unixtime` 为 UTC epoch seconds。 |
| `timezone` | string | 否 | `GMT` | 时区名或 `auto`。 |
| `past_days` | integer 0-92 | 否 | `0` | 追加过去天数。 |
| `forecast_days` | integer 0-7 | 否 | `5` | 预报天数。 |
| `forecast_hours`,`past_hours` | integer >0 | 否 | - | 控制小时步数。 |
| `start_date`,`end_date` | `yyyy-mm-dd` | 否 | - | 日期区间。 |
| `start_hour`,`end_hour` | `yyyy-mm-ddThh:mm` | 否 | - | 小时区间。 |
| `cell_selection` | string | 否 | `nearest` | `land` / `sea` / `nearest`。文档默认列为 `nearest`。 |
| `apikey` | string | 否 | - | 商业保留资源所需。 |

## Hourly / Current 变量

### 污染物浓度
- `pm10`
- `pm2_5`
- `carbon_monoxide`
- `carbon_dioxide`
- `nitrogen_dioxide`
- `sulphur_dioxide`
- `ozone`
- `ammonia`（仅欧洲）
- `methane`
- `dust`

### 辐射与能见相关
- `aerosol_optical_depth`
- `uv_index`
- `uv_index_clear_sky`

### 花粉（仅欧洲花粉季）
- `alder_pollen`
- `birch_pollen`
- `grass_pollen`
- `mugwort_pollen`
- `olive_pollen`
- `ragweed_pollen`

### AQI 指数
- 欧洲：
  - `european_aqi`
  - `european_aqi_pm2_5`
  - `european_aqi_pm10`
  - `european_aqi_nitrogen_dioxide`
  - `european_aqi_ozone`
  - `european_aqi_sulphur_dioxide`
- 美国：
  - `us_aqi`
  - `us_aqi_pm2_5`
  - `us_aqi_pm10`
  - `us_aqi_nitrogen_dioxide`
  - `us_aqi_ozone`
  - `us_aqi_sulphur_dioxide`
  - `us_aqi_carbon_monoxide`

## AQI 分级

### 欧洲 AQI
- 0-20：good
- 20-40：fair
- 40-60：moderate
- 60-80：poor
- 80-100：very poor
- >100：extremely poor

### 美国 AQI
- 0-50：good
- 51-100：moderate
- 101-150：unhealthy for sensitive groups
- 151-200：unhealthy
- 201-300：very unhealthy
- 301-500：hazardous

## 当前条件说明

文档强调：
- 当前条件基于 **15 分钟级天气模型数据**
- 所有 hourly 变量理论上都可以作为 `current` 请求

## 返回结构

常见字段：
- `latitude`,`longitude`
- `elevation`
- `generationtime_ms`
- `utc_offset_seconds`
- `timezone`,`timezone_abbreviation`
- `current`,`current_units`
- `hourly`,`hourly_units`

## 示例

### 当前 + 小时 AQI

```text
https://air-quality-api.open-meteo.com/v1/air-quality?latitude=31.23&longitude=121.47&current=us_aqi,european_aqi,pm2_5,pm10&hourly=pm2_5,pm10,ozone,nitrogen_dioxide&timezone=auto
```

### 欧洲域花粉查询

```text
https://air-quality-api.open-meteo.com/v1/air-quality?latitude=48.85&longitude=2.35&domains=cams_europe&hourly=alder_pollen,birch_pollen,grass_pollen,european_aqi,pm2_5&timezone=Europe/Paris
```

## 常见坑

- 花粉不是全球可用，只在**欧洲花粉季**可用
- `auto` 域会综合欧洲/全球域，边界区域结果可能与显式选域不同
- 全球域不少数据源本身是 3 小时分辨率，Open-Meteo 会做对齐展示
- AQI 是规范化指数，别与原始浓度数值混为一谈
- 如果用户只关心“今天现在空气如何”，优先返回 `current`，不要一上来堆完整 `hourly`
