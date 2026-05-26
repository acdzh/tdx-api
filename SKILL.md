---
name: tdx-api-usage
description: 当用户需要查询、调试或编写对接 TDX A 股股票数据 HTTP API 的代码、脚本或调用示例时使用，并优先为后续分析与决策提供准确、可解释、可比较的数据支持。
keywords: "tdx, tdx api, stock api, quote api, kline api, search api, stock-info api, batch-quote api, 股票, A股, 沪市, 深市, 北交所, 实时行情, 五档行情, K线, 分时, 逐笔成交, 个股信息, 股票代码, 指数, 交易日, 收益区间, 通达信"
---

# TDX API Usage

按本 skill 对接 TDX A 股股票数据 API。目标是稳定地拿到数据、解释字段、校验数据边界，并输出有利于后续分析与决策的数据支持结果。

## 何时使用

- 用户要查询或调试 `quote`、`kline`、`minute`、`trade`、`trade-history`、`search`、`stock-info`、`codes`、`index`、`workday`、`income` 等股票 HTTP API。
- 用户要生成 Python、curl、JavaScript 等调用示例。
- 用户要核对接口参数、响应结构、单位换算或错误处理方式。
- 用户希望让 AI 基于更准确的股票数据做后续分析、比较、归纳或决策。

## 角色定位

- 本 skill 是面向上层 AI 的股票数据支持层，不替代上层 AI 的综合判断。
- 本 skill 应全力为 AI 提供准确、可解释、可比较、可用于后续分析与决策的股票数据结果。
- 本 skill 不应只返回未经整理的原始接口内容；应尽可能把关键字段、单位、时间范围、数据状态和异常信息整理清楚，降低上层 AI 的二次处理成本。

## 不适用场景

- 不用于基金净值、基金档案、债券、期货、宏观经济、外汇、数字货币等非股票场景。
- 当用户只给出六码纯数字代码时，不要默认它一定是股票代码；例如 `001309` 既可能被用户当作基金代码，也会被本 API 解析为股票 `德明利`。
- 如果用户明确要查基金，应该改用基金数据源，而不是继续调用本 skill 中的股票接口。

## 硬性约束

- host 固定为 `tdx.acdzh.xyz`。
- 若用户没有额外指定协议或端口，示例默认使用 `http://tdx.acdzh.xyz` 作为 base URL。
- 命令行脚本或 Python 客户端访问业务接口时，建议显式携带 `User-Agent` 请求头；实测 `/api/search`、`/api/quote`、`/api/kline` 等接口在缺少常见浏览器式请求头时可能返回 `403`。
- 不要把一次性抓数脚本写进仓库。
- 如果需要写一次性的 Python 脚本，必须先生成 UUID，并把脚本写入系统临时目录，文件名使用该 UUID。
- `quote`、`kline`、`minute`、`trade`、`trade-history`、`index` 中的大多数价格字段默认单位为 `厘`，展示给用户时通常需要换算为 `元`，即 `value / 1000`。
- `income` 接口返回的价格字段实测已经是 `元`，不要再次除以 `1000`。
- 返回成交量默认单位通常为 `手`，若需要换算成 `股`，使用 `value * 100`。

## 推荐流程

1. 明确用户要的接口、资产类型、股票代码、日期、周期和返回格式。
2. 若用户只给出六码纯数字代码，先确认这是股票还是基金；未确认前不要直接调用股票接口。
3. 若股票代码不明确，优先调用 `/api/search`。
4. 若需要检查服务可用性，先调用 `/api/server-status`。
5. 组装请求 URL 或请求体，优先给出最小可执行示例，并在示例里带上 `User-Agent`。
6. 读取统一响应结构中的 `code`、`message`、`data`。
7. 对价格、成交量、成交额做必要单位换算，并在结果里显式说明具体接口的单位前提。
8. 在输出前先整理关键字段、时间范围、数据状态和异常信息，避免把未经解释的原始字段直接抛给上层 AI。
9. 若 `code != 0` 或列表为空，明确区分“接口报错”和“该日期无数据”。

