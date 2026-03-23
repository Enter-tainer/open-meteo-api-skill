---
name: open-meteo-api
description: >
  Comprehensive guide for Open-Meteo APIs (Weather Forecast, Ensemble, Air Quality, Geocoding). Use this skill when the user asks you to: 1. 获取全球任意地点的实时/历史天气或未来天气预报; 2. 获取AQI、PM2.5等空气污染及花粉预报; 3. 探究未来天气的概率与不确定性，或对比多气象成员模型; 4. 把地名串转换为经纬度坐标; 5. 询问如何构造 API URL、参数搭配或解析响应结构。
---

# Open-Meteo API Skill

本指南详细说明如何构建与调用 Open-Meteo 的四类核心 API：**Geocoding API**, **Weather Forecast API**, **Ensemble API**, 和 **Air Quality API**。

## 💡 核心工作流与判断逻辑

当用户询问天气、空气质量或坐标时，请遵循以下处理逻辑：
1. **是否需要查询坐标？**
   - 如果用户提供的是地名（如“巴黎”、“Tokyo”）、邮编或行政区名，**必须**先调用 **Geocoding API** 将其转换为 `latitude` 和 `longitude`。
2. **选择目标 API：**
   - **常规天气/历史天气**（如温度、降水、风速、气压层）：使用 **Weather Forecast API**。
   - **不确定性/概率/多模型对比**（如不同 member 的预报、置信区间、最长 35 天长期预报）：使用 **Ensemble API**。
   - **空气质量/AQI/污染物/花粉**：使用 **Air Quality API**（注意：花粉主要限欧洲花粉季）。
3. **时间与时区处理（极易错点）：**
   - 请求时**强烈建议**带上 `timezone=auto`（根据坐标自动推断时区）或明确的时区名称（如 `Europe/Berlin`），否则默认使用 `GMT`，这会导致 `daily` 变量的日边界切分不符合当地情况。
   - 如果用户需要 UTC 时间戳，传递 `timeformat=unixtime`，但展示给用户按天聚合的数据时，必须根据返回结果中的 `utc_offset_seconds` 做修正计算。
4. **多地点请求：**
   - 所有 API（Geocoding 除外）都支持通过**逗号分隔**的 `latitude` 和 `longitude` 一次查询多个地点。
   - **坑**：多地点请求会使返回的 JSON 结构从单一对象变为**对象数组**。处理响应时必须予以区分！

---

## 1. Geocoding API (地名转坐标)

**Endpoint:** `https://geocoding-api.open-meteo.com/v1/search`

将地点名称解析为精确的 WGS84 坐标，必须作为后续气象查询的前置步骤。

### 请求参数
| 参数 | 类型 | 必填 | 说明 |
|---|---|:---:|---|
| `name` | string | **是** | 搜索词。建议至少 3 个字符以启用模糊匹配。可传地名或邮编。 |
| `count` | int | 否 | 返回结果数量，默认 `10`，最大 `100`。 |
| `language` | string | 否 | 返回地点名的翻译语言（如 `zh`, `en`, `fr`）。无翻译时回退本名。 |
| `format` | string | 否 | 默认 `json`。 |
| `countryCode` | string | 否 | ISO-3166-1 alpha-2 两位国家码过滤，例如 `DE`, `CN`, `US`。 |

### 返回结构解析
- 成功时返回包含 `results` 数组的 JSON 对象。
- 重要字段：`id`, `name`, `latitude`, `longitude`, `timezone`, `country`, `admin1`(州/省)。
- **注意**：如果搜索结果可能存在歧义，应向用户展示候选列表（显示国家/地区和 `population` 帮助区分），或自动选择 `population` 最大的相关项。

---

## 2. Weather Forecast API (常规天气预报)

**Endpoint:** `https://api.open-meteo.com/v1/forecast`

提供当前天气、逐小时（hourly）、15分钟级（minutely_15）和逐日（daily）的天气预报。默认预报 7 天，最多可达 16 天。

