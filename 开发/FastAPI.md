安装`pip install fastapi`

项目启动：在工程目录下`uvicorn main:app --reload`

交互式文档：启动后的地址(http://127.0.0.1:8000/docs)

## 路由

路由就是 IJRL 地址和处理函数之间的映射关系，它决定了当用户访问某个特定网址时，服务器应该执行哪段代码来返回结果。

![image-20260607143835805](./assets/image-20260607143835805.png)

## 参数

### 路径参数

例如：http://localhost:8000/items/3   3就是是参数

![image-20260607144307613](./assets/image-20260607144307613.png)

#### 类型注解

FastAPI 允许为参数声明额外的信息和校验

##### 原生类型：

```python
from fastapi import FastAPI

app = FastAPI()

# 路径参数：直接写在函数参数里 + 类型注解
@app.get("/items/{item_id}")
def get_item(item_id: int, q: str = None):  # q 是查询参数，默认 None
    return {"item_id": item_id, "q": q}
```

##### Pydantic 模型注解

```python
from pydantic import BaseModel

# 定义请求体结构 + 类型注解
class Item(BaseModel):
    name: str          # 必传字符串
    price: float       # 必传数字
    is_offer: bool = False  # 可选布尔值，默认 False
    
    
    
@app.post("/items/")
def create_item(item: Item):  # 直接注解为 Item 模型
    return item
```

##### 高级注解：

- 支持混合注解，路径 + 查询 + 请求体
  FastAPI 能自动区分它们，**完全不用配置**：

- 支持列表注解
  ```python
  from typing import List
  
  @app.get("/items/")
  def read_items(ids: List[int] = []):  # ids=[1,2,3]
      return {"ids": ids}
  ```

  

- 支持嵌套模型注解
  ```python
  class User(BaseModel):
      name: str
      age: int
  
  class Order(BaseModel):
      user: User       # 嵌套模型
      items: List[Item]
  ```

##### 返回值注解

不仅可以注解**入参**，还能注解**返回值**，FastAPI 会自动格式化：

```python
@app.get("/items/{item_id}", response_model=Item)
def get_item(item_id: int):
    return {
        "name": "test",
        "price": 10.5
    }
```

##### Path类型注解

只能用在路径参数

```python
from fastapi import FastAPI, Path  # 导入 Path

#最基础用法（给路径参数加文档）
@app.get("/items/{item_id}")
def get_item(
    # 路径参数 + Path 注解
    item_id: int = Path(..., title="商品ID", description="商品的唯一编号")
):
    return {"item_id": item_id}

# 常用校验规则（最实用）
@app.get("/items/{item_id}")
def get_item(
    item_id: int = Path(
        ...,
        title="商品ID",
        ge=1,        # 大于等于 1
        le=1000,     # 小于等于 1000
        example=123  # 文档示例值
    )
):
    return {"item_id": item_id}
```

- `...` 表示**必传**（路径参数本来就必传，写 ... 更规范）
- 文档 `/docs` 会自动显示 title/description

**Path 支持的校验参数**

- `gt`：大于
- `ge`：大于等于
- `lt`：小于
- `le`：小于等于
- `min_length`：字符串最小长度
- `max_length`：字符串最大长度
- `regex`：正则匹配
- `title` / `description`：接口文档描述
- `example`：文档示例



**新版推荐写法**

```python
from typing import Annotated

@app.get("/items/{item_id}")
def get_item(item_id: Annotated[int, Path(..., ge=1, le=1000)]):
    return {"item_id": item_id}
```



### 查询参数

声明的参数不是路径参数时，路径操作函数会把该参数自动解释为查询参数

l例如：http://localhost:8000/items?name=apple&price=10

![image-20260607150243117](./assets/image-20260607150243117.png)

注：上述路径参数的注解方式，除了Path方式，其他方式通用

#### Query参数注解

`Query` 用来：

- 加**校验规则**
- 加**接口文档描述**
- 设置**默认值、是否必传**
- 限制**长度、范围、正则**

```python
from fastapi import Query


# 最常用 Query 写法
@app.get("/items")
def get_items(
    name: str = Query(
        None,          # 默认值 None = 可选
        title="商品名称",
        description="要搜索的商品名称",
        min_length=2,  # 最短2字符
        max_length=20  # 最长20字符
    )
):
    return {"name": name}

# 数值校验（int /float）
@app.get("/items")
def get_items(
    price: float = Query(
        ...,
        gt=0,    # 必须 > 0
        lt=9999  # 必须 < 9999
    )
):
    return {"price": price}
```

- 用 `...` 表示**必须传**：

##### 列表参数注解

```python
@app.get("/items")
def get_items(ids: List[int] = Query([])):
    return {"ids": ids}
```

##### Python 3.10+ 推荐写法（Annotated）

```python
from typing import Annotated

@app.get("/items")
def get_items(name: Annotated[str, Query(min_length=2, max_length=20)]):
    return {"name": name}
```





### 请求体参数

就是 **POST / PUT / PATCH** 里传的 **JSON 数据**。GET 不能用请求体

![image-20260607151148523](./assets/image-20260607151148523.png)

在 HTTP协 议中，一个完整的请求由三部分组成，

- 请求行：包含方法、 IJRLV 协议版本
- 请求头：元数据信息〈 Content—Type 、 Authorization 等
- 请求体：实际要发送的数据内容

**规则**：

- **GET 不能用请求体**
- **POST / PUT / DELETE** 常用
- 请求体必须用 **Pydantic Model** 或 **Body()** 注解
- FastAPI 会**自动校验、自动生成文档、自动转换类型**



#### 标准写法

同样可以使用原生注解

```python
from pydantic import BaseModel

from fastapi import FastAPI

app = FastAPI()

class User(BaseModel):
    username: str       # 必传
    age: int | None = None  # 可选
    email: str         # 必传
    
@app.post("/users")
def create_user(user: User):  # 直接注解为模型
    return user
```

#### Field

```python
from pydantic import BaseModel, Field

class User(BaseModel):
    username: str = Field(..., min_length=3, default="admin", max_length=10, description="用户名")       # 必传
    age: int | None = None  # 可选
    email: str = Field(..., regex="^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$")        # 必传
```



#### 使用 Body ()

如果你**不想定义模型**，只想接收**一个 JSON 字段**，用 `Body`。

```python
from fastapi import Body

@app.post("/users")
def create_user(
    username: str = Body(..., min_length=3, max_length=20),
    age: int = Body(..., ge=18, le=100)
):
    return {"username": username, "age": age}
```



#### 混合：路径参数 + 查询参数 + 请求体

```python
@app.put("/users/{user_id}")
def update_user(
    user_id: int,                  # 路径参数
    q: str | None = None,         # 查询参数
    user: User                    # 请求体
):
    return {"user_id": user_id, "q": q, "user": user}
```



#### Python 3.10+ 优雅写法 Annotated

```python
from typing import Annotated

@app.post("/users")
def create_user(
    username: Annotated[str, Body(min_length=3)]
):
    return {"username": username}
```

## 响应类型

默认情况下， FastAPl 会自动将路径操作函数返回的 Python 对象（字典、列表、 Pydantic 模型等〕，经由 jsonable-encoder 转挨为JSON 兼容格式，并包装为 JSONResponse 返回。这省去了手动序列化的步骤，让开发者能更专注于业务逻辑。如果需要返回非 JSON 数据（如 HTML 、文件流 ), FastAPl 提供了丰富的响应类型来返回不同数据

