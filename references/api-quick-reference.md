# API Quick Reference

本文件是 `tdx-api-usage` skill 的速查表，按常见股票任务给出推荐接口、关键参数和注意事项。

## 适用范围

- 本速查表面向 TDX A 股股票 API，不覆盖基金净值、基金档案、债券、期货、宏观经济等场景。
- 若用户只给出六码纯数字代码，先确认资产类型；例如 `001309` 在本 API 中会命中股票 `德明利`，但用户也可能想查询同代码基金。

## Base URL

- 默认：`http://tdx.acdzh.xyz`
- host 固定：`tdx.acdzh.xyz`
- 命令行脚本或 Python 客户端访问业务接口时，建议显式携带 `User-Agent` 请求头；实测部分接口在缺少常见浏览器式请求头时可能返回 `403`

## 统一响应

```json
{
  "code": 0,
  "message": "success",
  "data": {}
}
```

- 业务接口若返回 HTTP `403`，先检查请求头，尤其是 `User-Agent`

## 常用读接口

| 场景 | 方法 | 路径 | 关键参数 | 备注 |
|---|---|---|---|---|
| 服务状态 | GET | `/api/server-status` | 无 | 用于检查服务是否在线 |
| 五档行情 | GET | `/api/quote` | `code` | 支持多个代码，逗号分隔 |
| K 线 | GET | `/api/kline` | `code`, `type` | 常用 `day`, `minute30`, `week`, `month` |
| 分时 | GET | `/api/minute` | `code`, `date` | `date` 格式 `YYYYMMDD` |
| 逐笔成交 | GET | `/api/trade` | `code`, `date` | 当日最多 1800 条 |
| 历史逐笔成交 | GET | `/api/trade-history` | `code`, `date`, `start`, `count` | 单次最多 2000 条 |
| 搜索股票 | GET | `/api/search` | `keyword` | 最多 50 条 |
| 股票概览 | GET | `/api/stock-info` | `code` | 包含 quote、day kline、minute |
| 股票列表 | GET | `/api/codes` | `exchange` | `sh` / `sz` / `bj` / `all` |
| 指数 | GET | `/api/index` | `code`, `type` | 例：`sh000001` |
| 交易日信息 | GET | `/api/workday` | `date`, `count` | 支持 `YYYYMMDD` 和 `YYYY-MM-DD` |
| 收益区间 | GET | `/api/income` | `code`, `start_date`, `days` | `days` 可逗号分隔 |

## 常用写接口

| 场景 | 方法 | 路径 | 请求体 |
|---|---|---|---|
| 批量行情 | POST | `/api/batch-quote` | `{"codes":["000001","600519"]}` |
| 创建 K 线任务 | POST | `/api/tasks/pull-kline` | `codes`, `tables`, `dir`, `limit`, `start_date` |
| 创建成交任务 | POST | `/api/tasks/pull-trade` | `code`, `dir`, `start_year`, `end_year` |
| 取消任务 | POST | `/api/tasks/{task_id}/cancel` | 无 |

## 明确数据源的 K 线接口

| 场景 | 方法 | 路径 | 备注 |
|---|---|---|---|
| 通达信原始全量 K 线 | GET | `/api/kline-all/tdx` | 支持分钟、小时、日、周、月、季、年 |
| 同花顺前复权全量 K 线 | GET | `/api/kline-all/ths` | 只支持 `day` / `week` / `month` |

## 任务到接口选择矩阵

