$$
R∈R^{m×n}
$$



# 快速上手

```python
import multiprocessing as mp
import time

# 1. 定义一个任务函数
def worker(name):
    print(f"我是进程 {name}，我开始干活了")
    time.sleep(2)  # 模拟干活耗时
    print(f"我是进程 {name}，我干完了")

if __name__ == '__main__':
    # 2. 创建两个进程
    p1 = mp.Process(target=worker, args=("张三",))
    p2 = mp.Process(target=worker, args=("李四",))

    # 3. 启动进程
    p1.start()
    p2.start()

    # 4. 等待结束
    p1.join()
    p2.join()
    
    print("主进程：所有工人都干完了，收工！")
```

### **核心概念：**

死磕这三个概念，它们是 `multiprocessing` 的骨架：

#### 1. `Process` 类（造人的模具）

- `target`：你要让这个人干什么活（函数名）。
- `args`：给这个人的工具包（参数，记得加逗号变成元组）。
- `start()`：喊这个人开始干活。
- `join()`：老板在旁边等着，直到这个人干完活才继续下一步。

#### 2. `Pool` 进程池（招工队）

如果你要招 100 个工人，一个一个用 `Process` 去招太慢了，这时候用 `Pool`。

```python
if __name__ == '__main__':
    # 创建一个有4个工人的池子
    with mp.Pool(processes=4) as pool:
        # 让这4个工人轮流干10件事
        pool.map(worker, [f"任务{i}" for i in range(10)])
    print("全部完成")
```

- `pool.map(func, iterable)`：阻塞式，返回结果列表。
- `pool.apply_async(func, args)`：非阻塞，返回 `AsyncResult` 对象。
- `pool.starmap(func, iterable_of_tuples)`：类似 `map`，但支持多参数。



#### 3. `Queue` 队列（传话筒）

进程之间默认是“老死不相往来”的（内存隔离）。如果张三算出了结果想告诉李四，必须通过 `Queue`。

```python
def producer(q):
    q.put("张三说：算完了！")

def consumer(q):
    msg = q.get() # 这里会阻塞等待，直到有人放消息进来
    print(msg)

if __name__ == '__main__':
    q = mp.Queue() # 创建一个公共传话筒
    p1 = mp.Process(target=producer, args=(q,))
    p2 = mp.Process(target=consumer, args=(q,))
    p1.start(); p2.start()
    p1.join(); p2.join()
```





# pickle与unpickle

`pickle` 和 `unpickle` 是 Python 中用于**对象序列化与反序列化**的核心机制，主要用于将 Python 对象转换为字节流（保存或传输），再从字节流还原为原始对象。

Pickle 生成的是二进制数据，不像 JSON 那样可读。不适合做配置文件。

| 术语                 | 含义                                                         |
| -------------------- | ------------------------------------------------------------ |
| **Pickle（腌制）**   | 将 Python 对象 **序列化** 为字节流（`bytes`），以便存储到文件、通过网络发送等。 |
| **Unpickle（解腌）** | 将字节流 **反序列化** 回原来的 Python 对象。                 |

> 📦 类比：
>
> - Pickle ≈ 把活鱼冷冻成冰块（便于运输）
> - Unpickle ≈ 把冰块解冻回活鱼（恢复原状）

## 基本用法

1. 使用 `pickle.dump()` 和 `pickle.load()`（操作文件）
   ```python
   import pickle
   
   # 要保存的对象
   data = {'name': 'Alice', 'scores': [90, 85, 95]}
   
   # Pickle：写入文件
   with open('data.pkl', 'wb') as f:
       pickle.dump(data, f)
   
   # Unpickle：从文件读取
   with open('data.pkl', 'rb') as f:
       loaded_data = pickle.load(f)
   
   print(loaded_data)  # {'name': 'Alice', 'scores': [90, 85, 95]}
   ```

2. 使用 `pickle.dumps()` 和 `pickle.loads()`（操作字节）
   ```python
   import pickle
   
   obj = [1, 2, {'a': 3}]
   
   # 序列化为 bytes
   serialized = pickle.dumps(obj)
   print(type(serialized))  # <class 'bytes'>
   
   # 反序列化回对象
   restored = pickle.loads(serialized)
   print(restored)  # [1, 2, {'a': 3}]
   ```



## 为什么需要 Pickle？

 ✅ 典型应用场景：

1. **保存程序状态**（如机器学习模型、缓存数据）
2. **进程间通信**（`multiprocessing` 传递函数/参数时自动使用 pickle）
3. **分布式计算**（如通过网络发送任务和数据）
4. **临时存储复杂对象**（比 JSON 更强大，支持自定义类）

✅ 支持大多数 Python 内置类型和自定义对象：

- 基本类型：`int`, `str`, `list`, `dict`, `tuple`, `set`
- 函数（顶层定义的）、类、类实例（需可导入）
- 递归结构、共享引用（会正确保留）

❌ **不支持**：

- Lambda 函数
- 嵌套函数（闭包）
- 模块、文件句柄、线程锁等运行时资源
- 未在顶层定义的类或函数（无法被 `import`）

> ⚠️ 注意：对象必须能被 **`import` 到相同名称空间**，否则 unpickle 会失败。

## 在多进程中的关键作用

当你使用 `multiprocessing.Process` 或 `ProcessPoolExecutor` 时，**传递给子进程的函数和参数会自动被 pickle**，然后在子进程中 unpickle。

这就是为什么：

- 不能传 lambda 函数（无法 pickle）
- 自定义函数必须在模块顶层定义（子进程要能 import）

```python
from concurrent.futures import ProcessPoolExecutor

# 必须在顶层定义！
def square(x):
    return x * x

# ✅ 可以被 pickle
with ProcessPoolExecutor() as executor:
    result = list(executor.map(square, [1, 2, 3]))
```

| 操作                    | 函数                               | 用途           |
| ----------------------- | ---------------------------------- | -------------- |
| 序列化（对象 → 字节）   | `pickle.dumps()` / `pickle.dump()` | 保存或传输对象 |
| 反序列化（字节 → 对象） | `pickle.loads()` / `pickle.load()` | 恢复对象       |





# 概述

## 基本概述