### 常用请求参数
| 参数 | 必填 | 默认 | 说明 |
|---|:---:|---|---|
| `latitude`, `longitude` | **是** | - | 坐标（WGS84）。多地点可用逗号分隔。 |
| `current` | 否 | - | 需要的当前天气变量列表（逗号分隔）。 |
| `hourly` | 否 | - | 需要的逐小时变量列表。 |
| `daily` | 否 | - | 需要的逐日变量列表。必须配合 `timezone` 使用！ |
| `timezone` | 否 | `GMT` | 设为 `auto` 则由 API 自动根据经纬度推断。 |
| `forecast_days` | 否 | `7` | 预报天数（0 到 16天）。 |
| `past_days` | 否 | `0` | 包含过去的天数（0 到 92天）。 |
| `models` | 否 | `auto` | 指定天气模型（如 `icon_global`, `gfs_global`, `ecmwf_ifs04`）。默认自动融合最优。 |
| `temperature_unit` | 否 | `celsius` | 支持 `celsius` (默认) 或 `fahrenheit`。 |
| `wind_speed_unit` | 否 | `kmh` | 支持 `kmh`, `ms`, `mph`, `kn`。 |
| `precipitation_unit`| 否 | `mm` | 支持 `mm` 或 `inch`。 |

### 常见核心变量
- **Current / Hourly**: `temperature_2m` (温度), `relative_humidity_2m` (湿度), `apparent_temperature` (体感), `precipitation_probability` (降水概率), `precipitation` (降水量), `weather_code` (WMO天气代码), `wind_speed_10m` (风速), `cloud_cover` (云量), `pressure_msl` (海面气压)。
- **Daily**: `weather_code`, `temperature_2m_max`, `temperature_2m_min`, `sunrise`, `sunset`, `precipitation_sum`, `precipitation_probability_max`, `wind_speed_10m_max`。

*详细的全量变量清单，请需要时查阅 `references/forecast-api.md`。*

### 请求示例
```text
https://api.open-meteo.com/v1/forecast?latitude=52.52&longitude=13.41&current=temperature_2m,wind_speed_10m&hourly=temperature_2m,precipitation_probability&daily=weather_code,temperature_2m_max,temperature_2m_min&timezone=auto
```

---

## 3. Ensemble API (集合模型与不确定性预报)

**Endpoint:** `https://ensemble-api.open-meteo.com/v1/ensemble`

用于评估天气预报的**不确定性**和**概率**。它返回一个集合中每个成员（member）各自独立的预报数据列。

### 核心区别与参数
1. **必须显式提供 `models` 参数**：如 `models=gfs05` (35天长预报, 31成员), `ecmwf_ifs04` (15天, 51成员), `icon_eps` (7.5天, 40成员)。
2. 返回结果不是单一时间序列，而是包含多个成员的数据阵列（例如 `temperature_2m_member01`, `temperature_2m_member02` ... 对应不同 member）。
3. 时间步长会被统一插值为 1 小时展示，但远期原生数据分辨率可能是 3 小时或 6 小时。

### 常用请求参数
除 Forecast API 通用参数外：
| 参数 | 必填 | 说明 |
|---|:---:|---|
| `models` | **是** | 选择的集合模型，支持逗号分隔多个。 |
| `hourly` | 否 | 选择主要基础气象变量（通常支持温度、降水、风、辐射等）。 |
| `forecast_days`| 否 | 天数最高可达 **35** 天（依据选用的模型而定，如 `gfs05` 可达35天）。 |

*完整支持的模型和变量细目可查阅 `references/ensemble-api.md`。*

### 请求示例
```text
https://ensemble-api.open-meteo.com/v1/ensemble?latitude=35.68&longitude=139.76&models=gfs05,ecmwf_ifs04&hourly=temperature_2m,precipitation&forecast_days=14&timezone=auto
```
*提示：在将数据解释给最终用户时，建议对多个 member 取平均值、中位数或给出置信范围（最小值 - 最大值）。*

---

## 4. Air Quality API (空气质量与花粉)

**Endpoint:** `https://air-quality-api.open-meteo.com/v1/air-quality`

提供全球与欧洲区域的污染物、AQI 指数、AOD 以及花粉预报。默认预报 5 天，最大可达 7 天。

