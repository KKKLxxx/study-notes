# RIP、OSPF、BGP

三者均为路由选择协议，其中

RIP、OSPF为**内部网关协议**，用于一个自治系统内的路由器之间互相通信

BGP为**外部网关协议**，用于不同自治系统之间的通信

## 一、RIP(Open Shortest Path First，路由信息协议)

### 1、缺陷 

1、以跳数评估路由，可能并非最佳路径

2、最大跳数为 15，有网络规模限制（用 16 表示无穷、不可达）

3、每次要发送自己全部的路由信息，浪费网络资源（但也有水平分割可以进行优化）

4、只能定期（30s）发送路由信息，有产生环路、信息更新不及时的风险

## 二、OSPF(Open Shortest Path First，开放式最短路径优先)

### 1、工作流程

1、发现邻居

2、广播型网络会选举 DR 和 BDR（用于减少信息交换次数），但在 PPP 网络中不会选举

3、向邻居或 DR 和 BDR 传递 LSA（Link State Advertisement，链路状态公告）摘要，邻居反馈自己需要的 LSA 信息，再根据需要传递完整 LSA（可以减少网络资源消耗）

4、每个路由器均有一个 LSDB（Link State Database，链路状态数据库），由 LSA 组成。多次交换后，一个区域内的所有路由器 LSDB 相同，即每个路由器都知道全网状态

5、根据带宽进行路由计算

6、触发更新或定期更新，如果网络状态有变可以立即更新，如果没有变化就每 30min 更新一次

### 2、优点

1、按带宽计算最短路径，更准确

2、每次并非发送全部的路由信息，减少网络消耗

3、没有网络规模限制

## 三、BGP(Border Gateway Protocol，边界网关协议)

BGP是一种外部路由协议，与OSPF、RIP不同，其着眼点不在于**发现和计算路由**，而在于**交换路由信息**

BGP只能是力求寻找一条能够到达目的网络且比较好的路由（不能兜圈子），而并非要寻找一条最佳路由

## 四、对比

| 协议         | RIP  | OSPF | BGP  |
| ------------ | ---- | ---- | ---- |
| **类型**     | 内部 | 内部 | 内部 |
| **传递协议** | UDP  | IP   | TCP  |

