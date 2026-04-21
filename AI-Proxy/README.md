# AI-Proxy

一键部署 Xray 代理服务的自动化脚本，支持中转机+落地机架构。

## 特性

- 🚀 **一键安装**：全自动部署，无需手动配置
- 🔒 **安全加固**：自动配置防火墙、SSH、系统更新
- 🎯 **智能优化**：自动检测系统环境，优化配置
- 📊 **可视化管理**：可选安装 1Panel 管理面板
- 🌐 **安全访问**：集成 Tailscale 虚拟内网

## 架构说明

```
用户设备 → 中转机(国内) → 落地机(国外) → 目标网站
```

- **中转机**：部署在国内，接收用户连接，转发到落地机
- **落地机**：部署在国外，实际访问目标网站

## 快速开始

### 1. 部署落地机（国外VPS）

```bash
bash <(curl -sL https://raw.githubusercontent.com/vpn3288/AI-Proxy/main/install_landing.sh)
```

安装完成后会显示：
- 落地机IP地址
- 端口号（默认8443）
- UUID

**可选组件**：
- **Tailscale**：创建虚拟内网，安全访问管理面板
- **1Panel**：Docker管理面板，可一键安装OpenClaw等应用

### 2. 部署中转机（国内VPS）

```bash
bash <(curl -sL https://raw.githubusercontent.com/vpn3288/AI-Proxy/main/install_transit.sh)
```

安装时需要输入：
- 落地机IP地址
- 落地机端口
- 落地机UUID

## 1Panel 管理面板

### 功能特性

- 🐳 **Docker管理**：容器、镜像、网络、卷管理
- 📦 **应用商店**：一键安装OpenClaw、数据库等应用
- 📁 **文件管理**：Web界面管理服务器文件
- 📊 **系统监控**：CPU、内存、磁盘、网络实时监控
- 🖥️ **终端工具**：Web SSH终端

### 安全访问

推荐使用 Tailscale 虚拟内网访问：

1. 安装时选择安装 Tailscale
2. 按提示完成授权
3. 获取 Tailscale IP（如 100.x.x.x）
4. 通过 `http://100.x.x.x:端口/安全入口` 访问

### 查看访问信息

```bash
1pctl user-info
```

## OpenClaw 部署

通过 1Panel 应用商店一键安装：

1. 登录 1Panel 管理面板
2. 进入"应用商店"
3. 搜索"OpenClaw"
4. 点击"安装"
5. 配置 Telegram Bot Token
6. 启动容器

## 系统要求

### 最低配置
- CPU: 1核
- 内存: 512MB
- 硬盘: 10GB
- 系统: Ubuntu 20.04+ / Debian 10+

### 推荐配置（含1Panel）
- CPU: 1核
- 内存: 1GB+
- 硬盘: 20GB+
- 系统: Ubuntu 22.04 / Debian 12

## 常见问题

### 1. 如何查看服务状态？

```bash
systemctl status xray
```

### 2. 如何重启服务？

```bash
systemctl restart xray
```

### 3. 如何查看日志？

```bash
journalctl -u xray -f
```

### 4. 如何卸载？

```bash
bash <(curl -sL https://raw.githubusercontent.com/vpn3288/AI-Proxy/main/install_landing.sh) --remove
```

### 5. 1Panel 忘记密码？

```bash
1pctl reset-password
```

## 安全建议

1. ✅ 修改SSH默认端口
2. ✅ 禁用SSH密码登录，使用密钥
3. ✅ 启用防火墙，只开放必要端口
4. ✅ 定期更新系统
5. ✅ 使用Tailscale访问管理面板，不要暴露在公网

## 技术栈

- **代理核心**：Xray-core
- **传输协议**：VLESS + Reality
- **管理面板**：1Panel
- **虚拟内网**：Tailscale
- **容器平台**：Docker

## 许可证

MIT License

## 致谢

- [Xray-core](https://github.com/XTLS/Xray-core)
- [1Panel](https://github.com/1Panel-dev/1Panel)
- [Tailscale](https://tailscale.com)
- [mack-a](https://github.com/mack-a/v2ray-agent)

## 免责声明

本项目仅供学习交流使用，请遵守当地法律法规。使用本项目所产生的一切后果由使用者自行承担。
