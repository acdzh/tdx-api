# API Quick Reference

本文件是 `tdx-api-usage` skill 的速查表，按常见任务给出推荐接口、关键参数和注意事项。

## Base URL

- 默认：`http://tdx.acdzh.xyz`
- host 固定：`tdx.acdzh.xyz`

## 统一响应

```json
{
  "code": 0,
  "message": "success",
  "data": {}
}
```

## 常用读接口

| 场景 | 方法 | 路径 | 关键参数 | 备注 |
|---|---|---|---|---|
| 五档行情 | GET | `/api/quote` | `code` | 支持多个代码，逗号分隔 |
| K 线 | GET | `/api/kline` | `code`, `type` | 常用 `day`, `minute30`, `week`, `month` |
| 分时 | GET | `/api/minute` | `code`, `date` | `date` 格式 `YYYYMMDD` |
| 逐笔成交 | GET | `/api/trade` | `code`, `date` | 当日最多 1800 条 |
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

- 价格：`厘 / 1000 = 元`
- 成交额：`厘 / 1000 = 元`
- 成交量：`手 * 100 = 股`

## 典型示例

```bash
curl "http://tdx.acdzh.xyz/api/quote?code=000001"
curl "http://tdx.acdzh.xyz/api/kline?code=000001&type=day"
curl "http://tdx.acdzh.xyz/api/minute?code=000001&date=20241103"
curl "http://tdx.acdzh.xyz/api/search?keyword=平安"
curl -X POST "http://tdx.acdzh.xyz/api/batch-quote" \
  -H "Content-Type: application/json" \
  -d '{"codes":["000001","600519"]}'
```

## 使用建议

- 代码不明确时，先 `search` 再调用其他接口。
- 需要“完整历史”且关心数据源时，直接用 `/api/kline-all/tdx` 或 `/api/kline-all/ths`。
- `minute` 返回空列表时，应视为该日期无数据，而不是请求失败。
- 大结果集优先使用 `limit`，避免无意义的超大响应。
