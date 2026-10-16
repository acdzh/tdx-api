# GET /api/workday

查询交易日信息，判断某天是否为交易日，并获取前后相邻交易日。

## 基本信息

| 项目 | 值 |
|---|---|
| 方法 | GET |
| 路径 | `/api/workday` |
| 认证 | 无 |
| 请求头 | `User-Agent: Mozilla/5.0`（建议携带） |

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `date` | string | 是 | 日期，支持 `YYYYMMDD` 和 `YYYY-MM-DD` 两种格式 |
| `count` | int | 否 | 返回前后相邻交易日的数量，默认不限 |

## 返回格式

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "date": {
      "iso": "2026-05-20",
      "numeric": "20260520"
    },
    "is_workday": true,
    "next": [
      {"iso": "2026-05-21", "numeric": "20260521"}
    ],
    "previous": [
      {"iso": "2026-05-19", "numeric": "20260519"}
    ]
  }
}
```

## 字段说明

| 字段 | 类型 | 说明 |
|---|---|---|
| `data.date` | object | 查询日期的标准化表示 |
| `data.date.iso` | string | ISO 格式日期 `YYYY-MM-DD` |
| `data.date.numeric` | string | 数字格式日期 `YYYYMMDD` |
| `data.is_workday` | boolean | 是否为交易日 |
| `data.next` | array | 之后的相邻交易日列表（按时间正序） |
| `data.previous` | array | 之前的相邻交易日列表（按时间倒序，最近的在前） |

### next/previous 元素结构

| 字段 | 类型 | 说明 |
|---|---|---|
| `iso` | string | ISO 格式日期 |
| `numeric` | string | 数字格式日期 |

## 返回示例

### 查询交易日

请求：`GET /api/workday?date=20260520&count=3`

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "date": {
      "iso": "2026-05-20",
      "numeric": "20260520"
    },
    "is_workday": true,
    "next": [
      {"iso": "2026-05-21", "numeric": "20260521"},
      {"iso": "2026-05-22", "numeric": "20260522"},
      {"iso": "2026-05-25", "numeric": "20260525"}
    ],
    "previous": [
      {"iso": "2026-05-19", "numeric": "20260519"},
      {"iso": "2026-05-18", "numeric": "20260518"},
      {"iso": "2026-05-15", "numeric": "20260515"}
    ]
  }
}
```

### 查询非交易日（周末）

请求：`GET /api/workday?date=20260531&count=2`

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "date": {
      "iso": "2026-05-31",
      "numeric": "20260531"
    },
    "is_workday": false,
    "next": [
      {"iso": "2026-06-01", "numeric": "20260601"},
      {"iso": "2026-06-02", "numeric": "20260602"}
    ],
    "previous": [
      {"iso": "2026-05-29", "numeric": "20260529"},
      {"iso": "2026-05-28", "numeric": "20260528"}
    ]
  }
}
```

### 使用 YYYY-MM-DD 格式

请求：`GET /api/workday?date=2026-06-03&count=3`

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "date": {
      "iso": "2026-06-03",
      "numeric": "20260603"
    },
    "is_workday": true,
    "next": [],
    "previous": [
      {"iso": "2026-06-02", "numeric": "20260602"},
      {"iso": "2026-06-01", "numeric": "20260601"},
      {"iso": "2026-05-29", "numeric": "20260529"}
    ]
  }
}
```

## 代码示例

### Python

```python
import requests

url = "http://tdx.acdzh.com/api/workday"
headers = {"User-Agent": "Mozilla/5.0"}
params = {"date": "20260520", "count": 5}

response = requests.get(url, headers=headers, params=params)
data = response.json()

if data["code"] == 0:
    result = data["data"]
    date_str = result["date"]["iso"]
    is_workday = result["is_workday"]

    print(f"日期: {date_str}")
    print(f"是否交易日: {'是' if is_workday else '否'}")

    if result["previous"]:
        print(f"前 {len(result['previous'])} 个交易日:")
        for d in result["previous"]:
            print(f"  {d['iso']}")

    if result["next"]:
        print(f"后 {len(result['next'])} 个交易日:")
        for d in result["next"]:
            print(f"  {d['iso']}")
```

### Shell (xh)

```bash
# YYYYMMDD 格式
xh GET "http://tdx.acdzh.com/api/workday?date=20260520&count=3" User-Agent:"Mozilla/5.0"

# YYYY-MM-DD 格式
xh GET "http://tdx.acdzh.com/api/workday?date=2026-06-03&count=5" User-Agent:"Mozilla/5.0"
```

### Shell (curl)

```bash
curl -H "User-Agent: Mozilla/5.0" "http://tdx.acdzh.com/api/workday?date=20260520&count=3"
```

## 注意事项

- 支持 `YYYYMMDD` 和 `YYYY-MM-DD` 两种日期格式
- `is_workday` 为 `false` 时表示非交易日（周末或节假日）
- `next` 数组在查询当天或未来日期时可能为空（因为未来交易日尚未确定或已是最新日期）
- `previous` 数组按时间倒序排列，最近的交易日在前
- `count` 控制 `next` 和 `previous` 各返回多少个相邻交易日
- 跳过了周末和法定假日，只列出实际交易日