### HTML响应格式

在`@app.get("/html", response_class=HTMLResponse)`指定返回格式

```python
from fastapi import FastAPI
from fastapi.responses import HTMLResponse  # 关键

@app.get("/html", response_class=HTMLResponse)
def get_html():
    html = """
    <html>
        <head><title>FastAPI HTML</title></head>
        <body>
            <h1>Hello HTML</h1>
        </body>
    </html>
    """
    return HTMLResponse(content=html)
```



### 文件相应格式

```python
from fastapi.responses import FileResponse

@app.get("/file", response_class=FileResponse)
async def get_girl():
    path = "file/girl.jpg"
    return FileResponse(path)
```



### 自定义相应格式

```python
class News(BaseModel):
    id: int = Field(..., gt=0, lt=9999)
    title: str = Field(default="新闻标题", min_length=3, max_length=10)
    content: str = Field(..., min_length=3, max_length=10)

@app.get("/news/{id}", description="获取新闻列表", response_model=News)
def get_news(id: int):
    return {
        "id": id,
        "content": f"这是{id}号的新闻内容"
    }
```



### 异常处理

对于客户端引发的错误（ 4xx ，如资源未找到、认证失败），应使用 fastapi.HTTPException 来中断正常处理流程，并返回标准错误响应。

```python
from fastapi import HTTPException

class News(BaseModel):
    id: int = Field(..., gt=0, lt=999)
    title: str = Field(default="新闻标题", min_length=3, max_length=10)
    content: str = Field(..., min_length=3, max_length=10)


@app.get("/news/{id}", description="获取新闻列表", response_model=News)
def get_news(id: int):
    id_list = [i for i in range(0, 999)]
    if id not in id_list:
        raise HTTPException(status_code=404, detail="新闻不存在")
    return {
        "id": id,
        "content": f"这是{id}号的新闻内容"
    }
```





