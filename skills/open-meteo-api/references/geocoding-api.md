# Geocoding API

## Endpoint

- 搜索：`https://geocoding-api.open-meteo.com/v1/search`
- 按 ID 获取：`https://geocoding-api.open-meteo.com/v1/get?id=...`

## 适用场景

- 把城市名、地区名、邮编转换成经纬度
- 为后续 Forecast / Ensemble / Air Quality API 提供 `latitude` / `longitude`
- 根据 `language` 本地化地点名
- 用 `countryCode` 限制搜索范围

## 请求参数

| 参数 | 类型 | 必填 | 默认 | 说明 |
|---|---|---:|---|---|
| `name` | string | 是 | - | 搜索词。空串或仅 1 个字符会返回空结果；2 个字符只做精确匹配；3 个及以上启用模糊匹配。可传地名或邮编。 |
| `count` | integer | 否 | `10` | 返回结果数量，最大 `100`。 |
| `format` | string | 否 | `json` | 可选 `json` 或 `protobuf`。 |
| `language` | string | 否 | `en` | 返回地点名翻译；无翻译则回退英文或原生名称。要求小写。 |
| `countryCode` | string | 否 | - | ISO-3166-1 alpha2 国家码过滤，如 `CN`、`JP`、`DE`。 |
| `apikey` | string | 否 | - | 商业保留资源所需。 |

## 返回结构

成功时返回：

```json
{
  "results": [
    {
      "id": 2950159,
      "name": "Berlin",
      "latitude": 52.52437,
      "longitude": 13.41053,
      "elevation": 74.0,
      "feature_code": "PPLC",
      "country_code": "DE",
      "admin1_id": 2950157,
      "admin2_id": 0,
      "admin3_id": 6547383,
      "admin4_id": 6547539,
      "timezone": "Europe/Berlin",
      "population": 3426354,
      "postcodes": ["10967", "13347"],
      "country_id": 2921044,
      "country": "Deutschland",
      "admin1": "Berlin",
      "admin2": "",
      "admin3": "Berlin, Stadt",
      "admin4": "Berlin"
    }
  ]
}
```

### 字段说明

- `id`：地点唯一 ID，可用于 `/v1/get?id=...`
- `name`：地点名，尽量按 `language` 本地化
- `latitude` / `longitude`：WGS84 坐标
- `elevation`：海拔
- `timezone`：IANA 时区标识
- `feature_code`：GeoNames 类型码
- `country_code`：两位国家码
- `country`：国家名
- `population`：人口
- `postcodes`：相关邮编数组
- `admin1..4` / `admin1_id..4_id`：行政层级与其 ID

> 注意：空字段不会返回；例如没有 `admin4` 时，该字段会直接缺失。

## 错误返回

```json
{ "error": true, "reason": "Parameter count must be between 1 and 100." }
```

## 实战建议

### 1. 给天气 API 做前置解析

先搜：

```text
https://geocoding-api.open-meteo.com/v1/search?name=Hangzhou&count=5&language=zh&format=json
```

再把选中的 `latitude` 和 `longitude` 带入天气 API。

### 2. 处理歧义

多个同名地点时，优先展示：
- `name`
- `country`
- `admin1`
- `timezone`
- `population`

### 3. 处理国际化

- 用户中文提问可尝试 `language=zh`
- 若翻译缺失，结果可能回退英文或本地原名

## 常见坑

- 1 个字符不会给可用结果
- 2 个字符往往只有精确匹配
- `countryCode` 是过滤条件，不是必填
- 结果可能为空，调用 Forecast 前先检查 `results` 是否存在且非空
