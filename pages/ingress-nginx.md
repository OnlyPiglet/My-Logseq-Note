- config-map 中设置容器的等级日志 error-log-level
- [error-log-level 参数说明](https://nginx.org/en/docs/ngx_core_module.html#error_log)
- `debug, info, notice, warn, error, crit, alert, or emerg`
- # ingress-nginx 管理端链接
	- 获取当前内存中的 后端backend
	- curl http://127.0.0.1:10246/configuration/backends
	- 获取域名对应的证书
	- curl http://127.0.0.1:10246/configuration/certs?hostname=
- # ingress-nginx 多个ingress 相同 host 打到不同的 service 会冲突吗
	- 在开启允许同域名路由规则存在的前提下，早创建的 ingress 会生效，代码见
	- ingress-nginx/internal/ingress/controller/controller.go
	- 初始化为 DefBackend ，当第一个的 ingress 设置生效之后 ，会讲IsDefBackend 设置为 false，当第二条冲突的 ingress 会直接 break 跳过
	- ```go
	  if !loc.IsDefBackend {
	  	klog.V(3).Infof("Location %q already configured for server %q with upstream %q (Ingress %q)",
	  	loc.Path, server.Hostname, loc.Backend, ingKey)
	      break
	  }
	  ```