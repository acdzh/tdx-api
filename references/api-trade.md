# GET /api/trade

获取股票当日逐笔成交数据。

## 基本信息

| 项目 | 值 |
|---|---|
| 方法 | GET |
| 路径 | `/api/trade` |
| 认证 | 无 |
| 请求头 | `User-Agent: Mozilla/5.0`（建议携带） |

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `code` | string | 是 | 股票代码，如 `000001` |
| `date` | string | 是 | 日期，格式 `YYYYMMDD`，如 `20260602` |

## 返回格式

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "Count": 4429,
    "List": [
      {
        "Time": "2026-06-02T09:25:00+08:00",
        "Price": 10980,
        "Volume": 5231,
        "Status": 2,
        "Number": 0
      }
    ]
  }
}
```

## 字段说明

| 字段 | 类型 | 说明 |
|---|---|---|
| `data.Count` | int | 成交记录总条数 |
| `data.List` | array | 成交记录列表 |
| `data.List[].Time` | string | 成交时间，ISO 8601 格式带时区 |
| `data.List[].Price` | int | 成交价格（单位：厘，除以 1000 得元） |
| `data.List[].Volume` | int | 成交量（单位：手） |
| `data.List[].Status` | int | 成交方向：`0` = 主动买入，`1` = 主动卖出，`2` = 集合竞价/中性 |
| `data.List[].Number` | int | 序号（当日成交中通常为 0） |

## 返回示例

### 有数据的交易日

请求：`GET /api/trade?code=000001&date=20260602`

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "Count": 4429,
    "List": [
      {
        "Time": "2026-06-02T09:25:00+08:00",
        "Price": 10980,
        "Volume": 5231,
        "Status": 2,
        "Number": 0
      },
      {
        "Time": "2026-06-02T09:30:00+08:00",
        "Price": 10960,
        "Volume": 3275,
        "Status": 1,
        "Number": 0
      },
      {
        "Time": "2026-06-02T09:30:00+08:00",
        "Price": 10960,
        "Volume": 2413,
        "Status": 1,
        "Number": 0
      },
      {
        "Time": "2026-06-02T09:30:00+08:00",
        "Price": 10950,
        "Volume": 1424,
        "Status": 1,
        "Number": 0
      },
      {
        "Time": "2026-06-02T09:30:00+08:00",
        "Price": 10950,
        "Volume": 1102,
        "Status": 0,
        "Number": 0
      }
    ]
  }
}
```

### 无数据（非交易日或当日尚未开盘）

请求：`GET /api/trade?code=000001&date=20260603`（尚未产生成交）

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "Count": 0,
    "List": null
  }
}
```

## 代码示例

### Python

```python
import requests

url = "http://tdx.acdzh.com/api/trade"
headers = {"User-Agent": "Mozilla/5.0"}
params = {"code": "000001", "date": "20260602"}

response = requests.get(url, headers=headers, params=params)
data = response.json()

if data["code"] == 0:
    trade_data = data["data"]
    if trade_data["Count"] == 0 or trade_data["List"] is None:
        print("该日期无成交数据")
    else:
        print(f"共 {trade_data['Count']} 笔成交")

        status_map = {0: "买", 1: "卖", 2: "中"}
        for trade in trade_data["List"][:10]:
            price = trade["Price"] / 1000
            volume = trade["Volume"]
            direction = status_map.get(trade["Status"], "未知")
            time = trade["Time"][11:19]
            print(f"  {time} | {price:.2f}元 | {volume}手 | {direction}")
```

### Shell (xh)

```bash
xh GET "http://tdx.acdzh.com/api/trade?code=000001&date=20260602" User-Agent:"Mozilla/5.0"
```

### Shell (curl)

```bash
curl -H "User-Agent: Mozilla/5.0" "http://tdx.acdzh.com/api/trade?code=000001&date=20260602"
```

## 注意事项

- 价格字段单位为**厘**，展示时需除以 1000 转换为元
- 成交量单位为**手**
- 当日最多返回约 1800 条（文档标注），实测可返回更多（如 4429 条）
- `Status` 字段：0=主动买入（外盘）、1=主动卖出（内盘）、2=集合竞价/中性盘
- 第一条记录通常是 09:25 的集合竞价结果（Status=2）
- 无数据时返回 `{"Count":0,"List":null}`，不等于接口失败
- 若需要更完整的历史逐笔数据，应使用 `/api/trade-history`
