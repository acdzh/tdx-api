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

## 单位换算

- `quote` / `kline` / `minute` / `trade` / `trade-history` / `index` 的大多数价格字段：`厘 / 1000 = 元`
- `income` 的价格字段实测已经是 `元`
- 成交额：多数行情接口可按 `厘 / 1000 = 元` 解释，但应结合具体接口字段核对
- 成交量：`手 * 100 = 股`

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