## 中间件与依赖注入

![image-20260607160318050](./assets/image-20260607160318050.png)

### 中间件

中间件 (Middleware) 是一个在**每次请求进入 FastAPI** 应用时都会被执行的函数。它在请求到达实际的路径操作（路由处理函数）之前运行，并且在响应返回给客户之前再运行一次．

![image-20260607160451605](./assets/image-20260607160451605.png)

- 执行顺序：在代码中，后定义的中间件先执行

- 作用是：为每个请求添加统一的处理逻辑（记录日志、身份认证、跨域、设置响应头、性能监控等）
- 跨域用 `CORSMiddleware`（前后端必备）

### 依赖注入

依赖注入 = 把公共代码抽出来，谁要用就 “注入” 给谁用。

![image-20260607162259680](./assets/image-20260607162259680.png)

**作用：**

- **代码复用**：一段公共代码，100 个接口都能用
- **结构清晰**：接口只写业务逻辑，公共逻辑抽出去
- **可测试**：单独测试公共代码，不用动接口
- **FastAPI 最强功能之一**：比中间件更精细、更灵活



**依赖注入 = 公共代码抽离 + 自动注入使用**

**关键语法：`Depends(函数名)`**

**最常用场景：**

- 登录鉴权（最常用）
- 公共参数
- 数据库连接
- 权限校验

#### 示例

##### “登录校验” 依赖

```python
from fastapi import HTTPException, Header

# 依赖：校验token
async def check_token(token: str = Header(None)):
    if not token or token != "valid_token":
        # 直接抛异常，接口会自动返回401
        raise HTTPException(status_code=401, detail="未登录")
    return token  # 把token返回给接口用

# 加了 Depends(check_token)，必须登录才能访问
@app.get("/user/info")
async def user_info(token: str = Depends(check_token)):
    return {"user_id": 123, "token": token}
```

没传 token → 自动 401

传错 token → 自动 401

所有需要登录的接口，**只需要加一行代码**！

##### 路径依赖（整个接口组统一登录）

如果你想**一整个路由都必须登录**，直接写在路径上：

```python
# 整个 /admin 下的接口都需要登录
@app.get("/admin", dependencies=[Depends(check_token)])
async def admin():
    return {"msg": "管理员页面"}
```

##### 类作为依赖（更高级）

```python
class CommonDeps:
    def __init__(self, q: str = None, skip: int = 0, limit: int = 10):
        self.q = q
        self.skip = skip
        self.limit = limit

@app.get("/books/")
async def read_books(commons: CommonDeps = Depends()):
    return {"books": [1,2], "params": commons}
```

把一堆查询参数 封装成一个对象，不用在每个接口里写所有参数，直接直接写对象

**与pydantic对比：**

- **前端传 JSON → 用 Pydantic**
- **前端传？参数 → 用 类依赖**
- **想封装方法和逻辑 → 必须用 类依赖**



## ORM

安装：`pip install aiomysql=0.3.2 sqlalchemy=2.0.50`

