# API参考

## Tiller版本
```
GET http://127.0.0.1:9855/tiller/v2/version/json
```

## Release汇总
```
# List releases with status `DEPLOYED` from all namespaces
GET http://127.0.0.1:9855/tiller/v2/releases/json

# List releases with status `DEPLOYED` from `default` namespace
GET http://127.0.0.1:9855/tiller/v2/releases/json?namespace=default

# List releases from all namespaces for a list of statuses
GET http://127.0.0.1:9855/tiller/v2/releases/json?status_codes=DEPLOYED&&status_codes=DELETED

# List releases from `default` namespace for a list of statuses
GET http://127.0.0.1:9855/tiller/v2/releases/json?namespace=default&status_codes=DEPLOYED&&status_codes=DELETED

# List releases with any status from all namespaces
GET http://127.0.0.1:9855/tiller/v2/releases/json?all=true

Available query parameters:
  namespace=<name of namespace>|EMPTY(for all namespaces)
  sort_by=NAME|LAST_RELEASED
  all=true|false
  sort_order=ASC|DESC
  status_codes=UNKNOWN, DEPLOYED, DELETED, SUPERSEDED, FAILED, DELETING
```

## Release状态
```
GET http://127.0.0.1:9855/tiller/v2/releases/my-release/status/json
```

## Release内容
```

GET http://127.0.0.1:9855/tiller/v2/releases/my-release/content/json
GET http://127.0.0.1:9855/tiller/v2/releases/my-release/content/json?format_values_as_json=true

```

## Release历史
```
GET http://127.0.0.1:9855/tiller/v2/releases/my-release/json?max=10
```

## Release回滚
```
GET http://127.0.0.1:9855/tiller/v2/releases/my-release/rollback/json
```

## 从URL安装Release

```
# 在default命名空间中安装Chart
POST http://127.0.0.1:9855/tiller/v2/releases/my-release/json

{
	"chart_url": "https://github.com/tamalsaha/test-chart/raw/master/test-chart-0.1.0.tgz",
	"values": {
		"raw": "{\"ns\":\"c10\",\"clusterName\":\"h505\"}"
	}
}

# 在自定义的kube-system命名空间中安装Chart
POST http://127.0.0.1:9855/tiller/v2/releases/my-release/json

{
	"chart_url": "https://github.com/tamalsaha/test-chart/raw/master/test-chart-0.1.0.tgz",
	"namespace": "kube-system",
	"values": {
		"raw": "{\"ns\":\"c10\",\"clusterName\":\"h505\"}"
	}
}

# 使用自定义的values.yaml，在自定义的kube-system命名空间中安装Chart

## values.yaml
proxy:
  secretToken: mytoken
rbac:
   enabled: false

## 把values.yaml转换为JSON格式，并以字符串形式传递给values.raw
{
  "proxy": {
    "secretToken": "mytoken"
  },
  "rbac": {
    "enabled": false
  }
}

POST http://127.0.0.1:9855/tiller/v2/releases/my-release/json

{
	"chart_url": "https://github.com/tamalsaha/test-chart/raw/master/test-chart-0.1.0.tgz",
	"namespace": "kube-system",
	"values": {
		"raw": "{ \"proxy\": { \"secretToken\": \"mytoken\" }, \"rbac\": { \"enabled\": false } }"
	}
}
```

## 从稳定的Kube应用安装最新版本的Release

```
POST http://127.0.0.1:9855/tiller/v2/releases/my-release/json

{
	"chart_url": "stable/fluent-bit"
}
```

## 从稳定的Kube应用安装特定版本的Release

```
POST http://127.0.0.1:9855/tiller/v2/releases/my-release/json

{
	"chart_url": "stable/fluent-bit/0.1.2"
}
```

## Release更新

```
PUT http://127.0.0.1:9855/tiller/v2/releases/my-release/json

{
	"chart_url": "https://github.com/tamalsaha/test-chart/raw/master/test-chart-0.1.0.tgz",
	"values": {
		"raw": "{\"ns\":\"c15\",\"clusterName\":\"h505\"}"
	}
}
```

## Release卸载

```
DELETE http://127.0.0.1:9855/tiller/v2/releases/my-release/json
```

## Release卸载及清除

```
DELETE http://127.0.0.1:9855/tiller/v2/releases/my-release/json?purge=true
```