## 一次性 Python 脚本规则

只有在以下情况才建议写一次性 Python 脚本：

- 需要连续调用多个接口。
- 需要对响应做筛选、排序、聚合或换算。
- 需要比 curl 更稳定的超时、异常处理和 JSON 输出。

脚本创建流程必须严格遵循下面的顺序：

1. 先生成 UUID。
2. 以该 UUID 作为文件名，写入系统临时目录。
3. 执行脚本。
4. 用完后可删除该临时脚本。

参考命令：

```bash
SCRIPT_UUID="$(python - <<'PY'
import uuid
print(uuid.uuid4())
PY
)"
SCRIPT_PATH="/tmp/${SCRIPT_UUID}.py"
echo "$SCRIPT_PATH"
```

Python 示例模板：

```python
import requests

BASE_URL = "http://tdx.acdzh.xyz"
HEADERS = {"User-Agent": "Mozilla/5.0"}

resp = requests.get(
    f"{BASE_URL}/api/quote",
    params={"code": "000001"},
    headers=HEADERS,
    timeout=10,
)
resp.raise_for_status()
payload = resp.json()

if payload["code"] != 0:
    raise RuntimeError(payload["message"])

quote = payload["data"][0]
print({
    "code": quote["Code"],
    "close_yuan": quote["K"]["Close"] / 1000,
})
```

## 统一响应处理

所有接口都按下面结构处理：

```json
{
  "code": 0,
  "message": "success",
  "data": {}
}
```

处理规则：

- `code == 0` 视为成功。
- `code != 0` 时，优先向用户透出 `message`。
- 只有在 `code == 0` 的前提下再解析 `data`。
- 业务接口可能返回 HTTP `403`；若出现这种情况，先检查是否缺少 `User-Agent` 等常见请求头，而不是立即认定服务不可用。

## 常用接口选择建议

- 实时五档行情：`GET /api/quote`
- K 线：`GET /api/kline`
- 分时：`GET /api/minute`
- 逐笔成交：`GET /api/trade`
- 搜索股票：`GET /api/search`
- 股票概览：`GET /api/stock-info`
- 股票列表：`GET /api/codes`
- 批量行情：`POST /api/batch-quote`
- 指数：`GET /api/index`
- 交易日：`GET /api/workday`
- 收益区间：`GET /api/income`

当数据源必须明确区分时，优先使用：

- 通达信原始全量 K 线：`GET /api/kline-all/tdx`
- 同花顺前复权全量 K 线：`GET /api/kline-all/ths`

## 易错点

- `/api/minute` 和 `/api/trade` 在指定日期无数据时，实测返回 `{"Count":0,"List":null}`；这不等于接口失败。
- `/api/trade-history` 单次最多返回 2000 条，需要通过 `start` 和 `count` 分页。
- `/api/kline-all` 数据量可能很大，优先带 `limit`。
- 当用户只给股票名称时，不要直接猜代码，先调用 `/api/search`。
- 当用户只给六码纯数字代码时，不要默认它一定是股票；先确认资产类型，避免把基金代码误判成股票代码。
- 当用户关心“前复权还是原始数据”时，不要只用 `/api/kline`，要改用显式数据源接口。
- `/api/income` 的价格字段实测已经是 `元`，不要沿用 `quote`/`kline` 的 `厘 -> 元` 规则。

## 输出要求

- 默认优先输出可供后续分析与决策直接使用的数据摘要，而不是大段原始 JSON。
- 结果中优先说明查询范围、关键数据、单位换算、时间范围、数据状态和异常提示。
- 当字段对后续分析有帮助时，要把原始字段翻译成易理解的数据含义，例如价格、涨跌、成交量、区间表现、是否复权、是否盘中。
- 仅在用户明确需要调试或实现接口调用时，再展开请求方法、路径、参数和示例响应关键字段。
- 如果用了临时 Python 脚本，明确说明脚本位于临时目录，且文件名来自 UUID。

## 参考资料

- 速查表：`references/api-quick-reference.md`
- 原始文档：仓库根目录 `API_接口文档.md`
