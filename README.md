# Chained-Proxy

**双重伪装的双层代理架构**：中转机（SNI 路由）+ 落地机（TLS 终端）

## 核心策略：欺上瞒下

- **欺上（对 GFW）**：中转机伪装成普通 CDN，使用 Apple CDN fallback + 流量特征优化
- **瞒下（对目标网站）**：落地机伪装成真实美国用户，使用 uTLS 指纹 + DoH + Nginx fallback

## 特性亮点

### 🎭 隐蔽性（Stealth）
- **uTLS 指纹伪装**：模拟 Chrome 浏览器的 TLS 握手特征
- **Apple CDN Fallback**：错误/空 SNI 路由到 Apple CDN IP 池（4个IP轮换）
- **Nginx Fallback**：落地机返回真实网站响应（403/robots.txt/favicon.ico）
- **流量特征优化**：proxy_timeout 120s，模拟正常 CDN 行为
- **DoH 加密 DNS**：使用 Cloudflare 1.1.1.1 和 Google 8.8.8.8

### 🔒 安全性（Security）
- **Cloudflare Token 加密存储**：AES-256-CBC 加密，使用 machine-id 作为密钥
- **证书私钥权限加强**：600 权限，仅 root 可读
- **防火墙白名单**：落地机仅允许中转机 IP 访问
- **DDoS 防护**：connlimit 800 + hashlimit 2000/sec
- **进程隔离**：NoNewPrivileges + ProtectSystem=strict

### 🛡️ 稳定性（Reliability）
- **证书监控增强**：30天到期告警 + 自动重试续期
- **完善回滚机制**：包含 IPv6 防火墙规则清理
- **端口冲突检测**：安装前集中检测所有端口
- **原子化写入**：所有配置通过 mktemp → mv 写入
- **幂等安装**：重复运行不产生重复规则

### ⚡ 性能（Performance）
- **BBR 拥塞控制**：自动启用 TCP BBR
- **TCP Fast Open**：减少握手延迟
- **连接优化**：支持 200 并发连接/IP
- **Keepalive 优化**：proxy_socket_keepalive + tcp_nodelay

## 架构

```
Client (TLS SNI)
      │
      ▼
┌─────────────────────────────────┐
│  中转机 Transit                  │
│  Nginx stream ssl_preread        │
│  读取 TLS ClientHello SNI 字段   │
│  → 路由到对应落地机 IP:443        │
│  不解密、不终止 TLS               │
└───────────────┬─────────────────┘
                │ 原始 TCP
                ▼
┌─────────────────────────────────┐
│  落地机 Landing                  │
│  Xray-core Trojan-TLS           │
│  Let's Encrypt ECDSA 证书        │
│  终结 TLS，接管真实流量           │
└─────────────────────────────────┘
```

## 安装

### 中转机（一台）

```bash
curl -sL https://raw.githubusercontent.com/vpn3288/Chained-Proxy/refs/heads/main/install_transit.sh | sudo bash
```

### 落地机（多台）

```bash
curl -sL https://raw.githubusercontent.com/vpn3288/Chained-Proxy/refs/heads/main/install_landing.sh | sudo bash -s -- --cloudflare <CF_TOKEN>
```

## 中转机管理

```bash
# 添加落地机节点（交互式）
sudo bash install_transit.sh

# 导入节点（从落地机安装输出复制）
sudo bash install_transit.sh --import

# 卸载
sudo bash install_transit.sh --uninstall
```

## 落地机管理

```bash
# 交互式安装
sudo bash install_landing.sh

# 添加节点
sudo bash install_landing.sh --add-node

# 变更端口
sudo bash install_landing.sh --set-port

# 卸载
sudo bash install_landing.sh --uninstall
```

## 端口与共存

| 组件 | 端口 | 说明 |
|------|------|------|
| Transit Nginx | 443 | SNI 路由入口 |
| Landing Xray | 8443 | Trojan TLS 监听 |
| SSH | 自动探测 | 保持原端口 |

与 mack-a/v2ray-agent 完全物理隔离，不共享文件、端口、进程。

## 文件布局

```
/etc/transit_manager/          # 中转机配置
  conf/*.meta                   # 节点元数据
  snippets/landing_*.map        # Nginx SNI 路由表
  nodes/*.conf                  # 节点连接信息
  tmp/                          # 原子写入临时目录

/etc/xray-landing/              # 落地机配置
  config.json                   # Xray 运行配置
  certs/<domain>/              # Let's Encrypt 证书
  nodes/*.conf                  # 节点信息

/var/log/
  xray-landing-access.log      # 访问日志
  xray-landing-error.log       # 错误日志
```

## 安全特性

