# Hugging Face

[HF镜像站](https://hf-mirror.com)

需要配置环境变量（详细查看hf镜像站）

在环境安装transformers模块

## 准备

- **GPU 显存**：推荐至少 8GB 显存（如 RTX 4060/3060 Ti），可流畅运行 3B/7B 量化模型；若仅用 CPU，需 32GB 以上内存，但推理速度会显著下降。
- **系统内存**：建议 16GB 起步，32GB 更佳，用于加载模型权重、缓存和运行推理框架。
- **Python 版本**：需安装 Python 3.8 或更高版本（推荐 3.10/3.11），避免版本不兼容导致库安装失败。

- 设置环境变量，使用国内镜像站

  > - *Linux*
  >   export HF_ENDPOINT=https://hf-mirror.com
  >
  > - *Windows Powershell*:
  >   $env:HF_ENDPOINT = "https://hf-mirror.com"
  >   建议将上面这一行写入 `~/.bashrc`。

- 安装包
  ```bash
  pip3 install torch torchvision --index-url https://download.pytorch.org/whl/cu126
  
  pip install transformers accelerate bitsandbytes
  ```

  



## 注意事项

- 下载的模型默认路径是：~/.cache/huggingface/hub/。下次加载相同模型时会直接读取缓存，不再联网下载。
- 模型可以联网加载也可以本地加载（需要指定路径，指定到包含模型的文件夹）

- 注意模型的名字写法，否则名字不一样就会重复下载相同的模型：
  - **`bert-base-chinese`** 是**旧式写法**（社区约定俗成的简称）。
  - **`google-bert/bert-base-chinese`** 是**新式写法**（官方推荐的完整路径）。

- AutoModel与AutoModelForMaskedLM加载模型的区别

  > 二者下载的模型一样（模型名一样），只不过在代码中加载的模型不一样（`MLM Head` 额外的预测头不一样)。Hugging Face 的缓存系统非常智能，它区分了“模型权重文件”和“模型架构代码”。
  >
  > huggingface通过from_pretrained加载的都是经过预训练的模型，但有些模型可能没有经过微调，也就是预测头的权重是随机的

- **`AutoModelForMaskedLM`** 与**`BertForMaskedLM`** 

  > - **`AutoModelForMaskedLM` (推荐 ✅)**：是**自动工厂**。它会根据你提供的模型名字，**自动判断**该用哪个具体的类（是 `BertForMaskedLM` 还是 `RobertaForMaskedLM`？），通用性最强。
  >
  > - **`BertForMaskedLM` (不推荐 ❌)**：是**指定型号**。你明确告诉代码“我要用 BERT 架构的完形填空模型”。如果你换个模型名字（比如换成 RoBERTa），代码就会报错。
  >
  >   
  >
  >  `AutoModelForMaskedLM` **(自动化/通用)**
  >
  > 这是官方**强烈推荐**的写法。
  >
  > - **原理**：它是一个“自动类”（Auto Class）。当你传入模型名称（如 `"bert-base-chinese"`）时，它会去读取该模型配置文件（`config.json`）中的 `architectures` 字段。
  > - **行为**：
  >   - 如果配置说它是 BERT，它就实例化 `BertForMaskedLM`。
  >   - 如果配置说它是 RoBERTa，它就实例化 `RobertaForMaskedLM`。
  >   - 如果配置说它是 ALBERT，它就实例化 `AlbertForMaskedLM`。
  > - **优点**：**代码复用性极高**。你写一套代码，可以加载任意架构的模型，只需要改模型名字字符串即可。

- huggingface上大多是微调模型，要自己费劲去找基座模型

  > 基座模型基本只有大企业会发布，可以下载别人已经微调好的模型（用AutoModelForxxx加载），但是可以使用AutoModel加载，只会加载基座模型的权重
  >
  > **微调模型 = 基座模型 + 训练好的头**
  > **基座模型 = 微调模型名字 - 任务后缀**
  >
  > 1. **首选官方源**：去 `google-bert`, `FacebookAI`, `meta-llama` 等官方组织下找。
  > 2. **看名字**：找带 `base`, `large` 但不带 `finetuned`, `sentiment` 的。







工具链（Libraries）

Hugging Face 提供了一套围绕预训练模型构建的工具库。这些组件彼此独立，又可以协同工作，覆盖了从数据处理到模型训练与推理的完整流程。

