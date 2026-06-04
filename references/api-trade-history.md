# GET /api/trade-history

获取股票历史逐笔成交数据，支持分页。

## 基本信息

| 项目 | 值 |
|---|---|
| 方法 | GET |
| 路径 | `/api/trade-history` |
| 认证 | 无 |
| 请求头 | `User-Agent: Mozilla/5.0`（建议携带） |

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `code` | string | 是 | 股票代码，如 `000001` |
| `date` | string | 是 | 日期，格式 `YYYYMMDD`，如 `20260602` |
| `start` | int | 否 | 起始位置（从 0 开始），用于分页 |
| `count` | int | 否 | 返回条数，单次最多 2000 条 |

## 返回格式

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "Count": 5,
    "List": [
      {
        "Time": "2026-06-02T14:56:00+08:00",
        "Price": 11080,
        "Volume": 1103,
        "Status": 0,
        "Number": 0
      }
    ]
  }
}
```

## 字段说明

| 字段 | 类型 | 说明 |
|---|---|---|
| `data.Count` | int | 本次返回的记录条数 |
| `data.List` | array | 成交记录列表 |
| `data.List[].Time` | string | 成交时间，ISO 8601 格式带时区 |
| `data.List[].Price` | int | 成交价格（单位：厘，除以 1000 得元） |
| `data.List[].Volume` | int | 成交量（单位：手） |
| `data.List[].Status` | int | 成交方向：`0` = 主动买入，`1` = 主动卖出，`2` = 集合竞价/中性 |
| `data.List[].Number` | int | 逐笔序号 |

## 返回示例

请求：`GET /api/trade-history?code=000001&date=20260602&start=0&count=5`

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "Count": 5,
    "List": [
      {
        "Time": "2026-06-02T14:56:00+08:00",
        "Price": 11080,
        "Volume": 1103,
        "Status": 0,
        "Number": 0
      },
      {
        "Time": "2026-06-02T14:56:00+08:00",
        "Price": 11070,
        "Volume": 1206,
        "Status": 1,
        "Number": 0
      },
      {
        "Time": "2026-06-02T14:56:00+08:00",
        "Price": 11070,
        "Volume": 230,
        "Status": 1,
        "Number": 0
      },
      {
        "Time": "2026-06-02T14:57:00+08:00",
        "Price": 11070,
        "Volume": 188,
        "Status": 1,
        "Number": 0
      },
      {
        "Time": "2026-06-02T15:00:00+08:00",
        "Price": 11080,
        "Volume": 13751,
        "Status": 2,
        "Number": 0
      }
    ]
  }
}
```

## 代码示例

### Python

```python
import requests

url = "http://tdx.acdzh.xyz/api/trade-history"
headers = {"User-Agent": "Mozilla/5.0"}

# 分页获取全部成交数据
all_trades = []
start = 0
batch_size = 2000

while True:
    params = {
        "code": "000001",
        "date": "20260602",
        "start": start,
        "count": batch_size
    }
    response = requests.get(url, headers=headers, params=params)
    data = response.json()

    if data["code"] != 0:
        print(f"请求失败: {data['message']}")
        break

    trade_data = data["data"]
    if trade_data["Count"] == 0 or trade_data["List"] is None:
        break

    all_trades.extend(trade_data["List"])
    start += trade_data["Count"]

    if trade_data["Count"] < batch_size:
        break

print(f"共获取 {len(all_trades)} 笔成交")

# 分析成交方向
status_map = {0: "买", 1: "卖", 2: "中"}
for trade in all_trades[:5]:
    price = trade["Price"] / 1000
    volume = trade["Volume"]
    direction = status_map.get(trade["Status"], "未知")
    time = trade["Time"][11:19]
    print(f"  {time} | {price:.2f}元 | {volume}手 | {direction}")
```

### Shell (xh)

```bash
# 获取前 5 条
xh GET "http://tdx.acdzh.xyz/api/trade-history?code=000001&date=20260602&start=0&count=5" User-Agent:"Mozilla/5.0"

# 获取第 2000-4000 条
xh GET "http://tdx.acdzh.xyz/api/trade-history?code=000001&date=20260602&start=2000&count=2000" User-Agent:"Mozilla/5.0"
```

### Shell (curl)

```bash
curl -H "User-Agent: Mozilla/5.0" "http://tdx.acdzh.xyz/api/trade-history?code=000001&date=20260602&start=0&count=2000"
```

## 注意事项

- 价格字段单位为**厘**，展示时需除以 1000 转换为元
- 成交量单位为**手**
- 单次最多返回 2000 条，需要通过 `start` 参数分页获取全部数据
- `Status` 字段：0=主动买入、1=主动卖出、2=集合竞价/中性盘
- 不要一次性输出大量明细给用户，应做汇总分析
- 与 `/api/trade` 的区别：`trade-history` 支持分页且适用于历史数据完整获取，`trade` 更适合快速查看当日概况
