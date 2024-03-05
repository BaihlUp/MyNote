---
title: Langchain 详解与实战
date: 2024-02-07
categories:
  - 人工智能&大数据
tags:
  - LangChain
published: false
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

# 1.4 

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

[提示工程课程](https://learn.deeplearning.ai/login?redirect_course=chatgpt-prompt-eng)

```python
# 导入LangChain中的提示模板
from langchain import PromptTemplate
# 创建原始模板
template = """您是一位专业的鲜花店文案撰写员。\n
对于售价为 {price} 元的 {flower_name} ，您能提供一个吸引人的简短描述吗？
"""
# 根据原始模板创建LangChain提示模板
prompt = PromptTemplate.from_template(template)
# 打印LangChain提示模板的内容
print(prompt)
```
提示模板的具体内容如下：
```bash
input_variables=['flower_name', 'price']
output_parser=None partial_variables={}
template='/\n您是一位专业的鲜花店文案撰写员。
\n对于售价为 {price} 元的 {flower_name} ，您能提供一个吸引人的简短描述吗？\n'
template_format='f-string'
validate_template=True
```
LangChain 提供了多个类和函数，也 **为各种应用场景设计了很多内置模板，使构建和使用提示变得容易**。

### 2.1.3 语言模型
LangChain中支持的模型有三大类。

1. 大语言模型（LLM） ，也叫Text Model，这些模型将文本字符串作为输入，并返回文本字符串作为输出。Open AI的text-davinci-003、Facebook的LLaMA、ANTHROPIC的Claude，都是典型的LLM。
2. 聊天模型（Chat Model），主要代表Open AI的ChatGPT系列模型。这些模型通常由语言模型支持，但它们的 API 更加结构化。具体来说，这些模型将聊天消息列表作为输入，并返回聊天消息。
3. 文本嵌入模型（Embedding Model），这些模型将文本作为输入并返回浮点数列表，也就是Embedding。而文本嵌入模型如OpenAI的text-embedding-ada-002，我们之前已经见过了。文本嵌入模型负责把文档存入向量数据库，和我们这里探讨的提示工程关系不大。

基于 OpenAI 的模型进行问答：
```python
from langchain import PromptTemplate  
# 创建原始模板  
template = """您是一位专业的鲜花店文案撰写员。\n  
对于售价为 {price} 元的 {flower_name} ，您能提供一个吸引人的简短描述吗？  
"""  
# 根据原始模板创建LangChain提示模板  
prompt = PromptTemplate.from_template(template)   
# 打印LangChain提示模板的内容  
print(prompt)  
  
# 导入LangChain中的OpenAI模型接口  
from langchain import OpenAI  
# 创建模型实例  
model = OpenAI(model_name='gpt-3.5-turbo-instruct')  
  
# 多种花的列表  
flowers = ["玫瑰", "百合", "康乃馨"]  
prices = ["50", "30", "20"]  
  
# 生成多种花的文案  
for flower, price in zip(flowers, prices):  
    # 使用提示模板生成输入  
    input_prompt = prompt.format(flower_name=flower, price=price)  
  
    # 得到模型的输出  
    output = model(input_prompt)  
  
    # 打印输出内容  
    print(output)
```

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

## 2.2 


