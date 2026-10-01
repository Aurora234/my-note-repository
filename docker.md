Docker的命令虽然很多，但在日常使用中，掌握核心的20%就能解决绝大部分问题。下面是一份高频命令的速查清单，按功能分类整理，方便你快速查阅。

## 疑问

**1.Docker一个容器可以运行多个服务吗？比如PostgreSQL和Milvus在同一个容器中**

> 不推荐
>
> **一个容器只能基于“一个”镜像运行。** 这是Docker的设计原则
>
> 不过需要了解的是，虽然技术上可行，但**在同一个容器里运行多个服务（比如Milvus和PostgreSQL）并不是推荐的做法**。Docker的最佳实践是“一个容器只运行一个服务”，这样能让服务更轻量、易于管理和扩展。



## 云服务器安装Docker

### 卸载

在腾讯云服务器上彻底卸载 Docker，建议按照以下顺序操作，确保软件、配置和数据都被清理干净。

#### 1. 停止 Docker 服务

首先，停止正在运行的 Docker 服务，并取消开机自启：

```bash
sudo systemctl stop docker
sudo systemctl disable docker
```

#### 2. 卸载 Docker 软件包

根据你的服务器系统，使用对应的命令卸载 Docker 及其相关组件：

- **如果是 CentOS / Alibaba Cloud Linux 系统**：

```bash
sudo yum remove -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

- **如果是 Ubuntu / Debian 系统**：

```bash
sudo apt-get purge -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### 3. 删除残留的数据和配置文件

卸载软件包并不会自动删除你之前下载的镜像、创建的容器以及配置文件。如果你想彻底清理，可以执行以下命令：

```bash
# 删除所有镜像、容器、数据卷等核心数据
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd

# 删除 Docker 的配置文件目录
sudo rm -rf /etc/docker
```

#### 4. 清理系统残留（可选）

最后，可以删除 Docker 的用户组，并清理系统中可能残留的依赖包：

```bash
# 删除 docker 用户组
sudo groupdel docker

# 清理不再需要的依赖包（Ubuntu/Debian 系统）
sudo apt-get autoremove -y
```

执行完以上步骤后，Docker 就已经从你的腾讯云服务器上彻底移除了。你可以通过运行 `docker --version` 命令来验证，如果提示“command not found”，说明卸载成功。



### 安装

在 Ubuntu 24.04 上安装 Docker，最稳妥的方式是通过官方仓库进行安装。考虑到你是在国内的腾讯云服务器上操作，直接使用官方源可能会遇到网络超时的问题。

因此，下面这套步骤专门结合了**阿里云镜像源**和**国内镜像加速器**，确保整个安装过程丝滑顺畅。

#### 1. 卸载旧版本（可选）

如果服务器上之前安装过旧版本的 Docker，建议先清理掉，避免冲突：

```bash
sudo apt remove -y docker docker-engine docker.io containerd runc
```

#### 2. 更新软件源并安装依赖

更新系统的软件包列表，并安装必要的证书和工具：

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg
```

#### 3. 配置 Docker 官方密钥（使用阿里云镜像）

为了安全，Docker 需要 GPG 密钥来验证软件包。这里使用阿里云的镜像地址，避免直接连接官方服务器导致网络卡顿：

```bash
# 创建存放密钥的目录
sudo install -m 0755 -d /etc/apt/keyrings

# 下载并转换密钥
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 设置读取权限
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

#### 4. 添加 Docker 软件源

将阿里云的 Docker 仓库添加到系统的软件源列表中。命令会自动识别你的 Ubuntu 24.04 版本代号（noble）：

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

#### 5. 安装 Docker 核心组件

再次更新索引，并安装 Docker 引擎、客户端以及 Compose 插件（部署 Milvus 等复杂应用时非常有用）：

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### 6. 配置国内镜像加速器（关键）

这一步至关重要！如果不配置，后续使用 `docker pull` 拉取镜像（比如 Nginx 或 Milvus）会极慢甚至频繁超时。

```bash
# 创建配置目录
sudo mkdir -p /etc/docker

# 写入加速器配置
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://mirror.baidubce.com",
    "https://docker.m.daocloud.io",
    "https://reg-mirror.k8s.io"
  ]
}
EOF

# 重启 Docker 服务使配置生效
sudo systemctl daemon-reload
sudo systemctl restart docker
```

#### 7. 配置免 sudo 执行（推荐）