Ø **Datasets**

Datasets 是用于加载和处理数据集的工具库。支持从在线仓库或本地文件（如 CSV、JSON）加载文本数据，并支持清洗、编码、切分等预处理操作。处理后的数据可直接用于模型训练，是连接原始数据与模型输入的重要桥梁。

Ø **Tokenizers**

Tokenizers 是用于将文本转换为模型输入的工具。它支持文本分词、编码为 token ID，同时自动处理特殊符号、填充（padding）、attention mask 和句子对标记（token type ID）。分词器通常与模型配套使用，可通过统一接口加载。

Ø **Transformers**

Transformers 是 Hugging Face 最核心的库，用于加载、使用和微调各种预训练模型。该库统一了模型接口，支持数百种模型结构，如 BERT、GPT 等，用户可以通过一行代码 from_pretrained()直接加载公开模型，快速用于推理或训练。









## 查看模型文档

第一种方式：**模型专属文档 (Model Card / README)**

第二种方式：**huggingface文档**

> 如果模型卡片写得不清楚，你需要去查**这个模型所属的架构类**的官方文档。
> 因为 `uer/roberta-base...` 本质上是 **RoBERTa** 架构，`chatglm3` 本质上是 **GLM** 架构。同一架构的模型，输入输出格式是**完全一样**的！
>
> 1. 在模型卡片上找到 **Tags** 或 **Base Model**（例如看到 `roberta` 或 `bert`）。
> 2. 前往 Hugging Face 官方文档库：https://huggingface.co/docs/transformers。
> 3. 在左侧菜单找到 **API/MODELS** -> 选择对应的架构（如 `RobertaModel` 或 `RobertaForSequenceClassification`）。
> 4. 点击具体的类（Class），你会看到极其详细的：
>    - **`forward` 方法签名**：明确列出所有输入参数（`input_ids`, `attention_mask`, `token_type_ids` 等）。
>    - **`Returns` (返回值)**：明确列出输出是什么对象（`SequenceClassifierOutput`），里面包含哪些字段（`logits`, `loss`, `hidden_states`）。
>    - **Example 代码块**：官方提供的标准调用示例。

第三种方式 **查看源码**

> 如果文档还没看懂，**代码本身就是最真实的文档**。
> 在模型卡片页面，点击 **"Files and versions"** 标签页：
>
> 1. 查看 `config.json`：
>    - 看 `architectures`: 确认类名。
>    - 看 `id2label`: **直接看到数字对应什么标签**（这是解决“输出是什么意思”的最快方法）。
> 2. 查看 `tokenizer_config.json` 或 `special_tokens_map.json`：
>    - 了解分词器的特殊设置。

第四种方法 help方法

```python
tokenizer = AutoTokenizer.from_pretrained("bert-base-chinese")
model = AutoModel.from_pretrained("bert-base-chinese")
help(model)
```







## pipeline

`pipeline` 是 Hugging Face `transformers` 库中**最强大、最易用**的功能，也是新手上手的最佳入口。

它的核心理念是：**“一键式”解决常见的深度学习任务**。你不需要关心底层的模型架构、分词细节、数据预处理或后处理逻辑，只需告诉它你想做什么（比如“情感分析”或“翻译”），**它会自动帮你加载合适的默认模型并返回结果。**

当你只传入任务名称（如 `"sentiment-analysis"`）而不指定模型时，`pipeline` 会加载一个**硬编码在库内部的默认模型**。这个默认模型通常是**英文的**，且针对通用数据集（如 SST-2）训练过。



### **1. 为什么使用 Pipeline？**

在没有 `pipeline` 之前，运行一个模型通常需要 5-10 行代码：

1. 加载 Tokenizer。
2. 加载 Model。
3. 对文本进行分词（Tokenize）。
4. 将输入转为 Tensor 并传入模型。
5. 处理输出（Softmax, 解码 ID 等）。

**使用 `pipeline` 后，只需要 1-2 行代码：**

```python
from transformers import pipeline
classifier = pipeline("sentiment-analysis")
classifier("I love AI!")
```

------

### **2. 核心用法详解**

#### **基础语法**

```python
from transformers import pipeline

# 格式：pipeline(任务名称, model=模型名称, device=设备)
pipe = pipeline("task_name", model="model_name_or_path")
result = pipe("你的输入数据")
```

