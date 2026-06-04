# GET /api/income

计算股票从指定日期开始的区间收益。

## 基本信息

| 项目 | 值 |
|---|---|
| 方法 | GET |
| 路径 | `/api/income` |
| 认证 | 无 |
| 请求头 | `User-Agent: Mozilla/5.0`（建议携带） |

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `code` | string | 是 | 股票代码，如 `000001` |
| `start_date` | string | 是 | 起始日期，格式 `YYYYMMDD` |
| `days` | string | 是 | 观察天数，多个用逗号分隔，如 `5,10,20` |

## 返回格式

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "count": 2,
    "list": [
      {
        "offset": 5,
        "source": {
          "close": 11.36,
          "high": 11.5,
          "low": 11.3,
          "open": 11.5
        },
        "current": {
          "close": 11.14,
          "high": 11.28,
          "low": 11.11,
          "open": 11.25
        },
        "rise": -0.22,
        "rise_rate": -0.019366197183098594,
        "time": "2026-05-13T15:00:00+08:00"
      }
    ]
  }
}
```

## 字段说明

| 字段 | 类型 | 说明 |
|---|---|---|
| `data.count` | int | 返回的收益区间数量 |
| `data.list` | array | 收益区间列表 |
| `data.list[].offset` | int | 观察区间天数（对应 `days` 参数中的值） |
| `data.list[].source` | object | 起点价格信息（**单位：元**） |
| `data.list[].source.open` | float | 起点日开盘价（元） |
| `data.list[].source.high` | float | 起点日最高价（元） |
| `data.list[].source.low` | float | 起点日最低价（元） |
| `data.list[].source.close` | float | 起点日收盘价（元） |
| `data.list[].current` | object | 终点价格信息（**单位：元**） |
| `data.list[].current.open` | float | 终点日开盘价（元） |
| `data.list[].current.high` | float | 终点日最高价（元） |
| `data.list[].current.low` | float | 终点日最低价（元） |
| `data.list[].current.close` | float | 终点日收盘价（元） |
| `data.list[].rise` | float | 区间涨跌额（元） |
| `data.list[].rise_rate` | float | 区间收益率（小数，乘以 100 得百分比） |
| `data.list[].time` | string | 终点日期时间，ISO 8601 格式 |

## 返回示例

### 平安银行

请求：`GET /api/income?code=000001&start_date=20260501&days=5,10,20`

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "count": 2,
    "list": [
      {
        "current": {
          "close": 11.14,
          "high": 11.28,
          "low": 11.11,
          "open": 11.25
        },
        "offset": 5,
        "rise": -0.22,
        "rise_rate": -0.019366197183098594,
        "source": {
          "close": 11.36,
          "high": 11.5,
          "low": 11.3,
          "open": 11.5
        },
        "time": "2026-05-13T15:00:00+08:00"
      },
      {
        "current": {
          "close": 10.77,
          "high": 10.88,
          "low": 10.76,
          "open": 10.86
        },
        "offset": 10,
        "rise": -0.59,
        "rise_rate": -0.05193661971830986,
        "source": {
          "close": 11.36,
          "high": 11.5,
          "low": 11.3,
          "open": 11.5
        },
        "time": "2026-05-20T15:00:00+08:00"
      }
    ]
  }
}
```

### 贵州茅台

请求：`GET /api/income?code=600519&start_date=20260501&days=5,10,20`

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "count": 2,
    "list": [
      {
        "current": {
          "close": 1344.09,
          "high": 1358.6,
          "low": 1338,
          "open": 1354.5
        },
        "offset": 5,
        "rise": -30.91,
        "rise_rate": -0.02248,
        "source": {
          "close": 1375,
          "high": 1379,
          "low": 1360.05,
          "open": 1365.1
        },
        "time": "2026-05-13T15:00:00+08:00"
      },
      {
        "current": {
          "close": 1315,
          "high": 1332.99,
          "low": 1314,
          "open": 1321
        },
        "offset": 10,
        "rise": -60,
        "rise_rate": -0.04363636363636364,
        "source": {
          "close": 1375,
          "high": 1379,
          "low": 1360.05,
          "open": 1365.1
        },
        "time": "2026-05-20T15:00:00+08:00"
      }
    ]
  }
}
```

## 代码示例

### Python

```python
import requests

url = "http://tdx.acdzh.xyz/api/income"
headers = {"User-Agent": "Mozilla/5.0"}
params = {
    "code": "000001",
    "start_date": "20260501",
    "days": "5,10,20"
}

response = requests.get(url, headers=headers, params=params)
data = response.json()

if data["code"] == 0:
    result = data["data"]
    print(f"共 {result['count']} 个观察区间")

    for item in result["list"]:
        offset = item["offset"]
        source_close = item["source"]["close"]
        current_close = item["current"]["close"]
        rise = item["rise"]
        rise_rate = item["rise_rate"] * 100  # 转百分比
        end_date = item["time"][:10]

        print(f"\n{offset} 个交易日后 ({end_date}):")
        print(f"  起点收盘: {source_close:.2f} 元")
        print(f"  终点收盘: {current_close:.2f} 元")
        print(f"  涨跌额: {rise:+.2f} 元")
        print(f"  收益率: {rise_rate:+.2f}%")
```

### Shell (xh)

```bash
xh GET "http://tdx.acdzh.xyz/api/income?code=000001&start_date=20260501&days=5,10,20" User-Agent:"Mozilla/5.0"
```

### Shell (curl)

```bash
curl -H "User-Agent: Mozilla/5.0" "http://tdx.acdzh.xyz/api/income?code=000001&start_date=20260501&days=5,10,20"
```

## 注意事项

- **价格字段单位为元**，不需要除以 1000（与其他接口不同！）
- `rise_rate` 为小数，展示时乘以 100 得百分比
- 如果请求的 `days` 中某个天数对应的终点日期尚未到达（还没有那么多交易日），则该 offset 不会出现在返回结果中
- `source` 是起始日期当天（或最近交易日）的价格
- 收益率基于 `source.close`（起点收盘价）计算
- `count` 可能小于 `days` 参数中逗号分隔的数量（尚未到期的区间不返回）
