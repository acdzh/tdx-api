---
name: tdx-api-usage
description: 当用户需要查询、调试或编写对接 TDX 股票数据 HTTP API 的代码、脚本或调用示例时使用。固定使用 host `tdx.acdzh.xyz`，并优先通过一次性 Python 脚本完成数据获取与验证。
---

# TDX API Usage

按本 skill 对接 TDX 股票数据 API。目标是稳定地拿到数据、解释字段，并给出可直接执行的调用示例。

## 何时使用

- 用户要查询或调试 `quote`、`kline`、`minute`、`trade`、`search`、`stock-info`、`codes`、`index`、`workday`、`income` 等 HTTP API。
- 用户要生成 Python、curl、JavaScript 等调用示例。
- 用户要核对接口参数、响应结构、单位换算或错误处理方式。

## 硬性约束

- host 固定为 `tdx.acdzh.xyz`。
- 若用户没有额外指定协议或端口，示例默认使用 `http://tdx.acdzh.xyz` 作为 base URL。
- 不要把一次性抓数脚本写进仓库。
- 如果需要写一次性的 Python 脚本，必须先生成 UUID，并把脚本写入系统临时目录，文件名使用该 UUID。
- 返回价格默认单位为 `厘`，展示给用户时通常需要换算为 `元`，即 `value / 1000`。
- 返回成交量默认单位为 `手`，若需要换算成 `股`，使用 `value * 100`。

## 推荐流程

1. 明确用户要的接口、股票代码、日期、周期和返回格式。
2. 若股票代码不明确，优先调用 `/api/search`。
3. 若需要检查服务可用性，先调用 `/api/server-status`。
4. 组装请求 URL 或请求体，优先给出最小可执行示例。
5. 读取统一响应结构中的 `code`、`message`、`data`。
6. 对价格、成交量、成交额做必要单位换算，并在结果里显式说明。
7. 若 `code != 0` 或列表为空，明确区分“接口报错”和“该日期无数据”。

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

resp = requests.get(
    f"{BASE_URL}/api/quote",
    params={"code": "000001"},
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

- `/api/minute` 指定日期可能返回空列表；空列表不等于接口失败。
- `/api/trade-history` 单次最多返回 2000 条，需要通过 `start` 和 `count` 分页。
- `/api/kline-all` 数据量可能很大，优先带 `limit`。
- 当用户只给股票名称时，不要直接猜代码，先调用 `/api/search`。
- 当用户关心“前复权还是原始数据”时，不要只用 `/api/kline`，要改用显式数据源接口。

## 输出要求

- 优先给出可直接运行的示例。
- 明确写出请求方法、路径、参数和示例响应关键字段。
- 解释数值时带上单位换算。
- 如果用了临时 Python 脚本，明确说明脚本位于临时目录，且文件名来自 UUID。

## 参考资料

- 速查表：`references/api-quick-reference.md`
- 原始文档：仓库根目录 `API_接口文档.md`
