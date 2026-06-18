# FastAPi-learning-way
FastApi学习路线
一些共享的资源,希望可以给大家一些帮助,资源有用给个star谢谢
# FastAPI 学习路线

> 核心理念：零基础起步，**讲透原理**，系统掌握 FastAPI 作为后端通用技能。
> 既能独立写后端服务，也能为 RAG / Agent 项目提供干净的 API 层。
> 本路线对照 FastAPI 官方文档（前置概念 + 用户指南 + 高级用户指南）整理，力求完整覆盖。

---

## 学习路线图

```
1. FastAPI 入门与环境          ⭐
   ↓
2. 类型注解与 async 基础       ⭐⭐
   ↓
3. 路由与请求参数             ⭐⭐
   ↓
4. Pydantic 数据校验 ★        ⭐⭐⭐
   ↓
5. 响应处理                  ⭐⭐
   ↓
6. 表单、文件与请求体进阶      ⭐⭐
   ↓
7. 依赖注入 ★               ⭐⭐⭐
   ↓
8. 数据库集成               ⭐⭐⭐
   ↓
9. 认证与安全 ★             ⭐⭐⭐
   ↓
10. 中间件与 CORS           ⭐⭐
   ↓
11. 异步与并发 ★            ⭐⭐⭐
   ↓
12. 流式输出 (SSE/WS)       ⭐⭐⭐
   ↓
13. 生命周期事件与应用状态    ⭐⭐
   ↓
14. OpenAPI 与接口文档定制    ⭐⭐
   ↓
15. 测试                   ⭐⭐
   ↓
16. 项目结构与工程化         ⭐⭐
   ↓
17. 部署                   ⭐⭐⭐
   ↓
18. 实战项目               ⭐⭐⭐⭐
```

**目标是系统掌握 FastAPI，并做出一个完整的后端项目。学习节奏由使用者自行规划。**

打 ★ 的章节（Pydantic、依赖注入、认证、异步、流式）是重点，会展开得更细。
另设**附录：进阶专题**，收纳官方文档中较小众的高级主题，按需了解。

---

## 1. FastAPI 入门与环境

### 学什么

- HTTP 基础：请求/响应、请求方法（GET/POST/PUT/DELETE）、状态码、Header
- ASGI 是什么：FastAPI 为什么是异步框架，和 WSGI（Flask/Django）的区别
- 环境搭建：虚拟环境、环境变量、安装 FastAPI 和 Uvicorn
- 第一个接口：写出能跑的 `Hello World`，理解自动生成的交互式文档

### 怎么学

1. 先搞懂 HTTP 请求-响应模型（这是所有 Web 框架的地基）
2. 理解 ASGI：服务器（Uvicorn）和应用（FastAPI）如何分工
3. 创建虚拟环境、理解环境变量的作用，装好依赖
4. 写第一个接口，打开 `/docs` 看自动文档

### 实践

```python
from fastapi import FastAPI
app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello FastAPI"}
# 运行：uvicorn main:app --reload
# 访问 http://127.0.0.1:8000/docs 看自动生成的交互文档
```

### 学习资源

- FastAPI 官方教程：https://fastapi.tiangolo.com/zh/tutorial/
- 虚拟环境与环境变量：https://fastapi.tiangolo.com/zh/environment-variables/
- ASGI 规范：https://asgi.readthedocs.io


---

## 2. 类型注解与 async 基础

### 学什么

- Python 类型注解：`int`、`str`、`list[int]`、`dict`、`Optional`、`Union`
- 类型注解为什么是 FastAPI 的核心：它驱动自动校验、转换、文档生成
- 同步 vs 异步：`def` 和 `async def` 的区别
- `async / await`：协程、事件循环、什么时候该用异步

### 怎么学

1. 把 Python 类型注解练熟，FastAPI 的一切都建立在它之上
2. 理解事件循环：一个线程如何处理大量并发请求
3. 搞清楚"IO 密集用异步、CPU 密集别用异步"的原则

