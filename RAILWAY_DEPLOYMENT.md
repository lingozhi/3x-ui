# Railway 部署指南

## 一、前置准备

1. 注册 [Railway](https://railway.app) 账号
2. 安装 Railway CLI（可选）：`npm i -g @railway/cli`

## 二、部署步骤

### 方式一：通过 Railway Web 界面部署

1. **创建新项目**
   - 登录 Railway 控制台
   - 点击 "New Project"
   - 选择 "Deploy from GitHub repo" 或 "Empty Project"

2. **配置环境变量**（在 Railway 控制台的 Variables 标签页）

   必需的环境变量：
   ```
   XRAY_VMESS_AEAD_FORCED=false
   XUI_ENABLE_FAIL2BAN=true
   ```

   可选的环境变量：
   ```
   XUI_DB_FOLDER=/data                    # 数据库文件夹路径（默认已是 /data）
   XUI_LOG_FOLDER=/var/log                # 日志文件夹路径
   XUI_BIN_FOLDER=bin                     # 二进制文件夹路径
   XUI_LOG_LEVEL=info                     # 日志级别：debug, info, notice, warning, error
   XUI_DEBUG=false                        # 是否开启调试模式
   TZ=Asia/Shanghai                       # 时区设置（默认是 Asia/Tehran）
   ```

3. **添加 Volume（持久化存储）**
   - 在 Railway 项目设置中，点击 "Volumes"
   - 点击 "New Volume"
   - 设置挂载路径为：`/data`
   - Railway 会自动创建并管理持久化卷

4. **配置端口**
   - Railway 会自动检测并暴露 `EXPOSE 2053` 端口
   - 你可以在 "Settings" → "Networking" 中查看分配的公网 URL

5. **部署**
   - Railway 会自动检测到 `Dockerfile` 并开始构建
   - 等待构建和部署完成

### 方式二：通过 Railway CLI 部署

```bash
# 1. 登录 Railway
railway login

# 2. 初始化项目（在项目根目录执行）
railway init

# 3. 添加 Volume
railway volume add

# 4. 设置环境变量
railway variables set XRAY_VMESS_AEAD_FORCED=false
railway variables set XUI_ENABLE_FAIL2BAN=true
railway variables set TZ=Asia/Shanghai

# 5. 部署
railway up
```

## 三、部署后配置

### 1. 访问面板

部署完成后，Railway 会提供一个公网 URL，例如：
```
https://your-app.up.railway.app
```

访问地址为：
```
https://your-app.up.railway.app:2053
```

### 2. 获取初始凭据

查看部署日志（Logs）获取随机生成的用户名和密码：
```
Username: xxxxxxxxxx
Password: xxxxxxxxxx
Port: 2053
WebBasePath: xxxxxxxxxxxxxxxxxx
```

### 3. SSL 证书（可选）

如果需要使用 SSL 证书，可以：

1. 将证书文件通过环境变量或其他方式注入
2. 或者使用 Railway 的自动 HTTPS（Railway 提供的域名已自带 SSL）

## 四、数据持久化说明

### Volume 挂载点：`/data`

此目录包含：
- **x-ui.db** - 面板数据库（用户、配置、inbound/outbound 等）
- 其他运行时数据

⚠️ **重要**：确保 Volume 已正确挂载到 `/data`，否则每次重启数据会丢失！

### 日志文件

日志默认存储在 `/var/log`，不会持久化。如需持久化日志，可以：
1. 设置 `XUI_LOG_FOLDER=/data/logs`
2. 或者使用 Railway 的日志服务

## 五、常见问题

### 1. Volume 未挂载导致数据丢失

**症状**：每次重启后，用户配置、inbound 等数据都消失

**解决方案**：
- 确保在 Railway 控制台中添加了 Volume
- 确认 Volume 挂载路径为 `/data`

### 2. 无法访问面板

**检查**：
- 确认 Railway 已分配公网 URL
- 检查防火墙和端口配置
- 查看部署日志是否有错误

### 3. Fail2ban 相关错误

如果 Fail2ban 导致启动失败，可以禁用：
```
XUI_ENABLE_FAIL2BAN=false
```

## 六、环境变量完整列表

| 变量名 | 默认值 | 说明 |
|--------|--------|------|
| `XRAY_VMESS_AEAD_FORCED` | `false` | 是否强制启用 VMess AEAD |
| `XUI_ENABLE_FAIL2BAN` | `true` | 是否启用 Fail2ban（Railway 上建议设为 false） |
| `XUI_DB_FOLDER` | `/data` | 数据库文件存储路径 |
| `XUI_LOG_FOLDER` | `/var/log` | 日志文件存储路径 |
| `XUI_BIN_FOLDER` | `bin` | 二进制文件路径 |
| `XUI_LOG_LEVEL` | `info` | 日志级别 |
| `XUI_DEBUG` | `false` | 调试模式 |
| `TZ` | `Asia/Tehran` | 时区设置 |

## 七、更新部署

### 方式一：自动部署（推荐）

如果通过 GitHub 连接：
1. 推送代码到 GitHub
2. Railway 会自动检测并重新构建部署

### 方式二：手动部署

```bash
railway up
```

## 八、备份和恢复

### 备份

Railway Volume 的数据可以通过以下方式备份：

```bash
# 使用 Railway CLI 连接到容器
railway run bash

# 然后备份数据库
cp /data/x-ui.db /tmp/x-ui-backup.db
```

### 恢复

将备份的数据库文件上传到 `/data` 目录即可。

## 九、性能优化建议

1. **地区选择**：选择离你的用户最近的 Railway 区域
2. **资源配置**：根据需要升级 Railway 的套餐
3. **监控**：使用 Railway 的监控功能查看资源使用情况

## 十、支持

- Railway 文档：https://docs.railway.app
- 3x-ui 项目：https://github.com/MHSanaei/3x-ui
- Railway Volumes 文档：https://docs.railway.com/reference/volumes
