# Weather Forecast API

## Endpoint

- `https://api.open-meteo.com/v1/forecast`

## 适用场景

- 常规天气预报
- 当前天气条件
- 逐小时 / 逐日天气
- 15 分钟粒度天气
- 压力层变量
- 多模型对比（通过 `models`）

## 顶层能力概览

- 默认返回 **7 天**，最多 **16 天**
- 支持 `current`、`hourly`、`daily`
- 支持 `minutely_15`（通过 `forecast_minutely_15` / `past_minutely_15` / `start_minutely_15` 等控制）
- 支持 pressure level variables
- 可按 `timezone` 返回本地时间
- 可一次请求多个地点（逗号分隔的纬经度）

## 请求参数

| 参数 | 类型 | 必填 | 默认 | 说明 |
|---|---|---:|---|---|
| `latitude`,`longitude` | float | 是 | - | WGS84 坐标；多地点可逗号分隔。美洲经度通常为负数。 |
| `elevation` | float | 否 | 自动 DEM | 统计降尺度所用海拔；传 `nan` 可禁用 downscaling，改用网格平均海拔。多地点可逗号分隔。 |
| `hourly` | string[] | 否 | - | 逐小时变量列表。可逗号分隔，或重复 `hourly=`。 |
| `daily` | string[] | 否 | - | 逐日聚合变量列表。通常应配合 `timezone`。 |
| `current` | string[] | 否 | - | 当前条件变量列表。 |
| `temperature_unit` | string | 否 | `celsius` | 设为 `fahrenheit` 可转华氏。 |
| `wind_speed_unit` | string | 否 | `kmh` | 还支持 `ms`、`mph`、`kn`。 |
| `precipitation_unit` | string | 否 | `mm` | 还支持 `inch`。 |
| `timeformat` | string | 否 | `iso8601` | 设为 `unixtime` 时返回 UTC epoch seconds。 |
| `timezone` | string | 否 | `GMT` | 时区名或 `auto`；影响时间戳与起始时间。多地点可逗号分隔。 |
| `past_days` | integer 0-92 | 否 | `0` | 追加历史天数。 |
| `forecast_days` | integer 0-16 | 否 | `7` | 预报天数。 |
| `forecast_hours`,`past_hours` | integer >0 | 否 | - | 以“当前小时”为基准控制小时步数。 |
| `forecast_minutely_15`,`past_minutely_15` | integer >0 | 否 | - | 以“当前 15 分钟步”为基准控制 15 分钟序列。 |
| `start_date`,`end_date` | `yyyy-mm-dd` | 否 | - | 按日期区间请求。 |
| `start_hour`,`end_hour` | `yyyy-mm-ddThh:mm` | 否 | - | 按小时区间请求。 |
| `start_minutely_15`,`end_minutely_15` | `yyyy-mm-ddThh:mm` | 否 | - | 按 15 分钟区间请求。 |
| `models` | string[] | 否 | `auto` | 手工指定一个或多个天气模型。 |
| `cell_selection` | string | 否 | `land` | `land` / `sea` / `nearest`。 |
| `apikey` | string | 否 | - | 商业保留资源所需。 |

## 时间与时区语义

- 不传 `timezone`：默认 `GMT`
- 传 `timezone=auto`：根据坐标自动推断当地时区
- 传 `timeformat=unixtime`：所有时间戳都变成 **UTC 秒**
- 处理 `daily` + `unixtime` 时，要重新应用 `utc_offset_seconds` 才能得到本地日期

## 返回结构

常见顶层字段：

- `latitude`,`longitude`：**实际使用的网格点中心**，可能与请求坐标略有偏差
- `elevation`
- `generationtime_ms`
- `utc_offset_seconds`
- `timezone`,`timezone_abbreviation`
- `current`,`current_units`
- `hourly`,`hourly_units`
- `daily`,`daily_units`
- `minutely_15`,`minutely_15_units`

> 多地点请求时，JSON 顶层会从单对象变成对象数组；CSV/XLSX 会增加 `location_id`。

## 变量清单

### Hourly 变量（文档可见主集合）

- 温度湿度：
  - `temperature_2m`
  - `relative_humidity_2m`
  - `dew_point_2m`
  - `apparent_temperature`
- 气压云量：
  - `pressure_msl`
  - `surface_pressure`
  - `cloud_cover`
  - `cloud_cover_low`
  - `cloud_cover_mid`
  - `cloud_cover_high`
- 风：
  - `wind_speed_10m`
  - `wind_speed_80m`
  - `wind_speed_120m`
  - `wind_speed_180m`
  - `wind_direction_10m`
  - `wind_direction_80m`
  - `wind_direction_120m`
  - `wind_direction_180m`
  - `wind_gusts_10m`