### 实践

```python
from fastapi import FastAPI
app = FastAPI()

@app.get("/sync")
def sync_route() -> dict:        # 同步：适合 CPU 计算
    return {"type": "sync"}

@app.get("/async")
async def async_route() -> dict:  # 异步：适合 IO（数据库、网络请求）
    return {"type": "async"}
```

### 学习资源

- Python typing 文档：https://docs.python.org/zh-cn/3/library/typing.html
- FastAPI 异步说明：https://fastapi.tiangolo.com/zh/async/


---

## 3. 路由与请求参数

### 学什么

- 路径参数（Path）：`/items/{item_id}`、路径参数预设值（Enum）
- 查询参数（Query）：`/items?skip=0&limit=10`、查询参数模型
- 请求体（Body）：POST/PUT 传 JSON、多个请求体参数
- Header 和 Cookie 参数、Header/Cookie 参数模型
- 参数校验：`Query(..., min_length=3)`、`Path(..., gt=0)`、数值校验
- 额外数据类型：`UUID`、`datetime`、`Decimal`、`bytes`、`frozenset`

### 怎么学

1. 把四种参数来源（路径、查询、请求体、头部）分清楚
2. 理解 FastAPI 如何根据类型注解自动判断参数从哪来
3. 给参数加上校验规则，看 `/docs` 里的效果
4. 用额外数据类型，理解 FastAPI 自动完成的解析和序列化

### 实践

```python
from fastapi import FastAPI, Query, Path
app = FastAPI()

@app.get("/items/{item_id}")
def get_item(
    item_id: int = Path(gt=0),               # 路径参数，必须 > 0
    q: str | None = Query(default=None, max_length=50)  # 查询参数，可选
):
    return {"item_id": item_id, "q": q}
```

### 学习资源

- FastAPI 查询参数：https://fastapi.tiangolo.com/zh/tutorial/query-params/
- 路径参数与数值校验：https://fastapi.tiangolo.com/zh/tutorial/path-params-numeric-validations/
- 额外数据类型：https://fastapi.tiangolo.com/zh/tutorial/extra-data-types/


---

## 4. Pydantic 数据校验 ★核心重点

> Pydantic 是 FastAPI 的灵魂。所有请求体、响应模型、配置管理都靠它。
> 这一章要花足够时间彻底搞懂。

### 学什么

- BaseModel：定义数据模型
- 字段类型与校验：`Field`、约束、默认值、字段元数据
- 嵌套模型：模型里套模型、列表里套模型
- 校验器：`field_validator`、`model_validator` 自定义校验
- 序列化与反序列化：模型 ↔ JSON ↔ dict
- 请求体更新：`PUT` vs `PATCH`、`exclude_unset`、`model_copy(update=...)`
- 声明请求体示例数据：`json_schema_extra`、`examples`
- Pydantic v2 的新特性和常见变化

### 怎么学

1. 先定义简单模型，理解"声明即校验"
2. 练嵌套模型，处理真实场景的复杂 JSON
3. 写自定义校验器，处理业务规则
4. 理解 `model_dump()` / `model_validate()` 的序列化机制
5. 用 `exclude_unset` 实现 PATCH 局部更新

### 实践

```python
from pydantic import BaseModel, Field, field_validator

class Address(BaseModel):
    city: str
    zipcode: str

class User(BaseModel):
    name: str = Field(min_length=1, max_length=50)
    age: int = Field(ge=0, le=150)
    address: Address                  # 嵌套模型
    tags: list[str] = []

    @field_validator("name")
    @classmethod
    def name_not_admin(cls, v: str) -> str:
        if v.lower() == "admin":
            raise ValueError("name cannot be admin")
        return v
```

### 学习资源

