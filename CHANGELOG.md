# 变更记录

## 2026-03-27

### 初始化项目

- [x] 确定需求:OpenClash + Docker + udpxy + igmpproxy + 旁路由
- [x] 确定源码选择:DHDAXCW/OpenWRT_x86_x64(基于 ImmortalWrt openwrt-24.10)
- [x] 确定硬件平台:i9-9900T 虚拟机,旁路由模式
- [x] 创建 GitHub 仓库 fork(lichuncat/OpenWRT_x86_x64)
- [x] 配置 PAT 并通过 API 操作 GitHub
- [x] 修改 defconfig:加 udpxy + igmpproxy
- [x] 修改 diy-part2.sh:默认 IP 改为 192.168.1.252
- [x] 触发第一次 GitHub Actions 编译(immortalwrt_x86_64_docker workflow)
  - Run ID: https://github.com/lichuncat/OpenWRT_x86_x64/actions/runs/23643329470
  - 结果:失败,6 小时超时被系统取消

## 2026-03-28

### 编译策略调整

- [x] 登录现网 OpenWrt(192.168.1.252)做只读盘点,对比当前真实使用功能
- [x] 确认保留方向:OpenClash、Docker、UPnP、udpxy、igmpproxy、vlmcsd、OAF、Diskman、SQM
- [x] 精简 `immortalwrt/x86_64/defconfig`
  - 删除大量重复代理栈:Passwall / Passwall2 / SSR Plus / HomeProxy / Nikki / SmartDNS / AdGuardHome
  - 删除大量无关大包:4G/5G modem、Wi-Fi 驱动、音频/AirPlay、Samba/NAS/下载/远控等
  - 保留 OpenClash 所需 iptables/tproxy/ipset/tun 相关底层组件
  - 明确补回 Docker 管理栈和 OAF
- [x] 提交并触发第二次 GitHub Actions 编译
  - Commit: `c47b213`
  - Run ID: https://github.com/lichuncat/OpenWRT_x86_x64/actions/runs/23671263252

### 第二次编译结果

- [x] 失败原因已定位:不是超时,而是 `mosdns/v2dat` 编译失败
- [x] 具体报错:`go.mod requires go >= 1.25.0`,而当前构建环境 Go 版本为 `1.23.12`
- [x] 结论:这是依赖/工具链版本冲突,和是否改为本地/self-hosted 编译无关

### 当前临时处理

- [x] 为了先拿到可刷固件,暂时从 `defconfig` 中移除 MosDNS 相关项
- [x] 提交并触发第三次 GitHub Actions 编译
  - Commit: `c49e43d`
  - Run ID: https://github.com/lichuncat/OpenWRT_x86_x64/actions/runs/23677759601
  - 结果:成功(139 MB)

## 2026-03-28 (第二次编译)

### 第四次编译 - 完整配置

- [x] 修改 defconfig:
  - 移除:aria2 全家桶、SQM 全家桶
  - 保留:udpxy + LuCI、igmpproxy、oaf + LuCI、OpenClash、Docker
  - 添加:MosDNS v5 + v2dat + LuCI
- [x] 修改 workflow 添加 Go 1.24.x 支持(解决 MosDNS v2dat 编译问题)
- [x] 提交并触发第四次 GitHub Actions 编译
  - Commit: `f526f53`
  - Run ID: https://github.com/lichuncat/OpenWRT_x86_x64/actions/runs/23683628828
  - 当前状态:进行中(预计 6 小时)

### 第五次编译 - 移除 MosDNS

- [x] 定位第四次编译失败原因：MosDNS/v2dat 编译依赖问题
- [x] 临时移除 MosDNS 相关配置（`CONFIG_PACKAGE_mosdns`、`v2dat`、`luci-app-mosdns`）
- [x] 提交 commit `77d93ea`
- [x] 触发第五次 GitHub Actions 编译
  - Run ID: https://github.com/lichuncat/OpenWRT_x86_x64/actions/runs/23695086883
  - 当前状态：进行中（预计 4-5 小时）

### 后续决策点

- [ ] 如果第五次编译成功：下载固件并刷入虚拟机验证
- [ ] 固件稳定后，再单独处理 MosDNS：
  - [ ] 方案 A：后装 MosDNS（推荐）
  - [ ] 方案 B：查找兼容当前 toolchain 的 MosDNS/v2dat 版本并重新编进固件

### 待完成

- [ ] 编译完成，下载固件
- [ ] 固件刷入虚拟机
- [ ] 配置旁路由 LAN IP 和 DHCP
- [ ] 配置 OpenClash 网关模式
- [ ] 配置 IGMP Proxy（如需 IPTV）
- [ ] Docker 部署 AirPrint
- [ ] 测试 AirPrint 打印机
- [ ] 验证 DNS 防泄露