- **Xray 隔离**: `NoNewPrivileges=true`, `ProtectSystem=strict`, `ProtectHome=true`
- **证书权限**: 私钥 `640 root:xray-landing`，acme.sh reload 失败时拒绝启动
- **防火墙**: iptables 自定义链，仅允许已知 Transit IP 连接落地机
- **原子化写入**: 所有配置通过 `mktemp → chmod → chown → mv` 写入
- **幂等安装**: 重复运行不产生重复规则或配置块

## 前置要求

- Debian 11+ / Ubuntu 20.04+
- root 权限
- 落地机需要可解析的域名 + Cloudflare API Token（DNS-01 验证）
- 中转机需要 443 端口未被占用

## 版本

**当前版本：v3.63**

### v3.63 更新内容（2025-04-21）

#### 隐蔽性增强
- ✅ 添加 uTLS Chrome 指纹伪装（落地机）
- ✅ Apple CDN IP 轮换池（4个IP随机选择）
- ✅ Nginx Fallback 真实网站模拟（403/robots.txt/favicon.ico）
- ✅ 流量特征优化（proxy_timeout 120s）

#### 安全性增强
- ✅ Cloudflare Token AES-256-CBC 加密存储
- ✅ 证书私钥权限加强（600）
- ✅ DDoS 防护加强（connlimit 800, hashlimit 2000/sec）

#### 稳定性增强
- ✅ 证书监控增强（30天告警 + 自动重试）
- ✅ 完善回滚机制（IPv6 防火墙规则清理）
- ✅ 端口冲突检测增强（集中检测）

#### 性能优化
- ✅ 连接限制提高（200 per IP）
- ✅ 速率限制优化（2000/sec）

详细变更记录请查看 [CHANGELOG.md](CHANGELOG.md)

## 故障排除

### 中转机

**问题：Nginx 启动失败**
```bash
# 检查端口占用
sudo netstat -tlnp | grep :443

# 检查配置语法
sudo nginx -t

# 查看错误日志
sudo journalctl -u nginx -n 50
```

**问题：无法连接到落地机**
```bash
# 检查防火墙规则
sudo iptables -L TRANSIT_STREAM_OUT -n -v

# 测试连接
telnet <落地机IP> 8443
```

### 落地机

**问题：证书申请失败**
```bash
# 检查 Cloudflare Token
sudo /root/.acme.sh/acme.sh --list

# 手动申请证书
sudo /root/.acme.sh/acme.sh --issue --dns dns_cf -d <域名> --keylength ec-256

# 查看 acme.sh 日志
sudo cat /root/.acme.sh/acme.sh.log
```

**问题：Xray 启动失败**
```bash
# 检查配置语法
sudo /usr/local/bin/xray-landing run -test -config /etc/xray-landing/config.json

# 查看错误日志
sudo journalctl -u xray-landing -n 50

# 检查证书权限
sudo ls -la /etc/xray-landing/certs/*/
```

**问题：防火墙规则导致 SSH 锁死**
```bash
# 通过 VNC/控制台登录，清理防火墙规则
sudo iptables -F XRAY_LANDING_IN
sudo ip6tables -F XRAY_LANDING_IN
sudo iptables -D INPUT -j XRAY_LANDING_IN
sudo ip6tables -D INPUT -j XRAY_LANDING_IN
```

## 监控建议

### 证书到期监控
```bash
# 检查所有证书到期时间
for cert in /etc/xray-landing/certs/*/cert.pem; do
  echo "$cert:"
  openssl x509 -enddate -noout -in "$cert"
done
```

### 服务状态监控
```bash
# 中转机
sudo systemctl status nginx
sudo netstat -tlnp | grep :443

# 落地机
sudo systemctl status xray-landing
sudo netstat -tlnp | grep :8443
```

### 日志监控
```bash
# 中转机（仅记录 emerg 级别错误）
sudo tail -f /var/log/transit_manager/transit_stream_error.log

# 落地机
sudo tail -f /var/log/xray-landing-error.log
```

## 安全建议

1. **定期更新系统**：`sudo apt update && sudo apt upgrade`
2. **监控证书到期**：建议设置 cron 任务每周检查
3. **备份配置文件**：定期备份 `/etc/transit_manager/` 和 `/etc/xray-landing/`
4. **限制 SSH 访问**：使用密钥认证，禁用密码登录
5. **监控异常流量**：定期检查 Xray 日志，发现异常连接

## 许可证

MIT License

## 贡献

欢迎提交 Issue 和 Pull Request！

## 相关文档

- [CHANGELOG.md](CHANGELOG.md) - 详细变更记录
- [PROTECTION_REQUIREMENTS.md](PROTECTION_REQUIREMENTS.md) - 核心需求文档
