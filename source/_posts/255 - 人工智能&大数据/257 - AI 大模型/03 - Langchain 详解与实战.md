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




