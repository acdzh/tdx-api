# GET /api/stock-info

获取单只股票概览信息，包含实时行情、近期日 K 线、当日分时数据。

## 基本信息

| 项目 | 值 |
|---|---|
| 方法 | GET |
| 路径 | `/api/stock-info` |
| 认证 | 无 |
| 请求头 | `User-Agent: Mozilla/5.0`（建议携带） |

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `code` | string | 是 | 股票代码，如 `000001` |

## 返回格式

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "quote": { ... },
    "kline_day": {
      "Count": 30,
      "List": [ ... ]
    },
    "minute": {
      "Count": 240,
      "List": [ ... ]
    }
  }
}
```

## 字段说明

### 顶层结构

| 字段 | 类型 | 说明 |
|---|---|---|
| `data.quote` | object | 实时行情数据，结构与 `/api/quote` 单条返回一致 |
| `data.kline_day` | object | 近期日 K 线数据 |
| `data.kline_day.Count` | int | 日 K 线条数（默认约 30 条） |
| `data.kline_day.List` | array | 日 K 线列表 |
| `data.minute` | object | 当日分时数据 |
| `data.minute.Count` | int | 分时数据条数 |
| `data.minute.List` | array | 分时列表 |

### quote 子结构（与 /api/quote 相同）

| 字段 | 类型 | 说明 |
|---|---|---|
| `Exchange` | int | 交易所，0=深圳，1=上海 |
| `Code` | string | 股票代码 |
| `K.Last` | int | 昨收价（厘） |
| `K.Open` | int | 今开价（厘） |
| `K.High` | int | 最高价（厘） |
| `K.Low` | int | 最低价（厘） |
| `K.Close` | int | 当前价（厘） |
| `TotalHand` | int | 成交量（手） |
| `Amount` | int | 成交额 |
| `BuyLevel` | array | 买盘五档 |
| `SellLevel` | array | 卖盘五档 |
| `Rate` | float | 涨跌幅 |

### kline_day.List[] 子结构（与 /api/kline 相同）

| 字段 | 类型 | 说明 |
|---|---|---|
| `Last` | int | 上一周期收盘价（厘） |
| `Open` | int | 开盘价（厘） |
| `High` | int | 最高价（厘） |
| `Low` | int | 最低价（厘） |
| `Close` | int | 收盘价（厘） |
| `Volume` | int | 成交量（手） |
| `Time` | string | 时间 ISO 8601 |

### minute.List[] 子结构（与 /api/minute 相同）

| 字段 | 类型 | 说明 |
|---|---|---|
| `Time` | string | 时间 `HH:MM` |
| `Price` | int | 价格（厘） |
| `Number` | int | 成交量（手） |

## 返回示例

请求：`GET /api/stock-info?code=000001`

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "quote": {
      "Exchange": 0,
      "Code": "000001",
      "Active1": 1594,
      "K": {
        "Last": 11080,
        "Open": 11030,
        "High": 11060,
        "Low": 10960,
        "Close": 10970
      },
      "ServerTime": "10553704",
      "TotalHand": 445147,
      "Amount": 489697856,
      "InsideDish": 263959,
      "OuterDisc": 181189,
      "BuyLevel": [
        {"Buy": true, "Price": 10960, "Number": 9253},
        {"Buy": true, "Price": 10950, "Number": 8518},
        {"Buy": true, "Price": 10940, "Number": 3800},
        {"Buy": true, "Price": 10930, "Number": 5279},
        {"Buy": true, "Price": 10920, "Number": 5531}
      ],
      "SellLevel": [
        {"Buy": false, "Price": 10970, "Number": 4008},
        {"Buy": false, "Price": 10980, "Number": 4997},
        {"Buy": false, "Price": 10990, "Number": 3059},
        {"Buy": false, "Price": 11000, "Number": 4693},
        {"Buy": false, "Price": 11010, "Number": 2547}
      ],
      "Rate": 0.09,
      "Active2": 1594
    },
    "kline_day": {
      "Count": 30,
      "List": [
        {
          "Last": 11090,
          "Open": 11090,
          "High": 11110,
          "Low": 10990,
          "Close": 11010,
          "Volume": 722530,
          "Amount": 0,
          "Time": "2026-04-17T15:00:00+08:00",
          "UpCount": 0,
          "DownCount": 0
        },
        {
          "Last": 11010,
          "Open": 11010,
          "High": 11070,
          "Low": 10960,
          "Close": 11060,
          "Volume": 630034,
          "Amount": 0,
          "Time": "2026-04-20T15:00:00+08:00",
          "UpCount": 0,
          "DownCount": 0
        }
      ]
    },
    "minute": {
      "Count": 86,
      "List": [
        {"Time": "09:31", "Price": 11010, "Number": 63944},
        {"Time": "09:32", "Price": 11010, "Number": 14480}
      ]
    }
  }
}
```

## 代码示例

### Python

```python
import requests

url = "http://tdx.acdzh.xyz/api/stock-info"
headers = {"User-Agent": "Mozilla/5.0"}
params = {"code": "000001"}

response = requests.get(url, headers=headers, params=params)
data = response.json()

if data["code"] == 0:
    info = data["data"]

    # 实时行情
    quote = info["quote"]
    current_price = quote["K"]["Close"] / 1000
    prev_close = quote["K"]["Last"] / 1000
    change = current_price - prev_close
    change_pct = (change / prev_close) * 100 if prev_close else 0
    print(f"股票: {quote['Code']}")
    print(f"当前价: {current_price:.2f} 元")
    print(f"涨跌: {change:+.2f} 元 ({change_pct:+.2f}%)")
    print(f"成交量: {quote['TotalHand']} 手")

    # 近期日 K 线
    kline = info["kline_day"]
    print(f"\n近期日K线: {kline['Count']} 条")
    if kline["List"]:
        latest = kline["List"][-1]
        print(f"  最近一日: {latest['Time'][:10]} 收:{latest['Close']/1000:.2f}")

    # 分时
    minute = info["minute"]
    print(f"\n当日分时: {minute['Count']} 条")
```

### Shell (xh)

```bash
xh GET "http://tdx.acdzh.xyz/api/stock-info?code=000001" User-Agent:"Mozilla/5.0"
```

### Shell (curl)

```bash
curl -H "User-Agent: Mozilla/5.0" "http://tdx.acdzh.xyz/api/stock-info?code=000001"
```

## 注意事项

- 这是一个聚合接口，内容较多，使用时应先提炼关键信息，不要原样展开全部数据
- 所有价格字段单位为**厘**，需除以 1000 转元
- `kline_day` 默认返回最近约 30 条日 K 线
- `minute` 在交易时段中返回截至当前的分时数据
- 适合快速了解单只股票的全面概况，避免多次调用不同接口
