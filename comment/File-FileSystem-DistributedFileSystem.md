### 架构

![img](./img/webp)

### rados观察命令

```shell
ls -l -i
printf '%x\n' inode_num
rados lspools
rados ls -p cephfs_metadata | grep inode_num #查找元数据inode
rados listomapkeys -p cephfs_metadata inode_num #查看inode元数据key
rados listomapvals -p cephfs_metadata inode_num #查看inode元数据
rados ls -p cephfs_data | grep inode_num # 查找文件内容保存的对象
ceph daemon /var/run/ceph/ceph-mds.node1.asok flush journal #强制应用日志到osd元数据池inode对象
```

### 源码路径：

```shell
/src/
	include/    cephfs API
	messages/   网络消息结构体
	mds/        mds服务端
	client/     cephfs客户端
	msg/        网络框架
	common/     ceph公用工具
```

### mds观察命令

```shell
ceph daemon mds.$(node) config set debug_mds 20/5 #mds debug日志 文件20级别，内存日志5级别
ll /var/log/ceph #日志默认路径
ceph daemon mds.$(node) dump_blocked_ops -f json #查看卡住的op
ls -l -i
ceph daemon mds.$(node) session ls #查看当前会话
ceph daemon mds.$(node) config show #查看配置
ceph daemon mds.$(node) config set #修改配置
ceph daemon mds.$(node) dump inode 111 #查看mds对指定inode的状态
cat /var/log/ceph/ceph-mds.$(node)...log #mds日志
ceph osd blocklist ls #黑名单列表
```

### 客户端打开日志

```ini
[global]
debug client = 20
[client]
log file = C:/ProgramData/ceph/aaa.log
```

### 相关参数：

mds_session_autoclose：多少秒没响应算作超时并剔除，已废弃，默认300秒

mds_reconnect_timeout：

mds_session_blocklist_on_evict：手工驱逐时加不加黑名单

mds_session_blocklist_on_timeout：超时无响应时加不加黑名单

client_reconnect_stale：客户端配置项，断链是否重连

mds_cap_revoke_eviction_timeout：回收caps多少秒没回就剔除该session，默认永远不踢

mds_reconnect_timeout

ms_dispatch_throttle_bytes：网络层流量限制

### 恢复mds

```shell
#mds启动不了，恢复mds
systemctl stop ceph-mds@pve04.service #停掉mds 
cephfs-journal-tool --rank=cephfs:0 event recover_dentries summary #从日志中恢复元数据
cephfs-journal-tool --rank=cephfs:0 journal reset  #把末尾那段损坏的日志丢掉
cephfs-table-tool all reset session #重启
systemctl start ceph-mds@pve04.service #重启 
ceph mds repaired 0 #恢复
```

![飞书20211228-105645](./img/飞书20211228-105645.png)

