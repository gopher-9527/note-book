# calico
## calico是什么
Calico 是一款网络及安全解决方案，可帮助 Kubernetes 工作负载与非 Kubernetes/传统工作负载实现无缝且安全的通信。
Calico由用于保护网路通信的网络解决方案和用于保护大规模的云原生微服务/应用的高级网络策略组成。
| 组件                          | 描述                                                                                                                                                          | 主要特性                                                                                                                                                                                                                                                                                                  |
|-------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Calico CNI（网络层）**       | Calico CNI 是一个可编程多个数据平面的控制平面，作为 L3/L4 网络解决方案，用于保护容器、Kubernetes 集群、虚拟机及基于主机的原生工作负载。                                                                                                             | • 内置数据加密<br>• 高级 IPAM 管理<br>• 支持覆盖/非覆盖网络模式<br>• 多数据平面选择：iptables、eBPF、Windows HNS 或 VPP                                                                                                                                                                                                                                                          |
| **Calico 网络策略套件**        | Calico 网络策略套件是 Calico CNI 的规则执行接口，包含数据平面的策略规则。<br><br>核心特性：<br>• 基于零信任安全模型（默认拒绝所有流量，仅允许必要通信）<br>• 与 Kubernetes API 服务器集成（兼容原生 Kubernetes 网络策略）<br>• 支持传统系统（裸机/非集群主机）使用统一策略模型 | • **命名空间 & 全局策略**：控制集群内流量、Pod 与外部通信、非集群主机的访问<br>• **网络集 (Network Sets)**：通过 IP 子网/CIDR/域名集合，限制工作负载的入口/出口流量范围<br>• **应用层策略 (L7)**：基于 HTTP 方法/路径等属性实施流量控制，支持加密安全身份验证                                                                                                                                  |

## kubernetes网络模型
**Kubernetes 网络模型定义如下**：
- **独立 IP 分配**  
  每个 Pod 拥有独立的 IP 地址
  
- **容器间直连通信**  
  同一 Pod 内的容器共享该 IP 地址，可直接相互通信

- **集群无 NAT 通信**  
  Pod 可通过 IP 地址直接与集群内其他 Pod 通信（无需网络地址转换 NAT）

- **策略驱动隔离**  
  通过[网络策略（Network Policies）](https://kubernetes.io/docs/concepts/services-networking/network-policies/)定义通信隔离规则，控制 Pod 间的访问权限
  
## Kubernetes 网络核心原则与 Calico 解决方案总结

#### 一、Kubernetes 基础网络模型
1. **扁平化 IP 架构**  
   - 每个 Pod 分配独立 IP 地址（IPv4/IPv6）
   - 容器共享 Pod IP（通过 localhost 通信）
   - 跨节点 Pod 直连通信（无需 NAT）

2. **默认全通策略**  
   - 集群内 Pod 默认可自由互访
   - 跨命名空间无隔离限制

3. **策略驱动隔离**  
   - 通过 [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) 实现细粒度控制
   - 支持基于标签的流量过滤（命名空间/Pod/端口等）

#### 二、Kubernetes 网络实现挑战
| 挑战维度       | 具体场景                                                                 |
|----------------|------------------------------------------------------------------------|
| **网络平面**    | Overlay/Underlay 网络选择与性能损耗平衡                                  |
| **策略扩展性**  | 原生 NetworkPolicy 对 L7 策略支持有限                                    |
| **混合环境**    | 虚拟机/裸机与传统容器网络互通需求                                         |
| **安全合规**    | 零信任架构下的微分段（Micro-segmentation）实施                           |

#### 三、Calico 网络解决方案
| 能力维度          | 技术实现                                                                                     | 核心价值                                                                 |
|-------------------|------------------------------------------------------------------------------------------|------------------------------------------------------------------------|
| **多数据平面**   | 支持 iptables/eBPF/Windows HNS/VPP 等多种引擎                                               | 适应不同环境性能需求（如 eBPF 实现高性能网络）                             |
| **高级网络策略**   | • L3-L7 策略控制（HTTP方法/路径）<br>• 加密身份认证<br>• 全局/命名空间级策略<br>• 网络集(IP/CIDR/域名) | 实现零信任安全架构                                                      |
| **混合云网络**     | 统一管理 Kubernetes Pod 与 VM/裸机工作负载                                                   | 消除传统-云原生网络边界                                                 |
| **可观测性**       | 动态流量可视化 + 实时策略模拟                                                              | 简化网络排错与合规审计                                                  |
