---
title: Langchain 详解与实战
date: 2024-02-07
categories:
  - 人工智能&大数据
tags:
  - LangChain
published: true
---
# 0 参考资料

## 0.1 网站

- LangChain 中文网：[https://www.langchain.asia/](https://www.langchain.asia/)
- LangChain Github：[https://github.com/langchain-ai/langchain](https://github.com/langchain-ai/langchain)
- LangChain API文档：[https://python.langchain.com/docs/get_started](https://python.langchain.com/docs/get_started)

## 0.2 课程
- [《LangChain 实战课》](https://time.geekbang.org/column/intro/655)

## 0.3 开源项目
1. AutoGPT：[https://github.com/Significant-Gravitas/AutoGPT](https://github.com/Significant-Gravitas/AutoGPT)
2. AgentBench：[https://github.com/THUDM/AgentBench](https://github.com/THUDM/AgentBench) （测试 Agent 性能）
3. ChatALL：[https://github.com/sunner/ChatALL](https://github.com/sunner/ChatALL) （对多个大模型进行整合输出）

# 1 LangChain 安装和使用

## 1.1 LangChain 安装

```bash
pip install langchain[llms]
# 升级到最新版本
pip install --upgrade langchain
```
## 1.2 OpenAI API

OpenAI API文档：[https://platform.openai.com/docs/introduction](https://platform.openai.com/docs/introduction)
### 1.2.1 调用 Text 模型
```python
from openai import OpenAI  
client = OpenAI()  
  
response = client.completions.create(  
  model="gpt-3.5-turbo-instruct",  
  temperature=0.5,  
  max_tokens=1000,  
  prompt="请给我的花店起个名")  
  
print(response.choices[0].text.strip())
```

在使用OpenAI的文本生成模型时，可以通过一些参数来控制输出的内容和样式。这里我总结为了一些常见的参数。

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240305153824.png)

调用 Text 模型后，响应对象的主要字段包括：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240305154036.png)

choices字段是一个列表，因为在某些情况下，你可以要求模型生成多个可能的输出。每个选择都是一个字典，其中包含以下字段：
- text：模型生成的文本。
- finish_reason：模型停止生成的原因，可能的值包括 stop（遇到了停止标记）、length（达到了最大长度）或 temperature（根据设定的温度参数决定停止）。
所以， `response.choices[0].text.strip()` 这行代码的含义是：从响应中获取第一个（如果在调用大模型时，没有指定n参数，那么就只有唯一的一个响应）选择，然后获取该选择的文本，并移除其前后的空白字符。这通常是你想要的模型的输出。

### 1.2.2 调用 Chat 模型
```python
from openai import OpenAI
client = OpenAI()

response = client.chat.completions.create(
  model="gpt-3.5-turbo",
  messages=[
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Who won the world series in 2020?"},
    {"role": "assistant", "content": "The Los Angeles Dodgers won the World Series in 2020."},
    {"role": "user", "content": "Where was it played?"}
  ]
)
```
在OpenAI的Chat模型中，system、user和assistant都是消息的角色。每一种角色都有不同的含义和作用。
- system：系统消息主要用于设定对话的背景或上下文。这可以帮助模型理解它在对话中的角色和任务。例如，你可以通过系统消息来设定一个场景，让模型知道它是在扮演一个医生、律师或者一个知识丰富的AI助手。系统消息通常在对话开始时给出。
- user：用户消息是从用户或人类角色发出的。它们通常包含了用户想要模型回答或完成的请求。用户消息可以是一个问题、一段话，或者任何其他用户希望模型响应的内容。
- assistant：助手消息是模型的回复。例如，在你使用API发送多轮对话中新的对话请求时，可以通过助手消息提供先前对话的上下文。然而，请注意在对话的最后一条消息应始终为用户消息，因为模型总是要回应最后这条用户消息。

在使用Chat模型生成内容后，返回的 **响应**，也就是response会包含一个或多个choices，每个choices都包含一个message。每个message也都包含一个role和content。role可以是system、user或assistant，表示该消息的发送者，content则包含了消息的实际内容。响应内容格式如下：
```bash
{
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "content": "The 2020 World Series was played in Texas at Globe Life Field in Arlington.",
        "role": "assistant"
      },
      "logprobs": null
    }
  ],
  "created": 1677664795,
  "id": "chatcmpl-7QyqpwdfhqwajicIEznoc6Q47XAyW",
  "model": "gpt-3.5-turbo-0613",
  "object": "chat.completion",
  "usage": {
    "completion_tokens": 17,
    "prompt_tokens": 57,
    "total_tokens": 74
  }
}
```

以下是个字段的含义：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240305154545.png)

## 1.3 通过 LangChain 调用 Text 模型和 Chat模型
### 1.3.1 调用 Text 模型

```python
from langchain.llms import OpenAI  
llm = OpenAI(    
    model="gpt-3.5-turbo-instruct",  
    temperature=0.8,  
    max_tokens=60,)  
response = llm.predict("请给我的花店起个名")  
print(response)
```

### 1.3.2 调用 Chat 模型
```python
from langchain_openai import ChatOpenAI  
chat = ChatOpenAI(model="gpt-4",  
                    temperature=0.8,  
                    max_tokens=60)  
from langchain.schema import (  
    HumanMessage,  
    SystemMessage  
)  
messages = [  
    SystemMessage(content="你是一个很棒的智能助手"),  
    HumanMessage(content="请给我的花店起个名")  
]  
response = chat(messages)  
print(response)
```
输出：
```bash
content='为您的花店起名时，我们可以考虑一些富有创意和意义的选项，同时也要易于记忆。这里有几个建议：\n\n1. **花语轩**：这个名字简洁优雅，寓意每一朵花都有其独特的语言和意义，适合一个提供精致花艺设计的花店。\n2. **绽放轨迹**：暗示着花朵从含苞待放到绽放的美丽过程，也象征着人生中美好瞬间的捕捉和珍藏。\n3. **彩云间花舍**：给人一种梦幻而温馨的感觉，像是在彩云之间开设的一家花店，充满了浪漫和想象空间。\n4. **时光花语**：这个名字传达了花朵与时间的关系，每一朵花都代表着一个特定的时刻或记忆。'
```

> LangChain 不止支持 OpenAI模型，可以试试 HuggingFace 开源社区的其他模型

```python
from langchain import HuggingFaceHub
llm = HuggingFaceHub(model_id="bigscience/bloom-1b7")
```

# 1.4 LangChain 构建问答系统

- [文档QA系统](https://github.com/huangjia2019/langchain/tree/main/02_%E6%96%87%E6%A1%A3QA%E7%B3%BB%E7%BB%9F)

# 2 LangChain 核心组件
## 2.1 模型 I/O

Model I/O 文档：[https://python.langchain.com/docs/modules/model_io/](https://python.langchain.com/docs/modules/model_io/)
### 2.1.1 Model I/O

可以把对模型的使用过程拆解成三块，分别是 **输入提示**（对应图中的Format）、 **调用模型**（对应图中的Predict）和 **输出解析**（对应图中的Parse）。这三块形成了一个整体，因此在LangChain中这个过程被统称为 **Model I/O**（Input/Output）。

在模型 I/O的每个环节，LangChain都为咱们提供了模板和工具，快捷地形成调用各种语言模型的接口。
1. **提示模板**：使用模型的第一个环节是把提示信息输入到模型中，你可以创建LangChain模板，根据实际需求动态选择不同的输入，针对特定的任务和应用调整输入。
2. **语言模型**：LangChain允许你通过通用接口来调用语言模型。这意味着无论你要使用的是哪种语言模型，都可以通过同一种方式进行调用，这样就提高了灵活性和便利性。
3. **输出解析**：LangChain还提供了从模型输出中提取信息的功能。通过输出解析器，你可以精确地从模型的输出中获取需要的信息，而不需要处理冗余或不相关的数据，更重要的是还可以把大模型给回的非结构化文本，转换成程序可以处理的结构化数据。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240305165848.png)

### 2.1.2 提示模板
提示框架包括如下部分：

- **指令**（Instuction）告诉模型这个任务大概要做什么、怎么做，比如如何使用提供的外部信息、如何处理查询以及如何构造输出。这通常是一个提示模板中比较固定的部分。一个常见用例是告诉模型“你是一个有用的XX助手”，这会让他更认真地对待自己的角色。
- **上下文**（Context）则充当模型的额外知识来源。这些信息可以手动插入到提示中，通过矢量数据库检索得来，或通过其他方式（如调用API、计算器等工具）拉入。一个常见的用例时是把从向量数据库查询到的知识作为上下文传递给模型。
- **提示输入**（Prompt Input）通常就是具体的问题或者需要大模型做的具体事情，这个部分和“指令”部分其实也可以合二为一。但是拆分出来成为一个独立的组件，就更加结构化，便于复用模板。这通常是作为变量，在调用模型之前传递给提示模板，以形成具体的提示。
- **输出指示器**（Output Indicator）标记​​要生成的文本的开始。这就像我们小时候的数学考卷，先写一个“解”，就代表你要开始答题了。如果生成 Python 代码，可以使用 “import” 向模型表明它必须开始编写 Python 代码（因为大多数 Python 脚本以import开头）。这部分在我们和ChatGPT对话时往往是可有可无的，当然LangChain中的代理在构建提示模板时，经常性的会用一个“Thought：”（思考）作为引导词，指示模型开始输出自己的推理（Reasoning）。

[提示工程课程](https://learn.deeplearning.ai/login?redirect_course=chatgpt-prompt-eng)

**LangChain 提示模板的类型：**
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240306092557.png)

#### 2.1.2.1 PromptTemplate
通过 PromptTemplate 构建提示模板，第一种方式，通过占位符的方式：
```python
# 导入LangChain中的提示模板  
from langchain.prompts import PromptTemplate  
# 创建原始模板  
template = """您是一位专业的鲜花店文案撰写员。\n  
对于售价为 {price} 元的 {flower_name} ，您能提供一个吸引人的简短描述吗？  
"""  
# 根据原始模板创建LangChain提示模板  
prompt = PromptTemplate.from_template(template)  
# 打印LangChain提示模板的内容  
print(prompt)  
print(prompt.format(flower_name='鲜花', price='50'))
```
提示模板的具体内容如下：
```bash
input_variables=['flower_name', 'price'] 
template='您是一位专业的鲜花店文案撰写员。\n\n对于售价为 {price} 元的 {flower_name} ，您能提供一个吸引人的简短描述吗？\n'
您是一位专业的鲜花店文案撰写员。

对于售价为 50 元的 鲜花 ，您能提供一个吸引人的简短描述吗？
```
另一种方式，通过 PromptTemplate 类的构造函数，如下：
```python
from langchain.prompts import PromptTemplate

prompt = PromptTemplate(  
    input_variables=["product", "market"],  
    template="你是业务咨询顾问。对于一个面向 {market} 市场的，专注于销售 {product} 的公司，你会推荐哪个名字？"  
)  
print(prompt.format(product="鲜花", market="高端"))
```
输出的提示内容：
```bash
你是业务咨询顾问。对于一个面向 高端 市场的，专注于销售 鲜花 的公司，你会推荐哪个名字？
```

LangChain 提供了多个类和函数，也 **为各种应用场景设计了很多内置模板，使构建和使用提示变得容易**。

#### 2.1.2.2 ChatPromptTemplate
ChatPromptTemplate 对应聊天模型的模板，提供一系列角色设计。
```python
# 导入聊天消息类模板  
from langchain.prompts import (  
    ChatPromptTemplate,  
    SystemMessagePromptTemplate,  
    HumanMessagePromptTemplate,  
)  
# 模板的构建  
template="你是一位专业顾问，负责为专注于{product}的公司起名。"  
system_message_prompt = SystemMessagePromptTemplate.from_template(template)  
human_template="公司主打产品是{product_detail}。"  
human_message_prompt = HumanMessagePromptTemplate.from_template(human_template)  
prompt_template = ChatPromptTemplate.from_messages([system_message_prompt, human_message_prompt])  
  
# 格式化提示消息生成提示  
prompt = prompt_template.format_prompt(product="鲜花装饰", product_detail="创新的鲜花设计。").to_messages()  
print(prompt)
```
输出的提示内容是一个列表：
```python
[SystemMessage(content='你是一位专业顾问，负责为专注于鲜花装饰的公司起名。'), HumanMessage(content='公司主打产品是创新的鲜花设计。。')]
```

#### 2.1.2.3 FewShotPromptTemplate
- FewShot 思想
Few-Shot（少样本）、One-Shot（单样本）和与之对应的 Zero-Shot（零样本）的概念都起源于机器学习。如何让机器学习模型在极少量甚至没有示例的情况下学习到新的概念或类别，对于许多现实世界的问题是非常有价值的，因为我们往往无法获取到大量的标签化数据。

FewShotPromptTemplate 的使用分为以下几步：
1. 创建示例样本
2. 创建一个提示模板：会根据指定的输入变量和模板产生提示
3. 创建 FewShotPromptTemplate 对象
4. 调用大模型创建新文案
```python
# 1. 创建一些示例  
samples = [  
  
  {  
    "flower_type": "玫瑰",  
    "occasion": "爱情",  
    "ad_copy": "玫瑰，浪漫的象征，是你向心爱的人表达爱意的最佳选择。"  
  },  
  {  
    "flower_type": "康乃馨",  
    "occasion": "母亲节",  
    "ad_copy": "康乃馨代表着母爱的纯洁与伟大，是母亲节赠送给母亲的完美礼物。"  
  },  
  {  
    "flower_type": "百合",  
    "occasion": "庆祝",  
    "ad_copy": "百合象征着纯洁与高雅，是你庆祝特殊时刻的理想选择。"  
  },  
  {  
    "flower_type": "向日葵",  
    "occasion": "鼓励",  
    "ad_copy": "向日葵象征着坚韧和乐观，是你鼓励亲朋好友的最好方式。"  
  }  
]  
  
# 2. 创建一个提示模板  
from langchain.prompts.prompt import PromptTemplate  
prompt_sample = PromptTemplate(input_variables=["flower_type", "occasion", "ad_copy"],   
                               template="鲜花类型: {flower_type}\n场合: {occasion}\n文案: {ad_copy}")  
print(prompt_sample.format(**samples[0]))  
  
# 3. 创建一个FewShotPromptTemplate对象  
from langchain.prompts.few_shot import FewShotPromptTemplate  
prompt = FewShotPromptTemplate(  
    examples=samples,  
    example_prompt=prompt_sample,  
    suffix="鲜花类型: {flower_type}\n场合: {occasion}",  
    input_variables=["flower_type", "occasion"]  
)  
print(prompt.format(flower_type="野玫瑰", occasion="爱情"))  
  
# 4. 把提示传递给大模型  
# import os  
# os.environ["OPENAI_API_KEY"] = '你的OpenAI API Key'  
from langchain_openai import OpenAI  
model = OpenAI(model_name=' gpt-3.5-turbo-0613')  
result = model(prompt.format(flower_type="野玫瑰", occasion="爱情"))  
print(result)
```
生成的 prompt 如下：
```bash
鲜花类型: 玫瑰
场合: 爱情
文案: 玫瑰，浪漫的象征，是你向心爱的人表达爱意的最佳选择。
鲜花类型: 玫瑰
场合: 爱情
文案: 玫瑰，浪漫的象征，是你向心爱的人表达爱意的最佳选择。

鲜花类型: 康乃馨
场合: 母亲节
文案: 康乃馨代表着母爱的纯洁与伟大，是母亲节赠送给母亲的完美礼物。

鲜花类型: 百合
场合: 庆祝
文案: 百合象征着纯洁与高雅，是你庆祝特殊时刻的理想选择。

鲜花类型: 向日葵
场合: 鼓励
文案: 向日葵象征着坚韧和乐观，是你鼓励亲朋好友的最好方式。

鲜花类型: 野玫瑰
场合: 爱情
```
#### 2.1.2.4 示例选择器SemanticSimilarityExampleSelector
如果示例很多，那么一次性把所有示例发送给模型是不现实而且低效的。另外，每次都包含太多的Token也会浪费流量。
LangChain提供了示例选择器，来选择最合适的样本。（注意，因为示例选择器使用向量相似度比较的功能，此处需要安装向量数据库，这里使用的是开源的Chroma，也可以选择其他如 Qdrant 等。）
先安装 Chroma 向量数据库：
```bash
pip install chromadb
```
使用示例选择器的示例代码：
```python
# 使用示例选择器  
from langchain.prompts.example_selector import SemanticSimilarityExampleSelector  
from langchain_community.vectorstores import Chroma  
from langchain_openai import OpenAIEmbeddings  
  
# 初始化示例选择器  
example_selector = SemanticSimilarityExampleSelector.from_examples(  
    samples,  
    OpenAIEmbeddings(),  
    Chroma,  
    k=1  
)  
  
# 创建一个使用示例选择器的FewShotPromptTemplate对象  
prompt = FewShotPromptTemplate(  
    example_selector=example_selector,  
    example_prompt=prompt_sample,  
    suffix="鲜花类型: {flower_type}\n场合: {occasion}",  
    input_variables=["flower_type", "occasion"]  
)  
print(prompt.format(flower_type="红玫瑰", occasion="爱情"))
```
输出：
```bash
鲜花类型: 玫瑰
场合: 爱情
文案: 玫瑰，浪漫的象征，是你向心爱的人表达爱意的最佳选择。

鲜花类型: 红玫瑰
场合: 爱情
```
1. 首先创建了SemanticSimilarityExampleSelector对象，这个对象可以根据语义相似性选择最相关的示例。
2. 创建了一个新的FewShotPromptTemplate对象，这个对象使用了上一步创建的选择器来选择最相关的示例生成提示。
3. 用这个模板生成了一个新的提示，提示中需要创建的是红玫瑰的文案，所以，示例选择器example_selector会根据语义的相似度（余弦相似度）找到最相似的示例，也就是“玫瑰”，并用这个示例构建一个FewShot模板。

#### 2.1.2.5 思维链 和 思维树
1. 思维链

Chain of Thought（CoT，即“思维链”）的核心思想是通过生成一系列中间推理步骤来增强模型的推理能力。在Few-Shot CoT 中通过提供链式思考示例传递给模型，根据示例模型可以生成正确的答案。Zero-Shot CoT 是需要告诉模型“ **让我们一步步的思考（Let’s think step by step）**”，模型就能够给出更好的答案。

**思维链 实战示例：**
```python
# 创建聊天模型  
from langchain_openai import ChatOpenAI  
llm = ChatOpenAI(temperature=0)  
  
# 设定 AI 的角色和目标  
role_template = "你是一个为花店电商公司工作的AI助手, 你的目标是帮助客户根据他们的喜好做出明智的决定"  
  
# CoT 的关键部分，AI 解释推理过程，并加入一些先前的对话示例（Few-Shot Learning）  
cot_template = """  
作为一个为花店电商公司工作的AI助手，我的目标是帮助客户根据他们的喜好做出明智的决定。   
我会按部就班的思考，先理解客户的需求，然后考虑各种鲜花的涵义，最后根据这个需求，给出我的推荐。  
同时，我也会向客户解释我这样推荐的原因。  
  
示例 1:  人类：我想找一种象征爱情的花。  
  AI：首先，我理解你正在寻找一种可以象征爱情的花。在许多文化中，红玫瑰被视为爱情的象征，这是因为它们的红色通常与热情和浓烈的感情联系在一起。因此，考虑到这一点，我会推荐红玫瑰。红玫瑰不仅能够象征爱情，同时也可以传达出强烈的感情，这是你在寻找的。  
  
示例 2:  人类：我想要一些独特和奇特的花。  
  AI：从你的需求中，我理解你想要的是独一无二和引人注目的花朵。兰花是一种非常独特并且颜色鲜艳的花，它们在世界上的许多地方都被视为奢侈品和美的象征。因此，我建议你考虑兰花。选择兰花可以满足你对独特和奇特的要求，而且，兰花的美丽和它们所代表的力量和奢侈也可能会吸引你。  
"""  
from langchain.prompts import ChatPromptTemplate, HumanMessagePromptTemplate, SystemMessagePromptTemplate  
system_prompt_role = SystemMessagePromptTemplate.from_template(role_template)  
system_prompt_cot = SystemMessagePromptTemplate.from_template(cot_template)  
  
# 用户的询问  
human_template = "{human_input}"  
human_prompt = HumanMessagePromptTemplate.from_template(human_template)  
  
# 将以上所有信息结合为一个聊天提示  
chat_prompt = ChatPromptTemplate.from_messages([system_prompt_role, system_prompt_cot, human_prompt])  
  
prompt = chat_prompt.format_prompt(human_input="我想为我的女朋友购买一些花。她喜欢粉色和紫色。你有什么建议吗?").to_messages()  
  
# 接收用户的询问，返回回答结果  
response = llm(prompt)  
print(response)
```
输出：
```bash
content='根据你女朋友喜欢粉色和紫色的喜好，我会推荐以下几种花给你：\n\n1. **粉色康乃馨（Carnation）**：康乃馨是一种美丽且经典的花朵，粉色的康乃馨通常象征着母爱、友谊和善良。它们的花语也包括关怀和感激之情，是一种很适合送给女朋友的花。\n\n2. **紫色勿忘我（Forget-Me-Not）**：勿忘我是一种小巧可爱的花朵，紫色的勿忘我通常象征着真爱和忠诚。这种花也代表着永恒的爱情和美好的回忆，是表达对女朋友真挚感情的不错选择。\n\n3. **粉色玫瑰（Pink Rose）**：粉色玫瑰是一种温柔和浪漫的花朵，通常代表着感恩、喜悦和甜蜜的情感。送粉色玫瑰给女朋友可以表达你对她的爱意和关怀。\n\n希望这些建议能帮助你选择合适的花束来表达对女朋友的感情。'
```

2. 思维树（Tree of Thoughts，ToT）

ToT是一种解决复杂问题的框架，它在需要多步骤推理的任务中，引导语言模型搜索一棵由连贯的语言序列（解决问题的中间步骤）组成的思维树，而不是简单地生成一个答案。ToT框架的核心思想是：让模型生成和评估其思维的能力，并将其与搜索算法（如广度优先搜索和深度优先搜索）结合起来，进行系统性地探索和验证。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240306144833.png)
ToT 框架为每个任务定义具体的思维步骤和每个步骤的候选项数量。例如，要解决一个数学推理任务，先把它分解为3个思维步骤，并为每个步骤提出多个方案，并保留最优的5个候选方案。然后在多条思维路径中搜寻最优的解决方案。
这种方法的优势在于，模型可以通过观察和评估其自身的思维过程，更好地解决问题，而不仅仅是基于输入生成输出。这对于需要深度推理的复杂任务非常有用。此外，通过引入强化学习、集束搜索等技术，可以进一步提高搜索策略的性能，并让模型在解决新问题或面临未知情况时有更好的表现。

思维树（ToT）的应用方法和示例：[https://github.com/kyegomez/tree-of-thoughts](https://github.com/kyegomez/tree-of-thoughts)

### 2.1.3 语言模型
LangChain中支持的模型有三大类。

1. 大语言模型（LLM） ，也叫Text Model，这些模型将文本字符串作为输入，并返回文本字符串作为输出。Open AI的text-davinci-003、Facebook的LLaMA、ANTHROPIC的Claude，都是典型的LLM。
2. 聊天模型（Chat Model），主要代表Open AI的ChatGPT系列模型。这些模型通常由语言模型支持，但它们的 API 更加结构化。具体来说，这些模型将聊天消息列表作为输入，并返回聊天消息。
3. 文本嵌入模型（Embedding Model），这些模型将文本作为输入并返回浮点数列表，也就是Embedding。而文本嵌入模型如OpenAI的text-embedding-ada-002，我们之前已经见过了。文本嵌入模型负责把文档存入向量数据库，和我们这里探讨的提示工程关系不大。

**预训练+微调的模式：**
- **预训练**：在大规模无标注文本数据上进行模型的训练，目标是让模型学习自然语言的基础表达、上下文信息和语义知识，为后续任务提供一个通用的、丰富的语言表示基础。
- **微调**：在预训练模型的基础上，可以根据特定的下游任务对模型进行微调。现在你经常会听到各行各业的人说： _我们的优势就是领域知识嘛！我们比不过国内外大模型，我们可以拿开源模型做垂直领域嘛！做垂类模型！_—— 啥叫垂类？指的其实就是根据领域数据微调开源模型这件事儿。
预训练+微调的大模型应用模式优势明显。首先，预训练模型能够将大量的通用语言知识迁移到各种下游任务上，作为应用人员，我们不需要自己寻找语料库，从头开始训练大模型，这减少了训练时间和数据需求；其次，微调过程可以快速地根据特定任务进行优化，简化了模型部署的难度；最后，预训练+微调的架构具有很强的可扩展性，可以方便地应用于各种自然语言处理任务，大大提高了NLP技术在实际应用中的可用性和普及程度，给我们带来了巨大的便利。

#### 2.1.3.1 用 HuggingFace 跑开源模型
- 注册并安装 HuggingFace

第一步，还是要登录 [HuggingFace](https://huggingface.co/) 网站，并拿到专属于你的Token。（如果你做了前面几节课的实战案例，那么你应该已经有这个API Token了）
第二步，用 `pip install transformers` 安装HuggingFace Library。详见 [这里](https://huggingface.co/docs/transformers/installation)。
第三步，在命令行中运行 `huggingface-cli login`，设置API Token，或者在程序中设置环境变量：
```python
# 导入HuggingFace API Token
import os
os.environ['HUGGINGFACEHUB_API_TOKEN'] = '你的HuggingFace API Token'
```



#### 2.1.3.2 LangChain 和 HuggingFace 的接口


#### 2.1.3.3 用 LangChain 调用自定义语言模型
### 2.1.4 输出解析
增加输出解析的完整示例：
```python
from langchain import PromptTemplate, OpenAI  
  
# 导入OpenAI Key  
import os  
os.environ["OPENAI_API_KEY"] = '你的OpenAI API Key'  
  
# 创建提示模板  
prompt_template = """您是一位专业的鲜花店文案撰写员。  
对于售价为 {price} 元的 {flower_name} ，您能提供一个吸引人的简短描述吗？  
{format_instructions}"""  
  
# 创建模型实例  
model = OpenAI(model_name='text-davinci-003')  
  
# 导入结构化输出解析器和ResponseSchema  
from langchain.output_parsers import StructuredOutputParser, ResponseSchema  
# 定义我们想要接收的响应模式  
response_schemas = [  
    ResponseSchema(name="description", description="鲜花的描述文案"),  
    ResponseSchema(name="reason", description="问什么要这样写这个文案")  
]  
# 创建输出解析器  
output_parser = StructuredOutputParser.from_response_schemas(response_schemas)  
  
# 获取格式指示  
format_instructions = output_parser.get_format_instructions()  
# 根据模板创建提示，同时在提示中加入输出解析器的说明  
prompt = PromptTemplate.from_template(prompt_template,   
                partial_variables={"format_instructions": format_instructions})   
  
# 数据准备  
flowers = ["玫瑰", "百合", "康乃馨"]  
prices = ["50", "30", "20"]  
  
# 创建一个空的DataFrame用于存储结果  
import pandas as pd  
df = pd.DataFrame(columns=["flower", "price", "description", "reason"]) # 先声明列名  
  
for flower, price in zip(flowers, prices):  
    # 根据提示准备模型的输入  
    input = prompt.format(flower_name=flower, price=price)  
  
    # 获取模型的输出  
    output = model(input)  
      
    # 解析模型的输出（这是一个字典结构）  
    parsed_output = output_parser.parse(output)  
  
    # 在解析后的输出中添加“flower”和“price”  
    parsed_output['flower'] = flower  
    parsed_output['price'] = price  
  
    # 将解析后的输出添加到DataFrame中  
    df.loc[len(df)] = parsed_output    
  
# 打印字典  
print(df.to_dict(orient='records'))  
  
# 保存DataFrame到CSV文件  
df.to_csv("flowers_with_descriptions.csv", index=False)
```
以上增加输出解析后，处理的 prompt 如下：
```bash
您是一位专业的鲜花店文案撰写员。
对于售价为 50 元的 玫瑰 ，您能提供一个吸引人的简短描述吗？
The output should be a markdown code snippet formatted in the following schema, including the leading and trailing "```json" and "```":

//```json
{
	"description": string  // 鲜花的描述文案
	"reason": string  // 问什么要这样写这个文案
}
//```
```
LangChain的输出解析对 prompt 增加特定处理。

## 2.2 链（Chain）
如果想开发更复杂的应用程序，那么就需要通过 “Chain” 来链接LangChain的各个组件和功能——模型之间彼此链接，或模型与其他组件链接。
**说到链的实现和使用，也简单。**
- 首先LangChain通过设计好的接口，实现一个具体的链的功能。例如，LLM链（LLMChain）能够接受用户输入，使用 PromptTemplate 对其进行格式化，然后将格式化的响应传递给 LLM。这就相当于把整个Model I/O的流程封装到链里面。
- 实现了链的具体功能之后，我们可以通过将多个链组合在一起，或者将链与其他组件组合来构建更复杂的链。
所以你看，链在内部把一系列的功能进行封装，而链的外部则又可以组合串联。 **链其实可以被视为LangChain中的一种基本功能单元。**
LangChain中提供了很多种类型的预置链，目的是使各种各样的任务实现起来更加方便、规范。

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240311151037.png)

LangChain中各种各样的链：[https://github.com/langchain-ai/langchain/tree/master/libs/langchain/langchain/chains](https://github.com/langchain-ai/langchain/tree/master/libs/langchain/langchain/chains)
### 2.2.1 Sequential Chain：顺序链
使用示例，我们的目标是这样的：
- 第一步，我们假设大模型是一个植物学家，让他给出某种特定鲜花的知识和介绍。
- 第二步，我们假设大模型是一个鲜花评论者，让他参考上面植物学家的文字输出，对鲜花进行评论。
- 第三步，我们假设大模型是易速鲜花的社交媒体运营经理，让他参考上面植物学家和鲜花评论者的文字输出，来写一篇鲜花运营文案。

首先，导入所有需要的库。
```python
# 设置OpenAI API密钥
import os
os.environ["OPENAI_API_KEY"] = '你的OpenAI API Key'

from langchain.llms import OpenAI
from langchain.chains import LLMChain
from langchain.prompts import PromptTemplate
from langchain.chains import SequentialChain
```
然后，添加第一个LLMChain，生成鲜花的知识性说明。
```python
# 这是第一个LLMChain，用于生成鲜花的介绍，输入为花的名称和种类
llm = OpenAI(temperature=.7)
template = """
你是一个植物学家。给定花的名称和类型，你需要为这种花写一个200字左右的介绍。

花名: {name}
颜色: {color}
植物学家: 这是关于上述花的介绍:"""
prompt_template = PromptTemplate(input_variables=["name", "color"], template=template)
introduction_chain = LLMChain(llm=llm, prompt=prompt_template, output_key="introduction")

```
接着，添加第二个LLMChain，根据鲜花的知识性说明生成评论。
```python
# 这是第二个LLMChain，用于根据鲜花的介绍写出鲜花的评论
llm = OpenAI(temperature=.7)
template = """
你是一位鲜花评论家。给定一种花的介绍，你需要为这种花写一篇200字左右的评论。

鲜花介绍:
{introduction}
花评人对上述花的评论:"""
prompt_template = PromptTemplate(input_variables=["introduction"], template=template)
review_chain = LLMChain(llm=llm, prompt=prompt_template, output_key="review")
```
接着，添加第三个LLMChain，根据鲜花的介绍和评论写出一篇自媒体的文案。
```python
# 这是第三个LLMChain，用于根据鲜花的介绍和评论写出一篇自媒体的文案
template = """
你是一家花店的社交媒体经理。给定一种花的介绍和评论，你需要为这种花写一篇社交媒体的帖子，300字左右。

鲜花介绍:
{introduction}
花评人对上述花的评论:
{review}

社交媒体帖子:
"""
prompt_template = PromptTemplate(input_variables=["introduction", "review"], template=template)
social_post_chain = LLMChain(llm=llm, prompt=prompt_template, output_key="social_post_text")
```
最后，添加SequentialChain，把前面三个链串起来。
```python
# 这是总的链，我们按顺序运行这三个链
overall_chain = SequentialChain(
    chains=[introduction_chain, review_chain, social_post_chain],
    input_variables=["name", "color"],
    output_variables=["introduction","review","social_post_text"],
    verbose=True)

# 运行链，并打印结果
result = overall_chain({"name":"玫瑰", "color": "黑色"})
print(result)

```
最终的输出如下：
```json
> Entering new  chain...

> Finished chain.
{'name': '玫瑰', 'color': '黑色',
'introduction': '\n\n黑色玫瑰，这是一种对传统玫瑰花的独特颠覆，它的出现挑战了我们对玫瑰颜色的固有认知。它的花瓣如煤炭般黑亮，反射出独特的微光，而花蕊则是金黄色的，宛如夜空中的一颗星，强烈的颜色对比营造出一种前所未有的视觉效果。在植物学中，黑色玫瑰的出现无疑提供了一种新的研究方向，对于我们理解花朵色彩形成的机制有着重要的科学价值。',
'review': '\n\n黑色玫瑰，这不仅仅是一种花朵，更是一种完全颠覆传统的艺术表现形式。黑色的花瓣仿佛在诉说一种不可言喻的悲伤与神秘，而黄色的蕊瓣犹如漆黑夜空中的一抹亮色，给人带来无尽的想象。它将悲伤与欢乐，神秘与明亮完美地结合在一起，这是一种全新的视觉享受，也是一种对生活理解的深度表达。',
'social_post_text': '\n欢迎来到我们的自媒体平台，今天，我们要向您展示的是我们的全新产品——黑色玫瑰。这不仅仅是一种花，这是一种对传统观念的挑战，一种视觉艺术的革新，更是一种生活态度的象征。
这种别样的玫瑰花，其黑色花瓣宛如漆黑夜空中闪烁的繁星，富有神秘的深度感，给人一种前所未有的视觉冲击力。这种黑色，它不是冷酷、不是绝望，而是充满着独特的魅力和力量。而位于黑色花瓣之中的金黄色花蕊，则犹如星星中的灵魂，默默闪烁，给人带来无尽的遐想，充满活力与生机。
黑色玫瑰的存在，不仅挑战了我们对于玫瑰传统颜色的认知，它更是一种生动的生命象征，象征着那些坚韧、独特、勇敢面对生活的人们。黑色的花瓣中透露出一种坚韧的力量，而金黄的花蕊则是生活中的希望，二者的结合恰好象征了生活中的喜怒哀乐，体现了人生的百态。'}
```
至此，就通过两个LLM链和一个顺序链，生成了一篇完美的文案。

### 2.2.2 RouterChain：路由链
RouterChain，也叫路由链，能动态选择用于给定输入的下一个链。我们会根据用户的问题内容，首先使用路由器链确定问题更适合哪个处理模板，然后将问题发送到该处理模板进行回答。如果问题不适合任何已定义的处理模板，它会被发送到默认链。
在这里，我们会用LLMRouterChain和MultiPromptChain（也是一种路由链）组合实现路由功能，该MultiPromptChain会调用LLMRouterChain选择与给定问题最相关的提示，然后使用该提示回答问题。
**具体步骤如下：**
1. 构建处理模板：为鲜花护理和鲜花装饰分别定义两个字符串模板。
2. 提示信息：使用一个列表来组织和存储这两个处理模板的关键信息，如模板的键、描述和实际内容。
3. 初始化语言模型：导入并实例化语言模型。
4. 构建目标链：根据提示信息中的每个模板构建了对应的LLMChain，并存储在一个字典中。
5. 构建LLM路由链：这是决策的核心部分。首先，它根据提示信息构建了一个路由模板，然后使用这个模板创建了一个LLMRouterChain。
6. 构建默认链：如果输入不适合任何已定义的处理模板，这个默认链会被触发。
7. 构建多提示链：使用MultiPromptChain将LLM路由链、目标链和默认链组合在一起，形成一个完整的决策系统。

#### 2.2.2.1 构建提示信息的模板
```python
# 构建两个场景的模板
flower_care_template = """你是一个经验丰富的园丁，擅长解答关于养花育花的问题。
                        下面是需要你来回答的问题:
                        {input}"""

flower_deco_template = """你是一位网红插花大师，擅长解答关于鲜花装饰的问题。
                        下面是需要你来回答的问题:
                        {input}"""
# 构建提示信息
prompt_infos = [
    {
        "key": "flower_care",
        "description": "适合回答关于鲜花护理的问题",
        "template": flower_care_template,
    },
    {
        "key": "flower_decoration",
        "description": "适合回答关于鲜花装饰的问题",
        "template": flower_deco_template,
    }]
```
循环prompt_infos这个列表，构建出两个目标链，分别负责处理不同的问题。
```python
# 构建目标链
from langchain.chains.llm import LLMChain
from langchain.prompts import PromptTemplate
chain_map = {}
for info in prompt_infos:
    prompt = PromptTemplate(template=info['template'],
                            input_variables=["input"])
    print("目标提示:\n",prompt)
    chain = LLMChain(llm=llm, prompt=prompt,verbose=True)
    chain_map[info["key"]] = chain
```
目标链提示是这样的：
```json
目标提示:
input_variables=['input']
output_parser=None partial_variables={}
template='你是一个经验丰富的园丁，擅长解答关于养花育花的问题。\n                        下面是需要你来回答的问题:\n
{input}' template_format='f-string'
validate_template=True

目标提示:
input_variables=['input']
output_parser=None partial_variables={}
template='你是一位网红插花大师，擅长解答关于鲜花装饰的问题。\n                        下面是需要你来回答的问题:\n
{input}' template_format='f-string'
validate_template=True
```
对于每个场景，创建一个 LLMChain（语言模型链）。每个链会根据其场景模板生成对应的提示，然后将这个提示送入语言模型获取答案。

#### 2.2.2.2 构建路由链
```python
# 构建路由链
from langchain.chains.router.llm_router import LLMRouterChain, RouterOutputParser
from langchain.chains.router.multi_prompt_prompt import MULTI_PROMPT_ROUTER_TEMPLATE as RounterTemplate
destinations = [f"{p['key']}: {p['description']}" for p in prompt_infos]
router_template = RounterTemplate.format(destinations="\n".join(destinations))
print("路由模板:\n",router_template)
router_prompt = PromptTemplate(
    template=router_template,
    input_variables=["input"],
    output_parser=RouterOutputParser(),)
print("路由提示:\n",router_prompt)
router_chain = LLMRouterChain.from_llm(llm,
                                       router_prompt,
                                       verbose=True)
```
输出：
````bash
路由模板:
 Given a raw text input to a language model select the model prompt best suited for the input. You will be given the names of the available prompts and a description of what the prompt is best suited for. You may also revise the original input if you think that revising it will ultimately lead to a better response from the language model.

<< FORMATTING >>
Return a markdown code snippet with a JSON object formatted to look like:
```json
{{
    "destination": string \ name of the prompt to use or "DEFAULT"
    "next_inputs": string \ a potentially modified version of the original input
}}
```

REMEMBER: "destination" MUST be one of the candidate prompt names specified below OR it can be "DEFAULT" if the input is not well suited for any of the candidate prompts.
REMEMBER: "next_inputs" can just be the original input if you don't think any modifications are needed.

<< CANDIDATE PROMPTS >>
flower_care: 适合回答关于鲜花护理的问题
flower_decoration: 适合回答关于鲜花装饰的问题

<< INPUT >>
{input}

<< OUTPUT >>

路由提示:
input_variables=['input'] output_parser=RouterOutputParser(default_destination='DEFAULT', next_inputs_type=<class 'str'>, next_inputs_inner_key='input')
partial_variables={}
template='Given a raw text input to a language model select the model prompt best suited for the input. You will be given the names of the available prompts and a description of what the prompt is best suited for. You may also revise the original input if you think that revising it will ultimately lead to a better response from the language model.\n\n
<< FORMATTING >>\n
Return a markdown code snippet with a JSON object formatted to look like:\n```json\n{{\n "destination": string \\ name of the prompt to use or "DEFAULT"\n    "next_inputs": string \\ a potentially modified version of the original input\n}}\n```\n\n
REMEMBER: "destination" MUST be one of the candidate prompt names specified below OR it can be "DEFAULT" if the input is not well suited for any of the candidate prompts.\n
REMEMBER: "next_inputs" can just be the original input if you don\'t think any modifications are needed.\n\n<< CANDIDATE PROMPTS >>\n
flower_care: 适合回答关于鲜花护理的问题\n
flower_decoration: 适合回答关于鲜花装饰的问题\n\n
<< INPUT >>\n{input}\n\n<< OUTPUT >>\n'
template_format='f-string'
validate_template=True
````
1. 路由模板的解释

路由模板是路由功能得以实现的核心。

2. 路由提示的解释

路由提示 (router_prompt）则根据路由模板，生成了具体传递给LLM的路由提示信息。
- 其中input_variables 指定模板接收的输入变量名，这里只有 `"input"`。
- output_parser 是一个用于解析模型输出的对象，它有一个默认的目的地和一个指向下一输入的键。
- template 是实际的路由模板，用于给模型提供指示。这就是刚才详细解释的模板内容。
- template_format 指定模板的格式，这里是 `"f-string"`。
- validate_template 是一个布尔值，如果为 True，则会在使用模板前验证其有效性。

这个构造允许你将用户的原始输入送入路由器，然后路由器会决定将该输入发送到哪个具体的模型提示，或者是否需要对输入进行修订以获得最佳的响应。

#### 2.2.2.3 构建默认链
除了处理目标链和路由链之外，还需要准备一个默认链。如果路由链没有找到适合的链，那么，就以默认链进行处理。
```python
# 构建默认链
from langchain.chains import ConversationChain
default_chain = ConversationChain(llm=llm,
                                  output_key="text",
                                  verbose=True)
```
#### 2.2.2.4 构建多提示链
使用MultiPromptChain类把前几个链整合在一起，实现路由功能。这个MultiPromptChain类是一个多路选择链，它使用一个LLM路由器链在多个提示之间进行选择。**MultiPromptChain中有三个关键元素。**
- router_chain（类型RouterChain）：这是用于决定目标链和其输入的链。当给定某个输入时，这个router_chain决定哪一个destination_chain应该被选中，以及传给它的具体输入是什么。
- destination_chains（类型Mapping[str, LLMChain]）：这是一个映射，将名称映射到可以将输入路由到的候选链。例如，你可能有多种处理文本输入的方法（或“链”），每种方法针对特定类型的问题。destination_chains可以是这样一个字典： `{'weather': weather_chain, 'news': news_chain}`。在这里，weather_chain可能专门处理与天气相关的问题，而news_chain处理与新闻相关的问题。
- default_chain（类型LLMChain）：当 router_chain 无法将输入映射到destination_chains中的任何一个链时，LLMChain 将使用此默认链。这是一个备选方案，确保即使路由器不能决定正确的链，也总有一个链可以处理输入。

**它的工作流程如下：**
1. 输入首先传递给router_chain。
2. router_chain根据某些标准或逻辑决定应该使用哪一个destination_chain。
3. 输入随后被路由到选定的destination_chain，该链进行处理并返回结果。
4. 如果router_chain不能决定正确的destination_chain，则输入会被传递给default_chain。

这样，MultiPromptChain就提供了一个在多个处理链之间动态路由输入的机制，以得到最相关或最优的输出。
```python
# 构建多提示链
from langchain.chains.router import MultiPromptChain
chain = MultiPromptChain(
    router_chain=router_chain,
    destination_chains=chain_map,
    default_chain=default_chain,
    verbose=True)
```

#### 2.2.2.5 运行路由链
通过下边的方式运行路由链：
```python
print(chain.run(“如何为玫瑰浇水？”))
```
输出：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240311155929.png)

**完整代码：** [https://github.com/huangjia2019/langchain/blob/main/09_%E9%93%BE%E4%B8%8B/Rounter_Chain.py](https://github.com/huangjia2019/langchain/blob/main/09_%E9%93%BE%E4%B8%8B/Rounter_Chain.py)
## 2.3 记忆（Memory）
在默认情况下，无论是LLM还是代理都是无状态的，每次模型的调用都是独立于其他交互的。但是在聊天中，为了对话的连贯性，是需要让大模型记住之前的内容，具体的实现方式有如下几种。

1. ConversationBufferMemory的 [实现细节](https://github.com/langchain-ai/langchain/blob/master/libs/langchain/langchain/memory/buffer.py)
2. ConversationSummaryMemory的 [实现细节](https://github.com/langchain-ai/langchain/blob/master/libs/langchain/langchain/memory/summary.py)

### 2.3.1 ConversationChain
这个Chain最主要的特点是，它提供了包含AI 前缀和人类前缀的对话摘要格式，这个对话格式和记忆机制结合得非常紧密。使用示例：
```python
# 设置OpenAI API密钥  
# import os  
# os.environ["OPENAI_API_KEY"] = 'Your Key'  
  
# 导入所需的库  
from langchain import OpenAI  
from langchain.chains import ConversationChain  
  
# 初始化大语言模型  
llm = OpenAI(  
    temperature=0.5,  
    model_name="text-davinci-003"  
)  
  
# 初始化对话链  
conv_chain = ConversationChain(llm=llm)  
  
# 打印对话的模板  
print(conv_chain.prompt.template)
```
输出：
```python
The following is a friendly conversation between a human and an AI. The AI is talkative and provides lots of specific details from its context. If the AI does not know the answer to a question, it truthfully says it does not know.

Current conversation:
{history}
Human: {input}
AI:
```
这里的提示为人类（我们）和人工智能（ text-davinci-003 ）之间的对话设置了一个基本对话框架：这是 **人类和** **AI** **之间的友好对话。AI** **非常健谈并从其上下文中提供了大量的具体细节。** (The following is a friendly conversation between a human and an AI. The AI is talkative and provides lots of specific details from its context. )
同时，这个提示试图通过说明以下内容来减少幻觉，也就是尽量减少模型编造的信息：
**“如果** **AI** **不知道问题的答案，它就会如实说它不知道。”**（If the AI does not know the answer to a question, it truthfully says it does not know.）之后，我们看到两个参数 {history} 和 {input}。
- **{history}** 是存储会话记忆的地方，也就是人类和人工智能之间对话历史的信息。
- **{input}** 是新输入的地方，你可以把它看成是和ChatGPT对话时，文本框中的输入。

>当有了 {history} 参数，以及 Human 和 AI 这两个前缀，我们就能够把历史对话信息存储在提示模板中，并作为新的提示内容在新一轮的对话过程中传递给模型。—— 这就是记忆机制的原理。
### 2.3.2 ConversationBufferMemory
在LangChain中，通过ConversationBufferMemory（ **缓冲记忆**）可以实现最简单的记忆机制。示例如下：
```python
# 设置OpenAI API密钥  
# import os  
# os.environ["OPENAI_API_KEY"] = 'Your Key'  
  
# 导入所需的库  
from langchain import OpenAI  
from langchain.chains import ConversationChain  
from langchain.chains.conversation.memory import ConversationBufferMemory  
  
# 初始化大语言模型  
llm = OpenAI(  
    temperature=0.5,  
)  
  
# 初始化对话链  
conversation = ConversationChain(  
    llm=llm,  
    memory=ConversationBufferMemory()  
)  
  
# 第一天的对话  
# 回合1  
conversation("我姐姐明天要过生日，我需要一束生日花束。")  
print("第一次对话后的记忆:", conversation.memory.buffer)  
  
# 回合2  
conversation("她喜欢粉色玫瑰，颜色是粉色的。")  
print("第二次对话后的记忆:", conversation.memory.buffer)  
  
# 回合3 （第二天的对话）  
conversation("我又来了，还记得我昨天为什么要来买花吗？")  
print("/n第三次对话后时提示:/n",conversation.prompt.template)  
print("/n第三次对话后的记忆:/n", conversation.memory.buffer)
```
在第三回合中模型的输出：
```bash
Human: 我姐姐明天要过生日，我需要一束生日花束。
AI:  哦，你姐姐明天要过生日，那太棒了！我可以帮你推荐一些生日花束，你想要什么样的？我知道有很多种，比如玫瑰、康乃馨、郁金香等等。
Human: 她喜欢粉色玫瑰，颜色是粉色的。
AI:  好的，那我可以推荐一束粉色玫瑰的生日花束给你，你想要多少朵？
Human: 我又来了，还记得我昨天为什么要来买花吗？
AI:  是的，我记得你昨天来买花是因为你姐姐明天要过生日，你想要买一束粉色玫瑰的生日花束给她。
```
实际上，这些聊天历史信息，都被传入了ConversationChain的提示模板中的 {history} 参数，构建出了包含聊天记录的新的提示输入。这样简单的存储之前的对话内容，新输入中也包含了更多的Token（所有的聊天历史记录），这意味着响应时间变慢和更高的成本。而且，当达到LLM的令牌数（上下文窗口）限制时，太长的对话无法被记住（对于text-davinci-003和gpt-3.5-turbo，每次的最大输入限制是4096个Token）。

### 2.3.3 ConversationBufferWindowMemory
ConversationBufferWindowMemory 是 **缓冲窗口记忆**，它的思路就是只保存最新最近的几次人类和AI的互动。因此，它在之前的“缓冲记忆”基础上增加了一个窗口值 k。这意味着我们只保留一定数量的过去互动，然后“忘记”之前的互动。使用示例：
```python
# 设置OpenAI API密钥  
import os  
os.environ["OPENAI_API_KEY"] = 'Your Key'  
  
# 导入所需的库  
from langchain import OpenAI  
from langchain.chains import ConversationChain  
from langchain.chains.conversation.memory import ConversationBufferWindowMemory  
  
# 初始化大语言模型  
llm = OpenAI(  
    temperature=0.5,  
)  
  
# 初始化对话链  
conversation = ConversationChain(  
    llm=llm,  
    memory=ConversationBufferWindowMemory(k=1)  
)  
  
# 第一天的对话  
# 回合1  
result = conversation("我姐姐明天要过生日，我需要一束生日花束。")  
print(result)  
# 回合2  
result = conversation("她喜欢粉色玫瑰，颜色是粉色的。")  
# print("\n第二次对话后的记忆:\n", conversation.memory.buffer)  
print(result)  
  
# 第二天的对话  
# 回合3  
result = conversation("我又来了，还记得我昨天为什么要来买花吗？")  
print(result)
```
第二回合输出：
```json
{'input': '她喜欢粉色玫瑰，颜色是粉色的。', 'history': 'Human: 我姐姐明天要过生日，我需要一束生日花束。\nAI:  好的，让我来帮助你选择一束生日花束吧！根据我所知，生日花束通常会选择一些鲜艳的颜色，比如红色、粉色或橙色的花朵。也可以根据你姐姐的喜好来选择，比如她喜欢什么样的花或颜色。另外，你可以选择一束混合花束，里面包含多种不同的花朵，这样会更加丰富多彩。你还有其他要求吗？', 'response': ' 好的，我会为你选择一束粉色玫瑰的花束。你姐姐一定会喜欢的！另外，你还可以选择一些附加的小礼物，比如巧克力或贺卡，来让这份礼物更加特别。我可以帮你一起挑选，如果你需要的话。'}
```
第三回合的输出：
```json
{'input': '我又来了，还记得我昨天为什么要来买花吗？', 'history': 'Human: 她喜欢粉色玫瑰，颜色是粉色的。\nAI:  好的，我会为你选择一束粉色玫瑰的花束。你姐姐一定会喜欢的！另外，你还可以选择一些附加的小礼物，比如巧克力或贺卡，来让这份礼物更加特别。我可以帮你一起挑选，如果你需要的话。', 'response': ' 是的，你昨天来买花是为了给你的母亲庆祝她的生日。你选择了一束红色康乃馨和一张生日贺卡。我希望你的母亲喜欢这份礼物。'}
```
在第三个回合，当我们询问“还记得我昨天为什么要来买花吗？”，由于我们只保留了最近的互动（k=1），模型已经忘记了正确的答案，回答也是错误的了。

### 2.3.4  ConversationSummaryMemory
ConversationSummaryMemory（ **对话总结记忆**）的思路就是将对话历史进行汇总，然后再传递给 {history} 参数。这种方法旨在通过对之前的对话进行汇总来避免过度使用 Token。
ConversationSummaryMemory有这么几个核心特点。
1. 汇总对话：此方法不是保存整个对话历史，而是每次新的互动发生时对其进行汇总，然后将其添加到之前所有互动的“运行汇总”中。
2. 使用LLM进行汇总：该汇总功能由另一个LLM驱动，这意味着对话的汇总实际上是由AI自己进行的。
3. 适合长对话：对于长对话，此方法的优势尤为明显。虽然最初使用的 Token 数量较多，但随着对话的进展，汇总方法的增长速度会减慢。与此同时，常规的缓冲内存模型会继续线性增长。

**代码示例：**
```python
# 设置OpenAI API密钥  
# import os  
# os.environ["OPENAI_API_KEY"] = 'Your Key'  
  
# 导入所需的库  
from langchain import OpenAI  
from langchain.chains import ConversationChain  
from langchain.chains.conversation.memory import ConversationSummaryMemory  
  
# 初始化大语言模型  
llm = OpenAI(  
    temperature=0.5,  
    # model_name="text-davinci-003"  
)  
  
# 初始化对话链  
conversation = ConversationChain(  
    llm=llm,  
    memory=ConversationSummaryMemory(llm=llm)  
)  
  
# 第一天的对话  
# 回合1  
result = conversation("我姐姐明天要过生日，我需要一束生日花束。")  
print(result)  
# 回合2  
result = conversation("她喜欢粉色玫瑰，颜色是粉色的。")  
# print("\n第二次对话后的记忆:\n", conversation.memory.buffer)  
print(result)  
  
# 第二天的对话  
# 回合3  
result = conversation("我又来了，还记得我昨天为什么要来买花吗？")  
print(result)
```
第二回合的输出：
```json
{'input': '她喜欢粉色玫瑰，颜色是粉色的。', 'history': '\nThe human asks the AI for help in choosing a birthday bouquet for their sister. The AI suggests a bouquet of pink roses and white daisies, symbolizing gentleness and purity, and recommends adding some green leaves for decoration. The AI also asks for the desired number of flowers to help select the appropriate size.', 'response': ' 那么我建议您选择一束粉红色的玫瑰和白色的雏菊，这象征着温柔和纯洁。您也可以在花束中加入一些绿叶作为装饰。请问您想要多少朵花呢？这样我可以帮您选择合适的大小。'}
```
第三回合的输出：
```json
{'input': '我又来了，还记得我昨天为什么要来买花吗？', 'history': '\nThe human asks the AI for help in choosing a birthday bouquet for their sister. The AI suggests a bouquet of pink roses and white daisies, symbolizing gentleness and purity, and recommends adding green leaves for decoration. The AI also asks for the desired number of flowers to help select the appropriate size. The human clarifies that their sister prefers pink roses and the AI suggests adding them to the bouquet. The AI also asks for the desired number of flowers to ensure the right size.', 'response': ' 当然记得！昨天您说您的妹妹生日快到了，想要给她一个特别的生日花束。您想要为她挑选什么样的花束呢？我可以帮您选择最合适的花束。'}
```
看得出来，这里的 `'history'`，不再是之前人类和AI对话的简单复制粘贴，而是经过了总结和整理之后的一个综述信息。
ConversationSummaryMemory的优点是对于长对话，可以减少使用的 Token 数量，因此可以记录更多轮的对话信息，使用起来也直观易懂。不过，它的缺点是，对于较短的对话，可能会导致更高的 Token 使用。另外，对话历史的记忆完全依赖于中间汇总LLM的能力，还需要为汇总LLM使用 Token，这增加了成本，且并不限制对话长度。

### 2.3.5 ConversationSummaryBufferMemory
ConversationSummaryBufferMemory，即 **对话总结缓冲记忆**，它是一种 **混合记忆** 模型，结合了上述各种记忆机制，包括ConversationSummaryMemory 和 ConversationBufferWindowMemory的特点。这种模型旨在在对话中总结早期的互动，同时尽量保留最近互动中的原始内容。
是通过max_token_limit这个参数做到这一点的。当最新的对话文字长度在300字之内的时候，LangChain会记忆原始对话内容；当对话文字超出了这个参数的长度，那么模型就会把所有超过预设长度的内容进行总结，以节省Token数量。
```python
# 设置OpenAI API密钥  
# import os  
# os.environ["OPENAI_API_KEY"] = 'Your Key'  
  
# 导入所需的库  
from langchain import OpenAI  
from langchain.chains import ConversationChain  
from langchain.chains.conversation.memory import ConversationSummaryBufferMemory  
  
# 初始化大语言模型  
llm = OpenAI(  
    temperature=0.5,  
    # model_name="gpt-4"  
)  
  
# 初始化对话链  
conversation = ConversationChain(  
    llm=llm,  
    memory=ConversationSummaryBufferMemory(  
        llm=llm,  
        max_token_limit=300  
    )  
)  
  
# 第一天的对话  
# 回合1  
result = conversation("我姐姐明天要过生日，我需要一束生日花束。")  
print(result)  
# 回合2  
result = conversation("她喜欢粉色玫瑰，颜色是粉色的。")  
# print("\n第二次对话后的记忆:\n", conversation.memory.buffer)  
print(result)  
  
# 第二天的对话  
# 回合3  
result = conversation("我又来了，还记得我昨天为什么要来买花吗？")  
print(result)
```
第三回合输出：
```json
{'input': '我又来了，还记得我昨天为什么要来买花吗？',
'history': "System: \nThe human asked the AI for advice on buying a bouquet for their sister's birthday. The AI suggested buying a vibrant bouquet as a representation of their wishes and blessings, and recommended looking for special bouquets like colorful roses or lilies for something more unique.\nHuman: 她喜欢粉色玫瑰，颜色是粉色的。\nAI:  好的，那粉色玫瑰就是一个很好的选择！你可以买一束粉色玫瑰花束，这样你姐姐会很开心的！你可以在花店里找到粉色玫瑰，也可以从网上订购，你可以根据你的预算，选择合适的数量。另外，你可以考虑添加一些装饰，比如细绳、彩带或者小礼品",
'response': ' 是的，我记得你昨天来买花是为了给你姐姐的生日。你想买一束粉色玫瑰花束来表达你的祝福和祝愿，你可以在花店里找到粉色玫瑰，也可以从网上订购，你可以根据你的预算，选择合适的数量。另外，你可以考虑添加一些装饰，比如细绳、彩带或者小礼品}
```
在第三回合，它察觉出前两轮的对话已经超出了300个字节，就把早期的对话加以总结，以节省Token资源。

### 2.3.6 四种记忆机制的比较
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240311150350.png)

在经过 K 轮对话后，对Token的消耗对比：ConversationSummaryBufferMemory和ConversationSummaryMemory，在对话轮次较少的时候可能会浪费一些Token，但是多轮对话过后，Token的节省就逐渐体现出来了。ConversationBufferWindowMemory对于Token的节省最为直接，但是它会完全遗忘掉K轮之前的对话内容，因此对于某些场景也不是最佳选择。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240311150409.png)

## 2.4 代理（Agents）
### 2.4.1 代理的作用
如果需要模型做自主判断、自行调用工具、自行决定下一步行动的时候，可以使用Agent（也就是代理）。代理就像一个多功能的接口，它能够接触并使用一套工具。根据用户的输入，代理会决定调用哪些工具。它不仅可以同时使用多种工具，而且可以将一个工具的输出数据作为另一个工具的输入数据。
在LangChain中使用代理，需要理解下面三个元素。
- **大模型**：提供逻辑的引擎，负责生成预测和处理输入。
- 与之交互的 **外部工具**：可能包括数据清洗工具、搜索引擎、应用程序等。
- 控制交互的 **代理**：调用适当的外部工具，并管理整个交互过程的流程。

![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240312090544.png)

这个过程有很多地方需要大模型自主判断下一步行为（也就是操作）要做什么，如果不加引导，那大模型本身是不具备这个能力的。比如下面这一系列的操作：

- 什么时候开始在本地知识库中搜索（这个比较简单，毕竟是第一个步骤，可以预设）？
- 怎么确定本地知识库的检索已经完成，可以开始下一步？
- 调用哪一种外部搜索工具（比如Google引擎）？
- 如何确定外部搜索工具返回了想要找的内容？
- 如何确定信息真实性的检索已经全部完成，可以开始下一步？

### 2.4.2 ReAct 框架
ReAct 框架的灵感来自“行动”和“推理”之间的协同作用，这种协同作用使得咱们人类能够学习新任务并做出决策或推理。如每天早上想知道如何为鲜花定价？我会去Google上面查一查今天的鲜花成本价啊（ **行动**），也就是我预计的进货的价格，然后我会根据这个价格的高低（ **观察**），来确定我要加价多少（ **思考**），最后计算出一个售价（ **行动**）！
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240312091016.png)

这个例子中有观察、有思考，然后才会具体行动。这里的观察和思考，我们统称为推理（Reasoning）过程，推理指导着你的行动（Acting）。
ReAct 指如何指导大语言模型推理和行动的一种思维框架。这个思维框架是Shunyu Yao等人在ICLR 2023会议论文《 [ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/pdf/2210.03629.pdf)》（ReAct：在语言模型中协同推理和行动）中提出的。

### 2.4.3 通过代理实现ReAct框架
下边使用LangChain中最为常用的 **ZERO_SHOT_REACT_DESCRIPTION** ——这种常用代理类型，看下LLM是如何在ReAct框架的指导下进行推理的。
要给代理一个任务，这个任务是找到玫瑰的当前市场价格，然后计算出加价15%后的新价格。

在开始之前，有一个准备工作，需要在 [serpapi.com](https://serpapi.com/) 注册一个账号，并且拿到你的 SERPAPI_API_KEY，这个就是我们要为大模型提供的 Google 搜索工具。安装 SerpAPI 的包：
```bash
pip install google-search-results
```

**代码示例：**
```python
# 设置OpenAI和SERPAPI的API密钥  
import os  
os.environ["OPENAI_API_KEY"] = 'Your OpenAI API Key'  
os.environ["SERPAPI_API_KEY"] = 'Your SerpAPI API Key'  
  
# 加载所需的库  
from langchain.agents import load_tools  
from langchain.agents import initialize_agent  
from langchain.agents import AgentType  
from langchain.llms import OpenAI  
  
# 初始化大模型  
llm = OpenAI(temperature=0)  
  
# 设置工具  
tools = load_tools(["serpapi", "llm-math"], llm=llm)  
  
# 初始化Agent  
agent = initialize_agent(tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True)  
  
# 跑起来  
agent.run("目前市场上玫瑰花的平均价格是多少？如果我在此基础上加价15%卖出，应该如何定价？")
```
**输出：**
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240312092314.png)

ZERO_SHOT_REACT_DESCRIPTION类型的智能代理在LangChain中，自动形成了一个完善的思考与行动链条，而且给出了正确的答案。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240312092332.png)

通过ReAct框架，大模型将被引导生成一个任务解决轨迹，即观察环境-进行思考-采取行动。观察和思考阶段被统称为推理（Reasoning），而实施下一步行动的阶段被称为行动（Acting）。在每一步推理过程中，都会详细记录下来，这也改善了大模型解决问题时的可解释性和可信度。
- 在推理阶段，模型对当前环境和状态进行观察，并生成推理轨迹，从而使模型能够诱导、跟踪和更新操作计划，甚至处理异常情况。
- 在行动阶段，模型会采取下一步的行动，如与外部源（如知识库或环境）进行交互并收集信息，或给出最终答案。
### 2.4.4 AgentExecutor 驱动模型和工具完成任务
在链中，一系列操作被硬编码（在代码中）。在代理中，语言模型被用作推理引擎来确定要采取哪些操作以及按什么顺序执行这些操作。
下面这个图，就展现出了Agent接到任务之后，自动进行推理，然后自主调用工具完成任务的过程。
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240312092610.png)

AgentExecutor中最重要的方法是步骤处理方法，`_take_next_step`方法。它用于在思考-行动-观察的循环中采取单步行动。先调用代理的计划，查找代理选择的工具，然后使用选定的工具执行该计划（此时把输入传给工具），从而获得观察结果，然后继续思考，直到输出是 AgentFinish 类型，循环才会结束。

### 2.4.5 Agent 的关键组件
在LangChain的代理中，有这样几个关键组件。
1. **代理**（Agent）：这个类决定下一步执行什么操作。它由一个语言模型和一个提示（prompt）驱动。提示可能包含代理的性格（也就是给它分配角色，让它以特定方式进行响应）、任务的背景（用于给它提供更多任务类型的上下文）以及用于激发更好推理能力的提示策略（例如ReAct）。LangChain中包含很多种不同类型的代理。
2. **工具**（Tools）：工具是代理调用的函数。这里有两个重要的考虑因素：一是让代理能访问到正确的工具，二是以最有帮助的方式描述这些工具。如果你没有给代理提供正确的工具，它将无法完成任务。如果你没有正确地描述工具，代理将不知道如何使用它们。LangChain提供了一系列的工具，同时你也可以定义自己的工具。
3. **工具包**（Toolkits）：工具包是一组用于完成特定目标的彼此相关的工具，每个工具包中包含多个工具。比如LangChain的Office365工具包中就包含连接Outlook、读取邮件列表、发送邮件等一系列工具。当然LangChain中还有很多其他工具包供你使用。
4. **代理执行器**（AgentExecutor）：代理执行器是代理的运行环境，它调用代理并执行代理选择的操作。执行器也负责处理多种复杂情况，包括处理代理选择了不存在的工具的情况、处理工具出错的情况、处理代理产生的无法解析成工具调用的输出的情况，以及在代理决策和工具调用进行观察和日志记录。

总的来说，代理就是一种用语言模型做出决策、调用工具来执行具体操作的系统。通过设定代理的性格、背景以及工具的描述，你可以定制代理的行为，使其能够根据输入的文本做出理解和推理，从而实现自动化的任务处理。而代理执行器（AgentExecutor）就是上述机制得以实现的引擎。


### 2.4.6 其他 Agent 类型
#### 2.4.6.1 结构化工具
通过指定AgentType.STRUCTURED_CHAT_ZERO_SHOT_REACT_DESCRIPTION 这个代理类型，代理能够调用包含一系列复杂工具的“ **结构化工具箱**”，组合调用其中的多个工具，完成批次相关的任务集合。
结构化工具的示例包括：
1. 文件管理工具集：支持所有文件系统操作，如写入、搜索、移动、复制、列目录和查找。
2. Web 浏览器工具集：官方的 PlayWright 浏览器工具包，允许代理访问网站、点击、提交表单和查询数据。

下边以 PlayWright 工具包为例，来实现一个结构化工具对话代理。
Playwright是一个开源的自动化框架，它可以让你模拟真实用户操作网页，帮助开发者和测试者自动化网页交互和测试。用简单的话说，它就像一个“机器人”，可以按照你给的指令去浏览网页、点击按钮、填写表单、读取页面内容等等，就像一个真实的用户在使用浏览器一样。
Playwright支持多种浏览器，比如Chrome、Firefox、Safari等，这意味着可以用它来测试你的网站或测试应用在不同的浏览器上的表现是否一致。
先用 `pip install playwright` 安装Playwright工具。然后还需要通过 `playwright install` 命令来安装三种常用的浏览器工具。
通过Playwright浏览器工具来访问一个测试网页：
```python
from playwright.sync_api import sync_playwright

def run():
    # 使用Playwright上下文管理器
    with sync_playwright() as p:
        # 使用Chromium，但你也可以选择firefox或webkit
        browser = p.chromium.launch()

        # 创建一个新的页面
        page = browser.new_page()

        # 导航到指定的URL
        page.goto('https://langchain.com/')

        # 获取并打印页面标题
        title = page.title()
        print(f"Page title is: {title}")

        # 关闭浏览器
        browser.close()

if __name__ == "__main__":
    run()
```
输出：
```bash
Page title is: LangChain
```

**使用结构化工具对话代理：**
使用的Agent类型是STRUCTURED_CHAT_ZERO_SHOT_REACT_DESCRIPTION。要使用的工具则是PlayWrightBrowserToolkit，这是LangChain中基于PlayWrightBrowser包封装的工具箱，它继承自 BaseToolkit类。
PlayWrightBrowserToolkit 为 PlayWright 浏览器提供了一系列交互的工具，可以在同步或异步模式下操作。
其中具体的工具就包括：
![](https://raw.githubusercontent.com/BaihlUp/Figurebed/master/2024/20240313102748.png)

#### 2.4.6.2 Self-Ask with Search 代理
Self-Ask with Search 也是LangChain中的一个有用的代理类型（SELF_ASK_WITH_SEARCH）。它利用一种叫做 “Follow-up Question（追问）”加“Intermediate Answer（中间答案）”的技巧，来辅助大模型寻找事实性问题的过渡性答案，从而引出最终答案。
如下使用SerpAPIWrapper作为工具，用OpenAI作为语言模型，创建Self-Ask with Search代理。
```python

```


#### 2.4.6.3 Plan and execute 代理