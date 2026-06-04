# GET /api/codes

获取股票列表（按交易所筛选或全市场）。

## 基本信息

| 项目 | 值 |
|---|---|
| 方法 | GET |
| 路径 | `/api/codes` |
| 认证 | 无 |
| 请求头 | `User-Agent: Mozilla/5.0`（建议携带） |

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `exchange` | string | 是 | 交易所筛选，可选值见下方 |

### exchange 可选值

| 值 | 说明 |
|---|---|
| `sh` | 上海证券交易所 |
| `sz` | 深圳证券交易所 |
| `bj` | 北京证券交易所 |
| `all` | 全部交易所 |

## 返回格式

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "total": 5526,
    "exchanges": {
      "bj": 317,
      "sh": 2315,
      "sz": 2894
    },
    "codes": [
      {
        "code": "000001",
        "exchange": "sz",
        "name": "平安银行"
      }
    ]
  }
}
```

## 字段说明

| 字段 | 类型 | 说明 |
|---|---|---|
| `data.total` | int | 筛选条件下的股票总数 |
| `data.exchanges` | object | 各交易所股票数量统计 |
| `data.exchanges.sh` | int | 上海交易所股票数 |
| `data.exchanges.sz` | int | 深圳交易所股票数 |
| `data.exchanges.bj` | int | 北京交易所股票数 |
| `data.codes` | array | 股票列表 |
| `data.codes[].code` | string | 股票代码 |
| `data.codes[].exchange` | string | 交易所标识 |
| `data.codes[].name` | string | 股票名称 |

## 返回示例

### 深圳交易所

请求：`GET /api/codes?exchange=sz`

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "total": 2894,
    "exchanges": {
      "bj": 317,
      "sh": 2315,
      "sz": 2894
    },
    "codes": [
      {"code": "000001", "exchange": "sz", "name": "平安银行"},
      {"code": "000002", "exchange": "sz", "name": "万 科Ａ"},
      {"code": "000004", "exchange": "sz", "name": "*ST国华"},
      {"code": "000006", "exchange": "sz", "name": "深振业Ａ"},
      {"code": "000007", "exchange": "sz", "name": "全新好"}
    ]
  }
}
```

### 全部交易所

请求：`GET /api/codes?exchange=all`

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "total": 5526,
    "exchanges": {
      "bj": 317,
      "sh": 2315,
      "sz": 2894
    },
    "codes": [
      {"code": "600000", "exchange": "sh", "name": "浦发银行"},
      {"code": "600004", "exchange": "sh", "name": "白云机场"},
      {"code": "600006", "exchange": "sh", "name": "东风股份"}
    ]
  }
}
```

## 代码示例

### Python

```python
import requests

url = "http://tdx.acdzh.xyz/api/codes"
headers = {"User-Agent": "Mozilla/5.0"}
params = {"exchange": "all"}

response = requests.get(url, headers=headers, params=params)
data = response.json()

if data["code"] == 0:
    result = data["data"]
    print(f"总计: {result['total']} 只股票")
    print(f"上海: {result['exchanges']['sh']} 只")
    print(f"深圳: {result['exchanges']['sz']} 只")
    print(f"北京: {result['exchanges']['bj']} 只")

    # 筛选特定条件
    codes = result["codes"]
    bank_stocks = [s for s in codes if "银行" in s["name"]]
    print(f"\n银行股: {len(bank_stocks)} 只")
    for s in bank_stocks[:5]:
        print(f"  {s['code']} {s['name']} ({s['exchange']})")
```

### Shell (xh)

```bash
# 深圳
xh GET "http://tdx.acdzh.xyz/api/codes?exchange=sz" User-Agent:"Mozilla/5.0"

# 全部
xh GET "http://tdx.acdzh.xyz/api/codes?exchange=all" User-Agent:"Mozilla/5.0"
```

### Shell (curl)

```bash
curl -H "User-Agent: Mozilla/5.0" "http://tdx.acdzh.xyz/api/codes?exchange=all"
```

## 注意事项

- `exchanges` 字段始终返回所有交易所的统计，不受 `exchange` 参数过滤影响
- `total` 字段为当前筛选条件下 `codes` 数组的实际长度
- 结果集可能较大（全市场 5000+ 只），客户端需考虑内存和渲染性能
- 适合先获取股票池范围，再做批量筛选或二次查询
- 不要混淆 `exchange` 参数值（小写 `sh/sz/bj/all`）
