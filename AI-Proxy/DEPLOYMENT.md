# 完整部署指南

本指南将带你从零开始，完成整个代理系统的部署。

## 准备工作

### 1. 注册云服务商

推荐选择：
- **落地机**：Oracle Cloud（甲骨文云）- 免费套餐，1核6GB内存
- **中转机**：腾讯云/阿里云 - CN2 GIA线路

### 2. 创建VPS实例

#### Oracle Cloud（落地机）

1. 访问 https://cloud.oracle.com
2. 注册账号（需要信用卡验证）
3. 创建实例：
   - 镜像：Ubuntu 22.04
   - 配置：VM.Standard.A1.Flex（1核6GB）
   - 网络：创建新的VCN
   - SSH密钥：上传你的公钥或生成新密钥
4. 记录实例的公网IP

#### 腾讯云/阿里云（中转机）

1. 选择CN2 GIA线路的VPS
2. 配置：1核1GB即可
3. 系统：Ubuntu 22.04
4. 记录实例的公网IP

## 部署流程

### 第一步：部署落地机

1. SSH连接到落地机：
```bash
ssh ubuntu@落地机IP
```

2. 运行安装脚本：
```bash
bash <(curl -sL https://raw.githubusercontent.com/vpn3288/AI-Proxy/main/install_landing.sh)
```

3. 按提示输入信息：
   - Trojan密码：至少16位，建议使用密码生成器
   - Cloudflare Token：从CF获取（可选）
   - 是否安装1Panel：选择"是"
   - 是否安装Tailscale：选择"是"

4. 等待安装完成（约5-10分钟）

5. 记录以下信息：
   ```
   落地机IP: xxx.xxx.xxx.xxx
   端口: 8443
   UUID: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
   Tailscale IP: 100.x.x.x
   ```

### 第二步：配置Tailscale

1. 在落地机上，Tailscale已自动安装
2. 按提示完成授权：
   ```bash
   tailscale up
   ```
3. 浏览器会打开授权页面
4. 登录Tailscale账号并授权设备
5. 记录Tailscale IP（100.x.x.x）

### 第三步：访问1Panel

1. 在你的电脑上安装Tailscale：
   - Windows/Mac: https://tailscale.com/download
   - 登录同一个Tailscale账号

2. 查看1Panel访问信息：
   ```bash
   1pctl user-info
   ```

3. 在浏览器访问：
   ```
   http://100.x.x.x:端口/安全入口
   ```

4. 使用显示的用户名和密码登录

### 第四步：部署中转机

1. SSH连接到中转机：
```bash
ssh root@中转机IP
```

2. 运行安装脚本：
```bash
bash <(curl -sL https://raw.githubusercontent.com/vpn3288/AI-Proxy/main/install_transit.sh)
```

3. 按提示输入落地机信息：
   - 落地机IP: xxx.xxx.xxx.xxx
   - 落地机端口: 8443
   - 落地机UUID: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

4. 等待安装完成（约3-5分钟）

5. 记录中转机的连接信息

### 第五步：配置客户端

1. 下载客户端：
   - Windows: v2rayN
   - Mac: V2rayU
   - iOS: Shadowrocket
   - Android: v2rayNG

2. 导入配置：
   - 扫描二维码
   - 或手动输入连接信息

3. 测试连接

## 可选：部署OpenClaw

### 通过1Panel安装

1. 登录1Panel管理面板
2. 进入"应用商店"
3. 搜索"OpenClaw"
4. 点击"安装"
5. 配置参数：
   - Telegram Bot Token: 从 @BotFather 获取
   - 其他参数保持默认
6. 点击"确认"开始安装
7. 等待容器启动（约1-2分钟）

### 配置Telegram Bot

1. 在Telegram中搜索 @BotFather
2. 发送 `/newbot` 创建新机器人
3. 按提示设置机器人名称和用户名
4. 获取Bot Token（格式：123456789:ABCdefGHIjklMNOpqrsTUVwxyz）
5. 将Token填入1Panel的OpenClaw配置中

### 连接OpenClaw

1. 在Telegram中搜索你的机器人
2. 发送 `/start` 开始对话
3. 现在可以通过Telegram管理你的VPS了！

## 故障排查

### 落地机无法连接

1. 检查防火墙规则：
```bash
sudo ufw status
```

2. 检查Xray服务状态：
```bash
systemctl status xray
```

3. 查看日志：
```bash
journalctl -u xray -n 50
```

### 中转机无法连接落地机

1. 测试网络连通性：
```bash
ping 落地机IP
telnet 落地机IP 8443
```

2. 检查Nginx配置：
```bash
nginx -t
systemctl status nginx
```

### 1Panel无法访问

1. 确认Tailscale已连接：
```bash
tailscale status
```

2. 检查1Panel服务：
```bash
systemctl status 1panel
```

3. 查看1Panel日志：
```bash
1pctl logs
```

### OpenClaw无法启动

1. 检查Docker容器状态：
```bash
docker ps -a | grep openclaw
```

2. 查看容器日志：
```bash
docker logs openclaw
```

3. 确认Bot Token正确：
   - Token格式：数字:字母数字组合
   - 没有多余的空格或换行

## 安全加固

### 1. 修改SSH端口

```bash
sudo nano /etc/ssh/sshd_config
# 修改 Port 22 为其他端口（如 2222）
sudo systemctl restart sshd
```

### 2. 禁用密码登录

```bash
sudo nano /etc/ssh/sshd_config
# 设置 PasswordAuthentication no
sudo systemctl restart sshd
```

### 3. 配置防火墙

```bash
# 只开放必要端口
sudo ufw allow 2222/tcp  # SSH
sudo ufw allow 443/tcp   # 中转机
sudo ufw allow 8443/tcp  # 落地机（如果需要）
sudo ufw enable
```

### 4. 定期更新

```bash
sudo apt update && sudo apt upgrade -y
```

## 维护建议

1. **每周检查**：
   - 服务运行状态
   - 磁盘空间使用
   - 系统日志

2. **每月更新**：
   - 系统软件包
   - Xray核心
   - 1Panel版本

3. **备份重要数据**：
   - Xray配置文件
   - 1Panel数据
   - SSH密钥

## 成本估算

- **落地机**：Oracle Cloud免费套餐 - $0/月
- **中转机**：腾讯云CN2 GIA - ¥30-50/月
- **Tailscale**：免费版（最多20台设备）- $0/月
- **总计**：约 ¥30-50/月

## 下一步

- 配置多个落地机实现负载均衡
- 使用CDN加速中转机
- 部署监控系统（Prometheus + Grafana）
- 配置自动备份

## 获取帮助

- GitHub Issues: https://github.com/vpn3288/AI-Proxy/issues
- Telegram群组: [待创建]

---

祝你部署顺利！🚀