### 常用请求参数
| 参数 | 必填 | 默认 | 说明 |
|---|:---:|---|---|
| `latitude`, `longitude` | **是** | - | 坐标（WGS84）。多地点可用逗号分隔。 |
| `current` | 否 | - | 当前污染物或 AQI 变量。任何 hourly 变量皆可用作 current 变量。 |
| `hourly` | 否 | - | 逐小时污染物或 AQI 变量列表。 |
| `domains` | 否 | `auto` | 数据域：`auto`, `cams_europe` (欧洲更高精度), `cams_global` (全球)。 |
| `timezone` | 否 | `GMT` | 设为 `auto` 让系统自动通过经纬度匹配当地时区。 |
| `forecast_days` | 否 | `5` | 预报天数（0 到 7天）。 |
| `past_days` | 否 | `0` | 包含过去的天数（0 到 92天）。 |

### 核心变量
- **污染物**: `pm10`, `pm2_5`, `carbon_monoxide`, `nitrogen_dioxide`, `sulphur_dioxide`, `ozone`, `aerosol_optical_depth`, `dust`, `uv_index`.
- **AQI (空气质量指数)**: `european_aqi`, `us_aqi` 及各子项 AQI（如 `us_aqi_pm2_5`）。
- **花粉**: `alder_pollen`(桤木), `birch_pollen`(桦树), `grass_pollen`(草本), `olive_pollen`(橄榄) 等。

*注意：`us_aqi` 的级别为 Good(0-50), Moderate(51-100), Unhealthy for Sensitive(101-150), Unhealthy(151-200), Very Unhealthy(201-300), Hazardous(>300)。*

### 请求示例
```text
https://air-quality-api.open-meteo.com/v1/air-quality?latitude=31.23&longitude=121.47&current=us_aqi,pm2_5,pm10&hourly=us_aqi,ozone&timezone=auto
```

---

## 5. 典型返回结构与数据解析 (Response Format)

为了让 AI 代理或业务层更容易抽取数据，请注意 Open-Meteo 的 JSON 返回往往是以**时间序列数组**为核心组织的：

### 常规 Forecast / Air Quality 响应结构
```json
{
  "latitude": 52.52,
  "longitude": 13.419,
  "timezone": "Europe/Berlin",
  "utc_offset_seconds": 7200,
  "hourly": {
    "time": ["2022-07-01T00:00", "2022-07-01T01:00", "2022-07-01T02:00"],
    "temperature_2m": [13.0, 12.7, 12.7],
    "precipitation_probability": [0, 10, 20]
  },
  "hourly_units": {
    "temperature_2m": "°C",
    "precipitation_probability": "%"
  }
}
```
*解析技巧：所有的气象变量数组（如 `temperature_2m`）与 `time` 数组都是**一一对应、等长同序**的。例如 `temperature_2m[5]` 就是 `time[5]` 时刻的温度。*

### Ensemble 响应结构差异
集合预报的返回会将不同的预报 member 作为单独的键名（追加后缀）区分：
```json
{
  "hourly": {
    "time": ["2022-07-01T00:00", ...],
    "temperature_2m_member01": [13.0, ...],
    "temperature_2m_member02": [12.5, ...]
  }
}
```
*解析技巧：提取不确定性数据时，需要遍历含有 `_memberXX` 后缀的键，或者在提取时对同一时间点的多成员数组求最值、均值。*

---

## ⚠️ 常见排错与避坑指南 (Troubleshooting)

1. **"找不到城市" 或 "查询不到天气的报错"**
   - **必纠**：必须先使用 **Geocoding API** 将文本地址转换为经纬度，API 无法直接接受地名串。
2. **"返回的 daily 聚合数据不属于当地的一天"**
   - **必纠**：未传入或传入了错误的 `timezone`。查询 daily 时强迫症般地加上 `timezone=auto`。
3. **"花粉预报找不到数据或报错"**
   - **必纠**：花粉数据主要依赖欧洲模型（`cams_europe` 域），对于亚洲、美洲或其他非欧洲城市请求花粉常常无效，请提示用户不支持此区域。
4. **"API 报错 HTTP 400"**
   - **必纠**：参数名拼写错误（如把 `temperature_2m` 错拼成 `temp_2m`），或者非法的时间范围配置。
5. **多个地点的 JSON 解析失败**
   - **必纠**：如果逗号分隔穿入了多组经纬度，API 最外层 JSON 也是一个**数组**（`[ {地点1响应}, {地点2响应} ]`），这与请求单一地点的普通对象有显著变更！在脚本中请做兼容处理。