#### **常用任务列表 (Task Names)**

Hugging Face 支持数十种任务，以下是最高频的：

| 任务名称 (Task)                    | 功能描述                      | 默认模型示例                                       |
| :--------------------------------- | :---------------------------- | :------------------------------------------------- |
| **`sentiment-analysis`**           | 情感分析 (正面/负面)          | `distilbert-base-uncased-finetuned-sst-2-english`  |
| **`text-generation`**              | 文本生成 (续写/聊天)          | `gpt2`, `Qwen/Qwen2.5-7B-Instruct`                 |
| **`translation`**                  | 翻译 (需指定语言对)           | `Helsinki-NLP/opus-mt-en-zh`                       |
| **`summarization`**                | 文本摘要                      | `facebook/bart-large-cnn`                          |
| **`question-answering`**           | 阅读理解 (从文中找答案)       | `distilbert-base-cased-distilled-squad`            |
| **`fill-mask`**                    | 完形填空 (预测掩码词)         | `bert-base-uncased`                                |
| **`ner`**                          | 命名实体识别 (人名/地名/机构) | `dbmdz/bert-large-cased-finetuned-conll03-english` |
| **`image-classification`**         | 图像分类                      | `google/vit-base-patch16-224`                      |
| **`object-detection`**             | 物体检测                      | `facebook/detr-resnet-50`                          |
| **`automatic-speech-recognition`** | 语音转文字                    | `openai/whisper-small`                             |
| **`text-to-speech`**               | 文字转语音                    | `microsoft/speecht5_tts`                           |
| **`feature-extraction`**           | 提取文本向量 (Embedding)      | `bert-base-uncased`                                |

------

### **3. 实战代码示例**

#### **示例 A：情感分析 (最简单)**

```python
from transformers import pipeline

# 初始化管道 (首次运行会自动下载模型)
classifier = pipeline("sentiment-analysis")

# 单条预测
result = classifier("Hugging Face is the best platform for AI!")
print(result)
# 输出: [{'label': 'POSITIVE', 'score': 0.9998}]

# 批量预测 (自动并行处理)
texts = [
    "I hate waiting in line.",
    "The movie was fantastic and exciting."
]
results = classifier(texts)
print(results)
```

#### **示例 B：指定模型和语言 (翻译)**

如果你想做特定的任务（如英译中），必须指定模型，因为默认模型可能不支持。

```python
from transformers import pipeline

# 指定一个英译中的模型
translator = pipeline("translation", model="Helsinki-NLP/opus-mt-en-zh")

text = "Artificial Intelligence is changing the world."
result = translator(text)
print(result[0]['translation_text'])
# 输出: 人工智能正在改变世界。
```

#### **示例 C：文本生成 (类似 ChatGPT)**

```python
from transformers import pipeline

# 使用一个较小的生成模型演示
generator = pipeline("text-generation", model="gpt2")

prompt = "Once upon a time, in a land far away,"
# max_new_tokens: 生成的新令牌数量
# num_return_sequences: 返回几个不同的结果
results = generator(prompt, max_new_tokens=20, num_return_sequences=2)

for i, res in enumerate(results):
    print(f"--- 结果 {i+1} ---")
    print(res['generated_text'])
```

#### **示例 D：图像处理 (需要安装 Pillow)**

```python
pip install pillow
```

```python
from transformers import pipeline
from PIL import Image
import requests

# 图像分类管道
classifier = pipeline("image-classification")

# 加载一张网络图片
img_url = "https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/pipeline-cat-chonk.jpeg"
image = Image.open(requests.get(img_url, stream=True).raw)

result = classifier(image)
print(result)
# 输出: [{'label': 'lynx, catamount', 'score': 0.95}, ...]
```

------

### **4. 高级参数配置**

`pipeline` 接受许多参数来优化性能或改变行为：

1. **`device`**: 指定运行设备。

   - `device=-1`: CPU (默认)
   - `device=0`: 第一个 GPU
   - `device_map="auto"`: 自动分配到大模型可用的所有显卡 (推荐用于大模型)

   ```python
   pipe = pipeline("text-generation", device_map="auto")
   ```

2. **`model` 和 `tokenizer`**:

   - 你可以手动加载复杂的模型配置，然后传给 pipeline。
   - 也可以直接传模型名字符串（如上例）。