为了避免每次使用 Docker 都需要输入 `sudo`，可以将你的当前用户添加到 `docker` 用户组：

```bash
# 将当前用户加入 docker 组
sudo usermod -aG docker $USER

# 刷新用户组权限（或者直接退出重新登录 SSH）
newgrp docker
```

#### 8. 验证安装

运行经典的测试镜像，如果看到 `Hello from Docker!` 字样，则表示全部安装成功：

```bash
docker run --rm hello-world
```







## 常用命令



### ⚙️ 镜像管理 (Images)

镜像像是创建容器的“模板”或“安装包”。

| 功能         | 命令示例                     | 说明                                       |
| :----------- | :--------------------------- | :----------------------------------------- |
| **拉取镜像** | `docker pull nginx:latest`   | 从仓库（默认Docker Hub）下载镜像。         |
| **列出镜像** | `docker images`              | 查看本地已有的镜像列表。                   |
| **删除镜像** | `docker rmi <镜像ID>`        | 删除本地指定镜像。需要先删除使用它的容器。 |
| **构建镜像** | `docker build -t myapp:v1 .` | 根据当前目录下的`Dockerfile`构建镜像。     |

### 🚀 容器生命周期管理 (Containers)

容器是镜像的运行实例，可以把它看作一个轻量级的、独立的运行环境。

| 功能         | 命令示例                                    | 说明                                                         |
| :----------- | :------------------------------------------ | :----------------------------------------------------------- |
| **运行容器** | `docker run -d -p 8080:80 --name web nginx` | **最核心的命令**。从镜像创建并启动一个新容器。<br />-d:Run container in background and print container ID<br />-p:Publish a container's port(s) to the host<br />注意数据挂载保证数据持久化 |
| **查看容器** | `docker ps`                                 | 列出**正在运行**的容器。                                     |
|              | `docker ps -a`                              | 列出**所有**容器（包括已停止的）。                           |
| **停止容器** | `docker stop <容器名>`                      | 优雅地停止一个运行中的容器。                                 |
| **启动容器** | `docker start <容器名>`                     | 启动一个已停止的容器。<容器名>可以是容器名或容器id（id可以只写前几位即可） |
| **重启容器** | `docker restart <容器名>`                   | 重启容器。<容器名>可以是容器名或容器id（id可以只写前几位即可） |
| **删除容器** | `docker rm <容器名>`                        | 删除一个**已停止**的容器。                                   |
|              | `docker rm -f <容器名>`                     | 强制删除一个**正在运行**的容器。                             |

### 🔍 排查与调试 (Debug)

当容器出现问题时，以下命令是排查故障的利器。

| 功能         | 命令示例                             | 说明                                        |
| :----------- | :----------------------------------- | :------------------------------------------ |
| **查看日志** | `docker logs -f <容器名>`            | 查看容器的输出日志。 `-f` 表示实时跟踪。    |
| **进入容器** | `docker exec -it <容器名> /bin/bash` | 进入容器内部执行命令，常用于调试。exit 退出 |
| **查看详情** | `docker inspect <容器名>`            | 查看容器或镜像的非常详细的配置信息。        |
| **文件拷贝** | `docker cp <文件> <容器名>:/路径/`   | 在宿主机和容器之间复制文件。                |

### 🧹 系统信息与维护

这部分命令帮助你了解Docker环境状态，并进行清理。

| 功能         | 命令示例              | 说明                                                       |
| :----------- | :-------------------- | :--------------------------------------------------------- |
| **查看版本** | `docker version`      | 显示客户端和服务端的版本信息。                             |
| **系统信息** | `docker info`         | 显示系统范围的信息，如容器/镜像数量等。                    |
| **资源统计** | `docker stats`        | 实时查看容器的CPU、内存等资源使用情况。                    |
| **系统清理** | `docker system prune` | **一键清理**：删除所有停止的容器、未使用的网络和悬空镜像。 |

### 🌐 网络与数据卷 (Networks & Volumes)

| 功能           | 命令示例                       | 说明                 |
| :------------- | :----------------------------- | :------------------- |
| **查看网络**   | `docker network ls`            | 列出所有Docker网络。 |
| **创建网络**   | `docker network create my-net` | 创建一个自定义网络。 |
| **查看数据卷** | `docker volume ls`             | 列出所有数据卷。     |
| **创建数据卷** | `docker volume create my-vol`  | 创建一个新的数据卷。 |