| 任务 | 首选接口 | 可补充接口 | 关键提醒 |
|---|---|---|---|
| 确认股票代码或消歧 | `GET /api/search` | `GET /api/codes` | 关键词过泛时结果会较多；六码纯数字代码先确认是否其实是基金 |
| 查看单只股票当前实时情况 | `GET /api/quote` | `GET /api/stock-info` | 实时行情价格字段通常是厘，展示前换算为元 |
| 一次性看单股概览 | `GET /api/stock-info` | `GET /api/quote`, `GET /api/kline` | 聚合接口内容较多，先提炼关键块，不要原样展开 |
| 查看单只股票近期日线走势 | `GET /api/kline` | `GET /api/kline-all/tdx`, `GET /api/kline-all/ths` | 若用户关心复权或完整历史，优先改用显式数据源接口 |
| 查看完整历史 K 线 | `GET /api/kline-all/tdx` | `GET /api/kline-all/ths` | `tdx` 是原始口径，`ths` 是前复权口径；必须显式说明 |
| 查看盘中分时走势 | `GET /api/minute` | `GET /api/trade` | 指定日期无数据时可能返回 `Count=0, List=null`，不等于接口失败 |
| 查看当日逐笔成交 | `GET /api/trade` | `GET /api/trade-history` | 适合看当日成交活跃度；若要更完整历史逐笔，改用 `trade-history` |
| 查看历史逐笔成交明细 | `GET /api/trade-history` | `GET /api/trade` | 单次最多 2000 条，需要分页；不要一次性原样输出大量明细 |
| 比较多只股票实时表现 | `POST /api/batch-quote` | `GET /api/quote` | 先统一单位和时间口径，再做并列比较 |
| 查看指数阶段表现 | `GET /api/index` | `GET /api/kline` | 指数字段里可能包含上涨/下跌家数，适合补充市场宽度信息 |
| 判断某天是否交易日 | `GET /api/workday` | 无 | 支持 `YYYYMMDD` 和 `YYYY-MM-DD` 两种日期格式 |
| 计算指定起点后的区间收益 | `GET /api/income` | `GET /api/kline` | `income` 价格字段实测已是元，不要重复除以 `1000` |
| 获取股票池或全市场代码列表 | `GET /api/codes` | `GET /api/search` | 适合先拿范围，再做批量筛选或二次查询 |

## 单位换算

- `quote` / `kline` / `minute` / `trade` / `trade-history` / `index` 的大多数价格字段：`厘 / 1000 = 元`
- `income` 的价格字段实测已经是 `元`
- 成交额：多数行情接口可按 `厘 / 1000 = 元` 解释，但应结合具体接口字段核对
- 成交量：`手 * 100 = 股`

## 字段语义映射

### 通用响应字段

| 字段 | 含义 | 说明 |
|---|---|---|
| `code` | 业务状态码 | `0` 表示成功，非 `0` 优先查看 `message` |
| `message` | 业务提示信息 | 用于说明成功、报错或异常原因 |
| `data` | 业务数据主体 | 只有 `code == 0` 时再继续解析 |
| `Count` / `count` | 记录数 | 常见于列表型响应，大小写需按实际接口区分 |
| `List` / `list` | 数据列表 | 常见于行情列表、K 线列表、收益区间列表 |

### `quote` / `batch-quote` 高频字段

| 字段 | 含义 | 单位或说明 |
|---|---|---|
| `Code` | 股票代码 | 例如 `000001` |
| `Exchange` | 交易所标识 | 实际数值需结合返回结果解释，常见为沪深区分 |
| `K.Last` | 最新价 | 原始单位通常为厘，展示时常换算为元 |
| `K.Open` | 今开 | 原始单位通常为厘 |
| `K.High` | 最高价 | 原始单位通常为厘 |
| `K.Low` | 最低价 | 原始单位通常为厘 |
| `K.Close` | 昨收或参考收盘价 | 原始单位通常为厘；在实时行情中常可作为对比基准 |
| `Rate` | 涨跌幅 | 返回值常可直接按百分比理解，如 `0.09` 表示 `0.09%` |
| `TotalHand` | 成交量 | 单位通常为手，若需换算成股乘以 `100` |
| `Amount` | 成交额 | 多数情况下可按厘换算为元，但需结合接口口径核对 |
| `BuyLevel` | 买盘五档 | 含价格和挂单数量 |
| `SellLevel` | 卖盘五档 | 含价格和挂单数量 |
| `ServerTime` | 行情时间 | 服务端返回的时间标记，不直接等同于标准日期时间字符串 |

### `kline` / `kline-all` / `index` 高频字段

