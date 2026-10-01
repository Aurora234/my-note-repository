- ins-rbfmy7ol

- Langchain1.2_demo

- Chl2421724656



# 腾讯云Linux的postgres操作

**安装**

````bash
sudo apt install postgresql
````

**验证**

```bash
psql --version
```

**查看状态与端口占用**

```bash
sudo systemctl status postgresql
```

```bash
sudo netstat -tunlp | grep postgres
```



PostgreSQL安装后会创建一个名为 `postgres` 的默认超级用户。我们需要先切换到该用户来操作数据库。

- **切换用户并进入命令行**:

  ```
  sudo -i -u postgres
  psql
  ```

  执行后，命令行提示符会变为 `postgres=#`，表示已进入PostgreSQL的交互式终端。

以下是进入 `psql` 终端后的一些常用操作：

- **修改 `postgres` 用户密码（安全建议）**:

  ```
  ALTER USER postgres WITH PASSWORD '你的强密码';
  ```

  **注意**：请务必将 `你的强密码` 替换为一个足够复杂的密码。

- **创建新的数据库**:

  sql

  ```
  CREATE DATABASE mydb;
  ```

  更详细的创建方式，可以指定编码和所有者：

  ```
  CREATE DATABASE mydb WITH ENCODING='UTF8' OWNER=postgres;
  ```

- **赋权**

  ```bash
  GRANT ALL PRIVILEGES ON DATABASE langchain_db TO langchain_user;
  ```

- **创建新的数据库用户**:

  ```
  CREATE USER myuser WITH PASSWORD '用户密码';
  ```

- **列出所有数据库**:

  ```
  \l
  ```

- **连接到指定数据库**:

  ```
  \c mydb
  ```

- **退出 `psql` 终端**:

  ```
  \q
  ```

另外，PostgreSQL也提供了一个命令行工具 `createdb`，可以不用进入 `psql` 终端，直接在Shell环境下创建数据库：

```
createdb mydb
```

**测试URL**

postgresql默认监听5432端口

```bash
psql "postgresql://langchain_user:123456@localhost:5432/langchain_db?sslmode=disable"
```





# **服务器协议设置**

**服务器安全组放通5432端口**



**PostgreSQL监听所有IP**

- 查看相关配置文件路径

  - ```bash
    sudo -u postgres psql -c "SHOW config_file;"
    ```

- 编写

  - ```bash
    sudo vim /etc/postgresql/16/main/postgresql.conf
    ```

在60行找到`listen_address = 'localhost'`，按下`yy`，然后按下p复制一行

打开复制那行的注释，并修改

```bash
listen_address = '*'
```

保存退出，esc后输入`:exit`



**添加允许规则**

```bash
sudo -u postgres psql -c "SHOW hba_file;"
```

在文件最后添加

```bash
host    langchain_db     langchain_user            0.0.0.0/0                 scram-sha-256
```

**重启服务**

```bash
sudo systemctl restart postgresql
```



**测试连接**

该操作将直接进入PostgreSQL数据库并切换到langchain_user用户的langchain_db数据库下

```bash
psql "postgresql://langchain_user:123456@58.87.103.143:5432/langchain_db?sslmode=disable"
```





# PostgreSQL数据库操作

### 📝 创建数据库 (CREATE)

创建数据库主要有两种方式：SQL命令和命令行工具。

- **SQL命令方式**：在`psql`终端中执行 `CREATE DATABASE` 语句。

  ```
  -- 基本创建
  CREATE DATABASE mydb;
  ```

  你还可以在创建时指定更多属性，例如：

  - **指定所有者**：`CREATE DATABASE mydb OWNER myuser;`
  - **指定编码**：`CREATE DATABASE mydb WITH ENCODING='UTF8';`
  - **指定模板**：`CREATE DATABASE mydb TEMPLATE template0;`

- **命令行工具方式**：在Shell环境下使用 `createdb` 命令。

  ```
  # 创建数据库
  createdb mydb
  # 创建并指定所有者
  createdb -O myuser mydb
  ```

> **注意**：创建数据库需要拥有超级用户权限或特殊的`CREATEDB`权限。

### ✏️ 修改数据库 (ALTER)

使用 `ALTER DATABASE` 语句可以修改数据库的各种属性。

- **重命名数据库**：

  ```
  ALTER DATABASE mydb RENAME TO newdb;
  ```

  注意，不能重命名当前正在连接的数据库。

- **修改数据库所有者**：

  ```
  ALTER DATABASE mydb OWNER TO new_owner;
  ```

- **修改默认表空间**：

  ```
  ALTER DATABASE mydb SET TABLESPACE new_tablespace;
  ```

  此操作会物理移动数据，执行时不能有人连接到该数据库。

- **修改连接数限制**：

  ```
  ALTER DATABASE mydb CONNECTION LIMIT 10;
  ```

  将最大并发连接数设为10，`-1`表示无限制。

- **修改会话配置**：为连接该数据库的所有新会话设置默认配置参数。

  ```
  ALTER DATABASE mydb SET timezone TO 'Asia/Shanghai';
  ```

> **注意**：修改数据库属性通常需要数据库所有者或超级用户权限。

### 🗑️ 删除数据库 (DROP)

使用 `DROP DATABASE` 命令可以永久删除数据库。

```
DROP DATABASE mydb;
```

- **危险操作**：此操作**不可撤销**，会一并删除库内所有对象。
- **权限要求**：只有数据库所有者或超级用户才能执行。
- **重要限制**：**不能**在连接到待删除的数据库时执行此命令。你需要先切换到其他数据库（如`postgres`）再操作。

> **安全建议**：从PostgreSQL 13开始，可以使用 `FORCE` 选项强制断开所有连接并删除数据库。

### 🔗 连接与切换数据库

- **在`psql`中切换**：使用元命令 `\c` 或 `\connect`。

  ```
  \c mydb
  ```

- **在Shell中连接**：使用 `psql` 命令的 `-d` 参数。

  ```
  psql -d mydb -U myuser
  ```

### 👀 查看数据库信息

- **列出所有数据库**：在`psql`终端中使用 `\l` 或 `\list` 元命令。

- **查看数据库大小**：使用 `pg_database_size()` 函数。

  ```
  SELECT pg_database_size('mydb');
  ```



### 查表

#### 核心元命令速查表

| 目标                     | 命令                | 说明                                                 |
| :----------------------- | :------------------ | :--------------------------------------------------- |
| **列出所有表**           | `\dt`               | 只列出当前schema（默认为`public`）中的普通表。       |
| **列出所有表（含详情）** | `\dt+`              | 额外显示表的大小和描述等信息。                       |
| **列出所有schema的表**   | `\dt *.*`           | 列出数据库中所有schema下的表。                       |
| **查看特定schema的表**   | `\dt schema_name.*` | 例如，`\dt public.*` 查看`public` schema下的所有表。 |
| **查看表结构**           | `\d table_name`     | 显示表的列、数据类型、约束、索引等详细信息。         |
| **查看表结构（含详情）** | `\d+ table_name`    | 显示更详细的信息，如列注释、数据存储分布等。         |
| **列出所有数据库对象**   | `\d`                | 列出当前数据库中所有的表、视图、序列等。             |
| **切换数据库**           | `\c database_name`  | 切换到另一个数据库。                                 |
