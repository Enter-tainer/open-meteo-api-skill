# Ensemble API

## Endpoint

- `https://ensemble-api.open-meteo.com/v1/ensemble`

## 适用场景

- 关注未来天气**不确定性**
- 需要多个 ensemble members
- 需要更长时段概率视角
- 需要比较不同集合模型的 forecast horizon / 分辨率 / member 数量

## 核心特点

- 返回的是 **每个 ensemble member 的逐小时预报**
- 必须显式传 `models`
- 默认 7 天，最长可到 **35 天**（取决于模型）
- 所有数据被插值成 **1 小时步长**；越往远期，一些模型可能退化为 6 小时有效分辨率

## 主要模型（文档列出）

| 机构 | 模型 | 区域 | 分辨率/时间步 | Members | 时长 | 更新 |
|---|---|---|---|---:|---|---|
| DWD | `icon_d2_eps` | 中欧 | 2 km / 1h | 20 | 2 天 | 每 3 小时 |
| DWD | `icon_eu_eps` | 欧洲 | 13 km / 1h | 40 | 5 天 | 每 6 小时 |
| DWD | `icon_eps` | 全球 | 26 km / 1h | 40 | 7.5 天 | 每 12 小时 |
| NOAA | `gfs025` ensemble | 全球 | 25 km / 3h | 31 | 10 天 | 每 6 小时 |
| NOAA | `gfs05` ensemble | 全球 | 50 km / 3h | 31 | 35 天 | 每 6 小时 |
| NOAA | `aigfs` | 全球 | 25 km / 6h | 31 | 16 天 | 每 6 小时 |
| ECMWF | `ecmwf_ifs04` ensemble | 全球 | 25 km / 3h | 51 | 15 天 | 每 6 小时 |
| ECMWF | `ecmwf_aifs04` ensemble | 全球 | 25 km / 6h | 51 | 15 天 | 每 6 小时 |
| CWS | `gem_global` ensemble | 全球 | 25 km / 3h | 21 | 16 天（部分批次 39 天） | 每 12 小时 |
| BOM | `access_ge` | 全球 | 40 km / 3h | 18 | 10 天 | 每 6 小时 |
| UKMO | `mogreps_uk` | 英国 | 2 km / 1h | 3 | 5 天 | 每小时 |
| UKMO | `mogreps_g` | 全球 | 20 km / 1h | 18 | 8 天 | 每 6 小时 |
| MeteoSwiss | `icon_ch1` | 中欧 | 1 km / 1h | 11 | 33 小时 | 每 3 小时 |
| MeteoSwiss | `icon_ch2` | 中欧 | 2 km / 1h | 21 | 12 小时 | 每 6 小时 |

> 实际可用 `models` 名称应以当前上游文档/接口为准；回答时若用户要求精确模型名，优先核对 upstream notes。

## 请求参数

| 参数 | 类型 | 必填 | 默认 | 说明 |
|---|---|---:|---|---|
| `latitude`,`longitude` | float | 是 | - | WGS84 坐标；支持多地点。 |
| `models` | string[] | 是 | - | 显式选择一个或多个 ensemble 模型。 |
| `elevation` | float | 否 | 自动 DEM | 统计降尺度海拔；`nan` 可关闭。 |
| `hourly` | string[] | 否 | - | 逐小时变量列表。 |
| `temperature_unit` | string | 否 | `celsius` | `fahrenheit` 可转华氏。 |
| `wind_speed_unit` | string | 否 | `kmh` | 还支持 `ms` / `mph` / `kn`。 |
| `precipitation_unit` | string | 否 | `mm` | 还支持 `inch`。 |
| `timeformat` | string | 否 | `iso8601` | `unixtime` 返回 UTC epoch seconds。 |
| `timezone` | string | 否 | `GMT` | 时区名或 `auto`。 |
| `past_days` | integer | 否 | `0` | 可包含过去数据。 |
| `forecast_days` | integer 0-35 | 否 | `7` | 预报天数。 |
| `forecast_hours`,`past_hours` | integer >0 | 否 | - | 以当前小时为基准控制步数。 |
| `start_date`,`end_date` | `yyyy-mm-dd` | 否 | - | 日期区间。 |
| `start_hour`,`end_hour` | `yyyy-mm-ddThh:mm` | 否 | - | 小时区间。 |
| `cell_selection` | string | 否 | `land` | `land` / `sea` / `nearest`。 |
| `apikey` | string | 否 | - | 商业保留资源所需。 |

## Hourly 变量

文档给出的主要变量包括：

- 温度湿度：
  - `temperature_2m`
  - `relative_humidity_2m`
  - `dew_point_2m`
  - `apparent_temperature`
- 气压云量：
  - `pressure_msl`
  - `surface_pressure`
  - `cloud_cover`
- 风：
  - `wind_speed_10m`
  - `wind_speed_80m`
  - `wind_speed_100m`
  - `wind_speed_120m`
  - `wind_direction_10m`
  - `wind_direction_80m`
  - `wind_direction_100m`
  - `wind_direction_120m`
  - `wind_gusts_10m`
- 辐射：
  - `shortwave_radiation`
  - `direct_radiation`
  - `direct_normal_irradiance`
  - `diffuse_radiation`
  - `global_tilted_irradiance`
  - `sunshine_duration`
- 水文与地表：
  - `vapour_pressure_deficit`
  - `evapotranspiration`
  - `et0_fao_evapotranspiration`
  - `surface_temperature`
  - `soil_temperature_0_to_10cm`
  - `soil_temperature_10_to_40cm`
  - `soil_temperature_40_to_100cm`
  - `soil_temperature_100_to_200cm`
  - `soil_moisture_0_to_10cm`
  - `soil_moisture_10_to_40cm`
  - `soil_moisture_40_to_100cm`
  - `soil_moisture_100_to_200cm`
- 降水与对流：
  - `weather_code`
  - `precipitation`
  - `rain`
  - `snowfall`
  - `snow_depth`
  - `freezing_level_height`
  - `visibility`
  - `cape`

## Daily 变量

Ensemble API 文档也给出了 daily parameter definition。适合生成：
- 日最高/最低温
- 日总降水 / 降雪
- 日 dominant wind direction
- 日最大阵风
- 日 weather code

回答时只展开与用户任务相关的 daily 变量即可。

## 返回结构与解释建议

- 顶层会有网格点坐标、时区、偏移、生成耗时
- 核心数据在逐小时结构中
- 由于这是集合预报，实际消费时需要注意 **member 维度**
- 给用户解释结果时，优先转成：
  - 均值 / 中位数
  - 最小-最大区间
  - P10 / P50 / P90（如果你在应用层自己做统计）

## 示例

### 多模型 ensemble 请求

```text
https://ensemble-api.open-meteo.com/v1/ensemble?latitude=35.68&longitude=139.76&models=ecmwf_ifs04,gfs05&hourly=temperature_2m,precipitation,wind_speed_10m&forecast_days=10&timezone=auto
```

### 长时段不确定性

```text
https://ensemble-api.open-meteo.com/v1/ensemble?latitude=52.52&longitude=13.41&models=gfs05&hourly=temperature_2m,precipitation&forecast_days=16&timezone=Europe/Berlin
```

## 常见坑

- `models` 是必填，不像 Forecast API 默认 `auto`
- 不同模型支持的最长天数不同
- 返回值时间步统一为 1 小时展示，但远期信息可能本质上只有 3 小时或 6 小时分辨率
- 集合预报不是“最准单值预报”，而是用来表达不确定性范围
