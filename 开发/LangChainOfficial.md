# 模型 （欠缺）

模型可以通过两种方式使用：

1. **与代理一起使用** - 创建[代理](https://langchain-doc.cn/v1/python/langchain/agents#model)时可动态指定模型。
2. **独立使用** - 模型可直接调用（在代理循环之外），用于文本生成、分类或提取等任务，而无需代理框架。

同一模型接口在两种上下文中均适用，这为您提供了从简单开始并根据需要扩展到更复杂基于代理的工作流程的灵活性。

## 基本用法

### 初始化模型

在 LangChain 中开始使用独立模型的最简单方法是使用 [`init_chat_model`](https://reference.langchain.com/python/langchain/models/#langchain.chat_models.init_chat_model) 从您选择的[提供商](https://langchain-doc.cn/v1/python/integrations/providers/overview)初始化一个模型（以下示例）

```python
import os
import dotenv

dotenv.load_dotenv()

os.environ["OPENAI_API_KEY"] = os.getenv("OPENAI_API_KEY")
os.environ["OPENAI_BASE_URL"] = os.getenv("OPENAI_BASE_URL")
from langchain.chat_models import init_chat_model

model = init_chat_model(
    model="openai:gpt-4o-mini",
    temperature=0.5,
)
```

或从特定厂商集成包中初始化

```python
import os
import dotenv
from langchain_openai import ChatOpenAI

dotenv.load_dotenv()

os.environ["OPENAI_API_KEY"] = os.getenv("OPENAI_API_KEY")
os.environ["OPENAI_BASE_URL"] = os.getenv("OPENAI_BASE_URL")
model = ChatOpenAI(model="gpt-4o-mini", temperature=0.7)
```

### 关键调用方法

| 方法       | 说明                                               |
| ---------- | -------------------------------------------------- |
| **Invoke** | 模型接受消息作为输入，并在生成完整响应后输出消息。 |
| **Stream** | 调用模型，但实时流式传输生成的输出。               |
| **Batch**  | 将多个请求批量发送给模型，以实现更高效的处理。     |

## 参数

API文档：[模型 | LangChain 参考](https://reference.langchain.org.cn/python/langchain/models/)

| 参数          | 类型   | 必填 | 说明                                                         |
| ------------- | ------ | ---- | ------------------------------------------------------------ |
| `model`       | string | 是   | 您想使用的特定模型的名称或标识符。                           |
| `api_key`     | string | 否   | 用于向模型提供商进行身份验证的密钥。通常在注册访问模型时颁发。通常通过设置**环境变量**访问。 |
| `temperature` | number | 否   | 控制模型输出的随机性。值越高，响应越具创造性；值越低，响应越确定性。 |
| `timeout`     | number | 否   | 在取消请求之前等待模型响应的最大时间（秒）。                 |
| `max_tokens`  | number | 否   | 限制响应中的**令牌**总数，有效控制输出长度。                 |
| `max_retries` | number | 否   | 如果因网络超时或速率限制等问题而失败，系统将重新发送请求的最大尝试次数。 |

> 每个聊天模型集成可能具有用于控制提供商特定功能的额外参数。例如，[`ChatOpenAI`](https://reference.langchain.com/python/integrations/langchain_openai/ChatOpenAI/) 具有 `use_responses_api` 以决定是否使用 OpenAI Responses 或 Completions API。
> 要查找给定聊天模型支持的所有参数，请转到[聊天模型集成](https://langchain-doc.cn/v1/python/integrations/chat)页面。

## 调用

### Invoke

阻塞式调用

```python
response = model.invoke("为什么鹦鹉有五颜六色的羽毛？")
print(response)
```

```python
from langchain.messages import HumanMessage, AIMessage, SystemMessage

conversation = [
    {"role": "system", "content": "你是一个将英语翻译成法语的有用助手。"},
    {"role": "user", "content": "翻译：我喜欢编程。"},
    {"role": "assistant", "content": "J'adore la programmation."},
    {"role": "user", "content": "翻译：我喜欢构建应用程序。"}
]

response = model.invoke(conversation)
print(response)  # AIMessage("J'adore créer des applications.")
```

```python
from langchain_core.messages import HumanMessage, AIMessage, SystemMessage

conversation = [
    SystemMessage("你是一个将英语翻译成法语的有用助手。"),
    HumanMessage("翻译：我喜欢编程。"),
    AIMessage("J'adore la programmation。"),
    HumanMessage("翻译：我喜欢构建应用程序。")
]

response = model.invoke(conversation)
print(response)  # AIMessage("J'adore créer des applications。")
```

### Stream

流式调用

大多数模型可以在生成时流式传输其输出内容。通过逐步显示输出，流式传输显著改善了用户体验，尤其是对于较长的响应。

调用 [`stream()`](https://reference.langchain.com/python/langchain_core/language_models/#langchain_core.language_models.chat_models.BaseChatModel.stream) 返回一个**迭代器**，它在生成时逐块产生输出。您可以使用循环实时处理每个块：

```python
for chunk in model.stream("为什么鹦鹉有五颜六色的羽毛？"):
    print(chunk.text, end="", flush=True)
```

与返回单个 [`AIMessage`](https://reference.langchain.com/python/langchain/messages/#langchain.messages.AIMessage) 的 [`invoke()`](https://langchain-doc.cn/v1/python/langchain/models.html#invoke) 不同，`stream()` 返回多个 [`AIMessageChunk`](https://reference.langchain.com/python/langchain/messages/#langchain.messages.AIMessageChunk) 对象，每个对象包含一部分输出文本。重要的是，流中的每个块都设计为通过求和聚合成完整消息：

```python
full = None  # None | AIMessageChunk
for chunk in model.stream("天空是什么颜色？"):
    full = chunk if full is None else full + chunk
    print(full.text)
# 天空
# 天空是
# 天空通常
# 天空通常是蓝色
```

### Batch

将一组独立请求批量处理给模型可以显著提高性能并降低成本，因为处理可以并行进行：

```python
responses = model.batch([
    "为什么鹦鹉有五颜六色的羽毛？",
    "飞机是如何飞行的？",
    "什么是量子计算？"
])
for response in responses:
    print(response)
```

默认情况下，[`batch()`](https://reference.langchain.com/python/langchain_core/language_models/#langchain_core.language_models.chat_models.BaseChatModel.batch) 仅返回整个批次的最终输出。如果您希望在每个单独输入完成生成时接收输出，可以使用 [`batch_as_completed()`](https://reference.langchain.com/python/langchain_core/language_models/#langchain_core.language_models.chat_models.BaseChatModel.batch_as_completed) 流式传输结果：

```python
for response in model.batch_as_completed([
    "为什么鹦鹉有五颜六色的羽毛？",
    "飞机是如何飞行的？",
    "什么是量子计算？"
]):
    print(response)
```

## 工具调用

模型可以请求调用执行任务的工具，例如从数据库获取数据、搜索网络或运行代码。工具是以下内容的配对：

1. 架构，包括工具的名称、描述和/或参数定义（通常是 JSON 架构）
2. 要执行的函数或**协程**

要使您定义的工具可供模型使用，您必须使用 [`bind_tools()`](https://reference.langchain.com/python/langchain_core/language_models/#langchain_core.language_models.chat_models.BaseChatModel.bind_tools) 绑定它们。在后续调用中，模型可以根据需要选择调用任何绑定的工具。

```python
from langchain.tools import tool

@tool
def get_weather(location: str) -> str:
    """获取某个位置的天气。"""
    return f"{location} 天气晴朗。"

model_with_tools = model.bind_tools([get_weather])  # [!code highlight]

response = model_with_tools.invoke("波士顿的天气怎么样？")
for tool_call in response.tool_calls:
    # 查看模型发出的工具调用
    print(f"工具：{tool_call['name']}")
    print(f"参数：{tool_call['args']}")
```

**Agent与大模型区别**：绑定用户定义的工具时，模型的响应包括**请求**执行工具。当将模型与[代理](https://langchain-doc.cn/v1/python/langchain/agents)分开使用时，您需要执行请求的操作并将结果返回给模型以用于后续推理。请注意，当使用[代理](https://langchain-doc.cn/v1/python/langchain/agents)时，代理循环将为您处理工具执行循环。也就是说：大模型知道需要调用tool，但是无法真正的调用tool函数，而Agent自动执行tool

### 工具执行循环

当模型返回工具调用时，您需要执行工具并将结果传递回模型。这会创建一个对话循环，模型可以使用工具结果生成其最终响应。LangChain 包含[代理](https://langchain-doc.cn/v1/python/langchain/agents)抽象来为您处理此协调。

```python
from langchain_core.messages import AIMessage, HumanMessage
# 将（可能多个）工具绑定到模型
model_with_tools = model.bind_tools([get_weather])

# 步骤 1：模型生成工具调用
messages = [HumanMessage(content="波士顿的天气怎么样？")]
ai_msg = model_with_tools.invoke(messages)
messages.append(ai_msg)

# 步骤 2：执行工具并收集结果
for tool_call in ai_msg.tool_calls:
    # 使用生成的参数执行工具
    tool_result = get_weather.invoke(tool_call)
    messages.append(tool_result)

# 步骤 3：将结果传递回模型以获取最终响应
final_response = model_with_tools.invoke(messages)
print(final_response.text)
# "波士顿当前天气为 72°F，晴朗。"
```

### 强制工具调用

默认情况下，模型可以根据用户输入自由选择使用哪个绑定的工具。但是，您可能希望强制选择工具，确保模型使用特定工具或给定列表中的**任何**工具：

```python
model_with_tools = model.bind_tools([tool_1], tool_choice="any")
model_with_tools = model.bind_tools([tool_1], tool_choice="tool_1")
```

### 并行工具调用

许多模型支持在适当时并行调用多个工具。这允许模型同时从不同来源收集信息。

模型根据请求操作的独立性智能地确定何时适合并行执行。

```python
model_with_tools = model.bind_tools([get_weather])

response = model_with_tools.invoke(
    "波士顿和东京的天气怎么样？"
)

# 模型可能会生成多个工具调用
print(response.tool_calls)
# [
#   {'name': 'get_weather', 'args': {'location': 'Boston'}, 'id': 'call_1'},
#   {'name': 'get_weather', 'args': {'location': 'Tokyo'}, 'id': 'call_2'},
# ]

# 执行所有工具（可以使用 async 并行执行）
results = []
for tool_call in response.tool_calls:
    if tool_call['name'] == 'get_weather':
        result = get_weather.invoke(tool_call)
    ...
    results.append(result)
```



>**提示**
>大多数支持工具调用的模型默认启用并行工具调用。某些模型（包括 [OpenAI](https://langchain-doc.cn/v1/python/integrations/chat/openai) 和 [Anthropic](https://langchain-doc.cn/v1/python/integrations/chat/anthropic)）允许您禁用此功能。要执行此操作，请设置 `parallel_tool_calls=False`：

```python
model.bind_tools([get_weather], parallel_tool_calls=False)
```

### 流式传输工具调用

在流式传输响应时，工具调用通过 [`ToolCallChunk`](https://reference.langchain.com/python/langchain/messages/#langchain.messages.ToolCallChunk) 逐步构建。这允许您在生成工具调用时查看它们，而不是等待完整响应。

```python
for chunk in model_with_tools.stream(
    "波士顿和东京的天气怎么样？"
):
    # 工具调用块逐步到达
    for tool_chunk in chunk.tool_call_chunks:
        if name := tool_chunk.get("name"):
            print(f"工具：{name}")
        if id_ := tool_chunk.get("id"):
            print(f"ID：{id_}")
        if args := tool_chunk.get("args"):
            print(f"参数：{args}")

# 输出：
# 工具：get_weather
# ID：call_SvMlU1TVIZugrFLckFE2ceRE
# 参数：{"lo
# 参数：catio
# 参数：n": "B
# 参数：osto
# 参数：n"}
# 工具：get_weather
# ID：call_QMZdy6qInx13oWKE7KhuhOLR
# 参数：{"lo
# 参数：catio
# 参数：n": "T
# 参数：okyo
# 参数："}
```

## 结构化输出

可以请求模型以匹配给定架构的格式提供其响应。这对于确保输出易于解析并用于后续处理非常有用。LangChain 支持多种架构类型和强制执行结构化输出的方法。

- **Pydantic**（字段校验、描述、嵌套结构，功能最丰富）
-  **TypedDict（**轻量类型约束）
-  **JSON Schema**（与前后端/跨语言接口最通用） 
- **dataclass**

**只有Pydantic返回的是Schema类实例，其余三种方式返回的都是`dict`**

并不是所有模型都支持结构化输出，如果不支持，LangChain 会回退到提示词 + JSON 解析。

### Pydantic

[Pydantic 模型](https://docs.pydantic.dev/latest/concepts/models/#basic-model-usage) 提供最丰富的功能集，包括字段验证、描述和嵌套结构。

```python
from pydantic import BaseModel, Field

class Movie(BaseModel):
    """一部带有详细信息的电影。"""
    title: str = Field(..., description="电影标题")
    year: int = Field(..., description="电影上映年份")
    director: str = Field(..., description="电影导演")
    rating: float = Field(..., description="电影评分，满分 10 分")

model_with_structure = model.with_structured_output(Movie)
response = model_with_structure.invoke("提供关于电影《盗梦空间》的详细信息")
print(response)  # Movie(title="Inception", year=2010, director="Christopher Nolan", rating=8.8)
```

输出：

```python
title='盗梦空间' year=2010 director='克里斯托弗·诺兰' rating=8.8
```

### TypeDict

`TypedDict` 提供使用 Python 内置类型的更简单替代方案，适用于不需要运行时验证的情况。

```python
from typing_extensions import TypedDict, Annotated

class MovieDict(TypedDict):
    """一部带有详细信息的电影。"""
    title: Annotated[str, ..., "电影标题"]
    year: Annotated[int, ..., "电影上映年份"]
    director: Annotated[str, ..., "电影导演"]
    rating: Annotated[float, ..., "电影评分，满分 10 分"]

model_with_structure = model.with_structured_output(MovieDict)
response = model_with_structure.invoke("提供关于电影《盗梦空间》的详细信息")
print(response)  # {'title': 'Inception', 'year': 2010, 'director': 'Christopher Nolan', 'rating': 8.8}
```

输出

```python
{'title': '盗梦空间', 'year': 2010, 'director': '克里斯托弗·诺兰', 'rating': 8.8}
```

### Json Schema

为了获得最大控制或互操作性，您可以提供原始 JSON 架构。

```python
import json

json_schema = {
    "title": "Movie",
    "description": "一部带有详细信息的电影",
    "type": "object",
    "properties": {
        "title": {
            "type": "string",
            "description": "电影标题"
        },
        "year": {
            "type": "integer",
            "description": "电影上映年份"
        },
        "director": {
            "type": "string",
            "description": "电影导演"
        },
        "rating": {
            "type": "number",
            "description": "电影评分，满分 10 分"
        }
    },
    "required": ["title", "year", "director", "rating"]
}

model_with_structure = model.with_structured_output(
    json_schema,
    method="json_schema",
)
response = model_with_structure.invoke("提供关于电影《盗梦空间》的详细信息")
print(response)  # {'title': 'Inception', 'year': 2010, ...}
```

> **注意**
> **结构化输出的关键考虑因素：**
>
> - **方法参数**：某些提供商支持不同的方法（`'json_schema'`、`'function_calling'`、`'json_mode'`）
>   - `'json_schema'` 通常指提供商提供的专用结构化输出功能
>   - `'function_calling'` 通过强制[工具调用](https://langchain-doc.cn/v1/python/langchain/models.html#工具调用)遵循给定架构来派生结构化输出
>   - `'json_mode'` 是某些提供商提供的 `'json_schema'` 的前身——它生成有效的 JSON，但架构必须在提示中描述
> - **包含原始**：使用 `include_raw=True` 以同时获取已解析的输出和原始 AI 消息
> - **验证**：只有Pydantic 模型提供自动验证，而 `TypedDict` 和 JSON Schema 需要手动验证

#### 示例：消息输出与解析结构并存

返回原始 [`AIMessage`](https://reference.langchain.com/python/langchain/messages/#langchain.messages.AIMessage) 对象与解析表示一起以访问响应元数据（如[令牌计数](https://langchain-doc.cn/v1/python/langchain/models.html#令牌使用情况)）可能很有用。为此，在调用 [`with_structured_output`](https://reference.langchain.com/python/langchain_core/language_models/#langchain_core.language_models.chat_models.BaseChatModel.with_structured_output) 时设置 [`include_raw=True`](https://reference.langchain.com/python/langchain_core/language_models/#langchain_core.language_models.chat_models.BaseChatModel.with_structured_output(include_raw))：

```python
from pydantic import BaseModel, Field

class Movie(BaseModel):
    """一部带有详细信息的电影。"""
    title: str = Field(..., description="电影标题")
    year: int = Field(..., description="电影上映年份")
    director: str = Field(..., description="电影导演")
    rating: float = Field(..., description="电影评分，满分 10 分")

model_with_structure = model.with_structured_output(Movie, include_raw=True)  # [!code highlight]
response = model_with_structure.invoke("提供关于电影《盗梦空间》的详细信息")
response
# {
#     "raw": AIMessage(...),
#     "parsed": Movie(title=..., year=..., ...),
#     "parsing_error": None,
# }
```



#### 示例：嵌套结构

```python
from pydantic import BaseModel, Field

class Actor(BaseModel):
    name: str = Field(..., description="演员名称")
    role: str = Field(..., description="角色名称")

class MovieDetails(BaseModel):
    title: str = Field(..., description="电影标题")
    year: int = Field(..., description="电影上映年份")
    cast: list[Actor] = Field(..., description="主演列表")
    genres: list[str] = Field(..., description="电影类别")
    budget: float | None = Field(None, description="预算（百万美元）")

model_with_structure = model.with_structured_output(MovieDetails)
response = model_with_structure.invoke("关于电影《唐顿庄园》的详细信息")
print(response)
```

```python
title='唐顿庄园' year=2019 cast=[Actor(name='休·博纳维尔', role='罗伯特·克劳利'), Actor(name='玛吉·史密斯', role='维奥莱特·克劳利'), Actor(name='米歇尔·道克瑞', role='玛丽·克劳利'), Actor(name='吉姆·卡特', role='查尔斯·帕金斯'), Actor(name='伊丽莎白·麦戈文', role='柯拉·克劳利')] genres=['剧情', '历史', '爱情'] budget=0.0
```



## 高级主题

### 多模态

某些模型可以处理和返回非文本数据，如图像、音频和视频。您可以通过提供[内容块](https://langchain-doc.cn/v1/python/langchain/messages#message-content)将非文本数据传递给模型。

> **提示**
> 所有具有底层多模态功能的 LangChain 聊天模型都支持：
>
> 1. 跨提供商标准格式的数据（请参阅[我们的消息指南](https://langchain-doc.cn/v1/python/langchain/messages)）
> 2. OpenAI [聊天完成](https://platform.openai.com/docs/api-reference/chat)格式
> 3. 特定于该提供商的任何格式（例如，Anthropic 模型接受 Anthropic 原生格式）

有关详细信息，请参阅消息指南的[多模态部分](https://langchain-doc.cn/v1/python/langchain/messages#multimodal)。

> **提示**
> 并非所有 LLM 都是平等的！
> 某些模型可以作为其响应的一部分返回多模态数据。如果调用它们这样做，则生成的 [`AIMessage`](https://reference.langchain.com/python/langchain/messages/#langchain.messages.AIMessage) 将具有多模态类型的内容块。

```python
response = model.invoke("生成一张猫的图片", tools=[{"type": "image_generation"}])
print(response.content_blocks)

```





# 消息

消息是 LangChain 中模型上下文的基本单位。它们代表模型的输入和输出，携带内容和元数据，用于在与 LLM 交互时表示对话状态。

消息是包含以下内容的对象：

- **角色** - 标识消息类型（例如 `system`、`user`）
- **内容** - 表示消息的实际内容（例如文本、图像、音频、文档等）
- **元数据** - 可选字段，例如响应信息、消息 ID 和令牌使用情况

LangChain 提供了一种标准消息类型，可在所有模型提供商之间工作，确保无论调用哪个模型都能保持一致的行为。

## 基本用法

```python
from langchain_core.messages import (AIMessage, HumanMessage, SystemMessage)
messages = [
    SystemMessage(content="你是一个生物学专家"),
    HumanMessage(content="什么是杂交水稻？"),
]
stream = model.stream(messages)
full = next(stream)
for chunk in stream:
    full += chunk
    print(chunk.text, end="", flush=True)
```

- Invoke调用返回的是AIMessage
- stream调用返回AIMessageChunk

### 单文本格式

文本提示是字符串 - 适用于不需要保留对话历史的简单生成任务。

```python
response = model.invoke("Write a haiku about spring")
```

**何时使用文本提示：**

- 只有一个独立的请求
- 不需要对话历史
- 希望代码复杂度最小

### 对象格式

或者，您可以通过提供消息对象列表将消息列表传递给模型。

```python
from langchain.messages import SystemMessage, HumanMessage, AIMessage

messages = [
    SystemMessage("You are a poetry expert"),
    HumanMessage("Write a haiku about spring"),
    AIMessage("Cherry blossoms bloom...")
]
response = model.invoke(messages)
```

**何时使用消息提示：**

- 管理多轮对话
- 处理多模态内容（图像、音频、文件）
- 包含系统指令

### Json格式

您还可以直接以 OpenAI 聊天补全格式指定消息。

```python
messages = [
    {"role": "system", "content": "You are a poetry expert"},
    {"role": "user", "content": "Write a haiku about spring"},
    {"role": "assistant", "content": "Cherry blossoms bloom..."}
]
response = model.invoke(messages)
```

### 二元组格式

```python
messages = [
    ("system",  "You are a poetry expert"),
    ("user", "Write a haiku about spring"),
    ("ai", "Cherry blossoms bloom...")
]
response = model.invoke(messages)
print(response.content)
```

## 消息类型

- [系统消息](https://langchain-doc.cn/v1/python/langchain/messages.html#系统消息) - 告诉模型如何行为并为交互提供上下文
- [人类消息](https://langchain-doc.cn/v1/python/langchain/messages.html#人类消息) - 表示用户输入和与模型的交互
- [AI 消息](https://langchain-doc.cn/v1/python/langchain/messages.html#ai-消息) - 模型生成的响应，包括文本内容、工具调用和元数据
- [工具消息](https://langchain-doc.cn/v1/python/langchain/messages.html#工具消息) - 表示[工具调用](https://langchain-doc.cn/v1/python/langchain/models#tool-calling)的输出

### 系统消息

[`SystemMessage`](https://reference.langchain.com/python/langchain/messages/#langchain.messages.SystemMessage) 表示一组初始指令，用于引导模型的行为。您可以使用系统消息来设置语气、定义模型角色并建立响应指南。

### 人类消息

[`HumanMessage`](https://reference.langchain.com/python/langchain/messages/#langchain.messages.HumanMessage) 表示用户输入和交互。它们可以包含文本、图像、音频、文件以及任何其他多模态[内容](https://langchain-doc.cn/v1/python/langchain/messages.html#消息内容)。

#### 文本内容

```python
response = model.invoke([
    HumanMessage("What is machine learning?")
])
# 使用字符串是单个 HumanMessage 的快捷方式
response = model.invoke("What is machine learning?")
```

#### 消息元数据

```python
human_msg = HumanMessage(
    content="Hello!",
    name="alice",  # 可选：标识不同用户
    id="msg_123",  # 可选：用于追踪的唯一标识符
)
```

> **注意**
> `name` 字段的行为因提供商而异 - 有些用于用户识别，其他忽略它。要检查，请参考模型提供商的[参考文档](https://reference.langchain.com/python/integrations/)。

### AI消息

[`AIMessage`](https://reference.langchain.com/python/langchain/messages/#langchain.messages.AIMessage) 表示模型调用的输出。它们可以包含多模态数据、工具调用和提供商特定的元数据，您可以稍后访问。

```python
response = model.invoke("Explain AI")
print(type(response))  # <class 'langchain_core.messages.AIMessage'>
```

[`AIMessage`](https://reference.langchain.com/python/langchain/messages/#langchain.messages.AIMessage) 对象由调用模型时返回，其中包含响应中的所有关联元数据。

提供商对不同类型的消息的权重/上下文处理不同，这意味着有时手动创建新的 [`AIMessage`](https://reference.langchain.com/python/langchain/messages/#langchain.messages.AIMessage) 对象并将其插入消息历史中就像来自模型一样很有帮助。

```python
from langchain.messages import AIMessage, SystemMessage, HumanMessage

# 手动创建 AI 消息（例如，用于对话历史）
ai_msg = AIMessage("I'd be happy to help you with that question!")

# 添加到对话历史
messages = [
    SystemMessage("You are a helpful assistant"),
    HumanMessage("Can you help me?"),
    ai_msg,  # 插入就像来自模型一样
    HumanMessage("Great! What's 2+2?")
]

response = model.invoke(messages)
```



属性：

- **text** (`string`)
  消息的文本内容。
- **content** (`string | dict[]`)
  消息的原始内容。
- **content_blocks** (`ContentBlock[]`)
  消息的标准化[内容块](https://langchain-doc.cn/v1/python/langchain/messages.html#消息内容)。
- **tool_calls** (`dict[] | None`)
  模型进行的工具调用。如果没有调用工具，则为空。
- **id** (`string`)
  消息的唯一标识符（由 LangChain 自动生成或在提供商响应中返回）
- **usage_metadata** (`dict | None`)
  消息的使用元数据，可包含可用时的令牌计数。
- **response_metadata** (`ResponseMetadata | None`)
  消息的响应元数据。

#### 工具调用

当模型进行[工具调用](https://langchain-doc.cn/v1/python/langchain/models#tool-calling)时，它们包含在 [`AIMessage`](https://reference.langchain.com/python/langchain/messages/#langchain.messages.AIMessage) 中：

```python
def get_weather(location: str) -> str:
    """Get the weather at a location."""
    return f"{location} is sunny all week!"


model_with_tools = model.bind_tools([get_weather])
response = model_with_tools.invoke("What's the weather in Paris?")

for tool_call in response.tool_calls:
    print(f"Tool: {tool_call['name']}")
    print(f"Args: {tool_call['args']}")
    print(f"ID: {tool_call['id']}")
```

其他结构化数据（如推理或引用）也可以出现在消息[内容](https://langchain-doc.cn/v1/python/langchain/messages#消息内容)中。

#### token使用情况

[`AIMessage`](https://reference.langchain.com/python/langchain/messages/#langchain.messages.AIMessage) 可以在其 [`usage_metadata`](https://reference.langchain.com/python/langchain/messages/#langchain.messages.AIMessage.usage_metadata) 字段中保存令牌计数和其他使用元数据：

```python
response.usage_metadata
```

```txt
{'input_tokens': 8,
 'output_tokens': 304,
 'total_tokens': 312,
 'input_token_details': {'audio': 0, 'cache_read': 0},
 'output_token_details': {'audio': 0, 'reasoning': 256}}
```

#### 流式传输和块

在流式传输期间，您将收到可以组合成完整消息对象的 [`AIMessageChunk`](https://reference.langchain.com/python/langchain/messages/#langchain.messages.AIMessageChunk) 对象：

```python
chunks = []
full_message = None
for chunk in model.stream("Hi"):
    chunks.append(chunk)
    print(chunk.text, end='', flush=True)
    full_message = chunk if full_message is None else full_message + chunk
```



#### AIMessage内容

```txt
('content', 'AI回答用户内容')
('additional_kwargs', {'refusal': None, 'reasoning_content': '推理内容'})
('response_metadata', {'token_usage': {'completion_tokens': 293, 'prompt_tokens': 21, 'total_tokens': 314, 'completion_tokens_details': {'accepted_prediction_tokens': None, 'audio_tokens': None, 'reasoning_tokens': 272, 'rejected_prediction_tokens': None}, 'prompt_tokens_details': {'audio_tokens': None, 'cached_tokens': 0}}, 'model_provider': 'deepseek', 'model_name': 'deepseek-v4-flash', 'system_fingerprint': None, 'id': 'chatcmpl-2e536851-12f0-9ddb-8385-d0f5b4119bef', 'finish_reason': 'stop', 'logprobs': None})
('type', 'ai')
('name', None)
('id', 'lc_run--019e1fc0-61c6-7223-b9cc-96adc8fd8a2d-0')
('tool_calls', [])
('invalid_tool_calls', [])
('usage_metadata', {'input_tokens': 21, 'output_tokens': 293, 'total_tokens': 314, 'input_token_details': {'cache_read': 0}, 'output_token_details': {'reasoning': 272}})
```



### 工具消息

对于支持[工具调用](https://langchain-doc.cn/v1/python/langchain/models#tool-calling)的模型，AI 消息可以包含工具调用。工具消息用于将单个工具执行的结果传回模型。

[工具](https://langchain-doc.cn/v1/python/langchain/tools) 可以直接生成 [`ToolMessage`](https://reference.langchain.com/python/langchain/messages/#langchain.messages.ToolMessage) 对象。下面展示一个简单示例。有关更多信息，请阅读[工具指南](https://langchain-doc.cn/v1/python/langchain/tools)。

```python
# 模型进行工具调用后
ai_message = AIMessage(
    content=[],
    tool_calls=[{
        "name": "get_weather",
        "args": {"location": "San Francisco"},
        "id": "call_123"
    }]
)

# 执行工具并创建结果消息
weather_result = "Sunny, 72°F"
tool_message = ToolMessage(
    content=weather_result,
    tool_call_id="call_123"  # 必须匹配调用 ID
)

# 继续对话
messages = [
    HumanMessage("What's the weather in San Francisco?"),
    ai_message,  # 模型的工具调用
    tool_message,  # 工具执行结果
]
response = model.invoke(messages)  # 模型处理结果
```

属性：

- **content** (`string`, 必需)
  工具调用的字符串化输出。
- **tool_call_id** (`string`, 必需)
  此消息响应的工具调用的 ID。（必须匹配 [`AIMessage`](https://reference.langchain.com/python/langchain/messages/#langchain.messages.AIMessage) 中的工具调用 ID）
- **name** (`string`, 必需)
  被调用的工具的名称。ToolMessage必须紧邻匹配的AIMessage，和前者tool_calls中的id一 致。
- **artifact** (`dict`)
  不发送给模型但可以以编程方式访问的附加数据。

>**注意**
>`artifact` 字段存储不会发送给模型但可以以编程方式访问的补充数据。这对于存储原始结果、调试信息或下游处理的数据而不会使模型上下文杂乱很有用。



## 消息内容

### content属性

您可以将消息的内容视为发送给模型的数据负载。消息具有一个松散类型的 `content` 属性，支持字符串和未类型对象列表（例如字典）。这允许在 LangChain 聊天模型中直接支持提供商原生结构，例如[多模态](https://langchain-doc.cn/v1/python/langchain/messages.html#多模态)内容和其他数据。

LangChain 另外为文本、推理、引用、多模态数据、服务器端工具调用和其他消息内容提供了专用内容类型。请参阅下面的[标准内容块](https://langchain-doc.cn/v1/python/langchain/messages.html#标准内容块)。

LangChain 聊天模型接受 `content` 属性中的消息内容，可以包含：

1. 一个字符串
2. 提供商原生格式的内容块列表
3. [LangChain 的标准内容块](https://langchain-doc.cn/v1/python/langchain/messages.html#标准内容块)列表

```python
from langchain.messages import HumanMessage

# 字符串内容
human_message = HumanMessage("Hello, how are you?")

# 提供商原生格式（例如 OpenAI）
human_message = HumanMessage(content=[
    {"type": "text", "text": "Hello, how are you?"},
    {"type": "image_url", "image_url": {"url": "https://example.com/image.jpg"}}
])

# 标准内容块列表
human_message = HumanMessage(content_blocks=[
    {"type": "text", "text": "Hello, how are you?"},
    {"type": "image", "url": "https://example.com/image.jpg"},
])
```

> **提示**
> 在初始化消息时指定 `content_blocks` 仍将填充消息 `content`，但提供了一个类型安全的接口

### 标准内容块

LangChain 提供了一种跨提供商工作的消息内容的标准表示`content_blocks`。

消息对象实现了 `content_blocks` 属性，该属性将延迟解析 `content` 属性为标准、类型安全的表示。例如，从 [ChatAnthropic](https://langchain-doc.cn/v1/python/integrations/chat/anthropic) 或 [ChatOpenAI](https://langchain-doc.cn/v1/python/integrations/chat/openai) 生成的消息将以各自提供商的格式包含 `thinking` 或 `reasoning` 块，但可以延迟解析为一致的 [`ReasoningContentBlock`](https://langchain-doc.cn/v1/python/langchain/messages.html#内容块参考) 表示：

```python
from langchain.messages import AIMessage

message = AIMessage(
    content=[
        {"type": "thinking", "thinking": "...", "signature": "WaUjzkyp..."},
        {"type": "text", "text": "..."},
    ],
    response_metadata={"model_provider": "anthropic"}
)
message.content_blocks
```

```txt
[{'type': 'reasoning',
  'reasoning': '...',
  'extras': {'signature': 'WaUjzkyp...'}},
 {'type': 'text', 'text': '...'}]
```

```python
from langchain.messages import AIMessage

message = AIMessage(
    content=[
        {
            "type": "reasoning",
            "id": "rs_abc123",
            "summary": [
                {"type": "summary_text", "text": "summary 1"},
                {"type": "summary_text", "text": "summary 2"},
            ],
        },
        {"type": "text", "text": "...", "id": "msg_abc123"},
    ],
    response_metadata={"model_provider": "openai"}
)
message.content_blocks
```

```txt
[{'type': 'reasoning', 'id': 'rs_abc123', 'reasoning': 'summary 1'},
 {'type': 'reasoning', 'id': 'rs_abc123', 'reasoning': 'summary 2'},
 {'type': 'text', 'text': '...', 'id': 'msg_abc123'}]
```

> **序列化标准内容**
> 如果 LangChain 之外的应用程序需要访问标准内容块表示，您可以选择将内容块存储在消息内容中。
>
> 为此，您可以将 `LC_OUTPUT_VERSION` 环境变量设置为 `v1`。或者，使用 `output_version="v1"` 初始化任何聊天模型：
>
> ```python
> from langchain.chat_models import init_chat_model
> LC_OUTPUT_VERSION='v1'
> # 或者
> model = init_chat_model("openai:gpt-5-nano", output_version="v1")
> ```

- **`content`**：是**原始、松散类型**的存储字段，向后兼容。它的类型可以是
  - **字符串**：纯文本内容。
  - **字典列表**：提供商原生格式（如 OpenAI 的多模态内容块列表）。
- **`content_blocks`**：是LangChain**标准化、类型安全**的解析视图，面向现代多模态和工具调用场景。类型是`list[ContentBlock]`（类型安全的联合类型）

使用建议：

- **简单文本场景**：直接用 `content` 即可，例如 `print(msg.content)`。
- **多模态或工具调用场景**：推荐使用 `content_blocks`，因为它能保留完整的结构化信息。



### content_blocks包含字段

| `type` 值            | 含义                                               | 包含的字段                                      | 示例                                                         |
| -------------------- | -------------------------------------------------- | ----------------------------------------------- | ------------------------------------------------------------ |
| `"text"`             | 纯文本内容                                         | `text` (str)                                    | `{"type": "text", "text": "你好"}`                           |
| `"image"`            | 图片内容                                           | `url` (str), `mime_type` (str, 可选)            | `{"type": "image", "url": "data:image/png;base64,..."}`      |
| `"tool_use"`         | 模型发起的工具调用请求                             | `id` (str), `name` (str), `input` (dict)        | `{"type": "tool_use", "name": "get_weather", "input": {"city": "北京"}}` |
| `"tool_result"`      | 工具执行后的返回结果                               | `id` (str), `content` (str), `name` (str, 可选) | `{"type": "tool_result", "id": "call_xxx", "content": "晴天"}` |
| `"reasoning"`        | 模型的推理/思考过程（如 Anthropic 的 thinking 块） | `reasoning` (str), `extras` (dict, 可选)        | `{"type": "reasoning", "reasoning": "用户想查询天气..."}`    |
| `"code"`             | 代码块                                             | `code` (str), `language` (str, 可选)            | `{"type": "code", "code": "print('hello')", "language": "python"}` |
| `"execution_output"` | 代码执行后的输出结果                               | `output` (str)                                  | `{"type": "execution_output", "output": "hello"}`            |
| `"error"`            | 错误信息                                           | `error` (str)                                   | `{"type": "error", "error": "工具调用失败"}`                 |
| `"unknown"`          | 无法识别的原始内容块（兜底类型）                   | `raw` (dict)                                    | `{"type": "unknown", "raw": {"custom_field": "..."}}`        |



## 多模态消息

聊天模型可以接受多模态数据作为输入并生成作为输出。下面展示包含多模态数据的输入消息的简短示例。

> **注意**
> 额外键可以包含在内容块的顶层或嵌套在 `"extras": {"key": value}` 中。
>
> 例如，[OpenAI](https://langchain-doc.cn/v1/python/integrations/chat/openai#pdfs) 和 [AWS Bedrock Converse](https://langchain-doc.cn/v1/python/integrations/chat/bedrock) 对于 PDF 需要文件名。请参阅您选择的模型的[提供商页面](https://langchain-doc.cn/v1/python/integrations/providers/overview)以获取具体信息

### 传入图片

模型必须具有多模态能力

```python
# 方式 A：传入图片 URL
message = HumanMessage(
    content=[
        {"type": "text", "text": "请描述这张图片的内容。"},
        {"type": "image", "url": "https://pixnio.com/free-images/2026/01/20/2026-01-20-21-28-45-960x540.jpg"}
    ]
)

# 方式 B：传入本地图片（Base64 编码）
with open("girl.jpg", "rb") as f:
    image_data = base64.b64encode(f.read()).decode("utf-8")

message = HumanMessage(
    content=[
        {"type": "text", "text": "请描述这张图片的内容"},
        {"type": "image_url", "image_url": {"url": f"data:image/jpeg;base64,{image_data}"}}
    ]
)


response = model.stream([message])
full = next(response)
for chunk in response:
    full += chunk
    print(chunk.content, end="", flush=True)
    
    
# 方式3 从提供商管理的文件 ID
message = {
    "role": "user",
    "content": [
        {"type": "text", "text": "Describe the content of this image."},
        {"type": "image", "file_id": "file-abc123"},
    ]
}
```

### PDF文档输入

截止2026-5，绝大多数模型已经平台提供的模型不支持url方式输入PDF

```python
# 从 URL
message = {
    "role": "user",
    "content": [
        {"type": "text", "text": "Describe the content of this document."},
        {"type": "file", "url": "https://example.com/path/to/document.pdf"},
    ]
}

```

可行方式：

方式1 :需要在代码中**手动完成下载和解析**，再将文本内容喂给模型：

```python
import requests
from langchain_community.document_loaders import PyPDFLoader
from langchain_core.messages import HumanMessage

# 1. 下载PDF
pdf_url = "https://arxiv.org/pdf/1706.03762"
response = requests.get(pdf_url)
with open("temp.pdf", "wb") as f:
    f.write(response.content)

# 2. 解析PDF为文本
loader = PyPDFLoader("temp.pdf")
pages = loader.load()
full_text = "\n\n".join([page.page_content for page in pages])
message = HumanMessage(
    content=[
        {"type": "text", "text": "请总结以下内容："},
        {"type": "text", "text": full_text}
    ]
)
# 3. 喂给模型
result = model.invoke([message])
print(result.content)


```

方式2：**可以将 PDF 转为 Base64 格式后直接输入给模型**，而且这是目前很多多模态模型支持的标准做法。

```python
import base64
from langchain_core.messages import HumanMessage

location = "./example.pdf"
# 1. 读取 PDF 并转为 Base64
with open(location, "rb") as f:
    pdf_bytes = f.read()
    pdf_base64 = base64.b64encode(pdf_bytes).decode("utf-8")

# 2. 构造多模态消息（OpenAI 格式）
message = HumanMessage(
    content=[
        {
            "type": "file",
            "base64": "".join(pdf_base64),
            "mime_type": "application/pdf",
        },
        {
            "type": "text",
            "text": "这份文件大概再说什么？"
        }
    ]
)

# 3. 调用模型（需要支持多模态的模型）
response = model.invoke([message])
print(response.content)
```

**注意：**

1.Base64 编码会显著增加数据体积

- Base64 编码会使文件体积增加约 **33%**
- 例如：一个 10MB 的 PDF 编码后约为 **13.3MB**
- 这会导致 Token 消耗大幅增加，API 调用成本上升

**2.** 文件大小限制

- **GPT-4o 系列**：Base64 编码后的数据不能超过 **50MB**
- **Claude 系列**：Base64 编码后的数据不能超过 **32MB**
- 如果 PDF 过大，建议先分割成多个小文件

**3.** Token 消耗估算

- GPT-4o 处理 PDF 时，会将 PDF 的每一页转为图片再分析
- 每页大约消耗 **258 Token**（图片部分）+ 文本 Token
- 一个 10 页的 PDF 大约消耗 **2580 Token**（仅图片部分）

**4.** 模型兼容性

- **不是所有模型都支持** `input_file` 或 `document` 类型
- 纯文本模型（如 `gpt-3.5-turbo`、`deepseek-chat`）**不支持**这种方式
- 使用前请确认模型名称是否支持多模态

**5. 使用建议：**

- 小文件（< 10MB，< 20页）直接使用 Base64 编码传入，简单快捷。
- 大文件（> 10MB 或 > 20页）建议先提取文本，再喂给模型，避免 Token 浪费：
- 扫描件或图表密集的 PDF  . 使用 Base64 编码传入多模态模型，保留原始排版信息。



方式3

```python
# 从提供商管理的文件 ID
message = {
    "role": "user",
    "content": [
        {"type": "text", "text": "Describe the content of this document."},
        {"type": "file", "file_id": "file-abc123"},
    ]
}
```



**音频和视频是“流式信号”，模型天然能理解；而 PDF 是“复合文档”（包含图片，文本，矢量图等内容），模型需要先拆解才能读懂。**



### 视频输入

模型必须支持视频解析

#### 方式1 通过url



#### 方式2 通过base64

```python
import base64
from langchain_core.messages import HumanMessage

location = "C:\\Users\\aurora\\Videos\\test.mp4"
# 1. 读取视频文件并转为 Base64
with open(location, "rb") as f:
    audio_base64 = base64.b64encode(f.read()).decode("utf-8")

# 2. 构造多模态消息
message = HumanMessage(
    content=[
        {
            "type": "video",
            "base64": "".join(audio_base64),
            "mime_type": "video/mp4",
        },
        {
            "type": "text",
            "text": "这段视频中展示了哪些内容？"
        }
    ]
)
# 3. 调用模型
response = model.invoke([message])
print(response.content)
```



### 音频输入

#### 方式1 通过url



#### 方式2 通过base64



### 内容块参考

内容块在创建消息或访问 `content_blocks` 属性时表示为类型化字典列表。列表中的每个项目必须遵守以下块类型之一：

#### 核心

**TextContentBlock**

**用途：** 标准文本输出

- **type** (`string`, 必需)
  始终为 `"text"`
- **text** (`string`, 必需)
  文本内容
- **annotations** (`object[]`)
  文本的注释列表
- **extras** (`object`)
  额外的提供商特定数据

**示例：**

```python
{
    "type": "text",
    "text": "Hello world",
    "annotations": []
}
```

**ReasoningContentBlock**

**用途：** 模型推理步骤

- **type** (`string`, 必需)
  始终为 `"reasoning"`
- **reasoning** (`string`)
  推理内容
- **extras** (`object`)
  额外的提供商特定数据

**示例：**

```python
{
    "type": "reasoning",
    "reasoning": "The user is asking about...",
    "extras": {"signature": "abc123"},
}
```

#### 多模态

**ImageContentBlock**

**用途：** 图像数据

- **type** (`string`, 必需)
  始终为 `"image"`
- **url** (`string`)
  指向图像位置的 URL。
- **base64** (`string`)
  Base64 编码的图像数据。
- **id** (`string`)
  引用外部存储的图像的引用 ID（例如，在提供商的文件系统或存储桶中）。
- **mime_type** (`string`)
  图像 [MIME 类型](https://www.iana.org/assignments/media-types/media-types.xhtml#image)（例如，`image/jpeg`、`image/png`）

**AudioContentBlock**

**用途：** 音频数据

- **type** (`string`, 必需)
  始终为 `"audio"`
- **url** (`string`)
  指向音频位置的 URL。
- **base64** (`string`)
  Base64 编码的音频数据。
- **id** (`string`)
  引用外部存储的音频文件的引用 ID。
- **mime_type** (`string`)
  音频 [MIME 类型](https://www.iana.org/assignments/media-types/media-types.xhtml#audio)（例如，`audio/mpeg`、`audio/wav`）

**VideoContentBlock**

**用途：** 视频数据

- **type** (`string`, 必需)
  始终为 `"video"`
- **url** (`string`)
  指向视频位置的 URL。
- **base64** (`string`)
  Base64 编码的视频数据。
- **id** (`string`)
  引用外部存储的视频文件的引用 ID。
- **mime_type** (`string`)
  视频 [MIME 类型](https://www.iana.org/assignments/media-types/media-types.xhtml#video)（例如，`video/mp4`、`video/webm`）

**FileContentBlock**

**用途：** 通用文件（PDF 等）

- **type** (`string`, 必需)
  始终为 `"file"`
- **url** (`string`)
  指向文件位置的 URL。
- **base64** (`string`)
  Base64 编码的文件数据。
- **id** (`string`)
  引用外部存储的文件的引用 ID。
- **mime_type** (`string`)
  文件 [MIME 类型](https://www.iana.org/assignments/media-types/media-types.xhtml)（例如，`application/pdf`）

**PlainTextContentBlock**

**用途：** 文档文本（`.txt`、`.md`）

- **type** (`string`, 必需)
  始终为 `"text-plain"`
- **text** (`string`)
  文本内容
- **mime_type** (`string`)
  文本的 [MIME 类型](https://www.iana.org/assignments/media-types/media-types.xhtml)（例如，`text/plain`、`text/markdown`）

#### 工具调用

**ToolCall**

**用途：** 函数调用

- **type** (`string`, 必需)
  始终为 `"tool_call"`
- **name** (`string`, 必需)
  要调用的工具的名称
- **args** (`object`, 必需)
  要传递给工具的参数
- **id** (`string`, 必需)
  此工具调用的唯一标识符

**示例：**

```python
{
    "type": "tool_call",
    "name": "search",
    "args": {"query": "weather"},
    "id": "call_123"
}
```

**ToolCallChunk**

**用途：** 流式工具调用片段

- **type** (`string`, 必需)
  始终为 `"tool_call_chunk"`
- **name** (`string`)
  正在调用的工具的名称
- **args** (`string`)
  部分工具参数（可能是不完整的 JSON）
- **id** (`string`)
  工具调用标识符
- **index** (`number | string`)
  此块在流中的位置

**InvalidToolCall**

**用途：** 格式错误的调用，用于捕获 JSON 解析错误。

- **type** (`string`, 必需)
  始终为 `"invalid_tool_call"`
- **name** (`string`)
  未能调用的工具的名称
- **args** (`object`)
  要传递给工具的参数
- **error** (`string`)
  出错的描述

#### 服务器端工具执行

**ServerToolCall**

**用途：** 服务器端执行的工具调用。

- **type** (`string`, 必需)
  始终为 `"server_tool_call"`
- **id** (`string`, 必需)
  与工具调用关联的标识符。
- **name** (`string`, 必需)
  要调用的工具的名称。
- **args** (`string`, 必需)
  部分工具参数（可能是不完整的 JSON）

**ServerToolCallChunk**

**用途：** 流式服务器端工具调用片段

- **type** (`string`, 必需)
  始终为 `"server_tool_call_chunk"`
- **id** (`string`)
  与工具调用关联的标识符。
- **name** (`string`)
  正在调用的工具的名称
- **args** (`string`)
  部分工具参数（可能是不完整的 JSON）
- **index** (`number | string`)
  此块在流中的位置

**ServerToolResult**

**用途：** 搜索结果

- **type** (`string`, 必需)
  始终为 `"server_tool_result"`
- **tool_call_id** (`string`, 必需)
  对应服务器工具调用的标识符。
- **id** (`string`)
  与服务器工具结果关联的标识符。
- **status** (`string`, 必需)
  服务器端工具的执行状态。`"success"` 或 `"error"`。
- **output**
  执行工具的输出。


---





# 工具(欠缺)

**工具** 是 **[代理（agents）](https://langchain-doc.cn/v1/python/langchain/agents)** 调用来执行操作的组件。它们通过允许模型通过定义明确的输入和输出与世界交互来扩展模型的功能。工具封装了一个可调用的函数及其输入架构（schema）。这些可以传递给兼容的 **[聊天模型（chat models）](https://langchain-doc.cn/v1/python/langchain/models)**，让模型决定是否以及使用什么参数来调用工具。在这些场景中，**工具调用** 使模型能够生成符合指定输入架构的请求。

> **服务器端工具使用 (Server-side tool use)**
>
> 某些聊天模型（例如 **[OpenAI](https://langchain-doc.cn/v1/python/integrations/chat/openai)**、**[Anthropic](https://langchain-doc.cn/v1/python/integrations/chat/anthropic)** 和 **[Gemini](https://langchain-doc.cn/v1/python/integrations/chat/google_generative_ai)**）具有 **[内置工具](https://langchain-doc.cn/v1/python/langchain/models#server-side-tool-use)**，这些工具在服务器端执行，例如网络搜索和代码解释器。请参阅 **[提供商概览（provider overview）](https://langchain-doc.cn/v1/python/integrations/providers/overview)** 了解如何使用您的特定聊天模型访问这些工具。

## 创建工具

### 基本工具定义 (Basic tool definition)

创建工具最简单的方法是使用 **[`@tool`](https://langchain-doc.cn/v1/python/langchain/[https:/reference.langchain.com/python/langchain/tools/#langchain.tools.tool](https://reference.langchain.com/python/langchain/tools/#langchain.tools.tool))** 装饰器。默认情况下，函数的 **文档字符串**（docstring）会成为工具的描述，帮助模型理解何时使用它：

```python
from langchain.tools import tool
from langchain_core.messages import HumanMessage, AIMessage, SystemMessage

@tool(name_or_callable="get_weather")
def get_weather(location: str) -> str:
    """
    该工具可以获取location位置的天气信息。
    """
    return f"{location}接下来一周都是晴天！"

messages = [HumanMessage(content="北京的天气如何？")]

model_with_tools = model.bind_tools(tools=[get_weather])
ai_message : AIMessage = model_with_tools.invoke(messages)

messages.append(ai_message)
```

**类型提示** 是**必需的**，因为它们定义了工具的输入架构。文档字符串应该**信息丰富且简洁**，以帮助模型理解工具的用途。

`get_weather`函数类型变更为工具类`<class 'langchain_core.tools.structured.StructuredTool'>`

### 自定义工具属性 (Customize tool properties)

#### 自定义工具名称 (Custom tool name)

默认情况下，工具名称来自函数名称。当您需要更具描述性的名称时，可以通过`name_or_callable`覆盖它：

```python
@tool(name_or_callable="return_weather")
def get_weather(location: str) -> str:
    """
    该工具可以获取location位置的天气信息。
    """
    return f"{location}接下来一周都是晴天！"
```

#### 自定义工具描述 (Custom tool description)

覆盖自动生成的工具描述，以提供更清晰的模型指导`description`参数：

```python
@tool("calculator", description="Performs arithmetic calculations. Use this for any math problems.")
def calc(expression: str) -> str:
    """Evaluate mathematical expressions."""
    return str(eval(expression))
```

### 高级架构定义 (Advanced schema definition)

使用 Pydantic 模型 或 JSON 架构 定义复杂的输入：

```python
# json_schema
weather_schema = {
    "type": "object",
    "properties": {
        "location": {"type": "string"},
        "units": {"type": "string"},
        "include_forecast": {"type": "boolean"}
    },
    "required": ["location", "units", "include_forecast"]
}

@tool(args_schema=weather_schema,name_or_callable="get_weather_json")
def get_weather_json(location: str, units: str = "celsius", include_forecast: bool = False) -> str:
    """Get current weather and optional forecast."""
    temp = 22 if units == "celsius" else 72
    result = f"Current weather in {location}: {temp} degrees {units[0].upper()}"
    if include_forecast:
        result += "\nNext 5 days: rainy"
    return result
```

```python
from pydantic import BaseModel, Field
from typing import Literal

class WeatherInput(BaseModel):
    """Input for weather queries."""
    location: str = Field(description="City name or coordinates")
    units: Literal["celsius", "fahrenheit"] = Field(
        default="celsius",
        description="Temperature unit preference"
    )
    include_forecast: bool = Field(
        default=False,
        description="Include 5-day forecast"
    )

@tool(args_schema=WeatherInput,name_or_callable="get_weather_pydantic")
def get_weather_pydantic(location: str, units: str = "fahrenheit", include_forecast: bool = False) -> str:
    """Get current weather and optional forecast."""
    temp = 22 if units == "celsius" else 72
    result = f"Current weather in {location}: {temp} degrees {units[0].upper()}"
    if include_forecast:
        result += "\nNext 5 days: cloudy"
    return result
```

## 访问上下文 (Accessing Context)

**为什么这很重要：** 当工具可以访问**代理状态**、**运行时上下文**和**长期记忆**时，它们的功能最强大。这使得工具能够做出**上下文感知**的决策、**个性化**响应并在对话中**维护信息**。

工具可以通过 `ToolRuntime` 参数访问运行时信息，该参数提供：

- **State（状态）** - 流经执行的可变数据（消息、计数器、自定义字段）
- **Context（上下文）** - 不可变的配置，如用户 ID、会话详细信息或特定于应用程序的配置
- **Store（存储）** - 跨对话的持久长期记忆
- **Stream Writer（流写入器）** - 在工具执行时流式传输自定义更新
- **Config（配置）** - 执行的 `RunnableConfig`
- **Tool Call ID（工具调用 ID）** - 当前工具调用的 ID

#### ToolRuntime

使用 `ToolRuntime` 在一个参数中访问所有运行时信息。只需将 `runtime: ToolRuntime` 添加到您的工具签名中，它就会被自动注入，而**不会暴露给 LLM**。

> **`ToolRuntime`**: 一个统一的参数，为工具提供对**状态**、**上下文**、**存储**、**流式传输**、**配置**和**工具调用 ID** 的访问。这取代了使用单独的 **[`InjectedState`](https://langchain-doc.cn/v1/python/langchain/[https:/reference.langchain.com/python/langgraph/agents/#langgraph.prebuilt.tool_node.InjectedState](https://reference.langchain.com/python/langgraph/agents/#langgraph.prebuilt.tool_node.InjectedState))**、**[`InjectedStore`](https://langchain-doc.cn/v1/python/langchain/[https:/reference.langchain.com/python/langgraph/agents/#langgraph.prebuilt.tool_node.InjectedStore](https://reference.langchain.com/python/langgraph/agents/#langgraph.prebuilt.tool_node.InjectedStore))**、**[`get_runtime`](https://langchain-doc.cn/v1/python/langchain/[https:/reference.langchain.com/python/langgraph/runtime/#langgraph.runtime.get_runtime](https://reference.langchain.com/python/langgraph/runtime/#langgraph.runtime.get_runtime))** 和 **[`InjectedToolCallId`](https://langchain-doc.cn/v1/python/langchain/[https:/reference.langchain.com/python/langchain/tools/#langchain.tools.InjectedToolCallId](https://reference.langchain.com/python/langchain/tools/#langchain.tools.InjectedToolCallId))** 注释的旧模式。

**访问状态：**

工具可以使用 `ToolRuntime` 访问当前的图状态：



# 短期记忆

**记忆**是一个系统，用于记住关于先前交互的信息。对于 **AI 代理** (AI agents) 而言，记忆至关重要，因为它能让他们记住之前的交互、从反馈中学习并适应用户偏好。随着代理处理涉及大量用户交互的更复杂任务，这种能力对于**效率**和**用户满意度**都变得至关重要。

**短期记忆**让您的应用程序能够记住**单个线程**或**对话**中的先前交互。

聊天模型使用 **消息** (messages) 接受上下文，这些消息包括**指令**（系统消息 system message）和**输入**（人类消息 human messages）。在聊天应用程序中，消息在人类输入和模型响应之间交替，导致消息列表随着时间推移而变长。由于上下文窗口是有限的，许多应用程序可以受益于使用技术来**移除**或**“遗忘”**陈旧信息。

## 用法

要向代理添加短期记忆（线程级持久性），您需要在创建代理时指定一个 **`checkpointer`**。

> ℹ️ **信息：**
> LangChain 的代理将短期记忆作为代理**状态** (state) 的一部分进行管理。
>
> 通过将这些存储在图 (graph) 的状态中，代理可以访问给定对话的完整上下文，同时保持不同线程之间的分离。
>
> 状态使用 **checkpointer** 持久化到数据库（或内存）中，以便线程可以随时恢复。
>
> 当代理被调用或一个步骤（如工具调用）完成时，短期记忆会更新，并在每个步骤开始时读取状态。

```python
# 1 模型
from langchain.chat_models import init_chat_model
import dotenv
dotenv.load_dotenv()
model = init_chat_model(
    model="openai:gpt-4o-mini",
    temperature=0.7
)

# 2. 智能体
from langchain.agents import create_agent
# 短期记忆，内存实现
from langgraph.checkpoint.memory import InMemorySaver  # [!code highlight]
from langchain.tools import tool

@tool
def get_user_info(name: str) -> str:
    """
    获取用户信息。
    """
    return f"用户信息：{name}，19岁，性别男，现在正在读本科，正在备考CET-6。"

# agent创建时指定checkpointer的一种实现方式的实例即可
agent = create_agent(
    model,
    tools=[get_user_info],
    checkpointer=InMemorySaver(),  # [!code highlight]
)

# 3 调用
response = agent.invoke(
    {"messages": [{"role": "user", "content": "你好，我是小明。我想要学习英语，请你根据我的信息，生成一个学习英语的规划。"}]},
    {"configurable": {"thread_id": "1"}},  # [!code highlight]
)

response = agent.invoke(
    {"messages": [{"role": "user", "content": "我的弱势是口语，你能给制定一个更详细的口语计划吗？"}]},
    {"configurable": {"thread_id": "1"}},  # [!code highlight]
)
```

response包含从开始到最后的所有的Message

### 生产环境

在生产环境中，请使用由**数据库**支持的 checkpointer：

```bash
pip install langgraph-checkpoint-postgres
```

```python
from langchain.agents import create_agent

from langgraph.checkpoint.postgres import PostgresSaver  # [!code highlight]


DB_URI = "postgresql://postgres:postgres@localhost:5442/postgres?sslmode=disable"
with PostgresSaver.from_conn_string(DB_URI) as checkpointer:
    checkpointer.setup() # auto create tables in PostgresSql
    agent = create_agent(
        model,
        tools=[get_user_info],
        checkpointer=checkpointer,  # [!code highlight]
    )
```

## 自定义代理记忆（Customizing agent Memory）

默认情况下，代理使用 **`AgentState`** 来管理短期记忆，特别是通过 **`messages`** 键来管理会话历史记录。

您可以扩展 **`AgentState`** 以添加额外的字段。自定义状态模式通过 **`state_schema`** 参数传递给 **`create_agent`**。

```python
from langchain_core.messages import SystemMessage
from langchain.agents import AgentState
from langchain.tools import tool
from langchain.agents.middleware import AgentMiddleware


# 自定义 AgentState 扩展其字段
class CustomAgentState(AgentState):  # [!code highlight]
    user_id: str  # [!code highlight]
    preferences: dict  # [!code highlight]


@tool
def get_user_info(user_id: str) -> str:
    """
    根据用户id获取用户信息。
    """
    user_info = ""
    if user_id == "user01":
        user_info = "用户信息：张千，19岁，性别男，现在正在读本科，正在备考CET-6。"
    elif user_id == "user02":
        user_info = "用户信息：凌雨，23岁，性别女，现在本科毕业，正在备战考研，目标是上岸清华大学。"
    else:
        user_info = "用户信息未找到。"
    return user_info


# 自定义中间件强制获取用户信息
class UserInfoMiddleware(AgentMiddleware):
    """在模型调用前自动获取用户信息并注入系统提示"""

    def before_model(self, state: CustomAgentState, runtime):
        # 从状态中读取 user_id
        user_id = state.get("user_id")
        if not user_id:
            return None  # 没有 user_id，不做处理

        # 主动调用工具获取用户信息
        user_info = get_user_info.invoke({"user_id": user_id})

        # 读取偏好设置
        preferences = state.get("preferences", {})
        theme = preferences.get("theme", "normal")

        # 构建系统提示
        system_prompt = f"""你是一个智能学习规划助手。当前用户信息：
            {user_info}。输出风格偏好：{theme}
            - 如果 theme 为 "short"，请用简洁的语言回答
            - 如果 theme 为 "normal"，请提供详细的规划建议
            """

        # 将系统提示注入到消息列表最前面
        # 注意：这里不直接修改 state["messages"]，而是返回一个字典
        # LangGraph 会自动合并到状态中
        return {
            "messages": [SystemMessage(content=system_prompt)]
        }


agent = create_agent(
    model=model,
    tools=[get_user_info],
    state_schema=CustomAgentState,
    checkpointer=checkpointer,  # 关键：注入短期记忆
    middleware=[UserInfoMiddleware()]
)

message = HumanMessage(
    content=[
        {"type": "text", "text": "请你根据我的信息，帮我拟一份英语学习规划。"},
    ]
)
input = {
    "messages": [message],
    "user_id": "user02",  # [!code highlight]
    "preferences": {"theme": "short"}  # [!code highlight]
}
result = agent.invoke(
    input=input,
    config=config)

```

#### 重要补充

**默认状态的局限**：

- 只能存对话消息（`messages`）
- 无法区分“用户身份”、“任务进度”、“业务数据”
- 所有信息都混在消息文本里，Agent 难以精准提取

模型（LLM）的“感知”范围仅限于:

```txt
模型能看到的：
┌─────────────────────────────────────┐
│  messages 列表中的内容               │
│  ├── SystemMessage（系统提示）        │
│  ├── HumanMessage（用户输入）         │
│  ├── AIMessage（模型输出）            │
│  └── ToolMessage（工具结果）          │
└─────────────────────────────────────┘

模型看不到的：
┌─────────────────────────────────────┐
│  state["user_id"] = "user02"        │  ← 藏在状态里
│  state["preferences"] = {...}       │  ← 模型不知道
│  state["internal_notes"] = "..."    │  ← 完全不可见
└─────────────────────────────────────┘
```

自定义字段的价值在于**中间件和工具可以读写它们**，然后**把处理结果注入到 `messages` 中**，让模型间接“感知”。

**正确的工作流**

```txt
┌─────────────────────────────────────────┐
│  1. 用户输入                             │
│     user_id: "user02"                    │
│     preferences: {"theme": "short"}      │
│     messages: ["帮我拟学习计划"]          │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  2. 中间件（before_model）               │
│     ├── 读取 state["user_id"]            │
│     ├── 调用 get_user_info 工具          │
│     └── 将结果写入 messages              │
│         → 注入 SystemMessage             │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  3. 模型看到的消息列表                   │
│     SystemMessage: "用户信息：凌雨..."   │  ← 中间件注入的
│     HumanMessage: "帮我拟学习计划"       │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  4. 模型基于注入的信息生成回答            │
│     "凌雨同学，针对你考研英语的需求..."    │
└─────────────────────────────────────────┘
```



## 常见模式

启用**短期记忆**后，长对话可能会超出 LLM 的上下文窗口。常见的解决方案有：

| 模式                                 | 描述                                           |
| :----------------------------------- | :--------------------------------------------- |
| **修剪消息** (Trim messages) ✂️       | 移除最初或最后的 N 条消息（在调用 LLM 之前）。 |
| **删除消息** (Delete messages) 🗑️     | 从 LangGraph 状态中**永久**删除消息。          |
| **总结消息** (Summarize messages) 📝  | 总结历史记录中较早的消息，并用摘要替换它们。   |
| **自定义策略** (Custom strategies) ⚙️ | 自定义策略（例如：消息过滤等）。               |

### 修剪消息

大多数 LLM 都有一个最大支持的上下文窗口（以 token 计）。

决定何时截断消息的一种方法是**计算消息历史记录中的 token 数**，并在接近该限制时进行截断。如果您使用 LangChain，可以使用**修剪消息实用程序** (trim messages utility) 并指定要从列表中保留的 token 数量，以及用于处理边界的 `strategy`（例如：保留最后的 `max_tokens`）。

要在代理中修剪消息历史记录，请使用 **`@before_model`** 中间件装饰器：

```python
from langgraph.graph.message import REMOVE_ALL_MESSAGES
from langchain_core.messages import RemoveMessage
from langchain_core.runnables import RunnableConfig
from langchain.agents.middleware import before_model


@before_model
def trim_messages(state: AgentState, runtime):
    """Keep only the last few messages to fit context window."""
    messages = state["messages"]

    if len(messages) <= 3:
        return None  # No changes needed

    # 修剪消息，保留前四条消息
    first_msg = messages[0:4]
    recent_messages = messages[-6:]
    new_messages = first_msg + recent_messages

    return {
        "messages": [
            RemoveMessage(id=REMOVE_ALL_MESSAGES),
            *new_messages
        ]
    }

agent = create_agent(
    model=model,
    tools=[get_user_info],
    state_schema=CustomAgentState,
    checkpointer=checkpointer,  # 关键：注入短期记忆
    middleware=[UserInfoMiddleware(), trim_messages]
)

# 调用时传入 thread_id
config: RunnableConfig = {"configurable": {"thread_id": "session_001"}}
message = HumanMessage(
    content=[
        {"type": "text", "text": "给我用户id为'user01'的用户信息。"},
    ]
)
result = agent.invoke(
    input={"messages": [message]},
    config=config)
```

### 删除消息

您可以从图状态中删除消息以管理消息历史记录。

当您想要删除特定消息或清除整个消息历史记录时，这非常有用。

要从图状态中删除消息，可以使用 **`RemoveMessage`**。

要使 `RemoveMessage` 工作，您需要使用具有 **`add_messages`** [reducer] 的状态键。

默认的 **`AgentState`** 提供了此功能。

删除**特定**消息：

```python
from langchain.messages import RemoveMessage  # [!code highlight]

def delete_messages(state):
    messages = state["messages"]
    if len(messages) > 2:
        # remove the earliest two messages
        return {"messages": [RemoveMessage(id=m.id) for m in messages[:2]]}  # [!code highlight]
```

删除所有消息：

```python
from langgraph.graph.message import REMOVE_ALL_MESSAGES  # [!code highlight]

def delete_messages(state):
    return {"messages": [RemoveMessage(id=REMOVE_ALL_MESSAGES)]}  # [!code highlight]
```

> ⚠️ **警告：**
> 删除消息时，**请确保**生成的消息历史记录是**有效**的。请检查您正在使用的 LLM 提供商的限制。例如：
>
> - 一些提供商期望消息历史记录以 `user` 消息开始
> - 大多数提供商要求带有工具调用的 `assistant` 消息后跟相应的 `tool` 结果消息。

```python
@after_model
def delete_old_messages(state: AgentState, runtime: Runtime) -> dict | None:
    """Remove old messages to keep conversation manageable."""
    messages = state["messages"]
    if len(messages) > 2:
        # remove the earliest two messages
        return {"messages": [RemoveMessage(id=m.id) for m in messages[:2]]}
    return None


agent = create_agent(
    "openai:gpt-5-nano",
    tools=[],
    system_prompt="Please be concise and to the point.",
    middleware=[delete_old_messages],
    checkpointer=InMemorySaver(),
)
```

### 总结消息

如上所示，修剪或删除消息的问题是您可能会因为删除消息队列而**丢失信息**。因此，一些应用程序受益于使用**聊天模型**来总结消息历史记录的更复杂方法。

`SummarizationMiddleware` 自动解决这些问题：**检测到 Token 超限 → 压缩早期消息 → 保留最近对话**。

要在代理中总结消息历史记录，请使用内置的 **`SummarizationMiddleware`**：

```python
SummarizationMiddleware(
    model="gpt-4o-mini",                    # 必填：生成摘要的模型
    trigger=("tokens", 3500),               # 触发条件：Token 数超过 3500
    keep=("messages", 15),                  # 保留最近 15 条原始消息
    summary_prompt="请将以下对话压缩成简洁摘要，重点保留：用户身份、核心关注点、关键事实、未完成事项。\n\n{messages}",
    token_counter=my_custom_counter,        # 可选：自定义 Token 计数函数
)
```



```python
from langchain.agents import create_agent
from langchain.agents.middleware import SummarizationMiddleware
from langgraph.checkpoint.memory import InMemorySaver
from langchain_core.runnables import RunnableConfig


checkpointer = InMemorySaver()

summarization_middleware = SummarizationMiddleware(
    model="gpt-4o-mini",                    # 必填：生成摘要的模型
    trigger=("tokens", 3500),               # 触发条件：Token 数超过 3500
    keep=("messages", 15),                  # 保留最近 15 条原始消息
    summary_prompt="请将以下对话压缩成简洁摘要，重点保留：用户身份、核心关注点、关键事实、未完成事项。\n\n{messages}",
    token_counter=my_custom_counter,        # 可选：自定义 Token 计数函数
)

agent = create_agent(
    model="openai:gpt-4o",
    tools=[],
    middleware=[
        summarization_middleware
    ],
    checkpointer=checkpointer,
)

config: RunnableConfig = {"configurable": {"thread_id": "1"}}
agent.invoke({"messages": "hi, my name is bob"}, config)
agent.invoke({"messages": "write a short poem about cats"}, config)
agent.invoke({"messages": "now do the same but for dogs"}, config)
final_response = agent.invoke({"messages": "what's my name?"}, config)

final_response["messages"][-1].pretty_print()
"""
================================== Ai Message ==================================

Your name is Bob!
"""
```

## 访问记忆

### 工具

#### 在工具中读取短期记忆

使用 **`ToolRuntime`** 参数在工具中访问短期记忆（状态）。

`tool_runtime` 参数对工具签名是**隐藏**的（因此模型看不到它），但工具可以通过它访问状态。

```python
from langchain.agents import create_agent, AgentState
from langchain.tools import tool, ToolRuntime


class CustomState(AgentState):
    user_id: str

@tool
def get_user_info(
    runtime: ToolRuntime
) -> str:
    """Look up user info."""
    user_id = runtime.state["user_id"]
    return "User is John Smith" if user_id == "user_123" else "Unknown user"

agent = create_agent(
    model=model,
    tools=[get_user_info],
    state_schema=CustomState,
)

result = agent.invoke({
    "messages": "look up user information",
    "user_id": "user_123"
})
print(result["messages"][-1].content)
# > User is John Smith.
```

#### 从工具写入短期记忆

要在执行期间修改代理的短期记忆（状态），您可以直接从工具返回**状态更新** (state updates)。

这对于持久化中间结果或使信息可供后续工具或提示访问非常有用。

```python
from langchain.tools import tool, ToolRuntime
from langchain_core.runnables import RunnableConfig
from langchain.messages import ToolMessage
from langchain.agents import create_agent, AgentState
from langgraph.types import Command
from pydantic import BaseModel


class CustomState(AgentState):  # [!code highlight]
    user_name: str

class CustomContext(BaseModel):
    user_id: str

@tool
def update_user_info(
    runtime: ToolRuntime[CustomContext, CustomState],
) -> Command:
    """Look up and update user info."""
    user_id = runtime.context.user_id  # [!code highlight]
    name = "John Smith" if user_id == "user_123" else "Unknown user"
    return Command(update={
        "user_name": name,
        # update the message history
        "messages": [
            ToolMessage(
                "Successfully looked up user information",
                tool_call_id=runtime.tool_call_id
            )
        ]
    })

@tool
def greet(
    runtime: ToolRuntime[CustomContext, CustomState]
) -> str:
    """Use this to greet the user once you found their info."""
    user_name = runtime.state["user_name"]
    return f"Hello {user_name}!"
  # [!code highlight]
agent = create_agent(
    model="openai:gpt-5-nano",
    tools=[update_user_info, greet],
    state_schema=CustomState,
    context_schema=CustomContext,  # [!code highlight]
)

agent.invoke(
    {"messages": [{"role": "user", "content": "greet the user"}]},
    context=CustomContext(user_id="user_123"),
)
```

### 提示（Prompt）

`@dynamic_prompt` 是 LangChain Agent 中一个**非常实用的装饰器**，它的核心作用是：**在每次模型调用前，根据当前的运行时上下文（比如用户身份、对话状态、已执行的操作等）动态生成系统提示词（System Prompt）**。

简单来说，它让 Agent 不再是“一个角色演到底”，而是可以根据场景“随机应变”。

`@dynamic_prompt` 是一个**包装式（Wrap-style）中间件**，它会在 Agent 准备调用底层大语言模型（LLM）进行推理之前执行。

1. **执行时机**：每次模型调用前（`before_model` 钩子）。
2. **替换规则**：它返回的字符串会**完全替换**创建 Agent 时设置的静态 `system_prompt`。
3. **一次性有效**：这种替换只对**本次**模型调用生效。下一次调用如果没有再次触发 `@dynamic_prompt`，会恢复使用默认的静态提示词（如果有的话）。

```python
from typing import TypedDict
from langchain.agents import create_agent
from langchain.agents.middleware import dynamic_prompt, ModelRequest

# 1. 定义上下文结构（用于传入动态信息）
class Context(TypedDict):
    user_role: str

# 2. 定义动态提示词中间件
@dynamic_prompt
def user_role_prompt(request: ModelRequest) -> str:
    """根据用户角色生成不同的系统提示词"""
    # 从运行时上下文中获取 user_role
    user_role = request.runtime.context.get("user_role", "user")
    base_prompt = "You are a helpful assistant."

    if user_role == "expert":
        return f"{base_prompt} Provide detailed technical responses."
    elif user_role == "beginner":
        return f"{base_prompt} Explain concepts simply and avoid jargon."
    return base_prompt

# 3. 创建 Agent（可以同时设置静态提示词，但会被动态替换）
agent = create_agent(
    model=model,
    tools=[],  # 此处省略工具定义
    middleware=[user_role_prompt],  # 注册动态提示词中间件
    system_prompt="You are a default assistant.",  # 静态提示词（可选）
    context_schema=Context
)

# 4. 调用 Agent，传入不同的 context
# 第一次调用：专家模式
result1 = agent.invoke(
    {"messages": [{"role": "user", "content": "Explain transformer architecture."}]},
    context={"user_role": "expert"}
)
# 实际使用的系统提示词： "You are a helpful assistant. Provide detailed technical responses."

# 第二次调用：新手模式
result2 = agent.invoke(
    {"messages": [{"role": "user", "content": "Explain transformer architecture."}]},
    context={"user_role": "beginner"}
)
# 实际使用的系统提示词： "You are a helpful assistant. Explain concepts simply and avoid jargon."
```



### before model

在 **`@before_model`** 中间件中访问短期记忆（状态），以在模型调用之前处理消息。



### after model

在 **`@after_model`** 中间件中访问短期记忆（状态），以在模型调用之后处理消息。

































































# 智能体

智能体将语言模型与[工具](https://langchain-doc.cn/v1/python/langchain/tools)结合，创建能够对任务进行推理、决定使用哪些工具并迭代寻求解决方案的系统。

[`create_agent`](https://reference.langchain.com/python/langchain/agents/#langchain.agents.create_agent) 提供了一个生产就绪的智能体实现。

[LLM 智能体在循环中运行工具以实现目标](https://simonwillison.net/2025/Sep/18/agents/)。智能体会一直运行，直到满足停止条件——即模型输出最终结果或达到迭代次数限制。

> **信息**
> [`create_agent`](https://reference.langchain.com/python/langchain/agents/#langchain.agents.create_agent) 使用 [LangGraph](https://langchain-doc.cn/v1/python/langgraph/overview) 构建了一个**基于图**的智能体运行时。图由节点（步骤）和边（连接）组成，定义了智能体如何处理信息。智能体在图中移动，执行节点，如模型节点（调用模型）、工具节点（执行工具）或中间件。
>
> 了解更多关于 [Graph API](https://langchain-doc.cn/v1/python/langgraph/graph-api) 的信息。



## 核心组件

### 模型

模型是智能体的推理引擎。它可以通过多种方式指定，支持静态和动态模型选择。

#### 静态模型

静态模型在创建智能体时配置一次，并在整个执行过程中保持不变。这是最常见且直接的方法。

从 模型标识符字符串 初始化静态模型：

```python
from langchain.agents import create_agent

agent = create_agent(
    "openai:gpt-5",
    tools=tools
)
```

> **提示**
> 模型标识符字符串支持自动推断（例如 `"gpt-5"` 将被推断为 `"openai:gpt-5"`）。请参考[参考文档](https://reference.langchain.com/python/langchain/models/#langchain.chat_models.init_chat_model(model_provider)) 查看完整的模型标识符字符串映射列表。

为了对模型配置进行更多控制，可以直接使用提供商包初始化模型实例。在此示例中，我们使用 [`ChatOpenAI`](https://reference.langchain.com/python/integrations/langchain_openai/ChatOpenAI/)。请参阅 [聊天模型](https://langchain-doc.cn/v1/python/integrations/chat) 以了解其他可用的聊天模型类。

```python
from langchain.agents import create_agent
from langchain_openai import ChatOpenAI

model = ChatOpenAI(
    model="gpt-5",
    temperature=0.1,
    max_tokens=1000,
    timeout=30
    # ...（其他参数）
)
agent = create_agent(model, tools=tools)
```

模型实例为您提供对配置的完全控制。当您需要设置特定[参数](https://langchain-doc.cn/v1/python/langchain/models#parameters)，如 `temperature`、`max_tokens`、`timeouts`、`base_url` 和其他提供商特定设置时，请使用它们。请参考[参考文档](https://langchain-doc.cn/v1/python/integrations/providers/all_providers) 查看模型上可用的参数和方法。

#### 动态模型

动态模型在 运行时 根据当前 状态 和上下文进行选择。这支持复杂的路由逻辑和成本优化。

要使用动态模型，请使用 [`@wrap_model_call`](https://reference.langchain.com/python/langchain/middleware/#langchain.agents.middleware.wrap_model_call) 装饰器创建中间件，以修改请求中的模型：

```python
from langchain_openai import ChatOpenAI
from langchain.agents import create_agent
from langchain.agents.middleware import wrap_model_call, ModelRequest, ModelResponse


basic_model = ChatOpenAI(model="gpt-4o-mini")
advanced_model = ChatOpenAI(model="gpt-4o")

@wrap_model_call
def dynamic_model_selection(request: ModelRequest, handler) -> ModelResponse:
    """根据对话复杂性选择模型。"""
    message_count = len(request.state["messages"])

    if message_count > 10:
        # 对较长的对话使用高级模型
        model = advanced_model
    else:
        model = basic_model

    request.model = model
    return handler(request)

agent = create_agent(
    model=basic_model,  # 默认模型
    tools=tools,
    middleware=[dynamic_model_selection]
)
```

**警告**
在使用结构化输出时，不支持预绑定模型（已调用 [`bind_tools`](https://reference.langchain.com/python/langchain_core/language_models/#langchain_core.language_models.chat_models.BaseChatModel.bind_tools) 的模型）。如果您需要使用结构化输出的动态模型选择，请确保传递给中间件的模型未预绑定。

### 工具

工具赋予智能体执行行动的能力。智能体超越了简单的仅模型工具绑定，实现了：

- 序列中的多个工具调用（由单个提示触发）
- 适当的并行工具调用
- 基于先前结果的动态工具选择
- 工具重试逻辑和错误处理
- 工具调用之间的状态持久化

#### 定义工具

```python
from langchain.tools import tool
from langchain.agents import create_agent


@tool
def search(query: str) -> str:
    """搜索信息。"""
    return f"结果：{query}"

@tool
def get_weather(location: str) -> str:
    """获取位置的天气信息。"""
    return f"{location} 的天气：晴朗，72°F"

agent = create_agent(model, tools=[search, get_weather])
```

如果提供空工具列表，智能体将仅包含一个 LLM 节点，不具备工具调用能力。

#### 工具错误处理

要自定义工具错误的处理方式，请使用 [`@wrap_tool_call`](https://reference.langchain.com/python/langchain/middleware/#langchain.agents.middleware.wrap_tool_call) 装饰器创建中间件：

```python
from langchain.agents import create_agent
from langchain.agents.middleware import wrap_tool_call
from langchain_core.messages import ToolMessage


@wrap_tool_call
def handle_tool_errors(request, handler):
    """使用自定义消息处理工具执行错误。"""
    try:
        return handler(request)
    except Exception as e:
        # 向模型返回自定义错误消息
        return ToolMessage(
            content=f"工具错误：请检查您的输入并重试。({str(e)})",
            tool_call_id=request.tool_call["id"]
        )

agent = create_agent(
    model="openai:gpt-4o",
    tools=[search, get_weather],
    middleware=[handle_tool_errors]
)
```

当工具失败时，智能体将返回带有自定义错误消息的 [`ToolMessage`](https://reference.langchain.com/python/langchain/messages/#langchain.messages.ToolMessage)：

```python
[
    ...
    ToolMessage(
        content="工具错误：请检查您的输入并重试。(division by zero)",
        tool_call_id="..."
    ),
    ...
]
```

#### ReAct循环中的工具使用

智能体遵循 ReAct（“推理 + 行动”）模式，在简短的推理步骤与针对性的工具调用之间交替，并将结果观察反馈到后续决策中，直到能够提供最终答案。

提示：识别当前最受欢迎的无线耳机并验证其可用性。

- **推理**： “受欢迎程度具有时效性，我需要使用提供的搜索工具。”

- **行动**： 调用 `search_products("wireless headphones")`

  ```
  ================================== AI 消息 ==================================
  工具调用：
    search_products (call_abc123)
   调用 ID：call_abc123
    参数：
      query: wireless headphones
  ```

  ```
  ================================= 工具消息 =================================
  
  找到 5 个匹配“wireless headphones”的产品。前 5 个结果：WH-1000XM5, ...
  ```

- **推理**： “在回答前，我需要确认排名第一的商品的可用性。”

- **行动**： 调用 `check_inventory("WH-1000XM5")`

  ```
  ================================== AI 消息 ==================================
  工具调用：
    check_inventory (call_def456)
   调用 ID：call_def456
    参数：
      product_id: WH-1000XM5
  ```

  ```
  ================================= 工具消息 =================================
  
  产品 WH-1000XM5：库存 10 件
  ```

- **推理**： “我已经获取了最受欢迎的型号及其库存状态。现在可以回答用户的问题。”

- **行动**： 产生最终答案

  ```
  ================================== AI 消息 ==================================
  
  我找到了无线耳机（型号 WH-1000XM5），库存 10 件...
  ```



### 系统提示

您可以通过提供提示来塑造智能体处理任务的方式。[`system_prompt`](https://reference.langchain.com/python/langchain/agents/#langchain.agents.create_agent(system_prompt)) 参数可以作为字符串提供：

```python
agent = create_agent(
    model,
    tools,
    system_prompt="你是一个有帮助的助手。请简洁准确。"
)
```

当未提供 [`system_prompt`](https://reference.langchain.com/python/langchain/agents/#langchain.agents.create_agent(system_prompt)) 时，智能体将直接从消息中推断其任务。

#### 动态系统提示

对于需要根据运行时上下文或智能体状态修改系统提示的高级用例，您可以使用 [中间件](https://langchain-doc.cn/v1/python/langchain/middleware)。

[`@dynamic_prompt`](https://reference.langchain.com/python/langchain/middleware/#langchain.agents.middleware.dynamic_prompt) 装饰器创建中间件，根据模型请求动态生成系统提示：

```python
from typing import TypedDict

from langchain.agents import create_agent
from langchain.agents.middleware import dynamic_prompt, ModelRequest


class Context(TypedDict):
    user_role: str

@dynamic_prompt
def user_role_prompt(request: ModelRequest) -> str:
    """根据用户角色生成系统提示。"""
    user_role = request.runtime.context.get("user_role", "user")
    base_prompt = "你是一个有帮助的助手。"

    if user_role == "expert":
        return f"{base_prompt} 提供详细的技术响应。"
    elif user_role == "beginner":
        return f"{base_prompt} 简单解释概念，避免使用行话。"

    return base_prompt

agent = create_agent(
    model="openai:gpt-4o",
    tools=[web_search],
    middleware=[user_role_prompt],
    context_schema=Context
)

# 系统提示将根据上下文动态设置
result = agent.invoke(
    {"messages": [{"role": "user", "content": "解释机器学习"}]},
    context={"user_role": "expert"}
)
```

## 调用

您可以通过向其 [`State`](https://langchain-doc.cn/v1/python/langgraph/graph-api#state) 传递更新来调用智能体。所有智能体在其状态中包含[消息序列](https://langchain-doc.cn/v1/python/langgraph/use-graph-api#messagesstate)；要调用智能体，请传递一条新消息：

```python
result = agent.invoke(
    {"messages": [{"role": "user", "content": "旧金山天气如何？"}]}
)
```



## 高级概念

### 结构化输出

在某些情况下，您可能希望智能体以特定格式返回输出。LangChain 通过 [`response_format`](https://reference.langchain.com/python/langchain/middleware/#langchain.agents.middleware.ModelRequest(response_format)) 参数提供结构化输出策略。

#### ToolStrategy

`ToolStrategy` 是 LangChain 中用于实现**结构化输出**的核心策略之一，专门针对**不支持原生结构化输出**的模型（如 Claude 系列、部分开源模型）设计。

它的核心思路是：**把你想让模型输出的结构化 Schema（如 Pydantic 模型）伪装成一个“工具”**，然后利用模型的函数调用（Function Calling / Tool Calling）能力，让模型“调用”这个工具，从而生成符合你要求的 JSON 数据。

```python
from typing import Literal
from pydantic import BaseModel, Field
from langchain.agents import create_agent
from langchain.agents.structured_output import ToolStrategy
from langchain_openai import ChatOpenAI  # 假设使用 OpenAI 模型演示

# 1. 定义结构化输出的 Schema（Pydantic 模型）
class MeetingAction(BaseModel):
    """从会议记录中提取的行动项"""
    task: str = Field(description="需完成的具体任务")
    assignee: str = Field(description="任务负责人")
    priority: Literal["low", "medium", "high"] = Field(description="任务优先级")

# 2. 创建智能体，并指定使用 ToolStrategy
agent = create_agent(
    model=ChatOpenAI(model="gpt-4o", temperature=0),  # 即使模型支持原生，也可以用 ToolStrategy
    tools=[],  # 这里可以传入其他真实工具
    response_format=ToolStrategy(
        schema=MeetingAction,
        tool_message_content="行动项已提取并添加至会议纪要！",  # 可选：自定义工具提示信息
        handle_errors=True  # 可选：启用错误重试
    )
)

# 3. 运行智能体
result = agent.invoke({
    "messages": [
        {
            "role": "user",
            "content": "从会议内容提取信息：萨拉需要尽快更新项目时间线，优先级高"
        }
    ]
})

# 4. 获取结构化结果
print(result["structured_response"])
# 输出: MeetingAction(task='更新项目时间线', assignee='萨拉', priority='high')
```

#### ProviderStrategy

`ProviderStrategy` 使用模型提供商的原生结构化输出生成。这更可靠，但仅适用于支持原生结构化输出的提供商（例如 OpenAI）：

模型不会像ToolStrategy那样进行一次工具调用，而是由AI直接回答

```python
from langchain.agents.structured_output import ProviderStrategy

agent = create_agent(
    model="openai:gpt-4o",
    response_format=ProviderStrategy(ContactInfo)
)
```

> **注意**
> 从 `langchain 1.0` 开始，不再支持直接传递模式（例如 `response_format=ContactInfo`）。您必须明确使用 `ToolStrategy` 或 `ProviderStrategy`。

## 记忆

智能体通过消息状态自动维护对话历史。您还可以配置智能体使用自定义状态模式，在对话期间记住额外信息。

存储在状态中的信息可以被视为智能体的[短期记忆](https://langchain-doc.cn/v1/python/langchain/short-term-memory)：

自定义状态模式必须扩展 [`AgentState`](https://reference.langchain.com/python/langchain/agents/#langchain.agents.AgentState) 作为 `TypedDict`。

定义自定义状态有两种方式：

1. 通过 [中间件](https://langchain-doc.cn/v1/python/langchain/middleware)（推荐）
2. 通过 [`create_agent`](https://reference.langchain.com/python/langchain/agents/#langchain.agents.create_agent) 上的 [`state_schema`](https://reference.langchain.com/python/langchain/middleware/#langchain.agents.middleware.AgentMiddleware.state_schema)

> **注意**
> **推荐通过中间件定义自定义状态**，而不是通过 [`create_agent`](https://reference.langchain.com/python/langchain/agents/#langchain.agents.create_agent) 上的 [`state_schema`](https://reference.langchain.com/python/langchain/middleware/#langchain.agents.middleware.AgentMiddleware.state_schema)，因为它允许您将状态扩展概念上限定在相关中间件和工具的范围内。
>
> 仅仅创建扩展的AgentState实质上是为其添加更多的字段，模型是感知不到的，如果想要使用这些字段，必须要使用中间件或者其他方式
>
> [`state_schema`](https://reference.langchain.com/python/langchain/middleware/#langchain.agents.middleware.AgentMiddleware.state_schema) 仍支持用于向后兼容，在 [`create_agent`](https://reference.langchain.com/python/langchain/agents/#langchain.agents.create_agent) 上。

#### 通过中间件定义状态

当您的自定义状态需要被特定中间件钩子和附加到该中间件的工具访问时，使用中间件定义自定义状态。

```python
from langchain.agents import AgentState
from langchain.agents import create_agent
from langchain.agents.middleware import AgentMiddleware
from langchain.tools import tool

from langchain.chat_models import init_chat_model
import dotenv
dotenv.load_dotenv()
model = init_chat_model(
    model="openai:gpt-4o-mini",
    temperature=0.7
)


@tool
def search_tool(query: str) -> str:
    """搜索信息。"""
    return f"结果：{query}"

@tool(name_or_callable="get_weather")
def get_weather(location: str) -> str:
    """
    该工具可以获取location位置的天气信息。
    """
    return f"{location}接下来一周都是阴天！"

tools=[search_tool, get_weather]

class CustomState(AgentState):
    user_preferences: dict

class CustomMiddleware(AgentMiddleware):
    state_schema = CustomState
    tools = tools

    def before_model(self, state: CustomState, runtime):
        prefs = state.get("user_preferences", {})
        if prefs.get("style") == "technical":
            # 动态修改系统提示，要求模型用技术性语言回答
            state["system_prompt"] = "请用专业的技术术语回答，避免口语化表达。"
        return None

agent = create_agent(
    model,
    tools=tools,
    middleware=[CustomMiddleware()]
)



```

这段代码展示了 LangChain 中 **自定义中间件（Middleware）** 和 **自定义状态（Custom State）** 的典型用法，核心目的是让 Agent 在对话过程中能够**跟踪和管理额外的上下文信息**（比如用户的偏好设置）。

**1. 自定义状态类：**`CustomState`

- **`AgentState`** 是 LangChain 内置的状态基类，它已经包含了 `messages`（消息列表）等基础字段。
- **`CustomState`** 继承自 `AgentState`，并新增了一个 `user_preferences: dict` 字段。
- **作用**：这个字段用来存储用户的偏好信息（比如“喜欢技术性解释”、“详细程度要详细”），这些信息不属于对话消息本身，但会影响 Agent 的行为。

**2. 自定义中间件类：**`CustomMiddleware`

- **`state_schema = CustomState`**：这是最关键的一行。它告诉 LangChain 的 Agent 执行引擎：“我这个中间件需要访问的状态结构是 `CustomState`，里面除了消息列表，还有一个 `user_preferences` 字典。”

- **`tools = [tool1, tool2]`**：这个中间件可以绑定自己的工具集（比如一个“获取用户偏好”的工具）。

- **`before_model` 钩子**：这是一个**节点式钩子**，在每次模型调用之前执行。你可以在里面读取 `state.user_preferences`，然后动态修改系统提示词或调整工具行为。例如：

  ```python
  def before_model(self, state: CustomState, runtime):
      prefs = state.get("user_preferences", {})
      if prefs.get("style") == "technical":
          # 动态修改系统提示，要求模型用技术性语言回答
          state["system_prompt"] = "请用专业的技术术语回答，避免口语化表达。"
      return None
  ```

**3. 创建 Agent 并注入中间件**

- 通过 `middleware` 参数将 `CustomMiddleware` 实例传入 Agent。
- 当 Agent 运行时，LangChain 的执行引擎会自动识别 `CustomMiddleware` 中定义的 `state_schema`，并允许你在调用时传入 `user_preferences` 等额外字段。



**4. 调用 Agent 并传入额外状态**

- 这里的 `invoke` 参数除了标准的 `messages` 外，还多了一个 `user_preferences` 字段。
- 由于 `CustomMiddleware` 已经声明了 `state_schema = CustomState`，LangChain 知道这个字段是合法的，会把它合并到 Agent 的内部状态中。
- 在后续的 `before_model` 钩子中，你就可以通过 `state["user_preferences"]` 读取到这个值，并据此调整模型的行为。

 **5. 整体执行流程**

```txt
用户输入
    │
    ▼
before_agent 钩子（只执行一次）
    │
    ▼
before_model 钩子（CustomMiddleware 中的钩子）
    │   ├── 读取 state["user_preferences"]
    │   └── 根据偏好动态修改系统提示
    │
    ▼
模型调用（LLM）
    │
    ▼
after_model 钩子
    │
    ▼
（如果有工具调用，循环执行模型→工具→模型...）
    │
    ▼
after_agent 钩子（只执行一次）
    │
    ▼
最终输出
```

#### 通过`state_schema`定义状态

使用 [`state_schema`](https://reference.langchain.com/python/langchain/middleware/#langchain.agents.middleware.AgentMiddleware.state_schema) 参数作为快捷方式，定义仅在工具中使用的自定义状态。

```python
from langchain.agents import AgentState


class CustomState(AgentState):
    user_preferences: dict

agent = create_agent(
    model,
    tools=[tool1, tool2],
    state_schema=CustomState
)
# 智能体现在可以跟踪消息之外的额外状态
result = agent.invoke({
    "messages": [{"role": "user", "content": "我更喜欢技术性解释"}],
    "user_preferences": {"style": "technical", "verbosity": "detailed"},
})
```

## 流式输出

```python
for chunk in agent.stream({
    "messages": [{"role": "user", "content": "什么是LangChain？"}]
}, stream_mode="values"):
    # 每个块包含该时间点的完整状态
    latest_message = chunk["messages"][-1]
    if latest_message.content:
        print(f"智能体：{latest_message.content}")
    elif latest_message.tool_calls:
        print(f"正在调用工具：{[tc['name'] for tc in latest_message.tool_calls]}")
```





# 流式传输

流式传输对于增强基于 **LLM**（大型语言模型）构建的应用程序的**响应能力**至关重要。通过**渐进式**地显示输出，即使在完整的响应准备好之前，流式传输也能显著**改善用户体验 (UX)**，尤其是在处理 LLM 的延迟时。



LangChain 流式传输可能实现的功能：

- **流式传输代理进度** — 在每个代理步骤后获取状态更新。
- **流式传输 LLM 令牌** — 在语言模型生成令牌时进行流式传输。
- **流式传输自定义更新** — 发出用户定义的信号（例如，`"Fetched 10/100 records"`）。
- **流式传输多种模式** — 可选择 `updates`（代理进度）、`messages`（LLM 令牌 + 元数据）或 `custom`（任意用户数据）

## 智能体进度

要流式传输代理进度，请使用 `stream_mode="updates"` 的 [`stream`](https://langchain-doc.cn/v1/python/langchain/[https:/reference.langchain.com/python/langgraph/graphs/#langgraph.graph.state.CompiledStateGraph.stream](https://reference.langchain.com/python/langgraph/graphs/#langgraph.graph.state.CompiledStateGraph.stream)) 或 [`astream`](https://langchain-doc.cn/v1/python/langchain/[https:/reference.langchain.com/python/langgraph/graphs/#langgraph.graph.state.CompiledStateGraph.astream](https://reference.langchain.com/python/langgraph/graphs/#langgraph.graph.state.CompiledStateGraph.astream)) 方法。这会在**每个代理步骤**后发出一个事件。

例如，如果您有一个调用工具一次的代理，您应该会看到以下更新：

- **LLM 节点**: 带有工具调用请求的 [`AIMessage`](https://langchain-doc.cn/v1/python/langchain/[https:/reference.langchain.com/python/langchain/messages/#langchain.messages.AIMessage](https://reference.langchain.com/python/langchain/messages/#langchain.messages.AIMessage))
- **工具节点**: 带有执行结果的 [`ToolMessage`](https://langchain-doc.cn/v1/python/langchain/[https:/reference.langchain.com/python/langchain/messages/#langchain.messages.ToolMessage](https://reference.langchain.com/python/langchain/messages/#langchain.messages.ToolMessage))
- **LLM 节点**: 最终的 AI 响应

```python
from langchain.agents import create_agent


def get_weather(city: str) -> str:
    """获取给定城市的天气。"""

    return f"It's always sunny in {city}!"

agent = create_agent(
    model=model,
    tools=[get_weather],
)
for chunk in agent.stream(  # [!code highlight]
    {"messages": [{"role": "user", "content": "What is the weather in SF?"}]},
    stream_mode="updates",
):
    for step, data in chunk.items():
        print(f"step: {step}")
        print(f"content: {data['messages'][-1].content_blocks}")
```

## LLM tokens

要在 LLM 生成令牌时流式传输它们，请使用 `stream_mode="messages"`。您可以在下面看到代理流式传输工具调用和最终响应的输出。

```python
from langchain.agents import create_agent


def get_weather(city: str) -> str:
    """获取给定城市的天气。"""

    return f"It's always sunny in {city}!"

agent = create_agent(
    model=model,
    tools=[get_weather],
)
for token, metadata in agent.stream(  # [!code highlight]
    {"messages": [{"role": "user", "content": "What is the weather in SF?"}]},
    stream_mode="messages",
):
    # print(f"node: {metadata['langgraph_node']}")
    print(token.content,end="")
    # print("\n")
```

## 自定义更新

要流式传输工具在执行过程中发出的更新，您可以使用 [`get_stream_writer`](https://langchain-doc.cn/v1/python/langchain/[https:/reference.langchain.com/python/langgraph/config/#langgraph.config.get_stream_writer](https://reference.langchain.com/python/langgraph/config/#langgraph.config.get_stream_writer))。

并设置`stream_mode="custom"`

```python
from langchain.agents import create_agent
from langgraph.config import get_stream_writer  # [!code highlight]


def get_weather(city: str) -> str:
    """获取给定城市的天气。"""
    writer = get_stream_writer()  # [!code highlight]
    # 流式传输任何任意数据
    writer(f"Looking up data for city: {city}")
    writer(f"Acquired data for city: {city}")
    return f"It's always sunny in {city}!"

agent = create_agent(
    model=model,
    tools=[get_weather],
)

for chunk in agent.stream(
    {"messages": [{"role": "user", "content": "What is the weather in SF?"}]},
    stream_mode="custom"  # [!code highlight]
):
    print(chunk)
```

## 流式传输多种模式

可以通过将流模式作为列表传递来指定多种流式传输模式：`stream_mode=["updates", "custom"]`：



## 禁用流式传输

在某些应用程序中，您可能需要为给定的模型**禁用单个令牌的流式传输**。



# 中间件

中间件提供了一种方法，可以更精细地控制**代理**（Agent）内部的执行流程。

核心的代理循环包括调用模型、让模型选择并执行工具，然后在模型不再调用工具时结束

![core_agent_loop.png](./assets/core_agent_loop.png)

中间件在这些步骤的**之前**和**之后**暴露了钩子 (hooks)：

![middleware_final.png](./assets/middleware_final.png)

## 中间件可以做什么？

- **监控 (Monitor)**
  追踪代理行为，包括日志记录、分析和调试。
- **修改 (Modify)**
  转换提示词（prompts）、工具选择和输出格式。
- **控制 (Control)**
  添加重试、回退和提前终止逻辑。
- **强制执行 (Enforce)**
  应用速率限制、安全防护（guardrails）和个人身份信息（PII）检测。

通过将其传递给 `create_agent` 函数来添加中间件：

## 预置中间件

LangChain 为常见用例提供了预构建的中间件：

### 摘要

在接近**上下文窗口**（token limits）限制时，自动对对话历史进行摘要。

> **完美适用于：**
>
> - 超过上下文窗口的**长期对话**
> - 具有大量历史记录的**多轮对话**
> - 需要保留完整对话上下文的应用









# 结构化输出

结构化输出允许 **智能体（agents）** 以特定的、可预测的格式返回数据。这样，您无需解析自然语言响应，即可获得 **JSON 对象**、**Pydantic 模型** 或 **数据类（dataclasses）** 形式的结构化数据，供您的应用程序直接使用。

LangChain 的 [`create_agent`](https://langchain-doc.cn/v1/python/langchain/[https:/reference.langchain.com/python/langchain/agents/#langchain.agents.create_agent](https://reference.langchain.com/python/langchain/agents/#langchain.agents.create_agent)) 会自动处理结构化输出。用户设置所需的结构化输出 **模式（schema）**，当模型生成结构化数据时，它会被捕获、验证，并作为智能体状态中 `'structured_response'` 键的值返回。

```python
def create_agent(
    ...
    response_format: Union[
        ToolStrategy[StructuredResponseT],
        ProviderStrategy[StructuredResponseT],
        type[StructuredResponseT],
    ]
```

## 相应格式（Response Format）

该参数控制智能体如何返回结构化数据：

- **`ToolStrategy[StructuredResponseT]`**: 使用 **工具调用（tool calling）** 实现结构化输出。
- **`ProviderStrategy[StructuredResponseT]`**: 使用 **提供商原生（provider-native）** 的结构化输出功能。
- **`type[StructuredResponseT]`**: **模式类型（Schema type）** - 会根据模型功能自动选择最佳策略。
- **`None`**: 不进行结构化输出。

当直接提供模式类型时，LangChain 会自动选择：

- **`ProviderStrategy`**: 适用于支持原生结构化输出的模型（例如 [OpenAI](https://langchain-doc.cn/v1/python/integrations/providers/openai)、[Grok](https://langchain-doc.cn/v1/python/integrations/providers/xai)）。
- **`ToolStrategy`**: 适用于所有其他模型。

结构化响应将在智能体最终状态的 **`structured_response`** 键中返回。



## 提供商策略（Provider strategy）

一些模型提供商通过其 **API** 原生支持结构化输出（目前仅限 OpenAI 和 Grok）。在可用时，这是最可靠的方法。

要使用此策略，请配置 `ProviderStrategy`：

```python
class ProviderStrategy(Generic[SchemaT]):
    schema: type[SchemaT]
```

**`schema`** (必需)

定义结构化输出格式的模式。支持：

- **Pydantic 模型**: 带有字段验证的 `BaseModel` 子类。
- **数据类 (Dataclasses)**: 带有类型注解的 Python 数据类。
- **TypedDict**: 类型化字典类。
- **JSON Schema**: 带有 JSON 模式规范的字典。





## 工具调用策略 (Tool calling strategy)

对于不支持原生结构化输出的模型，LangChain 使用 **工具调用（tool calling）** 来达到相同的效果。这适用于所有支持工具调用的模型，即大多数现代模型。

要使用此策略，请配置 `ToolStrategy`：

```python
class ToolStrategy(Generic[SchemaT]):
    schema: type[SchemaT]
    tool_message_content: str | None
    handle_errors: Union[
        bool,
        str,
        type[Exception],
        tuple[type[Exception], ...],
        Callable[[Exception], str],
    ]
```

> **`schema`** (必需)
>
> 定义结构化输出格式的模式。支持：
>
> - **Pydantic 模型**: 带有字段验证的 `BaseModel` 子类。
> - **数据类 (Dataclasses)**: 带有类型注解的 Python 数据类。
> - **TypedDict**: 类型化字典类。
> - **JSON Schema**: 带有 JSON 模式规范的字典。
> - **联合类型 (Union types)**: 多个模式选项。模型将根据上下文选择最合适的模式。

> **`tool_message_content`** (可选)
>
> 生成结构化输出时，返回的工具消息的自定义内容。
> 如果未提供，默认为显示结构化响应数据的消息。

> **`handle_errors`** (可选)
>
> 结构化输出验证失败的错误处理策略。默认为 `True`。
>
> - **`True`**: 捕获所有错误并使用默认错误模板。
> - **`str`**: 捕获所有错误并使用此自定义消息。
> - **`type[Exception]`**: 仅捕获此异常类型并使用默认消息。
> - **`tuple[type[Exception], ...]`**: 仅捕获这些异常类型并使用默认消息。
> - **`Callable[[Exception], str]`**: 返回错误消息的自定义函数。
> - **`False`**: 不重试，让异常传播。

```python
# Pydantic Model 示例
from pydantic import BaseModel, Field
from typing import Literal
from langchain.agents import create_agent
from langchain.agents.structured_output import ToolStrategy


class ProductReview(BaseModel):
    """对产品评论的分析。"""
    rating: int | None = Field(description="产品的评分", ge=1, le=5)
    sentiment: Literal["positive", "negative"] = Field(description="评论的情感倾向")
    key_points: list[str] = Field(description="评论的要点。小写，每条 1-3 个词。")

agent = create_agent(
    model="openai:gpt-5",
    tools=tools,
    response_format=ToolStrategy(ProductReview)
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "Analyze this review: 'Great product: 5 out of 5 stars. Fast shipping, but expensive'"}]
})
result["structured_response"]
# ProductReview(rating=5, sentiment='positive', key_points=['fast shipping', 'expensive'])
```



### 自定义工作消息内容

`tool_message_content` 参数允许您自定义生成结构化输出时，对话历史中显示的消息：

```python
from pydantic import BaseModel, Field
from typing import Literal
from langchain.agents import create_agent
from langchain.agents.structured_output import ToolStrategy


class MeetingAction(BaseModel):
    """从会议记录中提取的行动事项。"""
    task: str = Field(description="需要完成的具体任务")
    assignee: str = Field(description="负责该任务的人员")
    priority: Literal["low", "medium", "high"] = Field(description="优先级")

agent = create_agent(
    model="openai:gpt-5",
    tools=[],
    response_format=ToolStrategy(
        schema=MeetingAction,
        tool_message_content="行动事项已捕获并添加到会议记录中！"
    )
)

agent.invoke({
    "messages": [{"role": "user", "content": "From our meeting: Sarah needs to update the project timeline as soon as possible"}]
})
```

输出

```python
================================= Tool Message =================================
Name: MeetingAction

Action item captured and added to meeting notes!
```

如果没有 `tool_message_content`，最终将是：

```python
================================= Tool Message =================================
Name: MeetingAction

Returning structured response: {'task': 'update the project timeline', 'assignee': 'Sarah', 'priority': 'high'}
```

### 错误处理

模型在通过工具调用生成结构化输出时可能会出错。LangChain 提供了智能的重试机制来自动处理这些错误。
