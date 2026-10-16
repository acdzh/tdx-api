# GET /api/index

获取指数 K 线数据。

## 基本信息

| 项目 | 值 |
|---|---|
| 方法 | GET |
| 路径 | `/api/index` |
| 认证 | 无 |
| 请求头 | `User-Agent: Mozilla/5.0`（建议携带） |

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `code` | string | 是 | 指数代码，需带交易所前缀，如 `sh000001`（上证指数） |
| `type` | string | 是 | K 线类型，如 `day`、`week`、`month` |

### 常用指数代码

| 代码 | 名称 |
|---|---|
| `sh000001` | 上证指数 |
| `sh000300` | 沪深300 |
| `sz399001` | 深证成指 |
| `sz399006` | 创业板指 |

### type 可选值

| 值 | 说明 |
|---|---|
| `day` | 日 K 线 |
| `week` | 周 K 线 |
| `month` | 月 K 线 |

## 返回格式

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "Count": 100,
    "List": [
      {
        "Last": 3965120,
        "Open": 3968730,
        "High": 3977540,
        "Low": 3955490,
        "Close": 3968840,
        "Volume": 490751500,
        "Amount": 829511892992000,
        "Time": "2025-12-31T15:00:00+08:00",
        "UpCount": 1066,
        "DownCount": 1174
      }
    ]
  }
}
```

## 字段说明

| 字段 | 类型 | 说明 |
|---|---|---|
| `data.Count` | int | K 线数据条数 |
| `data.List` | array | K 线数据列表 |
| `data.List[].Last` | int | 上一周期收盘点位（单位：厘，除以 1000 得点） |
| `data.List[].Open` | int | 开盘点位（单位：厘） |
| `data.List[].High` | int | 最高点位（单位：厘） |
| `data.List[].Low` | int | 最低点位（单位：厘） |
| `data.List[].Close` | int | 收盘点位（单位：厘） |
| `data.List[].Volume` | int | 成交量 |
| `data.List[].Amount` | int | 成交额 |
| `data.List[].Time` | string | 时间 ISO 8601 格式 |
| `data.List[].UpCount` | int | 上涨家数 |
| `data.List[].DownCount` | int | 下跌家数 |

## 返回示例

### 上证指数日 K 线

请求：`GET /api/index?code=sh000001&type=day`

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "Count": 100,
    "List": [
      {
        "Last": 0,
        "Open": 3947870,
        "High": 3979990,
        "Low": 3947420,
        "Close": 3965120,
        "Volume": 525932600,
        "Amount": 887533338624000,
        "Time": "2025-12-30T15:00:00+08:00",
        "UpCount": 769,
        "DownCount": 1512
      },
      {
        "Last": 3965120,
        "Open": 3968730,
        "High": 3977540,
        "Low": 3955490,
        "Close": 3968840,
        "Volume": 490751500,
        "Amount": 829511892992000,
        "Time": "2025-12-31T15:00:00+08:00",
        "UpCount": 1066,
        "DownCount": 1174
      },
      {
        "Last": 3968840,
        "Open": 3986970,
        "High": 4025260,
        "Low": 3983580,
        "Close": 4023420,
        "Volume": 596950400,
        "Amount": 1067334172672000,
        "Time": "2026-01-05T15:00:00+08:00",
        "UpCount": 1772,
        "DownCount": 501
      }
    ]
  }
}
```

### 上证指数周 K 线

请求：`GET /api/index?code=sh000001&type=week`

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "Count": 100,
    "List": [
      {
        "Last": 0,
        "Open": 96050,
        "High": 109130,
        "Low": 95790,
        "Close": 109130,
        "Volume": 154400,
        "Amount": 594000000,
        "Time": "1990-12-21T15:00:00+08:00",
        "UpCount": 16,
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

url = "http://tdx.acdzh.com/api/index"
headers = {"User-Agent": "Mozilla/5.0"}
params = {"code": "sh000001", "type": "day"}

response = requests.get(url, headers=headers, params=params)
data = response.json()

if data["code"] == 0:
    index_data = data["data"]
    print(f"上证指数日K线，共 {index_data['Count']} 条")

    for kline in index_data["List"][-5:]:
        date = kline["Time"][:10]
        close = kline["Close"] / 1000  # 厘转点
        up = kline["UpCount"]
        down = kline["DownCount"]
        volume = kline["Volume"]
        print(f"  {date} | 收:{close:.2f} | 涨:{up} 跌:{down} | 量:{volume}")
```

### Shell (xh)

```bash
# 上证指数日K线
xh GET "http://tdx.acdzh.com/api/index?code=sh000001&type=day" User-Agent:"Mozilla/5.0"

# 上证指数周K线
xh GET "http://tdx.acdzh.com/api/index?code=sh000001&type=week" User-Agent:"Mozilla/5.0"
```

### Shell (curl)

```bash
curl -H "User-Agent: Mozilla/5.0" "http://tdx.acdzh.com/api/index?code=sh000001&type=day"
```

## 注意事项

- 指数代码必须带交易所前缀（如 `sh000001`），不能只传 `000001`
- 点位字段单位为**厘**，除以 1000 得实际点位（如 3968840 → 3968.84 点）
- `UpCount` / `DownCount` 表示上涨/下跌家数，适合补充市场宽度信息
- 成交量和成交额字段在指数中有实际值（与个股 kline 不同）
- `Amount` 数值较大，展示时建议换算为亿元