[multiprocessing --- 基于进程的并行 — Python 3.14.2 文档](https://docs.python.org/zh-cn/3/library/multiprocessing.html#programming-guidelines)

一个简单示例：显示所涉及的各个进程ID

```python
from multiprocessing import Process
import os

def info(title):
    print(title)
    print('module name:', __name__)
    print('parent process:', os.getppid())
    print('process id:', os.getpid())

def f(name):
    info('function f')
    print('hello', name)

if __name__ == '__main__':
    info('main line')
    p = Process(target=f, args=('bob',))
    p.start()
    p.join()
```

-  `if __name__ == '__main__'` 是必须的，在Windows平台上防止递归无限创造进程

### 上下文和启动方法

根据不同的平台， [`multiprocessing`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#module-multiprocessing) 支持三种启动进程的方法。这些 *启动方法* 有：

- *spawn*

  **python默认方式**。父进程会启动一个新的 Python 解释器进程。 子进程将只继承那些运行进程对象的 [`run()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Process.run) 方法所必须的资源。 特别地，来自父进程的非必需文件描述符和句柄将不会被继承。 使用此方法启动进程相比使用 *fork* 或 *forkserver* 要慢上许多。在 POSIX 和 Windows 平台上可用。 默认在 Windows 和 macOS 上。

- *fork*

  父进程使用 [`os.fork()`](https://docs.python.org/zh-cn/3/library/os.html#os.fork) 来产生 Python 解释器分叉。子进程在开始时实际上与父进程相同。父进程的所有资源都由子进程继承。请注意，安全分叉多线程进程是棘手的。

- *forkserver*

  当程序启动并选择 *forkserver* 启动方法时，将产生一个服务器进程。 从那时起，每当需要一个新进程时，父进程就会连接到该服务器并请求它分叉一个新进程。 分叉服务器进程是单线程的，除非因系统库或预加载导入的附带影响改变了这一点，因此使用 [`os.fork()`](https://docs.python.org/zh-cn/3/library/os.html#os.fork) 通常是安全的。 没有不必要的资源被继承。

  

要选择一个启动方法:

- （**不推荐**）在主模块的 `if __name__ == '__main__'` 中调用 - - [`set_start_method()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.set_start_method) 。
  ```python 
  import multiprocessing as mp
  
  def foo(q):
      q.put('hello')
  
  if __name__ == '__main__':
      mp.set_start_method('spawn')
      q = mp.Queue()
      p = mp.Process(target=foo, args=(q,))
      p.start()
      print(q.get())
      p.join()
  ```

  **在程序中 [`set_start_method()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.set_start_method) 不应该被多次调用。**



- （**推荐**）或者使用 [`get_context()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.get_context) 来获取上下文对象。上下文对象与 multiprocessing 模块具有相同的API，并允许在同一程序中使用多种启动方法。
  ```python
  import multiprocessing as mp
  
  def foo(q):
      q.put('hello')
  
  if __name__ == '__main__':
      ctx = mp.get_context('spawn')
      q = ctx.Queue()
      p = ctx.Process(target=foo, args=(q,))
      p.start()
      print(q.get())
      p.join()
  ```

  请注意，对象在不同上下文创建的进程间可能并不兼容。 特别是，使用 *fork* 上下文创建的锁不能传递给使用 *spawn* 或 *forkserver* 启动方法启动的进程。



### 在进程之间交换对象

[`multiprocessing`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#module-multiprocessing) 支持进程之间的两种通信通道：

- **队列**
  [`Queue`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Queue) 类是一个近似 [`queue.Queue`](https://docs.python.org/zh-cn/3/library/queue.html#queue.Queue) 的克隆。 

  ```python
  from multiprocessing import Process, Queue
  
  def f(q):
      q.put([42, None, 'hello'])
  
  if __name__ == '__main__':
      q = Queue()
      p = Process(target=f, args=(q,))
      p.start()
      print(q.get())    # 打印 "[42, None, 'hello']"
      p.join()
  ```

  队列是线程和进程安全的。 任何放入 [`multiprocessing`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#module-multiprocessing) 队列的对象都将被序列化。

- **管道**
  [`Pipe()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Pipe) 函数返回一个由管道连接的连接对象，默认情况下是双工（双向）。例如:

  ```python
  from multiprocessing import Process, Pipe
  
  def f(conn):
      conn.send([42, None, 'hello'])
      conn.close()
  
  if __name__ == '__main__':
      parent_conn, child_conn = Pipe()
      p = Process(target=f, args=(child_conn,))
      p.start()
      print(parent_conn.recv())   # 打印 "[42, None, 'hello']"
      p.join()
  ```

  返回的两个连接对象 [`Pipe()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Pipe) 表示管道的两端。每个连接对象都有 `send()` 和 `recv()` 方法（相互之间的）。请注意，如果两个进程（或线程）同时尝试读取或写入管道的 *同一* 端，则管道中的数据可能会损坏。当然，在不同进程中同时使用管道的不同端的情况下不存在损坏的风险。

  `send()` 方法将序列化对象而 `recv()` 将重新创建对象。



### 进程间同步

[`multiprocessing`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#module-multiprocessing) 包含来自 [`threading`](https://docs.python.org/zh-cn/3/library/threading.html#module-threading) 的所有同步原语的等价物。例如，可以使用锁来确保一次只有一个进程打印到标准输出:

```python
from multiprocessing import Process, Lock

def f(l, i):
    l.acquire()
    try:
        print('hello world', i)
    finally:
        l.release()

if __name__ == '__main__':
    lock = Lock()

    for num in range(10):
        Process(target=f, args=(lock, num)).start()
```

不使用锁的情况下，来自于多进程的输出很容易产生混淆。



### 进程间共享状态

在进行并发编程时，通常最好尽量避免使用共享状态。使用多个进程时尤其如此。但是，如果你真的需要使用一些共享数据，那么 [`multiprocessing`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#module-multiprocessing) 提供了两种方法。

- **共享内存**
  可以使用 [`Value`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Value) 或 [`Array`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Array) 将数据存储在共享内存映射中。例如，以下代码:

  ```python
  from multiprocessing import Process, Value, Array
  
  def f(n, a):
      n.value = 3.1415927
      for i in range(len(a)):
          a[i] = -a[i]
  
  if __name__ == '__main__':
      num = Value('d', 0.0)
      arr = Array('i', range(10))
  
      p = Process(target=f, args=(num, arr))
      p.start()
      p.join()
  
      print(num.value)
      print(arr[:])
  ```

  创建 `num` 和 `arr` 时使用的 `'d'` 和 `'i'` 参数是 [`array`](https://docs.python.org/zh-cn/3/library/array.html#module-array) 模块使用的类型的 typecode ： `'d'` 表示双精度浮点数， `'i'` 表示有符号整数。这些共享对象将是进程和线程安全的。

- **服务进程**
  由 [`Manager()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Manager) 返回的管理器对象控制一个服务进程，该进程保存Python对象并允许其他进程使用代理操作它们。

  [`Manager()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Manager) 返回的管理器支持类型： [`list`](https://docs.python.org/zh-cn/3/library/stdtypes.html#list) 、 [`dict`](https://docs.python.org/zh-cn/3/library/stdtypes.html#dict) 、 [`set`](https://docs.python.org/zh-cn/3/library/stdtypes.html#set)、[`Namespace`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.managers.Namespace) 、 [`Lock`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Lock) 、 [`RLock`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.RLock) 、 [`Semaphore`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Semaphore) 、 [`BoundedSemaphore`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.BoundedSemaphore) 、 [`Condition`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Condition) 、 [`Event`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Event) 、 [`Barrier`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Barrier) 、 [`Queue`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Queue) 、 [`Value`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Value) 和 [`Array`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Array)

  ```python
  from multiprocessing import Process, Manager
  
  def f(d, l, s):
      d[1] = '1'
      d['2'] = 2
      d[0.25] = None
      l.reverse()
      s.add('a')
      s.add('b')
  
  if __name__ == '__main__':
      with Manager() as manager:
          d = manager.dict()
          l = manager.list(range(10))
          s = manager.set()
  
          p = Process(target=f, args=(d, l, s))
          p.start()
          p.join()
  
          print(d)
          print(l)
          print(s)
  ```

  将打印：
  ```python
  {0.25: None, 1: '1', '2': 2}
  [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
  {'a', 'b'}
  ```

  

### 使用工作进程

[`Pool`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.Pool) 类表示一个工作进程池。它具有允许以几种不同方式将任务分配到工作进程的方法。

```python
from multiprocessing import Pool, TimeoutError
import time
import os

def f(x):
    return x*x

if __name__ == '__main__':
    # 启动 4 个工作进程
    with Pool(processes=4) as pool:

        # 打印 "[0, 1, 4,..., 81]"
        print(pool.map(f, range(10)))

        # 以任意顺序打印同样的数字
        for i in pool.imap_unordered(f, range(10)):
            print(i)

        # 异步地对 "f(20)" 求值
        res = pool.apply_async(f, (20,))      # runs in *only* one process
        print(res.get(timeout=1))             # prints "400"

        # 异步地对 "os.getpid()" 求值
        res = pool.apply_async(os.getpid, ()) # *仅在* 一个进程中运行
        print(res.get(timeout=1))             # 打印进程的 PID

        # 异步地进行多次求值 *可能* 会使用更多进程
        multiple_results = [pool.apply_async(os.getpid, ()) for i in range(4)]
        print([res.get(timeout=1) for res in multiple_results])

        # 让一个工作进程休眠 10 秒
        res = pool.apply_async(time.sleep, (10,))
        try:
            print(res.get(timeout=1))
        except TimeoutError:
            print("We lacked patience and got a multiprocessing.TimeoutError")

        print("For the moment, the pool remains available for more work")

    # 退出 'with' 代码块将停止进程池
    print("Now the pool is closed and no longer available")
```

- map方法会阻塞主进程
  - 把 `range(10)` 分发给多个 worker 并行计算；
  - **等待所有结果都回来**；
  - **按输入顺序组装成一个完整列表**；
  - **一次性返回给主进程**。
- AsyncResult.get方法会阻塞主进程，**等待该任务完成，并返回结果**。
  - **任务提交是并发的**
  - **结果获取是串行等待的**，即使子进程的执行时间不同



## 参考

### Process 和异常

```python
class multiprocessing.Process(group=None, target=None, name=None, args=(), kwargs={}, *, daemon=None)
```

构造器被调用时应当总是传入关键字参数。

- *group* 应当始终为 `None`；它的存在仅是为了与 [`threading.Thread`](https://docs.python.org/zh-cn/3/library/threading.html#threading.Thread) 兼容。
-  *target* 是由 [`run()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Process.run) 方法来唤起的可调用对象。 它的默认值为 `None`，表示不调用任何东西。可以传入函数名作为工作函数
-  *name* 是进程名称（请参阅 [`name`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Process.name) 了解详情）。
-  *args* 是针对目标调用的参数元组。
-  *kwargs* 是针对目标调用的关键字参数字典。 如果提供，则仅限关键字参数 *daemon* 会将进程的 [`daemon`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Process.daemon) 旗标设为 `True` 或 `False`。 如果为 `None` (默认值)，则该旗标将从创建方进程继承。

通常情况下，传递给 [`Process`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Process) 类的所有参数都必须是可序列化（picklable）的。

#### 内部方法与属性

- **run()**
  表示进程活动的方法。
  标准 [`run()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Process.run) 方法调用传递给对象构造函数的可调用对象作为目标参数（如果有），分别从 *args* 和 *kwargs* 参数中获取顺序和关键字参数。

  你可以在子类中重写此方法。继承 `multiprocessing.Process` 类创建自定义进程时，**重写 `run()` 方法**就相当于“告诉进程：你启动后该做什么”。

  也可以不继承 `Process`，而是用 `target` 参数传入函数：`p = mp.Process(target=os.getpid(), args=())`

- **start**()
  启动进程活动。
  这个方法每个进程对象最多只能调用一次。它会将对象的`run()`方法安排在一个单独的进程中调用。

- **join**([*timeout*])
  如果可选参数 *timeout* 是 `None` （默认值），则该方法将阻塞，直到调用 [`join()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Process.join) 方法的进程终止。如果 *timeout* 是一个正数，它最多会阻塞 *timeout* 秒。请注意，如果进程终止或方法超时，则该方法返回 `None`

  一个进程可以被 join 多次。

  进程无法join自身，因为这会导致死锁。尝试在启动进程之前join进程是错误的。

- **name**
  进程的名称。该名称是一个字符串，仅用于识别目的。它没有语义。可以为多个进程指定相同的名称。
  初始名称由构造器设定。 如果没有为构造器提供显式名称，则会构造一个形式为 'Process-N1:N2:...:Nk' 的名称，其中每个 Nk 是其父亲的第 N 个孩子。

- **is_alive**()
  返回进程是否还活着。

- **pid**
  返回进程ID。在生成该进程之前，这将是 `None` 。

- **exitcode**
  子进程的退出代码。如果该进程尚未终止则为 `None` 。

  如果子进程的 [`run()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Process.run) 方法正常返回，退出代码将是 0 。 如果它通过 [`sys.exit()`](https://docs.python.org/zh-cn/3/library/sys.html#sys.exit) 终止，并有一个整数参数 *N* ，退出代码将是 *N* 。

- **terminate**()
  终结进程。 

  请注意，进程的后代进程将不会被终止 —— 它们将简单地变成孤立的。

  警告: 如果在关联进程使用管道或队列时使用此方法，则管道或队列可能会损坏，并可能无法被其他进程使用。类似地，如果进程已获得锁或信号量等，则终止它可能导致其他进程死锁。

- **kill**()
  与*terminate*相同

- **close**()
  关闭 [`Process`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Process) 对象，释放与之关联的所有资源。如果底层进程仍在运行，则会引发 [`ValueError`](https://docs.python.org/zh-cn/3/library/exceptions.html#ValueError) 。一旦 [`close()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Process.close) 成功返回， [`Process`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Process) 对象的大多数其他方法和属性将引发 [`ValueError`](https://docs.python.org/zh-cn/3/library/exceptions.html#ValueError) 。

注意 [`start()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Process.start) 、 [`join()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Process.join) 、 [`is_alive()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Process.is_alive) 、 [`terminate()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Process.terminate) 和 [`exitcode`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Process.exitcode) 方法只能由创建进程对象的进程(父进程)调用。



### 管道和队列

使用多进程时，一般使用消息机制实现进程间通信，尽可能避免使用同步原语，例如锁。

消息机制包含：**Pipe()**(可以用于在两个进程间传递消息)，以及**队列**(能够在多个生产者和消费者之间通信)。

#### multiprocessing.**Pipe**(*duplex=True*)

返回一对 [`Connection`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.connection.Connection) 对象 `(conn1, conn2)` ， 分别表示管道的两端。

如果 *duplex* 被置为 `True` (默认值)，那么该管道是双向的。如果 *duplex* 被置为 `False` ，那么该管道是单向的，即 `conn1` 只能用于接收消息，而 `conn2` 仅能用于发送消息。



#### multiprocessing.**Queue**([*maxsize*])

返回一个使用一个管道和少量锁和信号量实现的共享队列实例。当一个进程将一个对象放进队列中时，一个写入线程会启动并将对象从缓冲区写入管道中。

一旦超时，将抛出标准库 [`queue`](https://docs.python.org/zh-cn/3/library/queue.html#module-queue) 模块中常见的异常 [`queue.Empty`](https://docs.python.org/zh-cn/3/library/queue.html#queue.Empty) 和 [`queue.Full`](https://docs.python.org/zh-cn/3/library/queue.html#queue.Full)。

- **qsize**()

  返回队列的大致长度。由于多线程或者多进程的上下文，这个数字是不可靠的。

- **empty**()

  如果队列是空的，返回 `True` ，反之返回 `False` 。 由于多线程或多进程的环境，该状态是不可靠的。在已关闭的队列上可能会引发 [`OSError`](https://docs.python.org/zh-cn/3/library/exceptions.html#OSError)。 （但不保证如此）

- **full**()

  如果队列是满的，返回 `True` ，反之返回 `False` 。 由于多线程或多进程的环境，该状态是不可靠的。

- **put**(*obj*[, *block*[, *timeout*]])

  - 将 obj 放入队列。

  - 如果可选参数 *block* 是 `True` (默认值) 而且 *timeout* 是 `None` (默认值), 将会阻塞当前进程，直到有空的缓冲槽。

  - 如果 *timeout* 是正数，将会在阻塞了最多 *timeout* 秒之后还是没有可用的缓冲槽时抛出 [`queue.Full`](https://docs.python.org/zh-cn/3/library/queue.html#queue.Full) 异常。

  - 反之 (*block* 是 `False` 时)，仅当有可用缓冲槽时才放入对象，否则抛出 [`queue.Full`](https://docs.python.org/zh-cn/3/library/queue.html#queue.Full) 异常 (在这种情形下 *timeout* 参数会被忽略)。

- **put_nowait**(*obj*)

  相当于 `put(obj, False)`。

- **get**([*block*[, *timeout*]])

  - 从队列中取出并返回对象。

  - 如果可选参数 *block* 是 `True` (默认值) 而且 *timeout* 是 `None` (默认值), 将会阻塞当前进程，直到队列中出现可用的对象。

  - 如果 *timeout* 是正数，将会在阻塞了最多 *timeout* 秒之后还是没有可用的对象时抛出 [`queue.Empty`](https://docs.python.org/zh-cn/3/library/queue.html#queue.Empty) 异常。

  - 反之 (*block* 是 `False` 时)，仅当有可用对象能够取出时返回，否则抛出 [`queue.Empty`](https://docs.python.org/zh-cn/3/library/queue.html#queue.Empty) 异常 (在这种情形下 *timeout* 参数会被忽略)。

- **get_nowait**()

  相当于 `get(False)` 。

[`multiprocessing.Queue`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Queue) 类有一些在 [`queue.Queue`](https://docs.python.org/zh-cn/3/library/queue.html#queue.Queue) 类中没有出现的方法。这些方法在大多数情形下并不是必须的。

- **close**()

  关闭队列：释放内部资源。队列在被关闭后就不可再被使用。 例如不可再调用 [`get()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Queue.get)、[`put()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Queue.put) 和 [`empty()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Queue.empty) 等方法。后台线程会在将所有缓冲数据刷新到管道后退出。当队列被垃圾回收时，此操作会自动执行。

- **join_thread**()

  等待后台线程。这个方法仅在调用了 [`close()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Queue.close) 方法之后可用。这会阻塞当前进程，直到后台线程退出，确保所有缓冲区中的数据都被写入管道中。默认情况下，如果一个不是队列创建者的进程试图退出，它会尝试等待这个队列的后台线程。这个进程可以使用 [`cancel_join_thread()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Queue.cancel_join_thread) 让 [`join_thread()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Queue.join_thread) 方法什么都不做直接跳过。

- **cancel_join_thread**()

  防止 [`join_thread()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Queue.join_thread) 方法阻塞当前进程。具体而言，这防止进程退出时自动等待后台线程退出。详见 [`join_thread()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Queue.join_thread)。这个方法更好的名字可能是 `allow_exit_without_flush()`。 这可能会导致已排入队列的数据丢失，几乎可以肯定你将不需要用到这个方法。 实际上它仅适用于当你需要当前进程立即退出而不必等待将已排入的队列更新到下层管道，并且你不担心丢失数据的时候。

### 杂项

- multiprocessing.**active_children**()

  返回当前进程存活的子进程的列表。调用该方法有“等待”已经结束的进程的副作用。

- multiprocessing.**cpu_count**()

  返回系统的CPU数量。

  该数值不等于当前进程可使用的 CPU 数量。 可用的 CPU 数量可以通过 [`os.process_cpu_count()`](https://docs.python.org/zh-cn/3/library/os.html#os.process_cpu_count) (或 `len(os.sched_getaffinity(0))`) 获得。

  当 CPU 的数量无法确定时，会引发 [`NotImplementedError`](https://docs.python.org/zh-cn/3/library/exceptions.html#NotImplementedError) 。

- multiprocessing.**current_process**()

  返回与当前进程相对应的 [`Process`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Process) 对象。和 [`threading.current_thread()`](https://docs.python.org/zh-cn/3/library/threading.html#threading.current_thread) 相同。

- multiprocessing.**parent_process**()

  返回父进程 [`Process`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Process) 对象，和父进程调用 [`current_process()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.current_process) 返回的对象一样。如果一个进程已经是主进程， `parent_process` 会返回 `None`.

- multiprocessing.**get_context**(*method=None*)

  返回一个 Context 对象。该对象具有和 [`multiprocessing`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#module-multiprocessing) 模块相同的API。

  如果 *method* 为 `None` 则将返回默认的上下文。 需要注意的是，如果全局启动方法尚未设置，此操作会将其设置为默认方法。 否则 *method* 应为 `'fork'`, `'spawn'`, `'forkserver'`。

- multiprocessing.**set_start_method**(*method*, *force=False*)

  设置应当被用于启动子进程的方法。 *method* 方法可以为 `'fork'`, `'spawn'` 或 `'forkserver'`。

  注意这最多只能调用一次，并且需要藏在 main 模块中，由 `if __name__ == '__main__'` 保护着。

- multiprocessing.**get_start_method**(*allow_none=False*)

  返回启动进程时使用的启动方法名。若全局启动方法尚未设置且 *allow_none* 参数为 `False`，则会将启动方法设为默认值并返回方法名称；若全局启动方法尚未设置但 *allow_none* 参数为 `True`，则直接返回 `None`。



### 连接对象（Connection）

Connection 对象允许收发可以序列化的对象或字符串。它们可以看作面向消息的连接套接字。

通常使用 [`Pipe`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Pipe) 创建 Connection 对象。

#### class multiprocessing.connection.**Connection**

- **send**(*obj*)

  将一个对象发送到连接的另一端，可以用 [`recv()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.connection.Connection.recv) 读取。发送的对象必须是可以序列化的，过大的对象 ( 接近 32MiB+ ，这个值取决于操作系统 ) 有可能引发 [`ValueError`](https://docs.python.org/zh-cn/3/library/exceptions.html#ValueError) 异常。

- **recv**()

  返回一个由另一端使用 [`send()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.connection.Connection.send) 发送的对象。**该方法会一直阻塞直到接收到对象**。 如果对端关闭了连接或者没有东西可接收，将抛出 [`EOFError`](https://docs.python.org/zh-cn/3/library/exceptions.html#EOFError) 异常。

- **fileno**()

  返回由连接对象使用的描述符或者句柄。

- **close**()

  关闭连接对象。当连接对象被垃圾回收时会自动调用。

- **poll**([*timeout*])

  返回连接对象中是否有可以读取的数据。如果未指定 *timeout* ，此方法会马上返回。如果 *timeout* 是一个数字，则指定了最大阻塞的秒数。如果 *timeout* 是 `None`，那么将一直等待，不会超时。

### 共享 ctypes对象

- multiprocessing.**Value**(*typecode_or_type*, **args*, *lock=True*)

  返回一个从共享内存上创建的 [`ctypes`](https://docs.python.org/zh-cn/3/library/ctypes.html#module-ctypes) 对象。默认情况下返回的对象实际上是经过了同步器包装过的。

  可以通过 [`Value`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Value) 的 *value* 属性访问这个对象本身。

  *typecode_or_type* 指明了返回的对象类型: 它可能是一个 ctypes 类型或者 [`array`](https://docs.python.org/zh-cn/3/library/array.html#module-array) 模块中每个类型对应的单字符长度的字符串。 

  **args* 会透传给这个类的构造函数。如果 *lock* 参数是 `True` （默认值）, 将会新建一个递归锁用于同步对于此值的访问操作。

   如果 *lock* 是 [`Lock`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Lock) 或者 [`RLock`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.RLock) 对象，那么这个传入的锁将会用于同步对这个值的访问操作，如果 *lock* 是 `False` , 那么对这个对象的访问将没有锁保护，也就是说这个变量**不是进程安全**的。

- multiprocessing.**Array**(*typecode_or_type*, *size_or_initializer*, ***, *lock=True*)

  从共享内存中申请并返回一个具有ctypes类型的数组对象。默认情况下返回值实际上是被同步器包装过的数组对象。

  *typecode_or_type* 指明了返回的数组中的元素类型: 它可能是一个 ctypes 类型或者 [`array`](https://docs.python.org/zh-cn/3/library/array.html#module-array) 模块中每个类型对应的单字符长度的字符串。

   如果 *size_or_initializer* 是一个整数，那就会当做数组的长度，并且整个数组的内存会初始化为0。否则，如果 *size_or_initializer* 会被当成一个序列用于初始化数组中的每一个元素，并且会根据元素个数自动判断数组的长度。

  如果 *lock* 为 `True` (默认值) 则将创建一个新的锁对象用于同步对值的访问。 如果 *lock* 为一个 [`Lock`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.Lock) 或 [`RLock`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.RLock) 对象则该对象将被用于同步对值的访问。 如果 *lock* 为 `False` 则对返回对象的访问将不会自动得到锁的保护，也就是说它**不是“进程安全的”**。



### 进程池

可以创建一个进程池，它将使用 [`Pool`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.Pool) 类执行提交给它的任务。

- class multiprocessing.pool.**Pool**([*processes*[, *initializer*[, *initargs*[, *maxtasksperchild*[, *context*]]]]])
  一个进程池对象，它控制可以提交作业的工作进程池。它支持带有超时和回调的异步结果，以及一个并行的 map 实现。

  *processes* 是要使用的工作进程数量。 如果 *processes* 为 `None` 则使用 [`os.process_cpu_count()`](https://docs.python.org/zh-cn/3/library/os.html#os.process_cpu_count) 所返回的数值。

  注意，进程池对象的方法只有创建它的进程能够调用。

  - **apply**(*func*[, *args*[, *kwds*]])

    使用 *args* 参数以及 *kwds* 命名参数调用 *func* , 它会返回结果前阻塞。这种情况下，[`apply_async()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.Pool.apply_async) 更适合并行化工作。另外 *func* 只会在一个进程池中的一个工作进程中执行。

  - **apply_async**(*func*[, *args*[, *kwds*[, *callback*[, *error_callback*]]]])

    [`apply()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.Pool.apply) 方法的一个变种，返回一个 [`AsyncResult`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.AsyncResult) 对象。

    如果指定了 *callback* , 它必须是一个接受单个参数的可调用对象。当执行成功时， *callback* 会被用于处理执行后的返回结果，否则，调用 *error_callback* 。如果指定了 *error_callback* , 它必须是一个接受单个参数的可调用对象。当目标函数执行失败时， 会将抛出的异常对象作为参数传递给 *error_callback* 执行。回调函数应该立即执行完成，否则会阻塞负责处理结果的线程。

  - **map**(*func*, *iterable*[, *chunksize*])

    内置 [`map()`](https://docs.python.org/zh-cn/3/library/functions.html#map) 函数的并行版本 (但它只支持一个 *iterable* 参数，对于多个可迭代对象请参阅 [`starmap()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.Pool.starmap))。 **它会保持阻塞直到获得结果**。这个方法会将可迭代对象分割为许多块，然后提交给进程池。

     可以将 *chunksize* 设置为一个正整数来指定每个块的（近似）大小。注意对于很长的迭代对象，可能消耗很多内存。

    可以考虑使用 [`imap()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.Pool.imap) 或 [`imap_unordered()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.Pool.imap_unordered) 并且显式指定 *chunksize* 以提升效率。

  - **map_async**(*func*, *iterable*[, *chunksize*[, *callback*[, *error_callback*]]])

    [`map()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.Pool.map) 方法的一个变种，返回一个 [`AsyncResult`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.AsyncResult) 对象。

    如果指定了 *callback* , 它必须是一个接受单个参数的可调用对象。当执行成功时， *callback* 会被用于处理执行后的返回结果，否则，调用 *error_callback* 。如果指定了 *error_callback* , 它必须是一个接受单个参数的可调用对象。当目标函数执行失败时， 会将抛出的异常对象作为参数传递给 *error_callback* 执行。回调函数应该立即执行完成，否则会阻塞负责处理结果的线程。

  - **imap**(*func*, *iterable*[, *chunksize*])

    [`map()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.Pool.map) 的延迟执行版本。

    *chunksize* 参数的作用和 [`map()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.Pool.map) 方法的一样。对于很长的迭代器，给 *chunksize* 设置一个很大的值会比默认值 `1` **极大** 地加快执行速度。

    同样，如果 *chunksize* 是 `1` , 那么 [`imap()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.Pool.imap) 方法所返回的迭代器的 `next()` 方法拥有一个可选的 *timeout* 参数： 如果无法在 *timeout* 秒内执行得到结果，则 `next(timeout)` 会抛出 [`multiprocessing.TimeoutError`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.TimeoutError) 异常。

  - **imap_unordered**(*func*, *iterable*[, *chunksize*])

    和 [`imap()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.Pool.imap) 相同，只不过通过迭代器返回的结果是任意的。（当进程池中只有一个工作进程的时候，返回结果的顺序才能认为是"有序"的）

  - **starmap**(*func*, *iterable*[, *chunksize*])

    和 [`map()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.Pool.map) 类似，不过 *iterable* 中的每一项会被解包再作为函数参数。比如可迭代对象 `[(1,2), (3, 4)]` 会转化为等价于 `[func(1,2), func(3,4)]` 的调用。**

  - **starmap_async**(*func*, *iterable*[, *chunksize*[, *callback*[, *error_callback*]]])

    相当于 [`starmap()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.Pool.starmap) 与 [`map_async()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.Pool.map_async) 的结合，迭代 *iterable* 的每一项，解包作为 *func* 的参数并执行，返回用于获取结果的对象。

  - **close**()

    阻止后续任务提交到进程池，当所有任务执行完成后，工作进程会退出。

  - **terminate**()

    不必等待未完成的任务，立即停止工作进程。当进程池对象被垃圾回收时，会立即调用 [`terminate()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.Pool.terminate)。

  - **join**()

    等待工作进程结束。调用 [`join()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.Pool.join) 前必须先调用 [`close()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.Pool.close) 或者 [`terminate()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.Pool.terminate) 。

  

- class multiprocessing.pool.**AsyncResult**

  [`Pool.apply_async()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.Pool.apply_async) 和 [`Pool.map_async()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.Pool.map_async) 返回对象所属的类。

  - **get**([*timeout*])

    用于获取执行结果。如果 *timeout* 不是 `None` 并且在 *timeout* 秒内仍然没有执行完得到结果，则抛出 [`multiprocessing.TimeoutError`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.TimeoutError) 异常。如果远程调用发生异常，这个异常会通过 [`get()`](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.pool.AsyncResult.get) 重新抛出。

    默认timeout=None，会阻塞进程**无限等待**，直到任务完成或发生异常。

  - **wait**([*timeout*])

    阻塞，直到返回结果，或者 *timeout* 秒后超时。

  - **ready**()

    返回执行状态，是否已经完成。

  - **successful**()

    判断调用是否已经完成并且未引发异常。 如果还未获得结果则将引发 [`ValueError`](https://docs.python.org/zh-cn/3/library/exceptions.html#ValueError)。

  ```python
  from multiprocessing import Pool
  import time
  
  def f(x):
      return x*x
  
  if __name__ == '__main__':
      with Pool(processes=4) as pool:         # 启动 4 个工作进程
          result = pool.apply_async(f, (10,)) # 在单个进程中异步地对 "f(10)" 求值
          print(result.get(timeout=1))        # 打印 "100" 除非你的计算机 *非常* 慢
  
          print(pool.map(f, range(10)))       # 打印 "[0, 1, 4,..., 81]"
  
          it = pool.imap(f, range(10))
          print(next(it))                     # 打印 "0"
          print(next(it))                     # 打印 "1"
          print(it.next(timeout=1))           # 打印 "4" 除非你的计算机 *非常* 慢
  
          result = pool.apply_async(time.sleep, (10,))
          print(result.get(timeout=1))        # 引发 multiprocessing.TimeoutError
  ```

  

















---

# 代码解释



## 一、基础概念

### **1. 为什么需要 multiprocessing？**

```python
import multiprocessing

print(f"CPU核心数: {multiprocessing.cpu_count()}")
# 通常输出：4、8、16等（取决于你的CPU）
```

- **绕过 GIL**：每个进程有独立的 Python 解释器和内存空间。
- **利用多核 CPU**：适合 CPU 密集型任务（比如科学计算、图像处理等）。
- **比 threading 更适合并行计算**（但开销更大）。

### **2. 与 threading 的区别**

Python 有一个著名的限制叫 **GIL（Global Interpreter Lock，全局解释器锁）**，它使得 **同一时刻只有一个线程能执行 Python 字节码**。

- **`threading`（多线程）**：受 GIL 限制，即使你有多个 CPU 核心，也无法同时运行多个 Python 线程做计算（只能并发，不能并行）。适用于 I/O 密集型任务。
- **`multiprocessing`（多进程）**：每个进程有**独立的 Python 解释器和内存空间**，因此**每个进程都有自己的 GIL**。这样，多个进程可以**同时在不同的 CPU 核心上运行**，实现真正的并行计算。

>  对于 CPU 密集型任务（如数学计算、图像处理），multiprocessing 是实现真正并行的正确选择。

| 特性         | multiprocessing      | threading        |
| :----------- | :------------------- | :--------------- |
| **并行类型** | 真正的并行（多进程） | 伪并行（多线程） |
| **内存**     | 每个进程独立内存空间 | 共享内存空间     |
| **GIL影响**  | 不受GIL限制          | 受GIL限制        |
| **创建开销** | 较大                 | 较小             |
| **适用场景** | CPU密集型任务        | I/O密集型任务    |

## 二、基本用法

### **1. 创建和启动进程**

python

```python
import multiprocessing
import time
import os

def worker(name, delay):
    """工作函数"""
    print(f"进程 {name} (PID: {os.getpid()}) 开始工作")
    time.sleep(delay)
    print(f"进程 {name} 完成工作")
    return f"{name} 结果"

if __name__ == "__main__":
    # 创建进程
    process1 = multiprocessing.Process(target=worker, args=("A", 2))
    process2 = multiprocessing.Process(target=worker, args=("B", 1))
    
    # 启动进程
    process1.start()
    process2.start()
    
    # 等待进程结束
    process1.join()
    process2.join()
    
    print("所有进程完成")
```



### **2. 使用进程池（推荐）**

python

```python
import multiprocessing
import time

def square(x):
    """计算平方"""
    print(f"计算 {x} 的平方")
    time.sleep(0.5)
    return x * x

if __name__ == "__main__":
    # 创建进程池（默认使用所有CPU核心）
    with multiprocessing.Pool() as pool:
        # 方法1: map - 同步执行
        results = pool.map(square, range(10))
        print(f"map结果: {results}")
        
        # 方法2: map_async - 异步执行
        async_result = pool.map_async(square, range(10, 20))
        print("正在异步计算...")
        print(f"async结果: {async_result.get()}")  # 获取结果
        
        # 方法3: apply - 单个任务同步
        result = pool.apply(square, (25,))
        print(f"apply结果: {result}")
        
        # 方法4: apply_async - 单个任务异步
        async_result = pool.apply_async(square, (30,))
        print(f"apply_async结果: {async_result.get()}")
```



## 三、进程间通信（IPC）

### **1. 队列（Queue）**

python

```python
import multiprocessing
import time

def producer(queue):
    """生产者进程"""
    for i in range(5):
        print(f"生产数据: {i}")
        queue.put(i)
        time.sleep(0.5)
    queue.put(None)  # 结束信号

def consumer(queue):
    """消费者进程"""
    while True:
        item = queue.get()
        if item is None:  # 收到结束信号
            break
        print(f"消费数据: {item}")
        time.sleep(1)

if __name__ == "__main__":
    # 创建队列
    queue = multiprocessing.Queue(maxsize=3)  # 最大容量
    
    # 创建进程
    p1 = multiprocessing.Process(target=producer, args=(queue,))
    p2 = multiprocessing.Process(target=consumer, args=(queue,))
    
    # 启动进程
    p1.start()
    p2.start()
    
    # 等待
    p1.join()
    p2.join()
```



### **2. 管道（Pipe）**

python

```python
import multiprocessing

def sender(conn):
    """发送端"""
    messages = ['Hello', 'World', 'From', 'Process']
    for msg in messages:
        conn.send(msg)
        print(f"发送: {msg}")
    conn.close()

def receiver(conn):
    """接收端"""
    while True:
        try:
            msg = conn.recv()
            print(f"接收: {msg}")
        except EOFError:
            print("连接关闭")
            break

if __name__ == "__main__":
    # 创建管道（返回两个连接对象）
    parent_conn, child_conn = multiprocessing.Pipe()
    
    p1 = multiprocessing.Process(target=sender, args=(child_conn,))
    p2 = multiprocessing.Process(target=receiver, args=(parent_conn,))
    
    p1.start()
    p2.start()
    
    p1.join()
    p2.join()
```



### **3. 共享内存**

python

```python
import multiprocessing
import time

# 使用Value和Array共享数据
def worker1(shared_value, shared_array):
    """进程1修改共享数据"""
    shared_value.value = 3.14
    for i in range(len(shared_array)):
        shared_array[i] = i * 2
    print(f"Worker1: value={shared_value.value}, array={list(shared_array)}")

def worker2(shared_value, shared_array):
    """进程2读取共享数据"""
    time.sleep(0.1)  # 等待worker1先执行
    print(f"Worker2: value={shared_value.value}, array={list(shared_array)}")

if __name__ == "__main__":
    # 创建共享值（'d'表示双精度浮点数）
    shared_value = multiprocessing.Value('d', 0.0)
    
    # 创建共享数组（'i'表示整数）
    shared_array = multiprocessing.Array('i', 5)  # 5个元素的整数数组
    
    p1 = multiprocessing.Process(target=worker1, args=(shared_value, shared_array))
    p2 = multiprocessing.Process(target=worker2, args=(shared_value, shared_array))
    
    p1.start()
    p2.start()
    
    p1.join()
    p2.join()
```



### **4. 管理器（Manager）**

python

```python
import multiprocessing

def worker(shared_dict, shared_list):
    """修改共享数据结构"""
    shared_dict['process_id'] = multiprocessing.current_process().pid
    shared_list.append(multiprocessing.current_process().name)

if __name__ == "__main__":
    # 创建管理器
    with multiprocessing.Manager() as manager:
        # 创建共享数据结构
        shared_dict = manager.dict()
        shared_list = manager.list()
        
        # 创建进程
        processes = []
        for i in range(3):
            p = multiprocessing.Process(
                target=worker, 
                args=(shared_dict, shared_list),
                name=f"Worker-{i}"
            )
            processes.append(p)
            p.start()
        
        # 等待所有进程
        for p in processes:
            p.join()
        
        print(f"共享字典: {shared_dict}")
        print(f"共享列表: {shared_list}")
```



## 四、进程同步

### **1. 锁（Lock）**

python

```python
import multiprocessing
import time

def withdraw(balance, lock):
    """取款"""
    for _ in range(1000):
        with lock:  # 使用上下文管理器自动获取/释放锁
            balance.value -= 1

def deposit(balance, lock):
    """存款"""
    for _ in range(1000):
        with lock:
            balance.value += 1

if __name__ == "__main__":
    # 共享余额和锁
    balance = multiprocessing.Value('i', 1000)
    lock = multiprocessing.Lock()
    
    # 创建进程
    p1 = multiprocessing.Process(target=withdraw, args=(balance, lock))
    p2 = multiprocessing.Process(target=deposit, args=(balance, lock))
    
    p1.start()
    p2.start()
    
    p1.join()
    p2.join()
    
    print(f"最终余额: {balance.value}")  # 应该是1000
```



### **2. 信号量（Semaphore）**

python

```python
import multiprocessing
import time

def worker(semaphore, worker_id):
    """有限资源的工作"""
    with semaphore:
        print(f"Worker {worker_id} 获取资源")
        time.sleep(1)
        print(f"Worker {worker_id} 释放资源")

if __name__ == "__main__":
    # 创建信号量（允许3个进程同时访问）
    semaphore = multiprocessing.Semaphore(3)
    
    # 创建10个进程
    processes = []
    for i in range(10):
        p = multiprocessing.Process(target=worker, args=(semaphore, i))
        processes.append(p)
        p.start()
        time.sleep(0.1)  # 稍微错开启动时间
    
    for p in processes:
        p.join()
```



### **3. 事件（Event）**

python

```python
import multiprocessing
import time

def waiter(event):
    """等待事件"""
    print("等待事件触发...")
    event.wait()  # 阻塞直到事件被设置
    print("事件已触发，继续执行")

def setter(event):
    """设置事件"""
    print("准备触发事件...")
    time.sleep(2)
    event.set()  # 触发事件
    print("事件已触发")

if __name__ == "__main__":
    event = multiprocessing.Event()
    
    p1 = multiprocessing.Process(target=waiter, args=(event,))
    p2 = multiprocessing.Process(target=setter, args=(event,))
    
    p1.start()
    p2.start()
    
    p1.join()
    p2.join()
```



## 五、进程池高级用法

### **1. 带进度的并行处理**

python

```python
import multiprocessing
import time
from tqdm import tqdm  # 需要安装: pip install tqdm

def process_item(item):
    """处理单个项目"""
    time.sleep(0.1)
    return item * item

if __name__ == "__main__":
    items = list(range(100))
    
    # 使用imap获取实时进度
    with multiprocessing.Pool(processes=4) as pool:
        results = []
        # imap保持输入顺序，但返回迭代器
        for result in tqdm(pool.imap(process_item, items), total=len(items)):
            results.append(result)
    
    print(f"处理了 {len(results)} 个项目")
```



### **2. 批量处理大文件**

python

```python
import multiprocessing
import os

def process_file(filename):
    """处理单个文件"""
    try:
        with open(filename, 'r') as f:
            content = f.read()
        # 模拟处理
        processed = content.upper()
        
        # 保存结果
        output_file = f"processed_{os.path.basename(filename)}"
        with open(output_file, 'w') as f:
            f.write(processed)
        
        return f"成功处理: {filename}"
    except Exception as e:
        return f"处理失败 {filename}: {str(e)}"

if __name__ == "__main__":
    # 假设有一批文件需要处理
    file_list = ["file1.txt", "file2.txt", "file3.txt"]  # 替换为实际文件
    
    # 使用进程池并行处理
    with multiprocessing.Pool(processes=2) as pool:
        results = pool.map(process_file, file_list)
    
    for result in results:
        print(result)
```



### **3. 超时控制**

python

```python
import multiprocessing
import time

def long_task(seconds):
    """长时间运行的任务"""
    print(f"开始运行 {seconds} 秒的任务")
    time.sleep(seconds)
    return f"完成 {seconds} 秒任务"

if __name__ == "__main__":
    with multiprocessing.Pool(processes=2) as pool:
        # 提交任务
        async_result = pool.apply_async(long_task, (5,))
        
        try:
            # 设置超时时间为3秒
            result = async_result.get(timeout=3)
            print(f"结果: {result}")
        except multiprocessing.TimeoutError:
            print("任务超时！")
            pool.terminate()  # 终止所有进程
            pool.join()
```



## 六、实际应用示例

### **示例1：并行网页下载**

python

```python
import multiprocessing
import requests
import time

def download_url(url):
    """下载单个URL"""
    try:
        start_time = time.time()
        response = requests.get(url, timeout=5)
        elapsed = time.time() - start_time
        return {
            'url': url,
            'status': response.status_code,
            'size': len(response.content),
            'time': elapsed
        }
    except Exception as e:
        return {
            'url': url,
            'error': str(e)
        }

if __name__ == "__main__":
    urls = [
        'https://www.example.com',
        'https://www.python.org',
        'https://www.github.com',
        'https://www.stackoverflow.com',
    ] * 3  # 重复3次增加任务量
    
    print(f"开始并行下载 {len(urls)} 个URL...")
    
    # 使用4个进程并行下载
    with multiprocessing.Pool(processes=4) as pool:
        results = pool.map(download_url, urls)
    
    # 分析结果
    success = [r for r in results if 'status' in r]
    failed = [r for r in results if 'error' in r]
    
    print(f"成功: {len(success)}, 失败: {len(failed)}")
    if success:
        avg_time = sum(r['time'] for r in success) / len(success)
        print(f"平均下载时间: {avg_time:.2f}秒")
```



### **示例2：并行数据处理管道**

python

```python
import multiprocessing
import time
import random

def stage1_process(data):
    """第一阶段处理"""
    time.sleep(random.uniform(0.1, 0.5))
    return data * 2

def stage2_process(data):
    """第二阶段处理"""
    time.sleep(random.uniform(0.1, 0.3))
    return data + 10

def stage3_process(data):
    """第三阶段处理"""
    time.sleep(random.uniform(0.05, 0.2))
    return data ** 2

def pipeline_worker(input_queue, output_queue, process_func):
    """管道工作进程"""
    while True:
        data = input_queue.get()
        if data is None:  # 结束信号
            output_queue.put(None)  # 传递结束信号
            break
        result = process_func(data)
        output_queue.put(result)

if __name__ == "__main__":
    # 创建队列
    queue1 = multiprocessing.Queue()
    queue2 = multiprocessing.Queue()
    queue3 = multiprocessing.Queue()
    
    # 创建3个阶段的进程
    stage1 = multiprocessing.Process(
        target=pipeline_worker,
        args=(queue1, queue2, stage1_process)
    )
    stage2 = multiprocessing.Process(
        target=pipeline_worker,
        args=(queue2, queue3, stage2_process)
    )
    stage3 = multiprocessing.Process(
        target=pipeline_worker,
        args=(queue3, multiprocessing.Queue(), stage3_process)
    )
    
    # 启动所有进程
    stage1.start()
    stage2.start()
    stage3.start()
    
    # 输入数据
    input_data = list(range(10))
    for data in input_data:
        queue1.put(data)
    
    # 发送结束信号
    queue1.put(None)
    
    # 收集结果
    results = []
    while True:
        result = queue3.get()
        if result is None:
            break
        results.append(result)
    
    # 等待进程结束
    stage1.join()
    stage2.join()
    stage3.join()
    
    print(f"输入: {input_data}")
    print(f"输出: {results}")
```



## 七、最佳实践和注意事项

### **1. 避免共享状态**

python

```python
# ❌ 不好的做法：过多共享状态
def bad_example():
    manager = multiprocessing.Manager()
    shared_data = manager.list()
    # 多个进程频繁修改共享数据 -> 性能差
    
# ✅ 好的做法：最小化共享
def good_example():
    # 每个进程独立处理，最后合并结果
    with multiprocessing.Pool() as pool:
        results = pool.map(process_function, input_data)
    # 主进程汇总结果
```



### **2. 正确的 `if __name__ == '__main__':`**

python

```python
# 必须使用这个保护，否则在Windows/Mac上可能出错
if __name__ == '__main__':
    main()
```



### **3. 资源管理**

python

```python
# 使用with语句确保资源正确释放
with multiprocessing.Pool(processes=4) as pool:
    results = pool.map(func, data)
# 离开with块后pool会自动关闭
```



### **4. 进程数选择**

python

```python
import multiprocessing
import os

def get_optimal_process_count():
    """获取最优进程数"""
    cpu_count = multiprocessing.cpu_count()
    
    # 根据任务类型调整
    if task_is_cpu_intensive:  # CPU密集型
        return cpu_count
    elif task_is_io_intensive:  # I/O密集型
        return min(cpu_count * 2, 20)  # 可以稍多
    else:
        return max(1, cpu_count - 1)  # 保留一个核心给系统
```



### **5. 错误处理**

python

```python
import multiprocessing
import traceback

def safe_worker(data):
    """安全的worker函数"""
    try:
        # 你的处理逻辑
        return process_data(data)
    except Exception as e:
        # 返回错误信息而不是崩溃
        return {
            'error': str(e),
            'traceback': traceback.format_exc(),
            'data': data
        }

if __name__ == "__main__":
    with multiprocessing.Pool() as pool:
        results = pool.map(safe_worker, data_list)
    
    # 检查结果中的错误
    for result in results:
        if 'error' in result:
            print(f"处理出错: {result}")
```



## 八、常见问题解决

### **1. Windows上的问题**

python

```python
# Windows需要将代码放在函数中并通过if __name__判断
def main():
    # 你的代码
    pass

if __name__ == '__main__':
    main()  # 必须通过函数调用
```



### **2. 大型数据传递优化**

python

```python
# 使用共享内存减少数据复制
import numpy as np
import multiprocessing as mp

def process_large_data(shared_array):
    """处理共享内存中的大数据"""python
    # 将共享数组转换为numpy数组（无复制）
    arr = np.frombuffer(shared_array.get_obj(), dtype=np.float32)
    # 直接操作共享内存
    arr *= 2

if __name__ == "__main__":
    # 创建共享数组
    shared_arr = mp.Array('f', 1000000)  # 100万个浮点数
    
    # 初始化数据
    arr = np.frombuffer(shared_arr.get_obj(), dtype=np.float32)
    arr[:] = np.random.randn(1000000)
    
    # 并行处理
    processes = []
    for i in range(4):
        p = mp.Process(target=process_large_data, args=(shared_arr,))
        processes.append(p)
        p.start()
    
    for p in processes:
        p.join()
```



### **3. 调试技巧**

python

```python
import multiprocessing
import logging

def worker():
    """带日志记录的worker"""
    logging.basicConfig(level=logging.DEBUG)
    logger = logging.getLogger(__name__)
    logger.info("Worker started")
    # ... 工作逻辑

if __name__ == "__main__":
    # 启用多进程日志
    multiprocessing.log_to_stderr()
    logger = multiprocessing.get_logger()
    logger.setLevel(logging.INFO)
    
    p = multiprocessing.Process(target=worker)
    p.start()
    p.join()
```



## 总结

`multiprocessing` 是Python中进行并行计算的核心工具，主要特点：

1. **适用场景**：CPU密集型任务、需要绕过GIL限制
2. **核心组件**：
   - `Process`：单个进程
   - `Pool`：进程池（最常用）
   - `Queue`/`Pipe`：进程间通信
   - `Lock`/`Event`/`Semaphore`：进程同步
   - `Value`/`Array`/`Manager`：共享数据
3. **最佳实践**：
   - 优先使用进程池（`Pool`）
   - 最小化进程间通信
   - 使用 `if __name__ == '__main__':` 保护
   - 合理设置进程数量（通常是CPU核心数）
4. **替代方案**：
   - `concurrent.futures.ProcessPoolExecutor`：更简单的接口
   - `joblib`：适用于机器学习任务
   - `dask`：大规模并行计算
   - `ray`：分布式计算框架

根据具体需求选择合适的工具，对于大多数并行任务，`multiprocessing.Pool` 是最简单有效的选择。

