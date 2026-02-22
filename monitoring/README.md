## 监控子系统说明（Prometheus + Grafana）

`hcp-deploy/monitoring` 目录用于部署整套 HCP 系统的监控组件，主要包含两个模块：

### 1. Prometheus（指标采集与存储）
- 作为「时序指标数据库」，负责从各个服务周期性拉取指标并保存：
  - 共识节点（hcp-consensus）：共识状态、区块高度、TPS 等
  - 网关（hcp-gateway）：HTTP / gRPC 请求量、延迟、错误率等
  - 后端服务（hcp-server）：业务接口 QPS、数据库相关指标等
- 当前配置文件：
  - `prometheus/prometheus.yml`：定义抓取目标、采集间隔等
  - `prometheus/rules/*.yml`：预警与聚合规则（如 P99 延迟过高告警）
- 主要作用：
  - 为性能分析与容量评估提供基础数据
  - 为告警系统提供数据来源

### 2. Grafana（可视化与仪表盘）
- 作为「统一监控大屏」，从 Prometheus 读取数据并以图表形式展示：
  - 共识性能：TPS、区块出块时间、共识延迟
  - 节点健康：CPU / 内存 / 网络等资源使用情况
  - 应用层：Gateway / Server 接口的响应时间、错误率
- 当前配置文件：
  - `grafana/provisioning/datasources/datasource.yml`：内置 Prometheus 数据源
  - `grafana/provisioning/dashboards/dashboard.yml`：自动加载仪表盘配置
  - `grafana/dashboards/*.json`：具体仪表盘定义
- 主要作用：
  - 支持答辩演示时展示实时性能
  - 帮助定位性能瓶颈与异常节点

### 3. 与主系统的集成方式
- 监控模块通过 Docker Compose 启动（见 `docker-compose.yml`），在独立的 `hcp-deploy_hcp-net` 网络中运行：
  - Prometheus：监听 `localhost:9090`
  - Grafana：监听 `localhost:3001`
- 容器内部通过 `host.docker.internal` 访问主机上运行的各个服务端口，从而采集本地启动的 HCP 系统指标。

### 4. 启动方式（两种）
1. 单独启动监控系统：
   - 在仓库根目录执行：
     - `bash hcp/start_monitoring.sh`
   - 适用于只想看监控、不必同时启动整套 HCP 系统的场景。

2. 随整套系统一起启动：
   - 在仓库根目录执行：
     - `bash hcp/start_hcp.sh`
   - 脚本会依次启动：
     - 共识节点（hcp-consensus）
     - 后端服务（hcp-server）
     - 网关（hcp-gateway）
     - 前端（hcp-ui）
     - **监控系统（Prometheus + Grafana，本目录）**

### 5. 访问入口汇总
- Prometheus UI：`http://localhost:9090`
- Grafana UI：`http://localhost:3001`

建议在本目录下只放与监控相关的配置（Prometheus / Grafana / 监控规则），其他服务的部署脚本统一放在上层 `hcp` 或 `hcp-deploy` 中，方便维护整体架构。

