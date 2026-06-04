# GET /api/kline-all/ths

获取同花顺前复权全量 K 线数据。

## 基本信息

| 项目 | 值 |
|---|---|
| 方法 | GET |
| 路径 | `/api/kline-all/ths` |
| 认证 | 无 |
| 请求头 | `User-Agent: Mozilla/5.0`（建议携带） |
| 超时建议 | ≥10 秒（数据量大时响应较慢，通常 2-8 秒） |

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `code` | string | 是 | 股票代码，如 `000001` |
| `type` | string | 是 | K 线类型，只支持 `day` / `week` / `month` |

### type 可选值

| 值 | 说明 |
|---|---|
| `day` | 日 K 线（前复权） |
| `week` | 周 K 线（前复权） |
| `month` | 月 K 线（前复权） |

## 返回格式

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "count": 8420,
    "list": [
      {
        "Last": 0,
        "Open": -2720,
        "High": -2720,
        "Low": -2720,
        "Close": -2720,
        "Volume": 1,
        "Amount": 0,
        "Time": "1991-04-03T15:00:00+08:00",
        "UpCount": 0,
        "DownCount": 0
      }
    ],
    "meta": {
      "batch_limit": 8420,
      "notes": [...],
      "source": "ths",
      "type": "day"
    }
  }
}
```

## 字段说明

| 字段 | 类型 | 说明 |
|---|---|---|
| `data.count` | int | K 线总条数 |
| `data.list` | array | K 线数据列表（全量，从上市至今，前复权） |
| `data.list[].Last` | int | 上一周期收盘价（单位：厘，前复权） |
| `data.list[].Open` | int | 开盘价（单位：厘，前复权） |
| `data.list[].High` | int | 最高价（单位：厘，前复权） |
| `data.list[].Low` | int | 最低价（单位：厘，前复权） |
| `data.list[].Close` | int | 收盘价（单位：厘，前复权） |
| `data.list[].Volume` | int | 成交量（手） |
| `data.list[].Amount` | int | 成交额（前复权数据中通常为 0） |
| `data.list[].Time` | string | 时间 ISO 8601 格式 |
| `data.list[].UpCount` | int | 上涨数（通常为 0） |
| `data.list[].DownCount` | int | 下跌数（通常为 0） |
| `data.meta` | object | 元数据 |
| `data.meta.batch_limit` | int | 本次返回数据量 |
| `data.meta.source` | string | 数据源标识 `"ths"` |
| `data.meta.type` | string | K 线类型 |
| `data.meta.notes` | array | 使用提示 |

## 返回示例

请求：`GET /api/kline-all/ths?code=000001&type=day`

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "count": 8420,
    "list": [
      {
        "Last": 0,
        "Open": -2720,
        "High": -2720,
        "Low": -2720,
        "Close": -2720,
        "Volume": 1,
        "Amount": 0,
        "Time": "1991-04-03T15:00:00+08:00",
        "UpCount": 0,
        "DownCount": 0
      },
      {
        "Last": -2720,
        "Open": -2730,
        "High": -2730,
        "Low": -2730,
        "Close": -2730,
        "Volume": 3,
        "Amount": 0,
        "Time": "1991-04-04T15:00:00+08:00",
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
      },
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
    ],
    "meta": {
      "batch_limit": 8420,
      "notes": [
        "同花顺接口一次性返回前复权数据，响应时长取决于网络与标的数据量（通常 2-8 秒）",
        "建议调用方在 Python 等客户端中设置 ≥10 秒超时时间，并准备兜底策略"
      ],
      "source": "ths",
      "type": "day"
    }
  }
}
```

## 代码示例

### Python

```python
import requests

url = "http://tdx.acdzh.xyz/api/kline-all/ths"
headers = {"User-Agent": "Mozilla/5.0"}
params = {"code": "000001", "type": "day"}

# 设置较长超时（建议 ≥10 秒）
response = requests.get(url, headers=headers, params=params, timeout=15)
data = response.json()

if data["code"] == 0:
    result = data["data"]
    klines = result["list"]
    print(f"数据源: {result['meta']['source']} (前复权)")
    print(f"类型: {result['meta']['type']}")
    print(f"总条数: {result['count']}")

    # 最新数据（前复权价格）
    last = klines[-1]
    print(f"最新: {last['Time'][:10]} 收:{last['Close']/1000:.2f} 元 (前复权)")

    # 注意：早期数据可能为负数（前复权后）
    first = klines[0]
    print(f"最早: {first['Time'][:10]} 收:{first['Close']/1000:.2f} (前复权，可能为负)")
```

### Shell (xh)

```bash
xh --timeout=15 GET "http://tdx.acdzh.xyz/api/kline-all/ths?code=000001&type=day" User-Agent:"Mozilla/5.0"
```

### Shell (curl)

```bash
curl --max-time 15 -H "User-Agent: Mozilla/5.0" "http://tdx.acdzh.xyz/api/kline-all/ths?code=000001&type=day"
```

## 与 /api/kline-all/tdx 的对比

| 对比项 | `/api/kline-all/tdx` | `/api/kline-all/ths` |
|---|---|---|
| 数据源 | 通达信 | 同花顺 |
| 复权方式 | 不复权（原始价格） | 前复权 |
| 支持类型 | day/week/month/quarter/year/分钟级 | 仅 day/week/month |
| Amount 字段 | 有实际值 | 通常为 0 |
| 早期数据 | 正常正值 | 可能出现负数（前复权导致） |
| 响应时间 | 1-5 秒 | 2-8 秒 |

## 注意事项

- **前复权数据**：历史价格已根据除权除息进行调整，适合做长期趋势分析和涨幅计算
- 只支持 `day` / `week` / `month` 三种类型，**不支持分钟级数据**
- 早期数据可能出现负数值（由于前复权计算导致），这是正常现象
- 价格字段单位为**厘**，需除以 1000 转元
- 响应时间 2-8 秒，客户端应设置 ≥10 秒超时
- `Amount` 字段在前复权数据中通常为 0，不可依赖
- 返回字段名为小写 `count`/`list`（不是大写 `Count`/`List`）
