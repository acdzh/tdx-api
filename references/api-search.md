# GET /api/search

按关键词搜索股票。

## 基本信息

| 项目 | 值 |
|---|---|
| 方法 | GET |
| 路径 | `/api/search` |
| 认证 | 无 |
| 请求头 | `User-Agent: Mozilla/5.0`（建议携带） |

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `keyword` | string | 是 | 搜索关键词，支持股票名称、代码、拼音首字母等 |

## 返回格式

```json
{
  "code": 0,
  "message": "success",
  "data": [
    {
      "code": "601318",
      "exchange": "sh",
      "name": "中国平安"
    }
  ]
}
```

## 字段说明

| 字段 | 类型 | 说明 |
|---|---|---|
| `data` | array | 搜索结果列表，最多 50 条 |
| `data[].code` | string | 股票代码 |
| `data[].exchange` | string | 交易所，`sh`=上海、`sz`=深圳、`bj`=北京 |
| `data[].name` | string | 股票名称 |

## 返回示例

请求：`GET /api/search?keyword=平安`

```json
{
  "code": 0,
  "message": "success",
  "data": [
    {
      "code": "601318",
      "exchange": "sh",
      "name": "中国平安"
    },
    {
      "code": "000001",
      "exchange": "sz",
      "name": "平安银行"
    },
    {
      "code": "001359",
      "exchange": "sz",
      "name": "平安电工"
    }
  ]
}
```

## 代码示例

### Python

```python
import requests

url = "http://tdx.acdzh.xyz/api/search"
headers = {"User-Agent": "Mozilla/5.0"}
params = {"keyword": "平安"}

response = requests.get(url, headers=headers, params=params)
data = response.json()

if data["code"] == 0:
    results = data["data"]
    print(f"搜索到 {len(results)} 只股票:")
    for stock in results:
        exchange_name = {"sh": "上海", "sz": "深圳", "bj": "北京"}.get(stock["exchange"], stock["exchange"])
        print(f"  {stock['code']} | {stock['name']} | {exchange_name}")
```

### Shell (xh)

```bash
xh GET "http://tdx.acdzh.xyz/api/search?keyword=平安" User-Agent:"Mozilla/5.0"
```

### Shell (curl)

```bash
curl -H "User-Agent: Mozilla/5.0" "http://tdx.acdzh.xyz/api/search?keyword=%E5%B9%B3%E5%AE%89"
```

## 注意事项

- 最多返回 50 条结果
- 关键词过泛时结果较多，建议尽量具体
- 六码纯数字代码搜索时需先确认是股票还是基金，本 API 只返回股票结果
- 搜索结果可用于后续调用其他接口（如 `quote`、`kline` 等）时确认准确代码
- 建议在代码不明确时先调用 search 确认，再调用其他接口
