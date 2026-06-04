# GET /api/minute

获取股票分时数据。

## 基本信息

| 项目 | 值 |
|---|---|
| 方法 | GET |
| 路径 | `/api/minute` |
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
    "Count": 240,
    "List": [
      {
        "Time": "09:31",
        "Price": 10980,
        "Number": 28453
      }
    ],
    "date": "20260602"
  }
}
```

## 字段说明

| 字段 | 类型 | 说明 |
|---|---|---|
| `data.Count` | int | 分时数据条数，完整交易日为 240 条（上午 120 + 下午 120） |
| `data.List` | array | 分时数据列表 |
| `data.List[].Time` | string | 时间点，格式 `HH:MM`，如 `"09:31"` |
| `data.List[].Price` | int | 当前价格（单位：厘，除以 1000 得元） |
| `data.List[].Number` | int | 该分钟成交量（单位：手） |
| `data.date` | string | 请求的日期 |

## 返回示例

### 正常交易日

请求：`GET /api/minute?code=000001&date=20260602`

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "Count": 240,
    "List": [
      {"Time": "09:31", "Price": 10980, "Number": 28453},
      {"Time": "09:32", "Price": 10990, "Number": 11626},
      {"Time": "09:33", "Price": 11000, "Number": 33855},
      {"Time": "09:34", "Price": 10990, "Number": 5196},
      {"Time": "09:35", "Price": 11000, "Number": 2913},
      {"Time": "11:29", "Price": 11010, "Number": 1108},
      {"Time": "11:30", "Price": 11000, "Number": 582},
      {"Time": "13:01", "Price": 11020, "Number": 5692},
      {"Time": "14:59", "Price": 11070, "Number": 0},
      {"Time": "15:00", "Price": 11080, "Number": 13751}
    ],
    "date": "20260602"
  }
}
```

### 非交易日（无数据）

请求：`GET /api/minute?code=000001&date=20260531`（周日）

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

url = "http://tdx.acdzh.xyz/api/minute"
headers = {"User-Agent": "Mozilla/5.0"}
params = {"code": "000001", "date": "20260602"}

response = requests.get(url, headers=headers, params=params)
data = response.json()

if data["code"] == 0:
    minute_data = data["data"]
    if minute_data["Count"] == 0 or minute_data["List"] is None:
        print("该日期无分时数据（可能为非交易日）")
    else:
        print(f"日期: {minute_data['date']}，共 {minute_data['Count']} 条分时数据")
        for item in minute_data["List"][:10]:
            price = item["Price"] / 1000
            volume = item["Number"]
            print(f"  {item['Time']} | 价格: {price:.2f} 元 | 成交: {volume} 手")
```

### Shell (xh)

```bash
xh GET "http://tdx.acdzh.xyz/api/minute?code=000001&date=20260602" User-Agent:"Mozilla/5.0"
```

### Shell (curl)

```bash
curl -H "User-Agent: Mozilla/5.0" "http://tdx.acdzh.xyz/api/minute?code=000001&date=20260602"
```

## 注意事项

- 价格字段单位为**厘**，展示时需除以 1000 转换为元
- 成交量 `Number` 单位为**手**
- 完整交易日返回 240 条数据：上午 9:31-11:30（120 条）+ 下午 13:01-15:00（120 条）
- 非交易日返回 `{"Count": 0, "List": null}`，**不等于接口失败**
- 14:59 的 `Number` 可能为 0（集合竞价期间）
- 15:00 的 `Number` 通常较大（收盘集合竞价）
- `date` 格式为 `YYYYMMDD`，不支持 `YYYY-MM-DD` 格式
