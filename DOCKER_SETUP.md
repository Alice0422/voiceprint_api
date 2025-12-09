# Docker 部署说明

## 前置准备

### 1. 创建配置文件目录

```bash
mkdir -p data
```

### 2. 创建配置文件

将 `voiceprint.yaml` 复制到 `data/.voiceprint.yaml`：

```bash
cp voiceprint.yaml data/.voiceprint.yaml
```

### 3. 配置 MySQL 连接

编辑 `data/.voiceprint.yaml`，修改 MySQL 配置：

**重要：** 如果 MySQL 运行在宿主机上，容器内需要使用以下方式访问：

#### 方式 1：使用 host.docker.internal（推荐）

在 `docker-compose.yml` 中取消注释：
```yaml
extra_hosts:
  - "host.docker.internal:host-gateway"
```

然后在 `data/.voiceprint.yaml` 中设置：
```yaml
mysql:
  host: "host.docker.internal"  # 容器内访问宿主机
  port: 3306
  user: "root"
  password: "your_password"
  database: "voiceprint_db"
```

#### 方式 2：使用宿主机 IP

如果方式 1 不工作，可以：
1. 查找宿主机 IP：`ipconfig` (Windows) 或 `ifconfig` (Linux/Mac)
2. 在配置文件中使用宿主机 IP

#### 方式 3：使用 Docker 网络（如果 MySQL 也在 Docker 中）

如果 MySQL 也在 Docker Compose 中，可以使用服务名作为 host。

### 4. 确保 MySQL 允许远程连接

如果 MySQL 在宿主机上，需要确保允许 Docker 容器连接：

```sql
-- 允许 root 用户从任何主机连接（仅用于开发环境）
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'your_password';
FLUSH PRIVILEGES;
```

## 启动服务

### 1. 拉取镜像

```bash
docker-compose pull
```

### 2. 启动容器

```bash
docker-compose up -d
```

### 3. 查看日志

```bash
docker-compose logs -f voiceprint-api
```

### 4. 停止服务

```bash
docker-compose down
```

## 验证服务

访问健康检查接口：
```
http://localhost:8005/voiceprint/health?key=<your_authorization_key>
```

API 文档：
```
http://localhost:8005/voiceprint/docs
```

## 常见问题

### 1. 配置文件未找到

错误：`配置文件 data/.voiceprint.yaml 未找到`

解决：确保已创建 `data/.voiceprint.yaml` 文件

### 2. 数据库连接失败

错误：`数据库连接失败`

解决：
- 检查 MySQL 是否运行
- 检查配置文件中的 host、port、user、password 是否正确
- 如果 MySQL 在宿主机，使用 `host.docker.internal` 或宿主机 IP
- 检查 MySQL 是否允许远程连接

### 3. 音频处理失败

错误：`libsndfile` 相关错误

解决：已修复，Dockerfile 已包含必要的系统库

### 4. 端口被占用

错误：`port is already allocated`

解决：修改 `docker-compose.yml` 中的端口映射，例如 `"8006:8005"`

