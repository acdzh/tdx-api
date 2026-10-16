# POST /api/tasks/pull-trade

创建拉取逐笔成交数据的后台任务。

## 基本信息

| 项目 | 值 |
|---|---|
| 方法 | POST |
| 路径 | `/api/tasks/pull-trade` |
| 认证 | 无 |
| Content-Type | `application/json` |
| 请求头 | `User-Agent: Mozilla/5.0`（建议携带） |

## 请求参数（JSON Body）

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `code` | string | 是 | 股票代码，如 `000001` |
| `dir` | string | 否 | 数据存储目录 |
| `start_year` | int | 否 | 起始年份，如 `2024` |
| `end_year` | int | 否 | 结束年份，如 `2026` |

### 请求体示例

```json
{
  "code": "000001",
  "dir": "/data/trade",
  "start_year": 2024,
  "end_year": 2026
}
```

## 返回格式

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "task_id": "abc123-def456-...",
    "status": "pending",
    "created_at": "2026-06-03T10:00:00+08:00"
  }
}
```

## 字段说明

| 字段 | 类型 | 说明 |
|---|---|---|
| `data.task_id` | string | 任务 ID，可用于查询或取消任务 |
| `data.status` | string | 任务状态：`pending`、`running`、`completed`、`cancelled` |
| `data.created_at` | string | 任务创建时间 |

## 代码示例

### Python

```python
import requests

url = "http://tdx.acdzh.com/api/tasks/pull-trade"
headers = {
    "User-Agent": "Mozilla/5.0",
    "Content-Type": "application/json"
}
payload = {
    "code": "000001",
    "dir": "/data/trade",
    "start_year": 2024,
    "end_year": 2026
}

response = requests.post(url, headers=headers, json=payload)
data = response.json()

if data["code"] == 0:
    task = data["data"]
    print(f"任务已创建: {task['task_id']}")
    print(f"状态: {task['status']}")
else:
    print(f"创建失败: {data['message']}")
```

### Shell (xh)

```bash
echo '{"code":"000001","dir":"/data/trade","start_year":2024,"end_year":2026}' | \
  xh POST "http://tdx.acdzh.com/api/tasks/pull-trade" User-Agent:"Mozilla/5.0" Content-Type:"application/json"
```

### Shell (curl)

```bash
curl -X POST "http://tdx.acdzh.com/api/tasks/pull-trade" \
  -H "User-Agent: Mozilla/5.0" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "000001",
    "dir": "/data/trade",
    "start_year": 2024,
    "end_year": 2026
  }'
```

## 注意事项

- 这是一个异步任务接口，创建后立即返回任务 ID
- 逐笔成交数据量非常大，拉取多年数据可能耗时很长
- 一次只能为一只股票创建任务（不像 pull-kline 支持多只）
- 使用 `POST /api/tasks/{task_id}/cancel` 可取消未完成的任务
- `start_year` 和 `end_year` 用于限定拉取的时间范围