- 辐射与太阳：
  - `shortwave_radiation`
  - `direct_radiation`
  - `direct_normal_irradiance`
  - `diffuse_radiation`
  - `global_tilted_irradiance`
- 水文与农业：
  - `vapour_pressure_deficit`
  - `evapotranspiration`
  - `et0_fao_evapotranspiration`
- 降水降雪：
  - `precipitation`
  - `precipitation_probability`
  - `rain`
  - `showers`
  - `snowfall`
  - `snow_depth`
- 其他：
  - `weather_code`
  - `freezing_level_height`
  - `visibility`
  - `cape`
  - `soil_temperature_0cm`
  - `soil_temperature_6cm`
  - `soil_temperature_18cm`
  - `soil_temperature_54cm`
  - `soil_moisture_0_to_1cm`
  - `soil_moisture_1_to_3cm`
  - `soil_moisture_3_to_9cm`
  - `soil_moisture_9_to_27cm`
  - `soil_moisture_27_to_81cm`
  - `is_day`

### 15 分钟变量

Forecast API 文档单列了 15-minutely 部分；使用时要注意：
- 变量不是所有模型都支持
- 时间窗口控制参数与 hourly 不同
- 返回字段一般落在 `minutely_15` / `minutely_15_units`

### Pressure level variables

Forecast API 支持 pressure level variables；典型用法：
- 明确 pressure level 变量名
- 用 pressure-level 专用变量集合请求
- 返回的是对应气压层上的变量值

由于 pressure level 变量种类较多，实际回答时如用户明确需要高空风、温度、位势高度等，再补充具体变量名并对照上游文档。

### Daily 聚合变量

文档给出了 daily parameter definition。常见包括：
- `weather_code`
- `temperature_2m_max`
- `temperature_2m_min`
- `apparent_temperature_max`
- `apparent_temperature_min`
- `sunrise`
- `sunset`
- `daylight_duration`
- `sunshine_duration`
- `precipitation_sum`
- `rain_sum`
- `showers_sum`
- `snowfall_sum`
- `precipitation_hours`
- `precipitation_probability_max`
- `wind_speed_10m_max`
- `wind_gusts_10m_max`
- `wind_direction_10m_dominant`
- `shortwave_radiation_sum`
- `et0_fao_evapotranspiration`

> 真正输出时，只列与用户需求相关的 daily 变量；不要把 URL 塞爆。

## 多模型说明

Open-Meteo 会自动融合多国家/地区模型。文档展示了这些代表性来源：
- DWD ICON
- NOAA GFS / HRRR
- Météo-France ARPEGE / AROME
- ECMWF IFS / AIFS
- UKMO
- KMA
- JMA
- MeteoSwiss
- MET Norway
- GEM Canada
- BOM Australia
- CMA China
- KNMI / DMI / ItaliaMeteo

如果用户要求“固定模型”，可传 `models=`；否则让 `auto` 选“最佳适配模型”。

## 示例

### 最小逐小时查询

```text
https://api.open-meteo.com/v1/forecast?latitude=39.90&longitude=116.40&hourly=temperature_2m,precipitation_probability&forecast_days=3&timezone=auto
```

### 当前 + 每日摘要

```text
https://api.open-meteo.com/v1/forecast?latitude=35.68&longitude=139.76&current=temperature_2m,wind_speed_10m,is_day&daily=weather_code,temperature_2m_max,temperature_2m_min&timezone=auto
```

### 指定模型

```text
https://api.open-meteo.com/v1/forecast?latitude=52.52&longitude=13.41&hourly=temperature_2m,wind_speed_10m&models=icon,ecmwf_ifs&timezone=Europe/Berlin
```

### Python 示例

```python
import requests

url = "https://api.open-meteo.com/v1/forecast"
params = {
    "latitude": 31.23,
    "longitude": 121.47,
    "hourly": "temperature_2m,precipitation_probability",
    "daily": "weather_code,temperature_2m_max,temperature_2m_min",
    "forecast_days": 3,
    "timezone": "auto",
}
resp = requests.get(url, params=params, timeout=30)
resp.raise_for_status()
print(resp.json())
```

## 常见坑

- `daily` 通常应带 `timezone`，否则按 GMT 对日边界进行切分
- `unixtime` 是 UTC 秒，不是本地秒
- 多地点请求会改变返回结构
- `latitude` / `longitude` 响应值是实际网格点，不一定等于请求值
- `elevation=nan` 会关闭统计降尺度，结果可能明显变化
- 并非每个模型都支持所有变量和所有时间粒度