### 🏗️ Docker Compose (多容器编排)

当你需要管理多个容器（如一个应用加一个数据库）时，`docker-compose`命令非常实用。

| 功能         | 命令示例               | 说明                             |
| :----------- | :--------------------- | :------------------------------- |
| **启动服务** | `docker-compose up -d` | 启动所有服务，`-d`表示后台运行。 |
| **停止服务** | `docker-compose down`  | 停止并移除所有容器、网络等。     |
| **查看状态** | `docker-compose ps`    | 查看所有服务的状态。             |

---

### 💡 常用选项与技巧

*   `docker run` 常用选项:
    *   `-d`：后台运行容器。
    *   `-p 宿主机端口:容器端口`：进行端口映射。
    *   `-v 宿主机路径:容器路径`：挂载数据卷，实现数据持久化。
    *   `--name 容器名`：为容器指定一个名称。
    *   `--restart=always`：设置容器在退出或Docker重启时自动启动。
*   **批量操作技巧**:
    *   结合`-q`参数（只返回ID）和`$()`命令，可以批量操作。例如，`docker rm -f $(docker ps -aq)` 会强制删除所有容器。

可以把这份清单收藏起来，当成一个速查手册。平时可以多用 `--help` 参数（例如 `docker run --help`）来查看每个命令更详细的用法。

---





## 卷映射与数据挂载

Docker的数据映射机制，主要解决了容器的两大核心问题：**数据持久化**（防止容器删除后数据丢失）和**数据共享**（在宿主机与容器、容器与容器之间共享数据）。

Docker提供了几种不同的数据管理方式，其中最主要的是**数据卷（Volumes）** 和**绑定挂载（Bind Mounts）**。

### 🆚 数据映射方式对比

为了帮你快速理解，以下是这两种主要方式的对比：

| 特性             | 数据卷 (Volumes)                                             | 绑定挂载 (Bind Mounts)                                       |
| :--------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **主机存储位置** | 由Docker管理，通常位于 `/var/lib/docker/volumes/`            | 由你指定，可以是主机上的任意路径                             |
| **管理方式**     | 通过 `docker volume` 命令独立管理                            | 直接依赖于主机文件系统                                       |
| **适用场景**     | **生产环境**首选，用于数据库、应用数据等需要持久化存储的场景 | **开发环境**首选，用于代码热加载、配置文件注入等             |
| **自动创建目录** | 是。如果卷不存在，Docker会自动创建                           | 否。如果主机路径不存在，`--mount` 会报错                     |
| **重要**         | Docker会自动创建且会将容器内的内容复制到主机的卷上面         | Docker会直接将主机上的路径与容器内的路径关联，其他什么都不会做。容器内原有的数据将被忽略 |
| **优点**         | 安全性高，易于备份和迁移，支持卷驱动                         | 性能好，直接反映主机文件变化，简单直观                       |



此外，还有两种特殊类型：
*   **tmpfs挂载 (tmpfs mounts)**：数据存储在宿主机内存中，容器停止后数据即被清除。适合存放临时、敏感或高性能要求的缓存数据。
*   **数据卷容器 (Data Volume Containers)**：这是一种较旧的方式，通过一个专门的容器来共享数据卷，现已被 `--volumes-from` 选项替代，社区已不推荐使用。

---

### 🛠️ 核心操作命令详解

#### 1. 管理数据卷 (Volumes)

*   **创建数据卷**：`docker volume create <卷名称>`
    ```bash
    docker volume create my-app-data
    ```
*   **列出数据卷**：`docker volume ls`
    ```bash
    docker volume ls
    ```
*   **查看数据卷详情**：`docker volume inspect <卷名称>`
    ```bash
    docker volume inspect my-app-data
    ```
*   **删除数据卷**：`docker volume rm <卷名称>`
    ```bash
    docker volume rm my-app-data
    ```
*   **清理未使用的数据卷**：`docker volume prune`
    ```bash
    docker volume prune
    ```

#### 2. 挂载数据卷或目录到容器

这是使用数据映射的最核心操作，可以在 `docker run` 或 `docker create` 时通过 `-v` 或 `--mount` 参数完成。

**方式一：使用 `-v` 或 `--volume` (简洁)**

