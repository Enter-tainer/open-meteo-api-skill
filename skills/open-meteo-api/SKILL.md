---
name: open-meteo-api
description: Open-Meteo APIs for weather forecast, ensemble forecast, air quality, and geocoding. Use when the task involves converting place names to coordinates, querying Open-Meteo forecast data, choosing forecast/current/daily/hourly variables, using ensemble members or probabilistic weather models, retrieving air-quality or pollen forecasts, explaining request parameters/response schemas/units/timezone handling, or generating example Open-Meteo request URLs and code snippets.
---

# Open-Meteo API

使用这个 skill 来稳定地讲解和调用 Open-Meteo 的四类 API：
- Weather Forecast API
- Ensemble API
- Air Quality API
- Geocoding API

## 快速工作流

1. **先确定输入是地名还是坐标**
   - 地名 / 邮编 → 先读 `references/geocoding-api.md`
   - 已有纬度经度 → 直接进入目标 API
2. **再确定数据类型**
   - 常规天气预报 / 当前天气 / 日聚合 / 15 分钟数据 / 压力层 → `references/forecast-api.md`
   - 多成员集合预报 / 不确定性 / 长时段概率视角 → `references/ensemble-api.md`
   - 空气质量 / 花粉 / AQI → `references/air-quality-api.md`
3. **输出时默认补齐这些关键点**
   - endpoint
   - 必填参数
   - 时间与时区语义
   - 变量名与单位
   - 返回结构关键字段
   - 1~3 个可直接复制的请求示例
   - 常见坑

## 先做的判断

### 1. 地名是否需要先转坐标

如果用户给的是城市名、地区名、邮编、模糊地点：
- 先用 Geocoding API 搜索
- 如结果歧义较大，列出候选（国家、admin1、timezone、population）
- 再拿选中的 `latitude` / `longitude` 去调天气或空气质量 API

### 2. 用户到底想要哪类“时间粒度”

- **current**：当前时刻条件
- **hourly**：逐小时序列
- **minutely_15**：15 分钟序列（仅 Forecast API）
- **daily**：日聚合序列
- **pressure_level**：气压层变量（仅 Forecast API）

### 3. 用户是要“最优单一预报”还是“概率/不确定性”

- 常规天气查询 → Weather Forecast API
- 想看多个 member、置信区间、长期不确定性 → Ensemble API

## 输出规范

回答 Open-Meteo 相关问题时，优先按下面格式组织：

### A. 最短可用答案
- 这是什么 API
- 该用哪个 endpoint
- 最小可工作的 URL 示例

### B. 参数解释
- 必填参数
- 常用可选参数
- 互相制约的参数（例如 `daily` 常常需要 `timezone`）
- 时间窗口参数的优先级与替代关系

### C. 返回结构
- 顶层字段
- `*_units`
- `hourly` / `daily` / `current` / `hourly_units` / `daily_units`
- 多地点请求时返回数组这一点必须特别说明

### D. 常见坑
- `timezone` 不传时默认 GMT
- `unixtime` 是 UTC 秒；按日数据要重新应用 `utc_offset_seconds`
- 多地点请求会改变 JSON 结构
- 部分变量仅特定域/地区/模型可用
- `cell_selection`、`elevation` 会影响结果代表的网格点

## 参考文件导航

- `references/geocoding-api.md`：地名搜索、ID 解析、语言/国家过滤
- `references/forecast-api.md`：Forecast API 全面说明，包括 hourly/daily/current/15-minutely/pressure level
- `references/ensemble-api.md`：集合预报模型、成员、时长、参数与变量
- `references/air-quality-api.md`：污染物、花粉、欧洲/美国 AQI、domain 选择

如果需要对照上游措辞或补缺少的细项，再读：
- `references/forecast.upstream-notes.md`
- `references/ensemble.upstream-notes.md`
- `references/air-quality.upstream-notes.md`
- `references/geocoding.upstream-notes.md`

## 示例输出风格

### 用户问“柏林未来 3 天天气”

先 Geocoding（如果没有坐标），再给：

```text
https://api.open-meteo.com/v1/forecast?latitude=52.52&longitude=13.41&hourly=temperature_2m,precipitation_probability&daily=weather_code,temperature_2m_max,temperature_2m_min&forecast_days=3&timezone=auto
```

并解释：
- `hourly` 给逐小时温度与降水概率
- `daily` 给每日摘要
- `timezone=auto` 让时间戳按当地时区返回

### 用户问“东京空气质量和花粉”

优先提醒：
- 花粉变量只在**欧洲花粉季**可用
- 东京可查 AQI / PM / O3 / NO2 等，但通常没有欧洲花粉变量

### 用户问“未来 15 天不确定性”

切到 Ensemble API，并解释：
- 需要显式传 `models`
- 看不同 ensemble model 的 forecast horizon
- 返回是 **按 ensemble member** 的逐小时数据

## 写示例时的要求

- 默认给 **curl URL** 与 **JavaScript fetch** 或 **Python requests** 二选一
- 示例变量只放用户真正需要的，避免把 URL 拉得过长
- 如果用户问“完整参数/完整变量”，再展开对应 reference 中的完整清单