3. **任务特定参数**:

   - 在调用 `pipe()` 时，可以传递该任务特有的参数。
   - 例如生成任务：`max_length`, `temperature`, `top_k`, `do_sample`。
   - 例如问答任务：`handle_impossible_answer=True`。

   ```python
   # 温度采样，让生成更多样化
   generator("Hello world", temperature=0.8, do_sample=True, max_new_tokens=50)
   ```

4. **`batch_size`**:

   - 处理大量数据时，设置批次大小可以提高吞吐量。
   - `pipe(data_list, batch_size=16)`

------

### **5. 如何查看支持的所有任务？**

如果你想知道当前版本支持哪些任务，可以运行：

```python
from transformers import pipeline

# 列出所有支持的任务
print(pipeline.tasks) 
# 或者查看帮助
help(pipeline)
```

*注：更准确的方法是查阅官方文档的任务列表，因为代码中的枚举可能随版本更新。*

### **6. 局限性与何时不使用 Pipeline**

虽然 `pipeline` 很方便，但在以下情况你可能需要**手动拆解**（使用 Tokenizer + Model）：

1. **极度定制化**：你需要修改模型的中间层输出，或者需要特殊的预处理/后处理逻辑。
2. **性能极致优化**：在生产环境中，为了减少延迟，可能需要手动管理批处理和显存，去掉 pipeline 的通用开销。
3. **复杂的多模态交互**：当输入输出非常复杂，不是标准的“文本进文本出”或“图片进标签出”时。
4. **微调训练**：`pipeline` 主要用于**推理 (Inference)**。训练模型时通常使用 `Trainer` 类或自定义训练循环。



---



## 定制化开发

一旦你需要**精细化定制**（例如：自定义损失函数、提取中间层特征、修改注意力机制、部署到特定硬件、或者优化推理速度），你就必须**拆解 `pipeline`**，手动控制每一个环节。

这就是所谓的 **"Native Transformers" (原生开发模式)**。

要把 `pipeline` 拆开，只需要掌握这四个组件的独立调用：

1. **`AutoTokenizer`**：负责文本 ➡️ 数字 (输入准备)
2. **`AutoModelFor...`**：负责加载具体的任务模型 (模型加载)
3. **`model(\**inputs)`**：负责核心计算 (前向传播)
4. **`Post-processing`**：负责数字 ➡️ 业务结果 (输出解析)

### **💻 实战代码：从零手写一个“情感分析”流程**

假设我们要复现 `pipeline("sentiment-analysis")` 的功能，但我们要完全掌控它。

#### **1. 导入与加载 (完全可控)**

```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification
import torch
import torch.nn.functional as F

# 模型名称
model_name = "uer/roberta-base-finetuned-jd-binary-chinese"

# 【步骤 1】加载分词器 (可以自定义参数)
# 比如：你可以强制设定最大长度，或者添加特殊令牌
tokenizer = AutoTokenizer.from_pretrained(model_name)

# 【步骤 2】加载模型 (可以自定义配置)
# 比如：你可以加载后修改 dropout，或者冻结某些层
model = AutoModelForSequenceClassification.from_pretrained(model_name)
model.eval()  # 切换到评估模式 (关闭 Dropout 等训练特性)

# 如果有 GPU，移动到 GPU
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
model.to(device)
```

#### **2. 数据预处理 (精细化控制的核心)**

在 `pipeline` 里，你看不见这里发生了什么。现在你可以控制：

- **Padding 策略**：是补到最长句 (`longest`) 还是固定长度 (`max_length`)？
- **截断策略**：是从头截断还是从尾截断？
- **返回类型**：要 PyTorch Tensor 还是 Numpy Array？

```python
texts = [
    "这手机太棒了，运行速度飞快！",   # 正面
    "垃圾产品，用了两天就坏了。",     # 负面
    "还可以吧，中规中矩。"           # 模糊
]

# 【步骤 3】手动分词 (这里是定制的关键点)
inputs = tokenizer(
    texts,
    padding=True,               # 自动填充到 batch 中最长的句子
    truncation=True,            # 超过最大长度自动截断
    max_length=512,             # 强制限制最大长度 (可调整)
    return_tensors="pt",        # 返回 PyTorch 张量 (也可以是 "np")
    return_attention_mask=True  # 显式要求返回 attention_mask
)

# 将数据移动到设备
inputs = {key: value.to(device) for key, value in inputs.items()}

print("Input IDs shape:", inputs['input_ids'].shape)
print("Attention Mask:\n", inputs['attention_mask'])
```

