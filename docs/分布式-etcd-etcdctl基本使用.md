# 分布式 etcdctl 基本使用

## 基本介绍

etcdctl 是 etcd3 的一个简单的命令行程序。

## 命令

命令|用途
---|:--:
get|获取键或一系列键
put|把给定的键放入到存储里
del|删除指定的键或者一系列的键
txn|Txn在一个事务里操作所有的请求
compaction|压缩 etcd 中的事件历史记录
alarm disarm|取消所有的闹钟
alarm list|列出所有的闹钟
defrag|对具有给定 endpoints 的 etcd 成员的存储进行碎片整理
endpoint health|检查 endpoints 的健康
endpoint status|打印 endpoint 的状态
watch|监视 keys 或 prefixes 的事件流
version|打印 etcdctl 的版本
lease grant|创建 leases
lease revoke|调用 leases
lease timetolive|获取 leases 的信息
lease keep-alive|保持 leases 存活（更新 leases）
member add|给集群添加一个成员
member remove|从集群中删除一个成员
member update|更新集群里的一个成员
member list|列出集群中所有的成员
snapshot save|将 etcd 节点后端快照存储到给定的文件里
snapshot restore|将 etcd 成员快照还原到 etcd 目录
snapshot status|获取给定文件的后端快照状态
make-mirror|在目标 etcd 集群上生成镜像
migrate|从 v2 版本存储中迁移 keys 到 mvcc 版本存储
lock|获取named lock
elect|观察并参与 leader 选举
auth enable|开启权限校验
auth disable|关闭权限校验
user add|添加一个用户
user delete|删除一个用户
user get|获取一个用户的详细信息
user list|列出所有的用户
user passwd|改变一个用户的密码
user grant-role|给一个用户赋予一个角色
user revoke-role|从用户侧收回角色
role add|添加一个角色
role delete|删除一个角色
role get|获取一个角色的详细信息
role list|获取所有的角色
role grant-permission|赋予角色 key
role revoke-permission|从角色处收回 key
check perf|检查 etcd 集群的性能

## 选项

命令|用途
---|:--:
--cacert=""|使用此 CA 包验证启用 TLS 的安全服务器证书
--cert=""|使用此 TLS 证书文件标识安全客户端
--command-timeout=5s|短时间运行命令超时（不包括拨号超时）
--dial-timeout=2s|客户端连接拨号超时时间
--endpoints=[127.0.0.1:2379]|gRPC endpoints
--hex[=false]|将字节字符串打印为十六进制编码字符串
--insecure-skip-tls-verify[=false]|跳过服务器证书校验
--insecure-transport[=true]|禁用客户端连接的传输安全
--key=""|用此TLS key 文件识别安全客户端
--user=""|用户名[:password]登陆认证（如果未提供会提示）
-w, --write-out="simple"|设置输出格式