# GET /api/kline-all/tdx

获取通达信原始全量 K 线数据（不复权）。

## 基本信息

| 项目 | 值 |
|---|---|
| 方法 | GET |
| 路径 | `/api/kline-all/tdx` |
| 认证 | 无 |
| 请求头 | `User-Agent: Mozilla/5.0`（建议携带） |
| 超时建议 | ≥10 秒（数据量大时响应较慢） |

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
| `quarter` | 季 K 线 |
| `year` | 年 K 线 |
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
    "count": 8381,
    "list": [
      {
        "Last": 0,
        "Open": 49000,
        "High": 49000,
        "Low": 49000,
        "Close": 49000,
        "Volume": 32768,
        "Amount": 5000000,
        "Time": "1991-04-03T15:00:00+08:00",
        "UpCount": 0,
        "DownCount": 0
      }
    ],
    "meta": {
      "batch_limit": 800,
      "notes": [...],
      "source": "tdx",
      "type": "day"
    }
  }
}
```

## 字段说明

| 字段 | 类型 | 说明 |
|---|---|---|
| `data.count` | int | K 线总条数 |
| `data.list` | array | K 线数据列表（全量，从上市至今） |
| `data.list[].Last` | int | 上一周期收盘价（单位：厘） |
| `data.list[].Open` | int | 开盘价（单位：厘） |
| `data.list[].High` | int | 最高价（单位：厘） |
| `data.list[].Low` | int | 最低价（单位：厘） |
| `data.list[].Close` | int | 收盘价（单位：厘） |
| `data.list[].Volume` | int | 成交量（手） |
| `data.list[].Amount` | int | 成交额（有实际值） |
| `data.list[].Time` | string | 时间 ISO 8601 格式 |
| `data.list[].UpCount` | int | 上涨数（个股通常为 0） |
| `data.list[].DownCount` | int | 下跌数（个股通常为 0） |
| `data.meta` | object | 元数据 |
| `data.meta.batch_limit` | int | 底层单次请求限制（800 条） |
| `data.meta.source` | string | 数据源标识 `"tdx"` |
| `data.meta.type` | string | K 线类型 |
| `data.meta.notes` | array | 使用提示 |

## 返回示例

### 日 K 线

请求：`GET /api/kline-all/tdx?code=000001&type=day`

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "count": 8381,
    "list": [
      {
        "Last": 0,
        "Open": 49000,
        "High": 49000,
        "Low": 49000,
        "Close": 49000,
        "Volume": 32768,
        "Amount": 5000000,
        "Time": "1991-04-03T15:00:00+08:00",
        "UpCount": 0,
        "DownCount": 0
      },
      {
        "Last": 49000,
        "Open": 48760,
        "High": 48760,
        "Low": 48760,
        "Close": 48760,
        "Volume": 4098,
        "Amount": 15000000,
        "Time": "1991-04-04T15:00:00+08:00",
        "UpCount": 0,
        "DownCount": 0
      },
      {
        "Last": 10990,
        "Open": 10980,
        "High": 11100,
        "Low": 10940,
        "Close": 11080,
        "Volume": 885428,
        "Amount": 978159360000,
        "Time": "2026-06-02T15:00:00+08:00",
        "UpCount": 0,
        "DownCount": 0
      }
    ],
    "meta": {
      "batch_limit": 800,
      "notes": [
        "通达信单次底层请求最多返回 800 条数据，服务端已顺序拼接全量结果",
        "对于上市时间较长的标的，请预估调用耗时（通常 1-5 秒），客户端可增加超时时间"
      ],
      "source": "tdx",
      "type": "day"
    }
  }
}
```

### 30 分钟 K 线

请求：`GET /api/kline-all/tdx?code=000001&type=minute30`

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "count": 3987,
    "list": [
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
      }
    ],
    "meta": {
      "batch_limit": 800,
      "notes": [
        "通达信单次底层请求最多返回 800 条数据，服务端已顺序拼接全量结果",
        "对于上市时间较长的标的，请预估调用耗时（通常 1-5 秒），客户端可增加超时时间"
      ],
      "source": "tdx",
      "type": "minute30"
    }
  }
}
```

## 代码示例

### Python

```python
import requests

url = "http://tdx.acdzh.com/api/kline-all/tdx"
headers = {"User-Agent": "Mozilla/5.0"}
params = {"code": "000001", "type": "day"}

# 设置较长超时
response = requests.get(url, headers=headers, params=params, timeout=30)
data = response.json()

if data["code"] == 0:
    result = data["data"]
    klines = result["list"]
    print(f"数据源: {result['meta']['source']}")
    print(f"类型: {result['meta']['type']}")
    print(f"总条数: {result['count']}")

    # 最早数据
    first = klines[0]
    print(f"最早: {first['Time'][:10]} 收:{first['Close']/1000:.2f}")

    # 最新数据
    last = klines[-1]
    print(f"最新: {last['Time'][:10]} 收:{last['Close']/1000:.2f}")

    # 计算某段时间的涨幅
    recent_30 = klines[-30:]
    start_price = recent_30[0]["Close"] / 1000
    end_price = recent_30[-1]["Close"] / 1000
    change_pct = (end_price - start_price) / start_price * 100
    print(f"近30日涨幅: {change_pct:+.2f}%")
```

### Shell (xh)

```bash
# 日K线全量
xh --timeout=30 GET "http://tdx.acdzh.com/api/kline-all/tdx?code=000001&type=day" User-Agent:"Mozilla/5.0"

# 30分钟K线全量
xh --timeout=30 GET "http://tdx.acdzh.com/api/kline-all/tdx?code=000001&type=minute30" User-Agent:"Mozilla/5.0"
```

### Shell (curl)

```bash
curl --max-time 30 -H "User-Agent: Mozilla/5.0" "http://tdx.acdzh.com/api/kline-all/tdx?code=000001&type=day"
```

## 注意事项

- **这是不复权的原始数据**，价格为通达信原始口径
- 价格字段单位为**厘**，需除以 1000 转元
- 数据从股票上市日起至今，数据量很大（如 000001 日K有 8000+ 条）
- 响应时间通常 1-5 秒，客户端应设置 ≥10 秒超时
- 服务端已将底层 800 条/次的限制自动拼接为全量结果
- `Amount` 在日K线中有实际值（与 `/api/kline` 不同）
- 若需要前复权数据，应使用 `/api/kline-all/ths`
- 注意：返回字段名为小写 `count`/`list`（不是大写 `Count`/`List`）