#### **3. 模型推理 (黑盒变白盒)**

这是最关键的一步。`pipeline` 内部调用的就是这个。

```python
# 【步骤 4】执行前向传播
# 使用 torch.no_grad() 禁止计算梯度，节省显存并加速
with torch.no_grad():
    outputs = model(**inputs)

# 查看 outputs 对象里有什么
# 对于分类任务，通常包含: loss (如果给了labels), logits
print("Outputs keys:", outputs.keys()) 
# 输出: odict_keys(['logits'])
```

#### **4. 后处理 (业务逻辑定制)**

`pipeline` 会自动帮你做 `Softmax` 并映射标签。现在你要自己做，这意味着你可以：

- 修改阈值 (Threshold)。
- 获取 Top-K 个结果。
- 提取未归一化的 `logits` 用于集成学习。

```python
# 【步骤 5】解析 Logits
logits = outputs.logits  # 形状: [batch_size, num_labels]

# 原始 logits (未归一化分数)
print("Raw Logits:", logits)

# 手动做 Softmax 获取概率
probabilities = F.softmax(logits, dim=-1)

# 获取预测类别 (取概率最大的索引)
predicted_class_ids = torch.argmax(probabilities, dim=-1)

# 映射回标签字符串
# 模型配置里存有 id 到 label 的映射关系
labels_map = model.config.id2label

results = []
for i, text in enumerate(texts):
    pred_id = predicted_class_ids[i].item()
    score = probabilities[i][pred_id].item()
    label = labels_map[pred_id]
    
    results.append({
        "text": text,
        "prediction": label,
        "confidence": f"{score:.4f}",
        "all_scores": probabilities[i].cpu().numpy() # 如果你想看所有类别的得分
    })

# 打印结果
for res in results:
    print(f"文本: {res['text']}")
    print(f"预测: {res['prediction']} (置信度: {res['confidence']})")
    print("-" * 30)
```

------

### **🚀 为什么要这么做？(精细化定制的场景)**

当你拆开 `pipeline` 后，你就可以做以下 `pipeline` 很难做到的事情：

#### **场景 1：提取中间层特征 (Feature Extraction)**

你想用 BERT 做向量检索，不需要分类结果，只需要最后一层的隐藏状态。

```python
# 加载基础模型 (没有分类头)
base_model = AutoModel.from_pretrained("bert-base-chinese")
outputs = base_model(**inputs)

# 提取 [CLS] 向量作为整句表示
cls_vector = outputs.last_hidden_state[:, 0, :] 
# 现在 cls_vector 就是你的句子嵌入，可以做余弦相似度计算了
```

#### **场景 2：自定义阈值与多标签逻辑**

`pipeline` 默认只给一个最高分的标签。如果你要做多标签分类（一个评论既是“物流差”又是“态度差”）：

```python
# 假设是 sigmoid 激活的多标签任务
probs = torch.sigmoid(logits)
threshold = 0.6
predictions = (probs > threshold).int()
# 现在你可以灵活控制每个标签的阈值
```

#### **场景 3：批量流式处理 (Streaming)**

`pipeline` 通常是一次性处理完。如果你有几百万条数据，需要边读边处理，避免内存爆炸：

```python
def process_large_file(file_path):
    batch_texts = []
    for line in open(file_path):
        batch_texts.append(line.strip())
        if len(batch_texts) == 32: # 满 32 条处理一次
            inputs = tokenizer(batch_texts, ..., return_tensors="pt")
            outputs = model(**inputs)
            # 保存结果到磁盘...
            batch_texts = [] # 清空
    
    # 处理剩余不足 32 条的
    if batch_texts:
        # ... 同上
```

#### **场景 4：模型微调 (Fine-tuning)**

`pipeline` 只能推理。如果要训练，必须用这种原生写法，配合 `optimizer` 和 `loss`：

```python
labels = torch.tensor([1, 0, 1]).to(device) # 真实标签
outputs = model(**inputs, labels=labels)    # 传入 labels
loss = outputs.loss                         # 自动计算交叉熵损失
loss.backward()                             # 反向传播
optimizer.step()                            # 更新权重
```