*   **挂载一个数据卷**：`-v <卷名称>:<容器内路径>`
    ```bash
    docker run -d --name app -v my-app-data:/app/data nginx
    ```
    此命令将名为 `my-app-data` 的卷挂载到容器的 `/app/data` 目录。

*   **挂载一个主机目录 (绑定挂载)**：`-v <主机绝对路径>:<容器内路径>`
    ```bash
    docker run -d --name dev -v /home/user/code:/app/code node:18
    ```
    此命令将主机上的 `/home/user/code` 目录挂载到容器的 `/app/code` 目录。

*   **挂载为只读**：在最后加上 `:ro`
    ```bash
    docker run -d --name config -v /host/config:/app/config:ro nginx
    ```
    容器内对 `/app/config` 的修改将被禁止。

**方式二：使用 `--mount` (更明确，推荐)**

`--mount` 的语法更清晰，适合复杂场景。

*   **挂载一个数据卷**：
    ```bash
    docker run -d --name app \
      --mount type=volume,source=my-app-data,target=/app/data \
      nginx
    ```
    `type=volume` 表示挂载类型为数据卷，`source` 指定卷名，`target` 指定容器内路径。

*   **挂载一个主机目录 (绑定挂载)**：
    ```bash
    docker run -d --name dev \
      --mount type=bind,source=/home/user/code,target=/app/code \
      node:18
    ```
    `type=bind` 表示挂载类型为绑定挂载。

*   **挂载为只读**：添加 `readonly` 参数
    ```bash
    docker run -d --name config \
      --mount type=bind,source=/host/config,target=/app/config,readonly \
      nginx
    ```
    功能与 `-v` 的 `:ro` 相同。

### 进阶技巧与注意事项

*   **理解路径**：`-v` 和 `--mount` 的 `source` **必须是绝对路径**。
*   **权限管理**：默认容器内以 `root` 用户访问挂载文件。如果容器内应用使用非 `root` 用户（如 `node`），可能遇到权限问题，可通过 `--user` 参数指定运行用户。
*   **数据持久性**：数据卷的生命周期独立于容器。删除容器（`docker rm`）**不会**自动删除数据卷。需要手动用 `docker volume rm` 清理。
*   **共享数据卷**：多个容器可以同时挂载同一个数据卷，实现数据共享。
*   **备份与恢复**：数据卷的备份和恢复通常通过 `--volumes-from` 选项配合 `tar` 命令来完成。

### 总结

简单来说，选择哪种方式取决于你的场景：

*   **生产环境的数据持久化（如数据库）**：请使用**数据卷 (Volumes)**。
*   **开发环境的代码同步或配置文件注入**：请使用**绑定挂载 (Bind Mounts)**。

在命令选择上，**推荐优先使用 `--mount`**，它的语法更清晰、更强大。





## 自定义网络







Docker 自定义网络的所有操作都集中在 `docker network` 命令下，你可以把它看作管理网络的“控制中心”。

下面是这些核心指令的速查清单：

### 📋 核心管理指令

*   **`docker network ls`**：查看所有网络列表。
    ```bash
    docker network ls          # 查看所有网络
    docker network ls -q       # 只显示网络ID
    docker network ls --filter driver=bridge # 只显示bridge类型的网络
    ```
*   **`docker network create`**：创建一个新的自定义网络。
    ```bash
    # 创建一个默认的桥接网络
    docker network create my-network
    
    # 创建时指定驱动、子网和网关
    docker network create \
      --driver bridge \
      --subnet 172.20.0.0/16 \
      --gateway 172.20.0.1 \
      my-custom-network
    ```
*   **`docker network inspect`**：查看网络的详细配置信息。
    ```bash
    docker network inspect my-network
    ```
*   **`docker network rm`**：删除一个自定义网络。
    > **注意**：删除前，请确保**没有容器正在连接**到该网络。
    ```bash
    docker network rm my-network
    ```
*   **`docker network prune`**：清理所有未被使用的网络。
    ```bash
    docker network prune
    ```

### 🔗 容器与网络的连接指令

*   **`docker network connect`**：将**已存在**的容器连接到一个网络。
    ```bash
    # 将 my-container 连接到 my-network
    docker network connect my-network my-container
    
    # 连接时指定IP地址（仅自定义bridge支持）
    docker network connect --ip 172.20.0.100 my-network my-container
    ```
*   **`docker network disconnect`**：将容器从一个网络中断开。
    ```bash
    docker network disconnect my-network my-container
    ```