- Pydantic 官方文档：https://docs.pydantic.dev
- FastAPI 请求体：https://fastapi.tiangolo.com/zh/tutorial/body/
- 请求体更新：https://fastapi.tiangolo.com/zh/tutorial/body-updates/
- 声明请求示例数据：https://fastapi.tiangolo.com/zh/tutorial/schema-extra-example/


---

## 5. 响应处理

### 学什么

- 响应模型 `response_model`：控制返回的数据结构和字段过滤
- 状态码：`status_code`、`status` 常量
- JSON 兼容编码器：`jsonable_encoder`（把模型转成可存 JSON 的数据）
- 自定义响应：`JSONResponse`、`PlainTextResponse`、`HTMLResponse`、直接返回 `Response`
- 响应中设置 Cookie 和 Header
- 错误处理：`HTTPException`、自定义异常处理器、额外状态码
- 响应模型的字段过滤（排除敏感字段，如密码）、`response_model_exclude_unset`

### 怎么学

1. 用 `response_model` 控制输出，理解输入模型和输出模型为什么要分开
2. 理解 `jsonable_encoder` 在"存数据库/存文件"时的作用
3. 练 `HTTPException` 抛业务错误，写全局异常处理器统一错误格式
4. 练在响应里设置 Cookie/Header、返回非 JSON 响应

### 实践

```python
from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel

class UserOut(BaseModel):     # 输出模型，不含密码
    id: int
    name: str

app = FastAPI()

@app.get("/users/{uid}", response_model=UserOut, status_code=status.HTTP_200_OK)
def get_user(uid: int):
    if uid != 1:
        raise HTTPException(status_code=404, detail="User not found")
    return {"id": 1, "name": "Alice", "password": "secret"}  # password 会被自动过滤
```

### 学习资源

- 响应模型：https://fastapi.tiangolo.com/zh/tutorial/response-model/
- JSON 兼容编码器：https://fastapi.tiangolo.com/zh/tutorial/encoder/
- 响应 Cookie/Header：https://fastapi.tiangolo.com/zh/advanced/response-cookies/
- 处理错误：https://fastapi.tiangolo.com/zh/tutorial/handling-errors/


---

## 6. 表单、文件与请求体进阶

> 官方文档把表单和文件单独成节，这是真实业务（登录表单、文件上传）的高频需求，也是 AI 项目上传文档的入口。

### 学什么

- 表单数据：`Form`，处理 `application/x-www-form-urlencoded`
- 表单模型：用 Pydantic 模型接收表单字段
- 文件上传：`File`、`UploadFile`，大文件流式读取
- 同时接收表单和文件：`Form` + `File` 混用
- 多文件上传
- 文件上传的注意点：内存 vs 磁盘、文件大小限制

### 怎么学

1. 写一个登录表单接口，理解表单和 JSON 请求体的区别（不能混用）
2. 用 `UploadFile` 接收文件，理解它和 `bytes` 的差异（流式 vs 一次性读入内存）
3. 实现一个"上传文档"的接口，为后续 RAG 项目的文档入口打基础

### 实践

```python
from fastapi import FastAPI, Form, File, UploadFile

app = FastAPI()

@app.post("/login")
def login(username: str = Form(), password: str = Form()):
    return {"username": username}

@app.post("/upload")
async def upload(file: UploadFile = File()):
    content = await file.read()          # 读取文件内容
    return {"filename": file.filename, "size": len(content)}
```

### 学习资源

- 表单数据：https://fastapi.tiangolo.com/zh/tutorial/request-forms/
- 请求文件：https://fastapi.tiangolo.com/zh/tutorial/request-files/
- 请求表单与文件：https://fastapi.tiangolo.com/zh/tutorial/request-forms-and-files/


---

## 7. 依赖注入 ★核心重点

> 依赖注入是 FastAPI 最强大、最优雅的特性。掌握它，代码复用和测试都会脱胎换骨。

### 学什么