ORM (Object-RelationalMapping, 对象关系映射）是一种编程技术，用于在面向对象编程语言和关系型数据库之间建立映射。它允许开发者通过操作对象的方式与数据库进行交互，而无需直接编写复杂的 SQL 语句。

![image-20260607163717475](./assets/image-20260607163717475.png)

ORM 就是把 **数据库** 和 **Python** 一一对应：

- 数据库 **表（table）** ←→ Python **类（Class）**
- 数据库 **行（row）** ←→ Python **对象（Object）**
- 数据库 **字段（column）** ←→ Python **属性（Attribute）**

**FastAPI + ORM 怎么配合**：

**ORM 模型**：操作数据库

**Pydantic 模型**：接口参数校验

**依赖注入**：自动获取数据库连接



### 连接和创建数据库表

FastAPI启动后自动创建该表，如果已存在，则跳过，

当 FastAPI 再次启动并执行 `Base.metadata.create_all` 时，SQLAlchemy 内部有一个关键的安全机制：它的 `checkfirst` 参数默认值为 `True`。这意味着，在尝试创建任何表之前，SQLAlchemy 会首先检查数据库中是否已经存在同名的表。如果发现该表已经存在，它会直接跳过创建操作，而不会引发任何错误或尝试重新创建表。因此，保留启动时的建表逻辑是完全安全的。

```python
from contextlib import asynccontextmanager

from fastapi import FastAPI

from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker, AsyncSession
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column
from datetime import datetime
from sqlalchemy import func, DateTime, Integer, String, Float

DATABASE_URL = "mysql+aiomysql://root:123456@localhost:3306/db_test?charset=utf8mb4"

# 1 创建异步引擎
async_engine = create_async_engine(
    DATABASE_URL,
    echo=True,  # 可选，输出SQL语句日志
    pool_size=10,  # 可选，设置连接池大小
    max_overflow=20  # 可选，设置连接池溢出时最大连接数
)


# 2 定义模型类：基类+表对应的模型类
# 基类：创建时间、更新时间、：书籍表：id、书名、作者、价格、出版社

class Base(DeclarativeBase):
    create_time: Mapped[datetime] = mapped_column(
        DateTime, server_default=func.now(), default=func.now(), comment="创建时间"
    )
    update_time: Mapped[datetime] = mapped_column(
        DateTime, server_default=func.now(), default=func.now(), onupdate=func.now(), comment="更新时间"
    )

class Book(Base):
    __tablename__ = "book"
    id: Mapped[int] = mapped_column(Integer,autoincrement=True,primary_key=True, comment="id")
    name: Mapped[str] = mapped_column(String(255), nullable=False,comment="书名")
    author: Mapped[str] = mapped_column(String(255), comment="作者")
    price: Mapped[float] = mapped_column(Float, comment="价格")
    publisher: Mapped[str] = mapped_column(String(255), comment="出版社")


# 3 创建表
async def create_tables():
    async with async_engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
--------------------------------------------------------
        
@asynccontextmanager
async def lifespan(app: FastAPI):
    '''
     yield 之前的代码 → 启动时执行
     yield 之后的代码 → 关闭时执行（资源清理）
    '''
    # 启动时：创建数据库表
    await create_tables()
    yield
    # 关闭时：释放数据库连接池，避免 "Event loop is closed" 错误
    await async_engine.dispose()


app = FastAPI(lifespan=lifespan)


```

### 查询

```python
# 创建异步会话工厂
AsyncSessionLocal = async_sessionmaker(
    bind=async_engine, # 绑定引擎
    class_=AsyncSession, # 指定异步会话
    expire_on_commit=False # 提交后会话不会自动过期
)

# 依赖项
async def get_database():
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
        finally:
            await session.close()

@app.get("/book/books")
async def get_book_list(db: AsyncSession = Depends(get_database)):
    result = await db.execute(select(Book))
    book = result.scalars().all()
    return book

```

#### 补充

 **FastAPI 底层是如何处理** `yield` **的**

当你把 `get_database` 作为 `Depends` 传入时，FastAPI 在接收到请求时，实际上做了以下事情：

1. **启动生成器**：FastAPI 调用 `get_database()`，拿到的是一个生成器对象（Generator Object）。
2. **获取资源（相当于第一次 `next()`）**：FastAPI 内部执行了类似 `await anext(generator)` 的操作，拿到了 `yield` 出来的 `session`，并把它注入给你的路由函数 `get_book_list`。
3. **执行业务逻辑**：你的 `get_book_list` 开始运行。
4. **恢复生成器（相当于第二次 `next()`）**：当你的路由函数执行完毕（或者抛出异常）后，**FastAPI 的依赖注入系统会再次介入**。它会在后台自动对这个生成器对象再次调用 `next()`（或 `anext()`）。

正是因为 FastAPI 在后台偷偷帮你执行了这“第二次调用”，`get_database` 函数才会从 `yield` 处苏醒，继续往下执行 `commit` 和 `close`。

#### 1.基础查询操作

```python
from sqlalchemy import select

# 1. 查询所有书籍
statement = select(Book)
result = await db.execute(statement)
books = result.scalars().all()  # 返回 Book 对象列表

# 2. 根据条件过滤（查询单条记录）
statement = select(Book).filter_by(name="Python入门")
result = await db.execute(statement)
book = result.scalars().first()  # 返回第一个匹配的 Book 对象，没有则返回 None

# 3. 复杂条件过滤
statement = select(Book).filter(Book.price > 50.0)
result = await db.execute(statement)
expensive_books = result.scalars().all()
```

#### 2. 排序、分页与限制

```python
# 按价格降序排列，并只取前 10 条（常用于分页）
statement = (
    select(Book)
    .order_by(Book.price.desc())  # 降序排列
    .limit(10)                    # 限制返回数量
    .offset(0)                    # 偏移量（跳过前0条）
)
result = await db.execute(statement)
books = result.scalars().all()
```

#### 3. 查询指定的列

```python
# 只查询书名和价格
statement = select(Book.name, Book.price)
result = await db.execute(statement)
rows = result.all()  # 返回 Row 对象列表，可以通过 row.name 访问
```

#### 4. 关联查询（Join）

```python
# 查询某位作者的所有书籍
statement = (
    select(Book)
    .join(Author)
    .filter(Author.name == "张三")
)
result = await db.execute(statement)
books = result.scalars().all()
```

#### 5. 增删改（CUD）操作

**查询操作不能像增删改那样直接通过 `session` 对象一步完成，而是必须借助 `select()` 构造器。**

除了查询，增删改操作通常直接通过 `session` 对象完成：

```python
# 新增 (Create)
new_book = Book(name="新书", price=39.9)
db.add(new_book)
# 注意：如果你在 get_database 的 yield 之后配置了自动 commit，这里无需手动 commit

# 修改 (Update)
book.name = "更新后的书名"
# 修改属性后，SQLAlchemy 会自动追踪状态，无需额外操作

# 删除 (Delete)
await db.delete(book)
```

### 新增

注意：数据库会话只能操作ORM对象

```python
class BookBase(BaseModel):
    name: str
    author: str
    price: float
    publisher: str

# 新增book操作
@app.post("/book/add")
async def add_book(book: BookBase, db: AsyncSession = Depends(get_database)):
    # db.add() 只能接收 SQLAlchemy ORM 实例，不能直接接收 Pydantic 对象
    new_book = Book(**book.model_dump()) # ① Pydantic → dict → ORM 对象（类型转换）
    db.add(new_book)  # ② 将 ORM 对象加入数据库会话
    await db.commit()
    return new_book

# 新增多个books操作
@app.post("/book/add_many")
async def add_many_books(books: list[BookBase], db: AsyncSession = Depends(get_database)):
    new_books = [Book(**book.model_dump()) for book in books]
    db.add_all(new_books)
    await db.commit()
```

### 修改

先查后改，通过id修改数据

```python
# 修改book操作
@app.put("/book/{book_id}")
async def update_book(book_id: int, book_data: BookBase, db: AsyncSession = Depends(get_database)):
    # ① 根据 id 查询记录
    existing_book: Book | None = await db.get(Book, book_id)
    if not existing_book:
        raise HTTPException(status_code=404, detail=f"书籍 id={book_id} 不存在")
    # ② 更新字段值
    # model_dump() 方法的默认返回值是一个 Python 原生字典（dict），其类型签名为 dict[str, Any]。
    for field, value in book_data.model_dump().items():
        setattr(existing_book, field, value)
    await db.commit()
    return existing_book
```



### 删除

先查后删，通过id删除

```python
# 删除book操作
@app.delete("/book/{book_id}")
async def delete_book(book_id: int, db: AsyncSession = Depends(get_database)):
    # ① 根据 id 查询记录
    book: Book | None = await db.get(Book, book_id)
    if not book:
        raise HTTPException(status_code=404, detail=f"书籍 id={book_id} 不存在")
    # ② 删除记录
    await db.delete(book)
    await db.commit()
    return {"detail": f"书籍 id={book_id} 已删除"}
```

