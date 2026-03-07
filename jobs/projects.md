一、dot1x/mac认证

1.802.1x协议介绍

802.1x协议是一种基于端口的网络接入控制协议，即在局域网接入设备的端口上对所接入的用户和设备进行认证，以便控制用户设备对网络资源的访问。

802.1x系统中包含三个实体：客户端（client）、设备端（device）和认证服务器（authentication server），如下图所示：

802.1x认证流程（中继方式）

802.1x认证触发方式
    客户端主动触发：组播触发/广播触发
    设备端主动触发：组播触发/单播触发

hostapd现状
    命令行启动，指定监听端口、后台运行、日志目录、全局socket路径、配置文件
    dot1x认证支持情况：
        不支持通过配置控制业务启停
        不支持arp报文触发认证
        不支持客户端带vlan报文的认证报文解析
        不支持数据恢复，进程重启数据丢失
        hostapd虽然支持802.1x认证，但也只是停留在协议层，下内核等操作并不支持，也就是完整的认证并可转发报文的功能并不支持

dot1x拉起
    systemctl start/stop/enable/disable dot1x.service
    使能过程中，确保hostapd在dot1xsyncd启动之后启动，通过容器的supervisor.conf控制

接口使能
    ![alt text](image.png)

接口去使能

用户上线
    hostapd将fdb信息推给dot1xsyncd，dot1xsyncd下发fdb(APPLDB)，同时创建acl rule(APPLDB)用于统计流量
    ![alt text](image-1.png)

用户下线
    hostapd下线用户，将fdb信息推给dot1xsyncd，dot1xsyncd删除fdb，同时删除acl rule

统计信息获取
    hostapd通过dot1xsyncd获取用户流量信息，dot1x负责从COUNTDB读取统计信息数据返回给hostapd

在线用户查询：
    cli从APPDB和COUNTERDB中读取数据

支持带vlan用户上线
    修改创建socket时监听所有二层协议
    修改recvmsg函数
    接口pvid变化

hostapd和dot1xsyncd交互
    用户fdb表项下发，数据恢复
    计费查询，轮询获取流量信息

通过netlink响应接口事件
    hostapd初始化时要监听netlink_route事件，当接口down时，会上报netlink事件

ACL修改
    增加acl表类型
    acl配置


二、ISIS项目

SPF算法
    核心逻辑：贪心策略
    核心思想：每次都选择当前已知的、离起点最近的节点，并以此位跳板去更新其他节点的距离

2. 数学表达在算法运行过程中，对于每一条边 $(u, v)$，如果通过节点 $u$ 到达 $v$ 的路径比当前记录的 $v$ 的路径更短，我们就进行更新。其判别式为：$$dist(v) > dist(u) + weight(u, v)$$如果上述条件成立，则更新：$$dist(v) = dist(u) + weight(u, v)$$