- `Depends` 的原理：把公共逻辑抽成可复用的依赖
- 依赖嵌套：依赖也可以依赖别的依赖（子依赖）
- 类作为依赖：用类组织复杂依赖
- 带 `yield` 的依赖：资源管理（数据库会话的开启/关闭）
- 路径操作依赖（不返回值，只做拦截）
- 全局依赖：作用于整个应用的依赖
- `Annotated` 写法：复用依赖声明（官方推荐写法）

### 怎么学

1. 先理解"为什么要依赖注入"——消除重复代码、解耦
2. 写一个获取数据库会话的依赖（带 yield）
3. 写一个鉴权依赖，理解依赖如何做权限拦截
4. 用 `Annotated` 抽出可复用的依赖类型别名
5. 理解依赖在测试中如何被替换（dependency_overrides）

### 实践

```python
from fastapi import FastAPI, Depends

def get_db():                    # 带 yield 的依赖：用于资源管理
    db = "fake_db_session"
    try:
        yield db                 # 请求处理期间提供 db
    finally:
        print("close db")        # 请求结束后清理

def common_params(skip: int = 0, limit: int = 10):
    return {"skip": skip, "limit": limit}

app = FastAPI()

@app.get("/items/")
def list_items(db=Depends(get_db), params: dict = Depends(common_params)):
    return {"db": db, "params": params}
```

### 学习资源

- FastAPI 依赖注入：https://fastapi.tiangolo.com/zh/tutorial/dependencies/
- 带 yield 的依赖：https://fastapi.tiangolo.com/zh/tutorial/dependencies/dependencies-with-yield/
- 全局依赖：https://fastapi.tiangolo.com/zh/tutorial/dependencies/global-dependencies/


---

## 8. 数据库集成

### 学什么

- SQLAlchemy ORM：模型定义、查询、关系
- 同步 vs 异步 ORM：`SQLAlchemy 2.0` 的异步支持（`async_session`）
- CRUD 操作：增删改查的完整实现
- 数据库连接管理：连接池、会话生命周期（结合第 7 章的 yield 依赖）
- 数据库迁移：Alembic 简介
- SQLModel：FastAPI 作者出品，ORM + Pydantic 二合一（了解）

### 怎么学

1. 先用同步 SQLAlchemy 跑通 CRUD
2. 换成异步 ORM，理解 `async with session`
3. 把数据库会话做成依赖，注入到路由里
4. 用 Alembic 管理表结构变更

### 实践

```python
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column

engine = create_async_engine("sqlite+aiosqlite:///./test.db")
SessionLocal = async_sessionmaker(engine, expire_on_commit=False)

class Base(DeclarativeBase): pass

class User(Base):
    __tablename__ = "users"
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]

async def get_db():              # 异步会话依赖
    async with SessionLocal() as session:
        yield session
```

### 学习资源

- SQLAlchemy 文档：https://docs.sqlalchemy.org
- FastAPI + SQL 教程：https://fastapi.tiangolo.com/zh/tutorial/sql-databases/
- SQLModel：https://sqlmodel.tiangolo.com


---

## 9. 认证与安全 ★核心重点

> 任何上线的项目都绕不开认证。这一章是后端工程师的必备能力，也是面试高频。

### 学什么

- OAuth2 密码模式：`OAuth2PasswordBearer`、`OAuth2PasswordRequestForm`
- JWT：生成、签名、验证 token
- 密码哈希：`passlib` / `bcrypt`，绝不明文存密码
- 权限控制：基于依赖的登录校验、角色权限、`Security` 与 scopes
- HTTP 基本认证、API Key 认证（了解）
- 安全要点：token 过期、刷新、HTTPS、敏感信息保护

### 怎么学

1. 理解 token 认证的完整流程：登录拿 token → 请求带 token → 服务端验证
2. 实现密码哈希存储与校验
3. 写 JWT 的生成和解析
4. 用依赖实现 `get_current_user`，保护需要登录的接口
5. 了解 OAuth2 scopes 做细粒度权限