| 字段 | 含义 | 单位或说明 |
|---|---|---|
| `Time` | 对应时间点 | 常见为带时区的 ISO 时间字符串 |
| `Last` | 上一周期收盘或参考价 | 原始单位通常为厘 |
| `Open` | 开盘价 | 原始单位通常为厘 |
| `High` | 最高价 | 原始单位通常为厘 |
| `Low` | 最低价 | 原始单位通常为厘 |
| `Close` | 收盘价 | 原始单位通常为厘 |
| `Volume` | 成交量 | 单位通常为手 |
| `Amount` | 成交额 | 部分接口有值，部分接口可能为 `0`，需按实际返回解释 |
| `UpCount` | 上涨家数或上涨数量 | 常见于指数类数据 |
| `DownCount` | 下跌家数或下跌数量 | 常见于指数类数据 |
| `meta.source` | 数据源 | 例如 `tdx` 或 `ths` |
| `meta.type` | K 线类型 | 例如 `day`、`week`、`month` |

### `minute` / `trade` / `trade-history` 高频字段

| 字段 | 含义 | 单位或说明 |
|---|---|---|
| `Time` | 分时点或成交时间 | 常见为带时区的 ISO 时间字符串 |
| `Price` | 成交价 | 原始单位通常为厘，展示时常换算为元 |
| `Volume` | 成交量 | 单位通常为手 |
| `Status` | 成交方向或状态标记 | 具体值含义需结合业务口径理解，不建议臆断 |
| `Number` | 逐笔序号或附加编号 | 常见于 `trade-history` |
| `Count` | 返回条数 | 当日无数据时可能为 `0` |
| `List` | 成交列表 | 无数据时实测可能为 `null` 而不是空数组 |

### `search` / `codes` 高频字段

| 字段 | 含义 | 说明 |
|---|---|---|
| `code` | 股票代码 | 例如 `601318` |
| `name` | 股票名称 | 例如 `中国平安` |
| `exchange` | 交易所代码 | 常见为 `sh` / `sz` / `bj` |
| `total` | 总数 | 常见于 `codes` 返回结果 |
| `exchanges` | 各交易所数量统计 | 常见于 `codes` 返回结果 |
| `codes` | 股票列表 | 常见于 `codes` 返回结果 |

### `stock-info` 高频字段

| 字段 | 含义 | 说明 |
|---|---|---|
| `quote` | 实时行情块 | 结构与 `quote` 接口相近 |
| `kline_day` | 近期日线块 | 常见包含 `Count` 和 `List` |
| `minute` | 分时块 | 用于单股概览中的分时数据 |

### `income` 高频字段

| 字段 | 含义 | 单位或说明 |
|---|---|---|
| `offset` | 观察区间天数 | 例如 `5`、`20` |
| `source` | 起点价格信息 | 字段内价格实测为元 |
| `current` | 终点价格信息 | 字段内价格实测为元 |
| `rise` | 区间涨跌额 | 实测单位为元 |
| `rise_rate` | 区间收益率 | 常见为小数，展示时可换算为百分比 |
| `time` | 对应终点时间 | 常见为带时区的 ISO 时间字符串 |

## 典型示例

```bash
curl -H "User-Agent: Mozilla/5.0" "http://tdx.acdzh.xyz/api/quote?code=000001"
curl -H "User-Agent: Mozilla/5.0" "http://tdx.acdzh.xyz/api/kline?code=000001&type=day"
curl -H "User-Agent: Mozilla/5.0" "http://tdx.acdzh.xyz/api/minute?code=000001&date=20241103"
curl -H "User-Agent: Mozilla/5.0" "http://tdx.acdzh.xyz/api/search?keyword=平安"
curl -X POST "http://tdx.acdzh.xyz/api/batch-quote" \
  -H "User-Agent: Mozilla/5.0" \
  -H "Content-Type: application/json" \
  -d '{"codes":["000001","600519"]}'
```

## 使用建议

- 代码不明确时，先 `search` 再调用其他接口。
- 只给六码数字代码时，先确认是股票还是基金，不要直接假定为股票。
- 需要“完整历史”且关心数据源时，直接用 `/api/kline-all/tdx` 或 `/api/kline-all/ths`。
- `minute` 和 `trade` 在无数据日期实测返回 `{"Count":0,"List":null}`，应视为该日期无数据，而不是请求失败。
- 大结果集优先使用 `limit`，避免无意义的超大响应。