### 🚀 启动容器时的网络指令

*   **`--network`**：在 `docker run` 中指定容器启动时要连接的网络。
    ```bash
    docker run -d --name my-container --network my-network nginx
    ```
*   **`--network-alias`**：在 `docker run` 中为容器在网络中设置一个或多个别名，方便其他容器通过别名访问。
    ```bash
    docker run -d --name web --network my-network --network-alias web1 --network-alias web2 nginx
    ```



### 自定义网络，容器之间域名访问

当你创建一个自定义网络时，Docker 会自动在该网络内启动一个**内置的 DNS 服务器**（地址固定为 `127.0.0.11`）。

1. **自动注册**：当容器加入这个网络并启动时，Docker 会自动把容器的名字注册到这个内置 DNS 中。
2. **域名解析**：当网络内的另一个容器尝试访问这个名字时，DNS 服务器会自动将其解析为该容器的内网 IP 地址。
3. **直接通信**：容器之间通过解析后的 IP 直接通信，无需经过宿主机端口映射。

**举例**

```bash
# 1. 创建自定义网络
docker network create my-net

# 2. 启动一个 Nginx 容器，命名为 web
docker run -d --name web --network my-net nginx

# 3. 启动一个测试容器，命名为 app
docker run -it --name app --network my-net alpine sh
```

此时，你在 `app` 容器内部，可以直接通过 `web` 这个名字来访问 Nginx 服务：

```bash
# 在 app 容器内执行
ping web
# 或者
curl http://web:80
```

 **⚠️ 关键注意事项**

- **必须是自定义网络**：Docker 默认的 `bridge` 网络（即 `docker0`）**不支持**通过容器名进行 DNS 解析。在默认网络下，容器之间只能通过 IP 地址互相访问。
- **网络别名**：除了容器名，你还可以使用 `--network-alias` 参数给容器起额外的“域名”。例如 `--network-alias backend`，这样其他容器既可以通过原容器名访问，也可以通过 `backend` 这个名字访问。
- **仅限同网络**：DNS 解析只在同一个自定义网络内的容器之间生效。不同网络下的容器默认是完全隔离的，无法直接通过名字访问。

这种机制在微服务架构中非常实用，比如你的 Java 后端服务需要连接 Milvus 数据库时，只需要将连接地址配置为 `http://milvus:19530`（假设 Milvus 容器名为 milvus），而完全不用担心 Milvus 容器的 IP 地址在重启后发生变化。





### ⚙️ `docker network create` 常用选项详解

*   `--driver, -d`：指定网络驱动，默认为 `bridge`。其他常见驱动有 `overlay`（用于Swarm集群）、`macvlan`等。
*   `--subnet`：指定网络的子网段，例如 `172.20.0.0/16`。
*   `--gateway`：指定网络的网关IP，例如 `172.20.0.1`。
*   `--ip-range`：限制分配给容器的IP地址范围。
*   `--label`：为网络添加元数据标签，便于分类和管理。
*   `--attachable`：用于 `overlay` 网络，允许普通容器（非Swarm服务）连接到该网络。

### 💡 核心概念与技巧

*   **网络驱动 (Driver)**：决定了网络的类型和行为。
    *   **bridge**：最常用，用于**单机**容器间通信。
    *   **overlay**：用于**跨主机**的容器通信，需配合Docker Swarm。
    *   **host**：容器直接使用**宿主机**的网络栈。
    *   **macvlan**：为容器分配独立的MAC地址，使其在网络上像一台物理设备。
    *   **none**：容器无网络。
*   **自定义网络的优势**：最主要的一点是，连接到同一个自定义网络的容器，**可以通过容器名直接互相访问**，Docker内置的DNS服务会帮你自动解析。这比使用已弃用的 `--link` 参数要强大和方便得多。
*   **指定静态IP**：只有**用户自定义的** `bridge` 网络才支持为容器指定静态IP。默认的 `bridge` 网络不支持此功能。







## Docker-compose

Docker Compose 是一个用于定义和运行多容器 Docker 应用程序的工具。你可以通过一个 `docker-compose.yml` 文件来配置应用的所有服务、网络和卷，然后使用一个命令就能启动或停止整个应用栈。

它的核心价值在于，能将之前学习到的**镜像、容器、网络、数据卷**等概念，用一种声明式的方式统一编排起来，极大地简化了复杂应用的部署和管理。

