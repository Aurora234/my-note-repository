



# GIL:全局解释器锁

它是一把**互斥锁（Mutex）**，它的存在强制要求在任何时刻，**只有一个线程能执行 Python 字节码**。

## 1 是什么？

你可以把 GIL 想象成一个**“话筒”**或者**“唯一的操作权”**。

- **场景：** 你有一个 Python 进程，里面有多个线程，同时你的电脑是多核 CPU。
- **如果没有 GIL：** 理论上，线程 A 可以在核心 1 上运行，线程 B 可以在核心 2 上同时运行（并行）。
- **有 GIL（现实）：** 无论你有多少个 CPU 核心，也无论你开了多少个线程，**同一时刻只有一个线程能抢到这个“话筒”（GIL），从而执行 Python 代码**。其他线程只能排队等待。

> **注意：** GIL 不是 Python 语言的特性，而是 **CPython** 解释器（我们最常用的 Python 实现）的特性。像 Jython（基于 Java）或 IronPython（基于 .NET）就没有 GIL。

## 2.GIL 是如何工作的？

GIL 并不是让一个线程一直霸占到死，它有两种主要的释放机制：

1. **主动释放（I/O 操作）：** 当线程遇到**输入/输出操作**（如读写文件、网络请求、`time.sleep()`）时，它会主动交出 GIL。因为 I/O 操作不需要 CPU 计算，这时候让其他线程去干活，效率最高。
2. **被动释放（时间片轮转）：** 如果线程一直在疯狂计算（CPU 密集型），它会霸占 GIL 一段时间（例如执行了 1000 个字节码指令或 5 毫秒），然后解释器会强制让它释放 GIL，去排队重新抢锁。

## 3. GIL 对性能的真实影响

#### 📉 CPU 密集型任务（受 GIL 严重拖累）

- **场景：** 大量数学计算、图像处理、循环遍历大数据。
- **现象：** 多线程**无法**利用多核 CPU。多个线程实际上是在“轮流”使用 CPU，加上线程切换的开销，多线程的执行时间可能**等于甚至慢于**单线程。
- **结论：** **不要用多线程做计算**。

#### 📈 I/O 密集型任务（GIL 的“豁免区”）

- **场景：** 爬虫（等待网页下载）、Web 服务器（等待数据库响应）、文件读写。
- **现象：** 当一个线程等待网络响应（I/O 阻塞）时，它会释放 GIL，其他线程立刻接手工作。这使得程序在等待期间能做更多事。
- **结论：** **多线程非常适合处理 I/O 任务**，能显著提升吞吐量。

## 4.如何绕过 GIL 的限制？

1. **多进程（Multiprocessing）：** 这是目前最主流的方案。每个进程都有独立的 Python 解释器和独立的 GIL，这样就能真正利用多核 CPU 并行计算。
2. **使用 C 扩展库：** 像 NumPy、SciPy 这样的库，在执行底层计算时会**释放 GIL**，直接调用 C 语言的并行能力，所以它们在多线程下依然能并行计算。
3. **异步编程（Asyncio）：** 对于 I/O 密集型任务，使用 `async/await` 可以避免线程切换的开销，效率比多线程更高。
4. **等待 Python 3.13+（未来方案）：** Python 核心团队已经官宣在 Python 3.13 中实验性地移除了 GIL（PEP 703）。这意味着在未来，Python 原生多线程将能真正并行，但目前（2025 年）主流生产环境大多还在使用旧版本。



---



# 快速上手

Python 标准库中的 `threading` 模块是我们的主要工具。总结三种常用的创建方式：

#### 方式一：直接使用 `Thread` 类（最常用）

这是最简单直接的方法，通过指定目标函数来创建线程。