### 实践

```python
from fastapi import Depends, HTTPException
from fastapi.security import OAuth2PasswordBearer
import jwt

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")
SECRET_KEY = "your-secret"   # 实际从环境变量读取

def get_current_user(token: str = Depends(oauth2_scheme)):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        return payload["sub"]
    except jwt.PyJWTError:
        raise HTTPException(status_code=401, detail="Invalid token")
```

### 学习资源

- FastAPI 安全教程：https://fastapi.tiangolo.com/zh/tutorial/security/
- OAuth2 + JWT：https://fastapi.tiangolo.com/zh/tutorial/security/oauth2-jwt/
- JWT 介绍：https://jwt.io/introduction


---

## 10. 中间件与 CORS

### 学什么

- 中间件原理：在请求到达路由前、响应返回前做统一处理
- 请求生命周期：请求 → 中间件 → 依赖 → 路由 → 响应 → 中间件
- 常见中间件：日志记录、请求计时、统一异常捕获
- CORS：跨域是什么、为什么会被浏览器拦、`CORSMiddleware` 如何配置
- 常用内置中间件：`GZipMiddleware`、`TrustedHostMiddleware`

### 怎么学

1. 写一个记录每个请求耗时的中间件
2. 理解中间件的执行顺序（洋葱模型）
3. 配置 CORS，解决前端跨域调用问题

### 实践

```python
import time
from fastapi import FastAPI, Request
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()
app.add_middleware(CORSMiddleware, allow_origins=["*"], allow_methods=["*"])

@app.middleware("http")
async def add_process_time(request: Request, call_next):
    start = time.perf_counter()
    response = await call_next(request)
    response.headers["X-Process-Time"] = str(time.perf_counter() - start)
    return response
```

### 学习资源

- FastAPI 中间件：https://fastapi.tiangolo.com/zh/tutorial/middleware/
- CORS 配置：https://fastapi.tiangolo.com/zh/tutorial/cors/


---

## 11. 异步与并发 ★核心重点

> AI 项目里调用大模型 API 是典型的 IO 密集场景，异步并发能大幅提升吞吐。这一章对接 AI 应用极其重要。

### 学什么

- asyncio 深入：事件循环、`await`、`gather` 并发
- 并发调用：同时发起多个外部请求（如批量调用 AI API）
- 后台任务：`BackgroundTasks`，请求返回后继续执行
- 异步陷阱：在 async 里调用阻塞代码会卡死事件循环
- `run_in_threadpool`：把阻塞代码丢到线程池

### 怎么学

1. 理解为什么异步能在等待 IO 时处理别的请求
2. 用 `asyncio.gather` 并发调用多个接口，对比串行耗时
3. 用 `BackgroundTasks` 做"发送通知、写日志"等收尾工作
4. 搞清楚什么代码会阻塞事件循环，如何用线程池规避

### 实践

```python
import asyncio
from fastapi import FastAPI, BackgroundTasks

app = FastAPI()

@app.get("/batch")
async def batch_call():
    async def fake_api(i):
        await asyncio.sleep(1)
        return i
    results = await asyncio.gather(*[fake_api(i) for i in range(5)])  # 并发，总耗时 ~1s
    return {"results": results}

@app.post("/send")
async def send(bg: BackgroundTasks):
    bg.add_task(lambda: print("后台发送通知"))   # 响应返回后才执行
    return {"status": "accepted"}
```

### 学习资源

- Python asyncio：https://docs.python.org/zh-cn/3/library/asyncio.html
- FastAPI 后台任务：https://fastapi.tiangolo.com/zh/tutorial/background-tasks/


---

## 12. 流式输出（SSE / WebSocket）

> 对接 AI 大模型的"打字机效果"必备。这是 AI 应用前后端联调的关键能力。

### 学什么

