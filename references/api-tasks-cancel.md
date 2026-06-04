# POST /api/tasks/{task_id}/cancel

取消一个正在执行或排队中的后台任务。

## 基本信息

| 项目 | 值 |
|---|---|
| 方法 | POST |
| 路径 | `/api/tasks/{task_id}/cancel` |
| 认证 | 无 |
| 请求头 | `User-Agent: Mozilla/5.0`（建议携带） |

## 路径参数

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `task_id` | string | 是 | 任务 ID，由创建任务接口返回 |

## 请求体

无请求体。

## 返回格式

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "task_id": "abc123-def456-...",
    "status": "cancelled"
  }
}
```

## 字段说明

| 字段 | 类型 | 说明 |
|---|---|---|
| `data.task_id` | string | 被取消的任务 ID |
| `data.status` | string | 任务状态，取消成功后为 `"cancelled"` |

## 代码示例

### Python

```python
import requests

task_id = "abc123-def456-..."
url = f"http://tdx.acdzh.xyz/api/tasks/{task_id}/cancel"
headers = {"User-Agent": "Mozilla/5.0"}

response = requests.post(url, headers=headers)
data = response.json()

if data["code"] == 0:
    print(f"任务 {task_id} 已取消")
else:
    print(f"取消失败: {data['message']}")
```

### Shell (xh)

```bash
xh POST "http://tdx.acdzh.xyz/api/tasks/abc123-def456/cancel" User-Agent:"Mozilla/5.0"
```

### Shell (curl)

```bash
curl -X POST "http://tdx.acdzh.xyz/api/tasks/abc123-def456/cancel" \
  -H "User-Agent: Mozilla/5.0"
```

## 注意事项

- 只能取消状态为 `pending` 或 `running` 的任务
- 已完成（`completed`）的任务无法取消
- 取消操作是尽力而为，已执行的部分不会回滚
- `task_id` 来自创建任务接口（`/api/tasks/pull-kline` 或 `/api/tasks/pull-trade`）的返回值