```python
import threading
import time

def worker(name, delay):
    print(f"线程 {name} 开始工作")
    time.sleep(delay)  # 模拟耗时操作
    print(f"线程 {name} 结束工作")

# 创建线程
t1 = threading.Thread(target=worker, args=("A", 2))
t2 = threading.Thread(target=worker, args=("B", 2))

# 启动线程
t1.start()
t2.start()

# 等待线程执行完毕（阻塞主线程）
t1.join()
t2.join()
print("所有线程执行完毕")
```

#### 方式二：继承 `Thread` 类

如果你需要封装复杂的逻辑，可以通过继承 `threading.Thread` 并重写 `run()` 方法来实现。

```python
class MyThread(threading.Thread):
    def __init__(self, name):
        super().__init__()
        self.name = name

    def run(self):
        print(f"我是线程 {self.name}，正在执行任务")

# 使用
thread = MyThread("Worker-1")
thread.start()
```

#### 方式三：使用线程池（推荐）

对于管理大量线程，手动创建和销毁开销很大。`concurrent.futures` 模块提供的线程池可以自动管理线程的生命周期，代码更简洁。

```python
from concurrent.futures import ThreadPoolExecutor

def task(n):
    print(f"执行任务 {n}")
    return f"任务 {n} 完成"

with ThreadPoolExecutor(max_workers=3) as executor: # 最多3个线程
    results = executor.map(task, range(5)) # 提交5个任务
    for result in results:
        print(result)
```

在多线程环境下，数据安全至关重要。

- **线程同步（Lock）：** 多个线程如果同时修改同一个全局变量，可能会导致数据错乱（竞态条件）。这时需要使用 `threading.Lock()`。就像上厕所要上锁一样，拿到锁的线程才能修改数据，其他线程必须排队等待。
- **线程间通信（Queue）：** 如果你需要一个线程生产数据，另一个线程消费数据，推荐使用 `queue.Queue`。它是线程安全的，非常适合用来协调生产者和消费者的关系。
- **守护线程（Daemon）：** 默认情况下，主线程会等待所有子线程结束后才退出。如果你希望主线程结束时，子线程也立即强制结束，可以将子线程设置为守护线程（`thread.daemon = True`）。





# Socket套接字

### 1. 先验知识

- **`AF_INET`**：表示使用 IPv4 地址。
- **`SOCK_STREAM`**：表示 TCP 协议，它是面向连接的、可靠的，像打电话。
- **`SOCK_DGRAM`**：表示 UDP 协议，它是无连接的、不可靠的，像发短信/广播。
- **`bind(address)`**：绑定 IP 地址和端口号。
- **`listen()` / `accept()`**：用于 TCP 服务器端，监听连接并接受客户端的握手。
- **`connect()`**：用于 TCP 客户端，主动连接服务器。
- **`send()` / `recv()`**：用于 TCP 发送和接收数据。
- **`sendto()` / `recvfrom()`**：用于 UDP 发送和接收数据（因为 UDP 无连接，所以发送时必须指定目标地址）。

### 2. TCP 编程（可靠传输

TCP 需要先建立连接（三次握手），保证数据按序、不丢失。它适用于文件传输、网页浏览、邮件等场景。

#### TCP 服务器端流程

1. 创建 socket
2. 绑定地址 `bind`
3. 监听 `listen`
4. 接受连接 `accept`（阻塞等待）
5. 收发数据 `recv` / `send`
6. 关闭连接

```python
import socket

# 1. 创建 TCP socket
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 设置端口复用（避免"地址已在使用"错误）
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

# 2. 绑定地址和端口
server_address = ('localhost', 12345)
server_socket.bind(server_address)

# 3. 开始监听，最大等待连接数为 5
server_socket.listen(5)
print("服务器启动，等待连接...")

try:
    while True:
        # 4. 等待客户端连接 (阻塞)
        connection, client_address = server_socket.accept()
        print(f"收到客户端 {client_address} 的连接")

        try:
            while True:
                # 5. 接收数据
                data = connection.recv(1024) # 缓冲区大小为 1024 字节
                if data:
                    print(f"收到: {data.decode('utf-8')}")
                    # 发送响应
                    connection.sendall(b"TCP: Message received")
                else:
                    # 客户端断开连接
                    break
        finally:
            connection.close()
            print(f"连接 {client_address} 已关闭")
except KeyboardInterrupt:
    print("服务器关闭")
finally:
    server_socket.close()
```