- `StreamingResponse`：流式返回数据
- SSE（Server-Sent Events）：服务端持续推送，对接 AI 流式输出
- WebSocket：双向实时通信、连接管理、广播
- 异步生成器：`async def` + `yield` 配合流式

### 怎么学

1. 用 `StreamingResponse` + 异步生成器做一个流式返回
2. 实现 SSE，模拟逐字返回 AI 回答
3. 写一个 WebSocket 的简单聊天 demo，理解连接的建立与断开

### 实践

```python
import asyncio
from fastapi import FastAPI
from fastapi.responses import StreamingResponse

app = FastAPI()

@app.get("/stream")
async def stream():
    async def gen():
        for word in "你好 我是 流式 输出".split():
            yield f"data: {word}\n\n"   # SSE 格式
            await asyncio.sleep(0.3)
    return StreamingResponse(gen(), media_type="text/event-stream")
```

### 学习资源

- FastAPI WebSocket：https://fastapi.tiangolo.com/zh/advanced/websockets/
- 自定义响应（含 StreamingResponse）：https://fastapi.tiangolo.com/zh/advanced/custom-response/


---

## 13. 生命周期事件与应用状态

> 官方文档专门讲了 lifespan。AI 项目里"启动时加载模型/向量库客户端、关闭时释放连接"全靠它，非常重要。

### 学什么

- `lifespan`：应用启动/关闭时执行的逻辑（推荐写法）
- 旧式 `startup` / `shutdown` 事件（了解，已不推荐）
- `app.state`：在应用级别共享对象（如全局客户端、连接池）
- 典型用途：启动时建立数据库连接池、加载 AI 模型、初始化 HTTP 客户端
- 子应用挂载：`app.mount`，把多个 FastAPI 应用组合在一起

### 怎么学

1. 用 `lifespan` 在启动时初始化资源、关闭时清理
2. 把初始化好的客户端存进 `app.state`，在路由里通过 `request.app.state` 取用
3. 理解为什么"在请求里临时创建客户端"是性能反模式

### 实践

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI

@asynccontextmanager
async def lifespan(app: FastAPI):
    app.state.client = "init_ai_client"   # 启动：加载一次，全程复用
    print("startup: 资源已加载")
    yield
    print("shutdown: 释放资源")           # 关闭：清理

app = FastAPI(lifespan=lifespan)

@app.get("/")
def root(): 
    return {"client": app.state.client}
```

### 学习资源

- Lifespan 事件：https://fastapi.tiangolo.com/zh/advanced/events/
- 子应用挂载：https://fastapi.tiangolo.com/zh/advanced/sub-applications/


---

## 14. OpenAPI 与接口文档定制

> FastAPI 自动生成的 `/docs` 和 `/redoc` 是它的招牌。这一章学会让文档专业、可读，对团队协作和前后端联调很有价值。

### 学什么

- 路径操作配置：`tags`（分组）、`summary`、`description`、`response_description`、`deprecated`
- 用 docstring 写接口说明（支持 Markdown）
- 应用元数据：`title`、`version`、`description`、标签元数据
- 文档 URL 定制：修改 `/docs`、`/redoc` 路径，或关闭
- 额外响应：在文档里声明多种状态码的响应模型
- OpenAPI 高级定制：`openapi_extra`、自定义 OpenAPI schema

### 怎么学

1. 给接口加 `tags` 和 `summary`，看 `/docs` 里分组的效果
2. 配置应用级元数据，让文档首页专业
3. 声明额外响应（如 404、422 的响应体），让文档更完整

### 实践

```python
from fastapi import FastAPI

app = FastAPI(title="我的 API", version="1.0.0", description="项目接口文档")

@app.get(
    "/items/",
    tags=["items"],
    summary="获取商品列表",
    response_description="商品数组",
)
def list_items():
    """
    返回所有商品。

    - 支持分页
    - 支持按分类筛选
    """
    return []