### 1. 准备工作：安装与版本确认

*   **安装**：最简单和推荐的方式是安装 **Docker Desktop**，它会自带 Docker Compose。
    *   **Linux 用户**：可以通过官方仓库安装 Docker Compose 插件。
*   **版本确认**：请确保你使用的是 **Compose V2**（命令为 `docker compose`，注意中间是空格，没有连字符）。旧的 V1 版本（`docker-compose`）已不再推荐使用。
    ```bash
    docker compose version
    ```

### 2. 核心：编写 `docker-compose.yml` 文件

这个 YAML 文件是 Compose 的核心。你需要在你的项目根目录下创建它，文件名可以是 `docker-compose.yml` 或 `compose.yaml`。

一个典型的文件包含以下主要部分：

*   **`services`**: **最核心的部分**，定义你要运行的所有容器（即“服务”）。
*   **`networks`**: （可选）定义自定义网络，方便服务间通信。
*   **`volumes`**: （可选）定义数据卷，用于持久化数据。

下面是一个 `docker-compose.yml` 文件的典型结构和配置项说明：

```yaml
# 1. 版本声明 (建议使用 '3.8' 或更高版本)
version: '3.8'

# 2. 定义数据卷 (可选)
volumes:
  db_data:

# 3. 定义网络 (可选)
networks:
  app_net:

# 4. 核心：定义所有服务
services:
  # 服务名称 (将成为网络中的主机名)
  web:
    # 指定使用的镜像，或使用 build 从 Dockerfile 构建
    image: nginx:alpine
    # 容器端口映射 (宿主机:容器)
    ports:
      - "80:80"
    # 数据卷挂载
    volumes:
      - ./html:/usr/share/nginx/html
    # 依赖声明 (控制启动顺序)
    depends_on:
      - db
    # 重启策略
    restart: unless-stopped
    # 连接到自定义网络
    networks:
      - app_net

  db:
    image: mysql:8.0
    # 环境变量
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE: mydb
    # 挂载命名卷以实现数据持久化
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - app_net
```

### 3. 实战：常用命令一览

所有命令都需要在包含 `docker-compose.yml` 文件的目录下执行。

| 命令                         | 说明                                 | 示例                                                         |
| :--------------------------- | :----------------------------------- | :----------------------------------------------------------- |
| **`docker compose up`**      | **启动**所有服务。                   | `docker compose up -d` (后台运行)<br />docker compose -f <你的yaml文件>  up -d |
| **`docker compose down`**    | **停止并移除**所有容器、网络等资源。 | `docker compose down`                                        |
| **`docker compose ps`**      | 列出当前项目的**容器状态**。         | `docker compose ps`                                          |
| **`docker compose logs`**    | 查看服务的**日志**输出。             | `docker compose logs -f web` (实时跟踪web服务日志)           |
| **`docker compose exec`**    | 在**运行中**的容器内执行命令。       | `docker compose exec web bash` (进入web容器)                 |
| **`docker compose stop`**    | **停止**服务，但保留容器。           | `docker compose stop web` (只停止web服务)                    |
| **`docker compose restart`** | **重启**服务。                       | `docker compose restart web`                                 |
| **`docker compose build`**   | 重新**构建**服务镜像。               | `docker compose build --no-cache web`                        |

### 4. 关键优势与最佳实践

*   **环境一致性**：将 `docker-compose.yml` 纳入版本控制，团队成员克隆代码后，执行一个命令即可启动完全一致的开发环境。
*   **自动网络**：默认情况下，Compose 会为你的项目创建一个独立的网络，同一项目中的服务可以通过**服务名**（如 `web`、`db`）直接互相访问。
*   **数据持久化**：使用 `volumes` 关键字定义数据卷，确保数据库等有状态服务的数据在容器重启后不会丢失。
*   **生产环境考量**：Compose 非常适合开发、测试和小型生产环境。对于大规模、需要复杂编排的生产环境，Kubernetes 是更合适的选择。

### 💎 总结

总的来说，使用 Docker Compose 的典型工作流是：
1.  在项目根目录编写 `docker-compose.yml` 文件。
2.  执行 `docker compose up -d` 一键启动所有服务。
3.  需要停止时，执行 `docker compose down` 进行清理。

它通过一个配置文件和一个命令，将你之前学习的所有 Docker 知识整合起来，让多容器应用的管理变得非常简单和高效。