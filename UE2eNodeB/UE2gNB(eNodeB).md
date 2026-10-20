# 终端 ↔ 星上处理信号 完整通信流程

> 
> 和透明转发对比：**基站会解调、解码、调度资源**，不是单纯中继。终端 = UE，基站 = eNodeB/gNB

## 整体阶段

1. 小区搜索 & 下行同步
2. 系统信息读取
3. 随机接入（RACH）
4. RRC 连接建立
5. 鉴权、安全激活
6. EPS/5G 会话建立（PDN/PDU 会话）
7. 用户面业务传输
8. 链路维护（测量、切换）
9. 连接释放

---

## 分步详解

### 1. 小区搜索 + 下行同步（开机第一件事）

UE 开机扫描频段，找小区：

- 捕获 **PSS 主同步信号、SSS 辅同步信号**：获取帧定时、小区 ID
- 读取 PBCH 广播信道：MIB 主信息块，得到系统带宽、子帧配置

### 2. 读取系统消息 SIB

从 PDSCH 读取 SIB1、SIB2 等：小区接入参数、邻区、随机接入参数、频段信息。

> 
> UE 判断这个小区是否允许接入。

### 3. 随机接入 RACH（上行第一步，非常关键）

UE 要发数据，但上行还没同步，用 RACH 申请上行资源：

1. UE 在 PRACH 信道发送**前导码 preamble**
2. 基站检测 preamble，计算 UE 的**上行时间提前量 TA**，分配上行资源
3. 基站下发 Msg2（RAR 随机接入响应）携带 TA + UL 授权
4. UE 用 TA 对齐上行时序，发送 Msg3（RRC 连接请求）
5. 基站回复 Msg4，冲突解决，RACH 完成

> 
> 作用：上行时间同步 + 请求信道资源。

### 4. RRC 连接建立

UE → gNB：RRC Connection Request
gNB → UE：RRC Connection Setup
UE → gNB：RRC Connection Setup Complete
👉 控制面链路建立完成。

### 5. 鉴权与安全

核心网参与：

- 鉴权：验证 SIM 卡身份，防止伪终端
- 安全模式激活：加密、完整性保护，后续信令加密传输

### 6. 会话建立（PDU 会话，建立用户面通道）

UE 发起 PDU 会话建立请求 → 核心网分配 IP、QoS
基站建立 DRB 数据无线承载，**用户面通道打通**。

> 
> 到此终端拿到 IP，可以上网。

### 7. 用户面业务传输（数据收发）

- 上行：UE IP 包 → MAC 复用 → 编码调制 → RF 发射 → 基站接收解调 → 转发到核心网
- 下行：核心网数据到基站 → 基站调度、编码调制 → 下发 UE → UE 解调解码得到 IP 包
基站做**资源调度**：给不同 UE 分配时隙 / 时频资源（和卫星透明转发最大区别）

### 8. 移动性管理

UE 持续测量本小区 + 邻小区 RSRP/SINR，上报测量报告；基站判决，触发切换（小区切换）。

### 9. 连接释放

业务结束 / 长时间无数据，基站释放 RRC 连接，进入空闲态，节省功耗。

# ✅【地面蜂窝 vs 卫星通信核心差异】

表格

| 项目 | 地面蜂窝（终端 - 基站） | 透明转发卫星通信（终端 - 卫星） |
| --- | --- | --- |
| 基站 / 卫星角色 | 基站解调、解码、资源调度 | 卫星只放大变频，不解码，纯弯管中继 |
| 同步 | RACH 做上行 TA | 终端天线搜星、载波同步 |
| 距离 | 几百米～几公里 | 几百～数万公里，巨大自由空间损耗 |
| 调度 | 基站本地实时调度 | 调度在地面网关，卫星不参与调度 |
| 移动性 | 小区切换 | 波束切换 / 星间切换（低轨） |

# 通信术语缩写对照表

表格

| 缩写 | 英文全称 | 中文释义 |
| --- | --- | --- |
| UE | User Equipment | 用户终端 |
| eNodeB | evolved NodeB | 4G 基站 |
| gNB | next generation NodeB | 5G 基站 |
| PSS | Primary Synchronization Signal | 主同步信号 |
| SSS | Secondary Synchronization Signal | 辅同步信号 |
| PBCH | Physical Broadcast Channel | 物理广播信道 |
| MIB | Master Information Block | 主信息块 |
| SIB | System Information Block | 系统信息块 |
| SIB1 | System Information Block Type1 | 系统信息块 1 |
| SIB2 | System Information Block Type2 | 系统信息块 2 |
| RACH | Random Access Channel | 随机接入信道 |
| PRACH | Physical Random Access Channel | 物理随机接入信道 |
| preamble | preamble | 随机接入前导码 |
| RAR | Random Access Response | 随机接入响应（Msg2） |
| TA | Timing Advance | 时间提前量 |
| Msg2 | Message2 | 随机接入消息 2 |
| Msg3 | Message3 | 随机接入消息 3 |
| Msg4 | Message4 | 随机接入消息 4 |
| RRC | Radio Resource Control | 无线资源控制 |
| EPS | Evolved Packet System | 4G 演进分组系统 |
| PDN | Packet Data Network | 分组数据网（4G） |
| PDU | Protocol Data Unit | 协议数据单元 |
| DRB | Data Radio Bearer | 数据无线承载 |
| MAC | Medium Access Control | 媒体接入控制层 |
| PDSCH | Physical Downlink Shared Channel | 物理下行共享信道 |
| RSRP | Reference Signal Received Power | 参考信号接收功率 |
| SINR | Signal to Interference plus Noise Ratio | 信干噪比 |
| VSAT | Very Small Aperture Terminal | 甚小口径卫星终端 |
| Gateway | Gateway | 信关站 / 网关 |
| FDD | Frequency Division Duplex | 频分双工 |
| HTS | High Throughput Satellite | 高通量卫星 |
| OBP | On-Board Processing | 星上处理 |
| RF | Radio Frequency | 射频 |