```

### 学习资源

- 路径操作配置：https://fastapi.tiangolo.com/zh/tutorial/path-operation-configuration/
- 元数据和文档 URL：https://fastapi.tiangolo.com/zh/tutorial/metadata/
- 额外响应：https://fastapi.tiangolo.com/zh/advanced/additional-responses/


---

## 15. 测试

### 学什么

- `TestClient`：FastAPI 内置的测试客户端
- pytest：测试框架基础、fixture
- 异步测试：`httpx.AsyncClient` + `pytest-asyncio`
- 依赖覆盖：`dependency_overrides` 在测试中替换真实依赖（如用测试数据库）
- 测试数据库、测试事件（lifespan）

### 怎么学

1. 用 `TestClient` 写第一个接口测试
2. 学 pytest 的 fixture，组织测试数据
3. 用 `dependency_overrides` 把数据库依赖换成测试库
4. 写异步测试

### 实践

```python
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_read_root():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Hello FastAPI"}
```

### 学习资源

- FastAPI 测试：https://fastapi.tiangolo.com/zh/tutorial/testing/
- 异步测试：https://fastapi.tiangolo.com/zh/advanced/async-tests/
- pytest 文档：https://docs.pytest.org


---

## 16. 项目结构与工程化

### 学什么

- 大型项目分层：`routers/`、`models/`、`schemas/`、`crud/`、`services/`
- `APIRouter`：拆分路由到多个文件、路由前缀和标签
- 配置管理：`pydantic-settings` 读取环境变量
- 日志：结构化日志、日志分级
- 项目骨架：一个可扩展的目录结构

### 怎么学

1. 把单文件应用拆成多模块，理解 `APIRouter` 的作用
2. 用 `pydantic-settings` 管理配置，敏感信息进 `.env`
3. 配置日志，区分开发/生产

### 实践

```
app/
├── main.py            # 应用入口
├── core/config.py     # 配置（pydantic-settings）
├── routers/           # 各模块路由
│   ├── users.py
│   └── items.py
├── models/            # 数据库模型
├── schemas/           # Pydantic 模型
├── crud/              # 数据库操作
└── services/          # 业务逻辑
```

### 学习资源

- FastAPI 大型应用：https://fastapi.tiangolo.com/zh/tutorial/bigger-applications/
- 配置管理：https://fastapi.tiangolo.com/zh/advanced/settings/


---

## 17. 部署

### 学什么

- Uvicorn / Gunicorn：开发服务器 vs 生产服务器、多 worker
- Docker：容器化 FastAPI 应用、编写 Dockerfile
- 生产配置：环境变量、关闭 reload、设置 worker 数
- HTTPS、代理之后（运行在 Nginx/Traefik 反向代理后面，`root_path`）
- 反向代理：Nginx 简介（了解即可）

### 怎么学

1. 理解 Uvicorn 单进程和 Gunicorn 多 worker 的区别
2. 写一个 Dockerfile，把应用打包成镜像
3. 用 docker-compose 把应用 + 数据库一起跑起来
4. 了解部署在反向代理之后需要的 `root_path` 配置

### 实践

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 学习资源

- FastAPI 部署：https://fastapi.tiangolo.com/zh/deployment/
- 容器部署：https://fastapi.tiangolo.com/zh/deployment/docker/
- 代理之后：https://fastapi.tiangolo.com/zh/advanced/behind-a-proxy/


---

## 18. 实战项目

> 把前面所有能力整合成一个完整、规范、能部署的后端项目。

### 项目建议

```
项目：完整的后端服务（可直接对接 RAG / Agent）
技术栈：FastAPI + SQLAlchemy(异步) + JWT 认证 + Docker

功能：
1. 用户注册/登录（JWT 认证、密码哈希）
2. 完整的 CRUD 业务模块（分层架构）
3. 文件上传接口
4. 流式输出接口（为对接 AI 预留）
5. lifespan 加载全局客户端 + 全局异常处理 + 日志
6. 完整的接口测试
7. 专业的 OpenAPI 文档
8. Docker 一键部署