#### TCP 客户端流程

1. 创建 socket
2. 连接服务器 `connect`
3. 收发数据 `send` / `recv`
4. 关闭连接

```python
import socket

# 1. 创建 TCP socket
client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 2. 连接服务器
server_address = ('localhost', 12345)
client_socket.connect(server_address)

try:
    # 3. 发送数据
    message = "Hello TCP Server!"
    client_socket.sendall(message.encode('utf-8'))
    
    # 接收响应
    response = client_socket.recv(1024)
    print(f"服务器回复: {response.decode('utf-8')}")
finally:
    # 4. 关闭连接
    client_socket.close()
```

### 3. UDP 编程（快速传输）

UDP 不需要建立连接，直接发送数据包。它不保证数据一定能到达，也不保证顺序。适用于视频直播、在线游戏、DNS 查询等。

#### UDP 服务器端

```python
import socket

# 创建 UDP socket
server_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

# 绑定地址
server_address = ('localhost', 12346)
server_socket.bind(server_address)

print("UDP 服务器启动...")

while True:
    # 接收数据和客户端地址
    data, address = server_socket.recvfrom(1024)
    print(f"收到来自 {address} 的消息: {data.decode('utf-8')}")
    
    # 发送响应 (需要指定对方地址)
    server_socket.sendto(b"UDP: Got it!", address)
```

####  UDP 客户端

```python
import socket

# 创建 UDP socket
client_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

# 目标服务器地址
server_address = ('localhost', 12346)

# 发送数据
message = "Hello UDP Server!"
client_socket.sendto(message.encode('utf-8'), server_address)

# 接收响应
data, server = client_socket.recvfrom(1024)
print(f"收到回复: {data.decode('utf-8')}")

client_socket.close()
```

### 4. TCP vs UDP 核心区别对比

| 特性            | TCP (传输控制协议)                            | UDP (用户数据报协议)                  |
| :-------------- | :-------------------------------------------- | :------------------------------------ |
| **连接方式**    | **面向连接** (需 `listen`/`connect`/`accept`) | **无连接** (直接 `sendto`)            |
| **可靠性**      | **可靠** (保证数据不丢失、不重复、按序到达)   | **不可靠** (只管发，不管对方收没收到) |
| **传输速度**    | 较慢 (因为有握手、确认、重传等机制)           | **极快** (头部开销小，无连接状态)     |
| **数据形式**    | 字节流 (Stream)                               | 数据报 (Datagram)                     |
| **典型应用**    | HTTP/HTTPS, FTP, SMTP (网页、文件、邮件)      | 视频流、语音通话、在线游戏、DNS       |
| **Python 方法** | `send()`, `recv()`                            | `sendto()`, `recvfrom()`              |

### 5. 问题

1. **端口被占用**： 如果你修改代码后重新运行服务器报错 `OSError: [Errno 48] Address already in use`，是因为旧的连接处于 `TIME_WAIT` 状态。
   - **解决方法**：在 `bind` 前加上 `server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)`。
2. **粘包问题（TCP 特有）**： TCP 是流协议，它不保证你发送的 `send("abc")` 和 `send("def")` 会被对方分两次 `recv` 到，对方可能会一次性收到 `abcdef`。
   - **解决方法**：在应用层定义协议，比如在数据包头加上长度信息，或者使用固定的分隔符。
3. **阻塞模式**： 默认情况下，`accept()` 和 `recv()` 都是阻塞的。如果想要同时处理多个客户端且不卡住，可以使用 **多线程**（每个连接一个线程）或者 **异步 I/O**（如 `select` 或 `asyncio`）。