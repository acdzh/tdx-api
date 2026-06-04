# GET /api/server-status

检查服务是否在线运行。

## 基本信息

| 项目 | 值 |
|---|---|
| 方法 | GET |
| 路径 | `/api/server-status` |
| 认证 | 无 |
| 请求头 | `User-Agent: Mozilla/5.0`（建议携带） |

## 请求参数

无参数。

## 返回格式

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "status": "running",
    "connected": true,
    "version": "1.0.0",
    "uptime": "unknown"
  }
}
```

## 字段说明

| 字段 | 类型 | 说明 |
|---|---|---|
| `data.status` | string | 服务状态，正常时为 `"running"` |
| `data.connected` | boolean | 是否已连接到数据源 |
| `data.version` | string | 服务版本号 |
| `data.uptime` | string | 运行时间，可能为 `"unknown"` |

## 返回示例

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "status": "running",
    "connected": true,
    "version": "1.0.0",
    "uptime": "unknown"
  }
}
```

## 代码示例

### Python

```python
import requests

url = "http://tdx.acdzh.xyz/api/server-status"
headers = {"User-Agent": "Mozilla/5.0"}

response = requests.get(url, headers=headers)
data = response.json()

if data["code"] == 0:
    status = data["data"]
    print(f"服务状态: {status['status']}")
    print(f"已连接: {status['connected']}")
    print(f"版本: {status['version']}")
else:
    print(f"请求失败: {data['message']}")
```

### Shell (xh)

```bash
xh GET "http://tdx.acdzh.xyz/api/server-status" User-Agent:"Mozilla/5.0"
```

### Shell (curl)

```bash
curl -H "User-Agent: Mozilla/5.0" "http://tdx.acdzh.xyz/api/server-status"
```

## 使用场景

- 部署后检查服务是否正常启动
- 定时健康检查
- 调用其他接口前先确认服务可用性
