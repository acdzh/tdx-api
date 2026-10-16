# POST /api/batch-quote

批量获取多只股票的实时行情数据。

## 基本信息

| 项目 | 值 |
|---|---|
| 方法 | POST |
| 路径 | `/api/batch-quote` |
| 认证 | 无 |
| Content-Type | `application/json` |
| 请求头 | `User-Agent: Mozilla/5.0`（建议携带） |

## 请求参数（JSON Body）

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `codes` | array[string] | 是 | 股票代码数组，如 `["000001", "600519"]` |

### 请求体示例

```json
{
  "codes": ["000001", "600519"]
}
```

## 返回格式

```json
{
  "code": 0,
  "message": "success",
  "data": [
    {
      "Exchange": 0,
      "Code": "000001",
      "K": { ... },
      "TotalHand": 448171,
      "Amount": 493012224,
      "BuyLevel": [ ... ],
      "SellLevel": [ ... ],
      "Rate": 0,
      ...
    },
    {
      "Exchange": 1,
      "Code": "600519",
      "K": { ... },
      ...
    }
  ]
}
```

## 字段说明

返回结构与 `GET /api/quote` 完全一致，`data` 为数组，每个元素对应一只股票。

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
| `InsideDish` | int | 内盘 |
| `OuterDisc` | int | 外盘 |
| `BuyLevel` | array | 买盘五档 |
| `SellLevel` | array | 卖盘五档 |
| `Rate` | float | 涨跌幅 |

## 返回示例

请求：`POST /api/batch-quote` Body: `{"codes":["000001","600519"]}`

```json
{
  "code": 0,
  "message": "success",
  "data": [
    {
      "Exchange": 0,
      "Code": "000001",
      "Active1": 1606,
      "K": {
        "Last": 11080,
        "Open": 11030,
        "High": 11060,
        "Low": 10960,
        "Close": 10970
      },
      "ServerTime": "10561604",
      "TotalHand": 448171,
      "Amount": 493012224,
      "InsideDish": 266937,
      "OuterDisc": 181235,
      "BuyLevel": [
        {"Buy": true, "Price": 10960, "Number": 6790},
        {"Buy": true, "Price": 10950, "Number": 8642},
        {"Buy": true, "Price": 10940, "Number": 3779},
        {"Buy": true, "Price": 10930, "Number": 5345},
        {"Buy": true, "Price": 10920, "Number": 5561}
      ],
      "SellLevel": [
        {"Buy": false, "Price": 10970, "Number": 4771},
        {"Buy": false, "Price": 10980, "Number": 4979},
        {"Buy": false, "Price": 10990, "Number": 3059},
        {"Buy": false, "Price": 11000, "Number": 4696},
        {"Buy": false, "Price": 11010, "Number": 2547}
      ],
      "Rate": 0,
      "Active2": 1606
    },
    {
      "Exchange": 1,
      "Code": "600519",
      "Active1": 1689,
      "K": {
        "Last": 1307220,
        "Open": 1304000,
        "High": 1304000,
        "Low": 1280900,
        "Close": 1281040
      },
      "ServerTime": "10434497",
      "TotalHand": 26281,
      "Amount": 3382818048,
      "InsideDish": 15336,
      "OuterDisc": 10945,
      "BuyLevel": [
        {"Buy": true, "Price": 1281030, "Number": 28},
        {"Buy": true, "Price": 1281020, "Number": 2},
        {"Buy": true, "Price": 1281010, "Number": 7},
        {"Buy": true, "Price": 1281000, "Number": 15},
        {"Buy": true, "Price": 1280990, "Number": 1}
      ],
      "SellLevel": [
        {"Buy": false, "Price": 1281040, "Number": 8},
        {"Buy": false, "Price": 1281050, "Number": 5},
        {"Buy": false, "Price": 1281070, "Number": 2},
        {"Buy": false, "Price": 1281080, "Number": 2},
        {"Buy": false, "Price": 1281150, "Number": 3}
      ],
      "Rate": 655.29,
      "Active2": 1689
    }
  ]
}
```

## 代码示例

### Python

```python
import requests

url = "http://tdx.acdzh.com/api/batch-quote"
headers = {
    "User-Agent": "Mozilla/5.0",
    "Content-Type": "application/json"
}
payload = {
    "codes": ["000001", "600519", "000858"]
}

response = requests.post(url, headers=headers, json=payload)
data = response.json()

if data["code"] == 0:
    for stock in data["data"]:
        code = stock["Code"]
        price = stock["K"]["Close"] / 1000
        prev = stock["K"]["Last"] / 1000
        change = price - prev
        pct = (change / prev * 100) if prev else 0
        volume = stock["TotalHand"]
        print(f"{code} | 现价:{price:.2f} | 涨跌:{change:+.2f} ({pct:+.2f}%) | 量:{volume}手")
```

### Shell (xh)

```bash
echo '{"codes":["000001","600519"]}' | xh POST "http://tdx.acdzh.com/api/batch-quote" User-Agent:"Mozilla/5.0" Content-Type:"application/json"
```

### Shell (curl)

```bash
curl -X POST "http://tdx.acdzh.com/api/batch-quote" \
  -H "User-Agent: Mozilla/5.0" \
  -H "Content-Type: application/json" \
  -d '{"codes":["000001","600519"]}'
```

## 注意事项

- 与 `GET /api/quote?code=000001,600519` 功能类似，但使用 POST + JSON Body
- 适合需要批量查询大量股票的场景
- 价格字段单位为**厘**，需除以 1000 转元
- 成交量单位为**手**，需乘以 100 转股
- 先统一单位和时间口径，再做并列比较
- 返回数组顺序与请求 `codes` 数组顺序一致
