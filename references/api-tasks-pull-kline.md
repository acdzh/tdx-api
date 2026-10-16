# POST /api/tasks/pull-kline

创建批量拉取 K 线数据的后台任务。

## 基本信息

| 项目 | 值 |
|---|---|
| 方法 | POST |
| 路径 | `/api/tasks/pull-kline` |
| 认证 | 无 |
| Content-Type | `application/json` |
| 请求头 | `User-Agent: Mozilla/5.0`（建议携带） |

## 请求参数（JSON Body）

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `codes` | array[string] | 是 | 股票代码数组，如 `["000001", "600519"]` |
| `tables` | array[string] | 否 | K 线类型数组，如 `["day", "week"]` |
| `dir` | string | 否 | 数据存储目录 |
| `limit` | int | 否 | 每只股票拉取的最大条数 |
| `start_date` | string | 否 | 起始日期，格式 `YYYYMMDD` |

### 请求体示例

```json
{
  "codes": ["000001", "600519"],
  "tables": ["day", "week"],
  "dir": "/data/kline",
  "limit": 1000,
  "start_date": "20240101"
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

url = "http://tdx.acdzh.com/api/tasks/pull-kline"
headers = {
    "User-Agent": "Mozilla/5.0",
    "Content-Type": "application/json"
}
payload = {
    "codes": ["000001", "600519", "000858"],
    "tables": ["day", "week"],
    "dir": "/data/kline",
    "limit": 500,
    "start_date": "20240101"
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
echo '{"codes":["000001","600519"],"tables":["day","week"],"dir":"/data/kline","limit":1000}' | \
  xh POST "http://tdx.acdzh.com/api/tasks/pull-kline" User-Agent:"Mozilla/5.0" Content-Type:"application/json"
```

### Shell (curl)

```bash
curl -X POST "http://tdx.acdzh.com/api/tasks/pull-kline" \
  -H "User-Agent: Mozilla/5.0" \
  -H "Content-Type: application/json" \
  -d '{
    "codes": ["000001", "600519"],
    "tables": ["day", "week"],
    "dir": "/data/kline",
    "limit": 1000,
    "start_date": "20240101"
  }'
```

## 注意事项

- 这是一个异步任务接口，创建后立即返回任务 ID
- 任务在后台执行，可通过 task_id 查询状态或取消
- 大量股票 + 多种 K 线类型组合时，任务执行时间可能较长
- `limit` 参数用于控制每只股票拉取的数据量，避免超大响应
- 使用 `POST /api/tasks/{task_id}/cancel` 可取消未完成的任务