亮点：
- 规范的分层项目结构
- 异步数据库 + 依赖注入
- 完整的认证授权
- 流式输出能力（直接对接 RAG/Agent 的 AI 回答）
```

### 项目要求

- 代码放 GitHub，README 写清楚架构和启动方式
- Docker 一键启动
- 有接口测试，关键路径有覆盖
- 能讲清楚分层设计和技术选型


---

## 附录：进阶专题（按需了解）

> 以下是官方文档「高级用户指南」里较小众的主题，不属于主线，遇到具体需求时再查。

| 专题                                             | 说明                                                        | 官方文档                        |
| ------------------------------------------------ | ----------------------------------------------------------- | ------------------------------- |
| 静态文件 / 模板                                  | `StaticFiles` 托管静态资源、Jinja2 模板（前后端不分离时用） | advanced/templates              |
| 数据类 (dataclasses)                             | 用 Python `dataclass` 替代 Pydantic 模型                    | advanced/dataclasses            |
| 高级依赖                                         | 可调用实例作为参数化依赖                                    | advanced/advanced-dependencies  |
| OpenAPI 回调 / Webhooks                          | 声明你的 API 会回调的外部接口                               | advanced/openapi-callbacks      |
| WSGI 集成                                        | 在 FastAPI 中挂载 Flask/Django 等 WSGI 应用                 | advanced/wsgi                   |
| 生成客户端 SDK                                   | 根据 OpenAPI 自动生成前端/客户端代码                        | advanced/generate-clients       |
| 使用 Request 对象                                | 直接操作原始请求（特殊场景）                                | advanced/using-request-directly |
| 测试中的依赖覆盖进阶 / 测试 WebSocket / 测试事件 | 高级测试技巧                                                | advanced/testing-*              |
| 条件 OpenAPI / 扩展 OpenAPI                      | 按环境开关文档、自定义 schema                               | how-to/*                        |
| GraphQL                                          | 集成 Strawberry 等 GraphQL 库（了解）                       | how-to/graphql                  |

**这些不需要系统学，做项目遇到了再查官方文档。**

---

## 学习建议

1. **每章都要有产出**：理论文档读懂 + Demo 跑通，缺一不可
2. **Pydantic、依赖注入、异步是绝对核心**：FastAPI 的精髓全在这三块，要花足够时间
3. **结合 AI 方向**：异步并发、流式输出、生命周期事件这三章直接服务于 RAG/Agent 项目，重点练
4. **零基础不要跳**：HTTP、类型注解、async 这些地基不打牢，后面会一直懵
5. **边做边查文档**：FastAPI 官方文档质量极高，遇到问题优先查官方
6. **项目要能部署**：最终目标是 Docker 一键启动、GitHub 上有完整代码

---

## 与 AI 应用路线的衔接

这份 FastAPI 路线和 `1.PythonAI应用开发学习路线.md` 是互补关系：

| FastAPI 章节              | 在 AI 项目中的作用                |
| ------------------------- | --------------------------------- |
| 第 6 章 表单与文件        | RAG 上传文档（PDF/Word）的入口    |
| 第 9 章 认证与安全        | AI 服务的用户鉴权、API 保护       |
| 第 8 章 数据库集成        | 存储对话历史、用户数据            |
| 第 11 章 异步与并发       | 高并发调用大模型 API              |
| 第 12 章 流式输出         | RAG/Agent 回答的打字机效果（SSE） |
| 第 13 章 生命周期事件     | 启动时加载模型/向量库客户端       |
| 第 16、17 章 工程化与部署 | 把 RAG/Agent 项目部署上线         |

**建议先把 FastAPI 第 1-7 章打好基础，再并行推进 AI 应用路线，遇到 API 层的需求回来深入对应章节。**

















