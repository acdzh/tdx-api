# GET /api/quote

获取股票五档实时行情。

## 基本信息

| 项目 | 值 |
|---|---|
| 方法 | GET |
| 路径 | `/api/quote` |
| 认证 | 无 |
| 请求头 | `User-Agent: Mozilla/5.0`（建议携带） |

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `code` | string | 是 | 股票代码，支持多个代码逗号分隔，如 `000001,600519` |

## 返回格式

```json
{
  "code": 0,
  "message": "success",
  "data": [
    {
      "Exchange": 0,
      "Code": "000001",
      "Active1": 1570,
      "K": {
        "Last": 11080,
        "Open": 11030,
        "High": 11060,
        "Low": 10960,
        "Close": 10970
      },
      "ServerTime": "10542204",
      "ReversedBytes0": 10542204,
      "ReversedBytes1": -1097,
      "TotalHand": 442576,
      "Intuition": 7,
      "Amount": 486879744,
      "InsideDish": 261833,
      "OuterDisc": 180744,
      "ReversedBytes2": 0,
      "ReversedBytes3": 115186,
      "BuyLevel": [...],
      "SellLevel": [...],
      "ReversedBytes4": 4748,
      "ReversedBytes5": 0,
      "ReversedBytes6": 0,
      "ReversedBytes7": 0,
      "ReversedBytes8": 0,
      "ReversedBytes9": 65528,
      "Rate": 655.28,
      "Active2": 1570
    }
  ]
}
```

## 字段说明

| 字段 | 类型 | 说明 |
|---|---|---|
| `Exchange` | int | 交易所标识。`0` = 深圳，`1` = 上海 |
| `Code` | string | 股票代码 |
| `Active1` | int | 活跃度标识 |
| `K.Last` | int | 昨收价（单位：厘，除以 1000 得元） |
| `K.Open` | int | 今开价（单位：厘） |
| `K.High` | int | 最高价（单位：厘） |
| `K.Low` | int | 最低价（单位：厘） |
| `K.Close` | int | 当前价/最新价（单位：厘） |
| `ServerTime` | string | 服务器行情时间 |
| `TotalHand` | int | 总成交量（单位：手，乘以 100 得股） |
| `Amount` | int | 成交额（需结合接口口径换算） |
| `InsideDish` | int | 内盘（主动卖出量） |
| `OuterDisc` | int | 外盘（主动买入量） |
| `BuyLevel` | array | 买盘五档，每档含 `Buy`(bool)、`Price`(厘)、`Number`(手) |
| `SellLevel` | array | 卖盘五档，每档含 `Buy`(bool)、`Price`(厘)、`Number`(手) |
| `Rate` | float | 涨跌幅。注意：当值大于 100 时，实际含义可能需要除以某个系数；常规行情下可直接视为百分比 |
| `Active2` | int | 活跃度标识 |

## 返回示例

### 单只股票

请求：`GET /api/quote?code=000001`

```json
{
  "code": 0,
  "message": "success",
  "data": [
    {
      "Exchange": 0,
      "Code": "000001",
      "Active1": 1570,
      "K": {
        "Last": 11080,
        "Open": 11030,
        "High": 11060,
        "Low": 10960,
        "Close": 10970
      },
      "ServerTime": "10542204",
      "ReversedBytes0": 10542204,
      "ReversedBytes1": -1097,
      "TotalHand": 442576,
      "Intuition": 7,
      "Amount": 486879744,
      "InsideDish": 261833,
      "OuterDisc": 180744,
      "ReversedBytes2": 0,
      "ReversedBytes3": 115186,
      "BuyLevel": [
        {"Buy": true, "Price": 10960, "Number": 10642},
        {"Buy": true, "Price": 10950, "Number": 8358},
        {"Buy": true, "Price": 10940, "Number": 3761},
        {"Buy": true, "Price": 10930, "Number": 5084},
        {"Buy": true, "Price": 10920, "Number": 5201}
      ],
      "SellLevel": [
        {"Buy": false, "Price": 10970, "Number": 3187},
        {"Buy": false, "Price": 10980, "Number": 6616},
        {"Buy": false, "Price": 10990, "Number": 3284},
        {"Buy": false, "Price": 11000, "Number": 4669},
        {"Buy": false, "Price": 11010, "Number": 2451}
      ],
      "ReversedBytes4": 4748,
      "ReversedBytes5": 0,
      "ReversedBytes6": 0,
      "ReversedBytes7": 0,
      "ReversedBytes8": 0,
      "ReversedBytes9": 65528,
      "Rate": 655.28,
      "Active2": 1570
    }
  ]
}
```

### 多只股票

请求：`GET /api/quote?code=000001,600519`

返回 `data` 数组包含两个元素，第一个为 000001（Exchange=0 深圳），第二个为 600519（Exchange=1 上海）。

## 代码示例

### Python

```python
import requests

url = "http://tdx.acdzh.com/api/quote"
headers = {"User-Agent": "Mozilla/5.0"}
params = {"code": "000001"}

response = requests.get(url, headers=headers, params=params)
data = response.json()

if data["code"] == 0:
    for stock in data["data"]:
        code = stock["Code"]
        last_price = stock["K"]["Close"] / 1000  # 厘转元
        open_price = stock["K"]["Open"] / 1000
        high_price = stock["K"]["High"] / 1000
        low_price = stock["K"]["Low"] / 1000
        prev_close = stock["K"]["Last"] / 1000
        volume = stock["TotalHand"] * 100  # 手转股

        print(f"代码: {code}")
        print(f"最新价: {last_price:.2f} 元")
        print(f"今开: {open_price:.2f} 元")
        print(f"最高: {high_price:.2f} 元")
        print(f"最低: {low_price:.2f} 元")
        print(f"昨收: {prev_close:.2f} 元")
        print(f"成交量: {volume} 股")

        # 五档买盘
        print("买盘五档:")
        for level in stock["BuyLevel"]:
            print(f"  {level['Price']/1000:.2f} 元 - {level['Number']} 手")
```

### Shell (xh)

```bash
# 单只股票
xh GET "http://tdx.acdzh.com/api/quote?code=000001" User-Agent:"Mozilla/5.0"

# 多只股票
xh GET "http://tdx.acdzh.com/api/quote?code=000001,600519" User-Agent:"Mozilla/5.0"
```

### Shell (curl)

```bash
curl -H "User-Agent: Mozilla/5.0" "http://tdx.acdzh.com/api/quote?code=000001,600519"
```

## 注意事项

- 价格字段单位为**厘**，展示时需除以 1000 转换为元
- 成交量单位为**手**，需乘以 100 转换为股
- `Rate` 字段在某些情况下返回值较大（如 655.28），可能因行情时段或特殊状态导致，常规使用时请结合实际行情判断
- 支持多只股票同时查询，逗号分隔代码即可
- `Exchange` 为 0 代表深圳，1 代表上海
