# GET /api/kline

获取股票 K 线数据（日线、周线、月线、分钟线等）。

## 基本信息

| 项目 | 值 |
|---|---|
| 方法 | GET |
| 路径 | `/api/kline` |
| 认证 | 无 |
| 请求头 | `User-Agent: Mozilla/5.0`（建议携带） |

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `code` | string | 是 | 股票代码，如 `000001` |
| `type` | string | 是 | K 线类型，见下方可选值 |

### type 可选值

| 值 | 说明 |
|---|---|
| `day` | 日 K 线 |
| `week` | 周 K 线 |
| `month` | 月 K 线 |
| `minute` | 1 分钟 K 线 |
| `minute5` | 5 分钟 K 线 |
| `minute15` | 15 分钟 K 线 |
| `minute30` | 30 分钟 K 线 |
| `minute60` | 60 分钟 K 线 |

## 返回格式

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "Count": 100,
    "List": [
      {
        "Last": 10990,
        "Open": 10980,
        "High": 11100,
        "Low": 10940,
        "Close": 11080,
        "Volume": 885428,
        "Amount": 0,
        "Time": "2026-06-02T15:00:00+08:00",
        "UpCount": 0,
        "DownCount": 0
      }
    ]
  }
}
```

## 字段说明

| 字段 | 类型 | 说明 |
|---|---|---|
| `Count` | int | K 线数据条数 |
| `List` | array | K 线数据列表 |
| `List[].Last` | int | 上一周期收盘价（单位：厘） |
| `List[].Open` | int | 开盘价（单位：厘） |
| `List[].High` | int | 最高价（单位：厘） |
| `List[].Low` | int | 最低价（单位：厘） |
| `List[].Close` | int | 收盘价（单位：厘） |
| `List[].Volume` | int | 成交量（单位：手） |
| `List[].Amount` | int | 成交额（部分类型可能为 0） |
| `List[].Time` | string | 时间，ISO 8601 格式带时区 |
| `List[].UpCount` | int | 上涨数（个股 K 线中通常为 0） |
| `List[].DownCount` | int | 下跌数（个股 K 线中通常为 0） |

## 返回示例

### 日 K 线

请求：`GET /api/kline?code=000001&type=day`

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "Count": 100,
    "List": [
      {
        "Last": 10990,
        "Open": 10980,
        "High": 11100,
        "Low": 10940,
        "Close": 11080,
        "Volume": 885428,
        "Amount": 0,
        "Time": "2026-06-02T15:00:00+08:00",
        "UpCount": 0,
        "DownCount": 0
      },
      {
        "Last": 10930,
        "Open": 10900,
        "High": 10990,
        "Low": 10810,
        "Close": 10990,
        "Volume": 954596,
        "Amount": 0,
        "Time": "2026-06-01T15:00:00+08:00",
        "UpCount": 0,
        "DownCount": 0
      }
    ]
  }
}
```

### 30 分钟 K 线

请求：`GET /api/kline?code=000001&type=minute30`

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "Count": 3987,
    "List": [
      {
        "Last": 0,
        "Open": 11000,
        "High": 11060,
        "Low": 10950,
        "Close": 10950,
        "Volume": 337194,
        "Amount": 370934144000,
        "Time": "2024-05-14T10:00:00+08:00",
        "UpCount": 0,
        "DownCount": 0
      },
      {
        "Last": 10950,
        "Open": 10950,
        "High": 10960,
        "Low": 10890,
        "Close": 10920,
        "Volume": 206390,
        "Amount": 225320048000,
        "Time": "2024-05-14T10:30:00+08:00",
        "UpCount": 0,
        "DownCount": 0
      }
    ]
  }
}
```

### 周 K 线

请求：`GET /api/kline?code=000001&type=week`

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "Count": 1789,
    "List": [
      {
        "Last": 0,
        "Open": -2720,
        "High": -2720,
        "Low": -2730,
        "Close": -2730,
        "Volume": 13,
        "Amount": 0,
        "Time": "1991-04-06T15:00:00+08:00",
        "UpCount": 0,
        "DownCount": 0
      }
    ]
  }
}
```

### 月 K 线

请求：`GET /api/kline?code=000001&type=month`

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "Count": 421,
    "List": [
      {
        "Last": 0,
        "Open": -2720,
        "High": -2720,
        "Low": -2770,
        "Close": -2770,
        "Volume": 134,
        "Amount": 0,
        "Time": "1991-04-30T15:00:00+09:00",
        "UpCount": 0,
        "DownCount": 0
      }
    ]
  }
}
```

## 代码示例

### Python

```python
import requests

url = "http://tdx.acdzh.com/api/kline"
headers = {"User-Agent": "Mozilla/5.0"}
params = {"code": "000001", "type": "day"}

response = requests.get(url, headers=headers, params=params)
data = response.json()

if data["code"] == 0:
    klines = data["data"]["List"]
    print(f"共 {data['data']['Count']} 条 K 线数据")

    for kline in klines[-5:]:  # 取最近 5 条
        time = kline["Time"][:10]
        open_p = kline["Open"] / 1000
        high_p = kline["High"] / 1000
        low_p = kline["Low"] / 1000
        close_p = kline["Close"] / 1000
        volume = kline["Volume"]
        print(f"{time} | 开:{open_p:.2f} 高:{high_p:.2f} 低:{low_p:.2f} 收:{close_p:.2f} 量:{volume}手")
```

### Shell (xh)

```bash
# 日 K 线
xh GET "http://tdx.acdzh.com/api/kline?code=000001&type=day" User-Agent:"Mozilla/5.0"

# 周 K 线
xh GET "http://tdx.acdzh.com/api/kline?code=000001&type=week" User-Agent:"Mozilla/5.0"

# 30分钟 K 线
xh GET "http://tdx.acdzh.com/api/kline?code=000001&type=minute30" User-Agent:"Mozilla/5.0"
```

### Shell (curl)

```bash
curl -H "User-Agent: Mozilla/5.0" "http://tdx.acdzh.com/api/kline?code=000001&type=day"
```

## 注意事项

- 价格字段单位为**厘**，展示时需除以 1000 转换为元
- 成交量单位为**手**
- `Amount` 在部分 type 下可能为 0（如 `day` 类型），不要依赖此字段做计算
- 早期历史数据中 `Open`/`Close` 等可能出现负数值（复权相关），属于数据源原始特性
- 日 K 线默认返回约 100 条最近数据
- 分钟级 K 线返回数据量更大（数千条）
- 若需完整历史 K 线，应使用 `/api/kline-all/tdx` 或 `/api/kline-all/ths`
