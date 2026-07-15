# 学习笔记（核心提炼）

LangChain 是目前大模型应用开发的主流开源框架。

## 开门见山：如何选择Langchain 和 Langgraph？

LangChain（Chains / LCEL）适合“单向直达”的线性任务
它就像一条单向的工厂流水线（A -> B -> C）。
比如：用户提问 -> 检索知识库 -> 拼接提示词 -> 大模型生成答案。
这种任务步骤固定、不需要回头，用 LangChain 的链式结构写起来最快、最简单。

LangGraph 适合“带循环、条件分支”的图式任务（尤其是复杂的 Agent）
它就像一个复杂的交通路网，允许状态流转、条件判断和无限循环。
比如：让大模型写一段代码 -> 运行测试 -> 如果报错，就把错误信息发回给大模型让它重写（形成循环） -> 直到测试通过再输出。
这种需要“反复横跳”和“记忆全局状态”的任务，LangChain 的单向链条很难搞定，必须用 LangGraph 来画状态图。

用传统的 Workflow（比如 Zapier、Airflow、甚至企业里的审批流引擎）不就行了吗？为什么非要用大模型专属的框架？

最核心的区别可以用一句话概括：传统的 Workflow 是“死”的（基于死规则），而 LangChain/LangGraph 的 Workflow 是“活”的（自带一个会思考的大脑）。
具体来说，有以下三大核心区别，这也是必须用它们的理由：

传统 Workflow 只能处理“死数据”，它能处理“活语言”
- 传统 Workflow：只能处理高度结构化的数据。比如：如果 订单金额 > 1000，就发邮件给经理。它看不懂一封充满抱怨的客户邮件。
- LangChain/LangGraph：专门用来处理非结构化数据（自然语言、文档、甚至图片）。你可以设定：读取客户邮件 -> 大模型判断客户情绪 -> 如果是“愤怒”，则生成安抚话术并走加急通道。
    大模型充当了 Workflow 中的“理解和判断节点”。

传统 Workflow 路线是写死的，它能“动态决定路线”
- 传统 Workflow：必须穷举所有的 If-Else 分支。如果遇到程序员没写过的情况，流程直接崩溃。
- LangGraph (Agent)：你不需要写死路线，你只需要给它一堆工具（比如：计算器、网页搜索、数据库查询）。当用户问“苹果公司昨天的股价和今天的天气”时，大模型会自己决定：“我先调用天气工具，再调用搜索工具，最后把结果拼起来给你。”——这种自主规划路线的能力，传统工作流根本做不到。

传统 Workflow 报错就停，它能“自我纠错”
- 传统 Workflow：执行到第 3 步，API 报错了，流程直接亮红灯，停机等待人工处理。
- LangGraph：它具备反思（Reflection）和纠错能力。比如让它写一段代码并运行，如果运行报错了，LangGraph 可以把“错误代码”传回给大模型，大模型会说：“哦，少了一个括号，我改一下再试。” 它可以在循环里不断试错，直到任务成功。

---

通俗比喻：
- 传统 Workflow 就像工厂的自动化传送带：效率极高，但只能按固定轨道走，一旦传送带上掉下来一个它没见过的零件，机器就卡死了。
- LangChain/LangGraph 就像你雇佣了一个有经验的数字员工：你不仅给了他一套标准SOP（链），还给了他一个大脑（大模型）。遇到没见过的问题，他会自己查资料、自己判断、自己纠正错误，最终把结果交给你。
- 
所以，只要你的业务流程中需要**“理解人类语言”、“模糊判断”或“自主试错”**，
用 LangChain/LangGraph 这样的 AI 框架，传统 Workflow 引擎是完全无能为力的。

## 对于langcain的一些特点：

1. 模块化设计（核心优势）
主打“搭积木”式开发。把模型调用、提示词、记忆管理等封装成了独立组件。不用重复造轮子，相比传统框架，开发效率能提升约 40%。
2. 开发者友好（上手快）
基于 Python，API 设计极简，官方文档和社区 Demo 丰富。据统计，上手速度快 ，代码维护成本降低。
3. 场景与数据接入（企业级刚需）
不仅支持无缝切换各种大模型（OpenAI、本地模型等），最强大的是内置了数据连接器。支持直接接入 20 多种企业私有数据源（数据库、本地文档等），解决数据隐私问题。
4. 顺应技术趋势（多模态与可控性）
不仅能处理文本，现在也能处理图像输入。内置了调试和监控工具，解决了 AI 应用经常出现的“黑盒”问题，让执行流程更透明、可控。

5. 生态与维护（行业标准）

社区极其庞大（GitHub 超 10 万 Star），有专业团队定期维护更新。2025 年企业采用率高达 68%，已经是大模型开发的**“事实标准”**，不用担心框架烂尾。

6. 环境准备简单（实操第一步）

要求：Python 3.8 及以上版本。
安装：直接通过 pip 或 conda 安装即可。

## LangChain 的最简结构其实就是一条数据处理的流水线。

为了让您一目了然，我们先看它的五大核心模块。

---

最直观的代码例子：搭建一条基础的“链 (Chain)”
在最新的 LangChain 中，最核心的结构体现为 LCEL (LangChain 表达式语言)。它使用管道符 | 把各个组件像拼图一样串起来。
假设我们要实现一个简单的功能：用户输入一个中文词，大模型把它翻译成英文，并严格以 JSON 格式输出。
以下是完整的结构代码示例：

```python
# 1. 引入需要的组件 (对应 Model I/O 模块)
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import JsonOutputParser

# 2. 准备组件 A：提示词模板 (Prompt)
# 告诉大模型它要做什么，并预留 {word} 作为变量
prompt = ChatPromptTemplate.from_template("请将以下中文翻译成英文，并以JSON格式输出，键名为'translation'。词语：{word}")

# 3. 准备组件 B：大语言模型 (LLM)
# 这里以 OpenAI 为例，实际也可以换成通义千问、文心一言等
model = ChatOpenAI(model="gpt-3.5-turbo")

# 4. 准备组件 C：输出解析器 (Output Parser)
# 强制把大模型输出的纯文本，转换成 Python 的字典(JSON)对象
parser = JsonOutputParser()

# 5. 核心结构：组装流水线 (Chain)
# 使用 | 符号，将 A -> B -> C 串联起来
chain = prompt | model | parser

# 6. 运行流水线
# 传入变量，启动流水线
result = chain.invoke({"word": "苹果"})

print(result)
# 输出结果: {'translation': 'Apple'}
```

## 结构解析（大白话版）

在这个例子中，您能清晰地看到 LangChain 的整体流转结构：
1. 数据流入：用户输入 {"word": "苹果"}。
2. 第一站 (Prompt)：把“苹果”填入模板，变成一段完整的指令：“请将以下中文翻译成英文...词语：苹果”。
3. 第二站 (Model)：大模型接收指令，进行思考，输出一段字符串 {"translation": "Apple"}。
4. 第三站 (Parser)：解析器把字符串清洗干净，变成程序可以直接使用的 JSON 数据。

5. 数据流出：最终返回给您干净的结果。

## 加入知识库和Memory的Chain

这就是 LangChain 最基础、最核心的结构！
如果您要加入本地知识库 (Retrieval)，只需要在这条流水线的最前面加一个“搜索文档”的节点；
如果要加入记忆 (Memory)，就在流水线旁边挂一个保存历史记录的组件。万变不离其宗，都是组件的拼接。

在最新的 LangChain 架构中，加入这两个功能依然是“拼图”的思想，
只是我们引入了 RunnablePassthrough（用于传递检索到的知识）和 RunnableWithMessageHistory（用于外挂记忆）。以下是最直观的代码例子：

```python
# 1. 引入需要的组件
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.output_parsers import JsonOutputParser
from langchain_core.runnables import RunnablePassthrough
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_community.chat_message_histories import ChatMessageHistory

# ==========================================
# 准备阶段：准备各个组件拼图
# ==========================================

# 2. 准备组件 A：本地知识库 (Retrieval)
# 模拟一段本地私有数据，并将其转化为向量数据库，生成一个“检索器”
texts = [
    "荣耀Magic6搭载了第三代骁龙8旗舰芯片。", 
    "荣耀公司的愿景是创造属于每个人的智慧新世界。"
]
vectorstore = FAISS.from_texts(texts, embedding=OpenAIEmbeddings())
retriever = vectorstore.as_retriever()

# 检索器：负责根据问题去数据库里找答案
# 3. 准备组件 B：记忆组件 (Memory)
# 创建一个字典，用于存储不同用户的历史聊天记录
store = {}
def get_session_history(session_id: str):
    if session_id not in store:
        store[session_id] = ChatMessageHistory()
    return store[session_id]

# 4. 准备组件 C：提示词模板 (Prompt)
# 模板中预留了三个变量：
# {context}(检索到的知识)、{chat_history}(历史记忆)、{question}(当前问题)
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个AI客服。请根据以下【参考资料】回答问题。必须以JSON格式输出，包含'answer'(回答内容)和'emotion'(当前情绪)两个键。\n\n【参考资料】：{context}"),
    MessagesPlaceholder(variable_name="chat_history"), # 动态插入历史记忆
    ("human", "{question}")
])

# 5. 准备组件 D：大语言模型 (LLM) 和 解析器 (Parser)
model = ChatOpenAI(model="gpt-3.5-turbo")
parser = JsonOutputParser()

# 6. 核心结构：组装基础流水线 (加入知识检索)
# 格式化检索到的文档，变成纯文本
def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)

# ==========================================
# 组装阶段：拼接流水线
# ==========================================

# 第一步：并行处理数据。根据 question 去检索 context，同时把 question 原封不动传下去
setup_and_retrieval = {
    "context": retriever | format_docs, 
    "question": RunnablePassthrough()
}

# 将检索、提示词、模型、解析器串联
base_chain = setup_and_retrieval | prompt | model | parser

# 7. 终极组装：给流水线外挂“记忆” (Memory)
# 用 RunnableWithMessageHistory 把基础流水线包裹起来，赋予它记忆能力
chain_with_memory = RunnableWithMessageHistory(
    base_chain,
    get_session_history, # 获取记忆的函数
    input_messages_key="question", # 告诉系统哪个变量是用户的输入
    history_messages_key="chat_history", # 告诉系统哪个变量用来放历史记录
)

# ==========================================
# 运行阶段：测试流水线
# ==========================================

# 第一次对话（测试知识库）
print("--- 第一轮对话 ---")
result1 = chain_with_memory.invoke(
    "荣耀Magic6用的是什么芯片？",
    config={"configurable": {"session_id": "user_001"}} # 传入用户ID，区分记忆
)
print(result1)
# 预期输出: {'answer': '荣耀Magic6搭载了第三代骁龙8旗舰芯片。', 'emotion': 'professional'}

# 第二次对话（测试记忆能力）
print("\n--- 第二轮对话 ---")
result2 = chain_with_memory.invoke(
    "那我刚才问了你什么手机？",
    config={"configurable": {"session_id": "user_001"}} # 使用同一个ID，读取刚才的记忆
)
print(result2)
# 预期输出: {'answer': '您刚才问的是荣耀Magic6手机。', 'emotion': 'friendly'}
```
## 结构解析（大白话进阶版）

在这个加入了知识库和记忆的完整例子中，数据流转变得更加智能，但依然遵循流水线的逻辑：

```text
                        [ 用户输入 ] 
                              |
                              | (传入: question="荣耀Magic6用的是什么芯片？", session_id="user_001")
                              v

+===================================================================================+
| 1. 记忆读取拦截 (RunnableWithMessageHistory - 进场)                               |
|    |-- 系统根据 session_id 去后台查找                                             |
|    |-- 提取之前的聊天记录，生成变量 **{chat_history}**                            |
+===================================================================================+
                              |
                              v

+===================================================================================+
| 2. 并行处理与知识检索 (setup_and_retrieval)                                       |
|    |-- 此时数据兵分两路：                                                         |
|    |                                                                              |
|    |---> [分支 A：检索] question -> **[向量数据库(FAISS)]** -> 找到相关文档       |
|    |                    -> 格式化提取纯文本 -> 生成变量 **{context}**             |
|    |                                                                              |
|    |---> [分支 B：透传] question -> [RunnablePassthrough] -> 原样传递             |
|    |                    -> 维持变量 **{question}**                                |
+===================================================================================+
                              |
                              | (此时已集齐三大核心变量：{chat_history}, {context}, {question})
                              v

+===================================================================================+
| 3. 提示词组装 (ChatPromptTemplate)                                                |
|    |-- 将上述三个变量填入预设的模板中                                             |
|    |-- 拼装成一段包含背景知识、历史记忆和当前问题的**完整超级指令**               |
+===================================================================================+
                              |
                              | (传递: 组装好的完整 Prompt 消息)
                              v

+===================================================================================+
| 4. 大语言模型思考 (ChatOpenAI)                                                    |
|    |-- 接收超级指令，结合知识和记忆进行推理                                       |
|    |-- 输出一段**原始的纯文本字符串** (例如带有 ```json 的文本)                   |
+===================================================================================+
                              |
                              | (传递: 原始纯文本字符串)
                              v

+===================================================================================+
| 5. 输出解析清洗 (JsonOutputParser)                                                |
|    |-- 剔除 Markdown 符号和废话                                                   |
|    |-- 将纯文本强制转化为**结构化的 Python 字典**                                 |
+===================================================================================+
                              |
                              v

+===================================================================================+
| 6. 记忆保存拦截 (RunnableWithMessageHistory - 出场)                               |
|    |-- 拦截最终结果，将本次的“用户问题”和“模型答案”打包                           |
|    |-- **自动存入 session_id 对应的记忆库**，供下一次对话使用                     |
+===================================================================================+
                              |
                              | (传递: 干净的 Python 字典 {'answer': '...', 'emotion': '...'})
                              v
                        [ 最终输出结果 ]
```

---

## 进阶过渡：从“手动挡流水线”到“自动挡智能体”的代码演进，可直接从此章节开始学习

在掌握了基础的 | (LCEL) 语法后，我们需要了解 LangChain 框架在面对**“复杂工具调用”**时，发生的一次极其优雅的架构升级。
这里要特别说明一句：你在最新博客里看到代码风格大变，绝对不是我们之前学的操作不对，而是官方工具本身在近期版本（特别是引入 LangGraph 生态后）进行了跨代升级。
虽然底层依然是我们熟悉的 Prompt、Model 和 Parser，但当我们需要让大模型自主使用工具（Tools）时，代码的编写范式发生了改变。

为了让你直观感受到这种工具升级带来的便利，我们假设有一个需求：“让大模型调用天气工具，查询北京天气并回复用户”。
我们来看看实现同样的功能，旧版本的“单向链条”和新版本的“Agent 智能体”在代码上有什么区别：
1. 以前的做法：LCEL 单向链条（手动挡）
在纯 LCEL 时代，因为流水线是单向的，不能自动回头。
如果大模型决定调用工具，开发者必须自己写代码去拦截、执行工具，然后再手动把结果塞回给大模型。
只是你之前学的是**“如何构建一条完美的单向流水线（Chain）”，
而现在的 create_agent，是为了解决“如何构建一个能自主使用工具的循环大脑（Agent）”**。两者是不同场景下的最优解！

```python
# 【手动挡模式】使用基础的 LCEL 逻辑
# 1. 把工具绑在模型上
model_with_tools = model.bind_tools([search_weather_tool])

# 2. 第一轮调用：模型不会直接给答案，而是输出一个“工具调用请求”
ai_msg = model_with_tools.invoke("北京今天天气怎样？")

# 3. 开发者必须手动写 if-else 来处理这个请求
if ai_msg.tool_calls:
    # 手动提取参数，手动执行工具
    tool_args = ai_msg.tool_calls[0]['args']
    tool_result = search_weather_tool.invoke(tool_args)

    # 4. 手动把工具的结果拼接到历史消息中，发起第二轮调用
    final_result = model.invoke([
        ("human", "北京今天天气怎样？"),
        ai_msg,  # 模型的工具调用请求
        ("tool", tool_result)  # 工具的返回结果
    ])
    print(final_result.content)
```
痛点：一旦业务复杂，需要调用多个工具或者报错重试时，这里的 if-else 和 while 循环会写得极其痛苦且容易崩溃。
2. 现在的做法：create_agent (自动挡)
为了解决上述痛点，LangChain 升级了底层架构，引入了 LangGraph，并提供了 create_agent（或 create_react_agent）这样的高级封装。
它在底层自动帮你画好了一个**“思考 -> 调用工具 -> 观察结果 -> 总结”的循环图**。
```python
from langgraph.prebuilt import create_react_agent

# 【自动挡模式】使用最新的 Agent 封装
# 1. 一行代码组装：把模型和工具交给 Agent，它会自动在底层构建循环图
agent = create_react_agent(model, tools=[search_weather_tool])

# 2. 直接调用：你只需要发号施令，中间的工具调用、循环、拼装，它全自动搞定
result = agent.invoke({"messages": [("human", "北京今天天气怎样？")]})

print(result["messages"][-1].content)

```
核心总结：为什么博客里全是 create_agent？
通过对比就能发现，create_agent 并没有推翻 LangChain 的基础组件，它肚子里装的依然是 Model 和 Tools。
它只是把 LCEL 时代那些繁琐的 if-else 判断、手动执行工具、以及循环重试的代码，全部封装进了一个标准化的图（Graph）结构里。

- 简单代码（RAG / 检索增强）： 是开发者提前把资料准备好，**“喂”**给大模型。这是一条单向直达的高速公路，用你写的 LCEL | 语法最完美、最简单。
- 复杂代码（Agent / 智能体）： 是开发者给大模型一堆工具，让大模型**“自己动手”**去查资料。这需要反复对话、循环试错（ReAct 理念）。在没有 create_agent 之前，用纯 LCEL 去写这种“循环”逻辑，就会出现我刚才演示的那种满篇 if-else 的痛苦代码。

所以在企业级 Agent 开发的博客中，我们不再使用基础的 | 去手动拼接复杂的工具链，而是直接享受工具升级的红利，使用 create_agent 召唤一个自带“自动驾驶”能力的智能体。
理解了这一层封装，我们再往下看 Agent 的中间件和高级玩法，就豁然开朗了！

---

一句话：如果业务流程是固定的、不需要大模型去动态做选择或判断分支，直接用 LCEL 拼一条单向流水线就是最高效、最优雅的做法！

---

## LangChain  2025 年 10 月 后 1.0 版本大升级

一、 什么是 LangChain？
- 行业地位：目前全球搞 Agent（智能体）开发最火、生态最庞大 的开源框架。
- 重要分水岭：在 2025 年 10 月，LangChain 升级了 1.0 大版本。这是一个破坏性更新，1.0 版本和以前的 0.x 老版本在底层架构上完全是两个东西（特别是引入了 LangGraph 处理复杂工作流）。
二、 LangChain 的七大核心组件
无论你的 Agent 有多复杂，基本都逃不出这几个积木块：
1. Agent（智能体）：大脑，负责思考下一步该干嘛。
2. LLM（大语言模型）：引擎，比如 GPT-4、豆包、DeepSeek。
3. Message（消息）：血液，在组件之间传递的对话数据。
4. Tools（工具）：手脚，比如联网搜索、查天气、读数据库。

5. 记忆（Memory）：让 AI 记住上下文，不至于“聊完就忘”。

6. Stream（流式输出）：像打字机一样一个字一个字往外蹦，提升用户体验。

7. 结构化输出：强制 AI 返回 JSON 等固定格式，方便程序解析。

---

## Langchain 的 Agent 消息格式拆解

当返回一段话时，它不仅仅返回了“文字”，而是返回了一个极其丰富的数据包。
这个数据包通常包含 HumanMessage（你发的话）和 AIMessage（AI 的回复）。
原文中的原始数据结构如下：
```python
{
    "messages": [
        HumanMessage(
            content="你是谁？",
            additional_kwargs={},
            response_metadata={},
            id="1952a4d9-6f93-4ba7-bd7d-46fe1d42393b",
        ),
        AIMessage(
            content="我是智能助手小宁，很高兴能为你提供帮助~",
            additional_kwargs={"refusal": None},
            response_metadata={
                "token_usage": {
                    "completion_tokens": 85,
                    "prompt_tokens": 46,
                    "total_tokens": 131,
                    "completion_tokens_details": {
                        "accepted_prediction_tokens": None,
                        "audio_tokens": None,
                        "reasoning_tokens": 71,
                        "rejected_prediction_tokens": None,
                    },
                    "prompt_tokens_details": {"audio_tokens": None, "cached_tokens": 0},
                },
                "model_provider": "openai",
                "model_name": "doubao-seed-1-6-251015",
                "system_fingerprint": None,
                "id": "0217684493419497d8ec4d988cc7ba637a273ca7a40c11bd8c5d4",
                "service_tier": "default",
                "finish_reason": "stop",
                "logprobs": None,
            },
            id="lc_run--019bbfcb-77f6-72f3-94e1-840b598f04dd-0",
            tool_calls=[],
            invalid_tool_calls=[],
            usage_metadata={
                "input_tokens": 46,
                "output_tokens": 85,
                "total_tokens": 131,
                "input_token_details": {"cache_read": 0},
                "output_token_details": {"reasoning": 71},
            },
        ),
    ]
}

```
以这段代码中的 AIMessage 为例，它的核心属性可以分为四大类：
1. 基础识别属性（我是谁，我说了啥）
- content：最核心的字段，也就是 AI 真正回复给你的文本内容（如："我是智能助手小宁..."）。
- id：这条消息的“身份证号”（如："lc_run--019bbfcb..."），用于在复杂的链路中精准追踪。
- additional_kwargs：额外参数，用来放一些特定大模型厂商独有的特殊数据（如：{"refusal": None}）。

2. usage_metadata（消耗统计：关乎你的钱包）
这部分记录了本次对话消耗的 Token（算力计费单位）：
- input_tokens / output_tokens / total_tokens：输入、输出和总消耗（如代码中的 46, 85, 131）。
- reasoning（推理 Token）：非常关键！ 现在的深度思考模型（如 DeepSeek-R1 或豆包的思考模型）在给出答案前会在后台“碎碎念”思考很久，这部分思考过程虽然不一定显示给用户，但也是要计费的，就记录在这里（如代码中的 "reasoning": 71）。

3. response_metadata（响应详情：底层快照）
这是模型供应商返回的原始底层信息：
- model_provider & model_name：你用的是哪家厂商的哪个具体模型（例如 "doubao-seed-1-6-251015"）。
- finish_reason（结束原因）：最理想的值是 "stop"，代表模型自然地说完了。如果是 length，说明字数超限被强行掐断了。

4. 工具与状态属性（Agent 自动化的关键）
当 AI 不只是聊天，而是作为 Agent 帮干活时，这几个字段起决定性作用：
- tool_calls：如果 AI 觉得需要调用工具（比如查天气），它不会直接回复文字，而是把要调用的工具名称和参数放在这里（代码中目前为空 []）。
- invalid_tool_calls：记录 AI 试图调用工具但格式写错了的失败记录。
- refusal：如果你的问题触发了安全限制，AI 拒绝回答的理由会写在这里。

总结：
在开发 LangChain 应用时，我们平时只看 content 就够了；但当你需要做成本核算（看 Token）、做复杂 Agent（看 tool_calls）、或者排查 Bug（看 finish_reason） 时，这些元数据就是你最好的帮手。

---

## Agent 动态选择 LLM：拦截Request请求，替换model参数

为了平衡成本和效果，我们通常不会从头到尾只用一个大模型。
比如：简单的闲聊用便宜的“基础模型”，当对话变得很长、逻辑变得复杂时，再切换成昂贵的“高级模型”。
这段代码就是用来实现这个功能的。我为你梳理了它的核心逻辑：
```python
@wrap_model_call
def dynamic_model_selection(request: ModelRequest, handler) -> ModelResponse:
    """动态模型选择"""

    # 修复1: 用 request.messages 获取当前对话的所有历史消息
    messages = request.messages
    message_count = len(messages)

    print(f"\n[中间件] 消息数: {message_count}")

    # 核心路由逻辑：根据上下文长度决定用哪个模型
    if message_count > 6:
        print("高级模型")
        model = advanced_model
    else:
        print("基础模型")
        model = basic_model

    # 修复2: 覆盖原始请求中的模型，并继续执行
    return handler(request.override(model=model))

```

1. @wrap_model_call (拦截器/中间件) 见后续生命周期钩子章节
这个装饰器的作用就像是一个**“收费站”或“安检口”**。
在你的代码真正把问题发送给大模型之前，它会先把请求拦截下来，交给你定义的 dynamic_model_selection 函数处理。处理完之后，再放行。

2. request.messages (获取上下文) 见下一章节 什么是 Request 
这里提取了当前对话中所有的历史消息。
代码通过 len(messages) 计算了消息的回合数。这是一种非常经典的判断标准：对话越长，往往意味着任务越复杂，越需要模型具备强大的长文本理解能力。

动态路由逻辑 (If-Else 切换)
- 基础模型 (basic_model)：当消息数 ≤6≤6 时，说明还在简单的开场或初步交流阶段，调用便宜、速度快的基础模型（比如豆包的 lite 版本）。
- 高级模型 (advanced_model)：当消息数 >6>6 时，说明上下文已经很长了，基础模型可能“记不住”或者“脑子不够用”了，立刻无缝切换到参数量更大、更聪明的高级模型（比如 DeepSeek-R1 或 GPT-4）。

3. request.override(model=model) (狸猫换太子)
这是最关键的一步。它把请求包裹里原本默认配置的模型，**强行替换（override）**成了你刚才通过 If-Else 选出来的模型。
最后通过 handler(...) 把修改后的请求发出去，完成调用。

这种“动态选择 LLM”的机制是企业级 Agent 开发的标配。
它不仅能根据消息长度切换，你以后还可以根据用户身份**（VIP用户用高级模型，普通用户用基础模型）或者任务类型（写代码用代码模型，画图用画图模型）来进行动态路由，极大提升系统的灵活性和性价比。

### Request 包含哪些信息？

起点（你调用 invoke）：
你在代码的最外层写下类似这样的代码：

```python
agent.invoke({"messages": ["帮我写一段Python代码"]})
```

这时候，你只提供了最基础的用户输入。

框架打包（生成 Request）：
当你按下回车，底层框架接管了工作。它会像一个尽职尽责的快递打包员，把你传入的 messages，连同你在 create_agent 时配置好的默认大模型（model）、给 Agent 配备的工具箱（tools）、以及当前的系统状态（state 和 config），全部整合在一起。
这个整合后的“超级大包裹”，就是你代码里看到的 request 对象。

1. request.messages (最常用) 这是最重要的属性，包含当前对话的完整消息列表。
- 内容：一个包含 HumanMessage、AIMessage 等对象的列表。
- 用途：你代码中判断 len(messages) 就是在统计这个列表的长度，以此决定对话的复杂度。
2. request.model 这是原始打算使用的模型对象。
- 内容：你在 create_agent(model=llm, ...) 中传入的那个 llm 对象。
- 用途：中间件可以读取它，也可以通过 request.override(model=new_model) 来替换它。
3. request.tools 这是 Agent 此时可以调用的工具列表。
- 内容：你在创建 Agent 时定义的 tools 数组。
- 用途：你可以根据请求的内容动态决定是否禁用某些工具，或者在高级模型下开启更多复杂工具。
4. request.state 这是当前的状态字典。
- 内容：除了消息之外的其他状态数据（例如：用户 ID、当前处理步骤、临时变量等）。
- 用途：如果你在 invoke 时传入了除了 messages 以外的其他键值对，它们通常会出现在这里。 5.其他控制参数 (如 request.config) 包含运行时的配置信息。
- 内容：例如 recursion_limit（递归深度限制）、configurable（可配置项）等。

---

## LangChain Message 消息格式规范

在 LangChain 中构建对话消息时，主要有两种格式。
对于简单的测试，可以使用字典格式；
但在构建复杂的生产级项目时，强烈推荐使用 LangChain 原生的 Message 类对象。
使用 LangChain 提供的 SystemMessage、HumanMessage、AIMessage 等类来显式定义消息角色。
```python
from langchain.chat_models import init_chat_model
from langchain.messages import HumanMessage, SystemMessage

model = init_chat_model("gpt-5-nano")

```
### 1. 实例化消息对象

```python
system_msg = SystemMessage("You are a helpful assistant.")
human_msg = HumanMessage("Hello, how are you?")

```
### 2. 传入模型

```python
messages = [system_msg, human_msg]
response = model.invoke(messages)  # 返回的结果也是一个 AIMessage 对象

```
字典格式（OpenAI 原生风格，适合简单场景）
直接使用包含 role 和 content 键的字典列表。
python
复制
```python
messages = [
    {"role": "system", "content": "You are a poetry expert"},
    {"role": "user", "content": "Write a haiku about spring"},
    {"role": "assistant", "content": "Cherry blossoms bloom..."}
]
response = model.invoke(messages)

```
---

### 复杂项目用 Message 类对象？（核心优势）

简单的 dict 字典在处理纯文本对话时没有问题，但在复杂的高级应用中会显得力不从心。使用 Message 类对象具有以下三大核心优势：
1. 支持多模态与结构化扩展
- 痛点：简单的 dict 很难优雅地表达复杂的嵌套结构。
- 优势：Message 对象可以完美封装高级属性。例如，返回的 AIMessage 不仅包含文本，还可以直接携带 tool_calls（工具调用参数）、usage_metadata（Token 消耗统计），甚至可以方便地封装多模态数据（如图片、音频）。

2. 抹平差异，保持接口一致性 (Interface Consistency)
- 痛点：不同的底层大模型厂商，其 API 要求的参数可能不同（比如有的要求 {"role": "user"}，有的要求 {"author": "human"}）。
- 优势：在 LangChain 业务代码层面，你永远只需要关心 HumanMessage。LangChain 底层的“适配器”会自动将其翻译为对应目标模型所需的正确格式，实现代码与底层模型的解耦。

3. 状态追踪与持久化
- 优势：Message 对象自带 id 和 metadata 属性。
- 应用场景：这在构建复杂的 LangGraph 工作流，或者需要将对话历史持久化保存到数据库（如 Redis, MongoDB）时至关重要，方便精准追踪每一条消息的来源和流转状态。

---

## LangChain 流式输出与四种核心模式

在构建复杂的 Agent 或大模型应用时，如果等模型完全思考并执行完所有步骤再返回结果，用户体验会非常卡顿（尤其是包含多个 Tool 调用的场景）。
流式输出（Streaming） 是降低用户感知延迟的核心技术。
在 LangGraph / LangChain 架构中，流式输出不仅限于“打字机效果”，而是对整个 Agent 运行状态的实时透视。
为什么需要高级的流式输出？
传统的 llm.stream() 只能实现单纯的文本“打字机”效果。但在 Agent 场景下，我们需要知道：
- Agent 现在走到哪个节点了？
- 全局状态（State）发生了什么变化？
- 工具调用的中间结果是什么？
为此，LangGraph 在调用 .stream() 时，提供了 stream_mode 参数，包含 四种核心模式，以满足不同的业务和调试需求。

---

### 流式输出的四种核心模式 (stream_mode)

1. stream_mode="messages" (🌟 推荐：面向终端用户的打字机效果)
- 核心作用：专门用于捕获大模型生成的 Token 级别 的流式数据。
- 适用场景：前端需要实现类似 ChatGPT 的实时打字机回复效果。
- 特点：它会过滤掉繁杂的图状态流转，只把底层的 AIMessageChunk 实时抛出来。

### 面向用户的实时打字机输出

```python
for chunk in agent.stream(
    {"messages": [HumanMessage(content="讲个长故事")]}, 
    stream_mode="messages"
):

    # chunk 是一个元组: (message_chunk, metadata)

    msg_chunk = chunk[0]
    if isinstance(msg_chunk, AIMessageChunk):
        print(msg_chunk.content, end="", flush=True)

```
2. stream_mode="updates" (🌟 推荐：面向业务逻辑的增量状态修改)
- 核心作用：每次某个节点（Node）执行完毕后，只返回该节点对全局状态的“增量修改”。
- 适用场景：需要在后端监控 Agent 执行进度（例如：UI 上显示“正在搜索天气...”、“正在总结...”）。
- 特点：数据量小，逻辑清晰，只告诉你“刚刚发生了什么改变”。

### 监控 Agent 节点的执行进度

```python
for chunk in agent.stream(inputs, stream_mode="updates"):
    for node_name, state_update in chunk.items():
        print(f"✅ 节点 [{node_name}] 执行完毕！")
        print(f"🔄 状态更新内容: {state_update}")

```
3. stream_mode="values" (全量状态快照)
- 核心作用：每次节点执行完毕后，返回当前全局 State 的完整快照。
- 适用场景：需要随时获取整个对话历史和所有上下文变量的场景。
- 特点：数据量较大，因为它不仅包含增量，还包含之前所有的历史消息和状态。
```python
for full_state in agent.stream(inputs, stream_mode="values"):

    # 每次打印的都是截至目前的完整 messages 列表

    print(f"📦 当前总消息数: {len(full_state['messages'])}")

```
4. stream_mode="debug" (底层调试模式)
- 核心作用：输出最底层的执行事件，包括节点进入、节点退出、边（Edge）的路由判断等。
- 适用场景：开发阶段排查死循环、路由条件判断错误等复杂 Bug。
- 特点：极其详细，相当于给 Agent 开启了最高级别的日志输出。
```python
for event in agent.stream(inputs, stream_mode="debug"):
    print(f"🐛 [DEBUG EVENT]: {event['type']} - {event['step']}")

```
---

总结与最佳实践
在实际做复杂项目时，通常会组合使用这些模式：
1. 给前端用户看：强制使用 stream_mode="messages"，只把文本 Token 推送给 WebSocket。
2. 给前端展示进度条/步骤：使用 stream_mode="updates"，当捕获到 node_name == "tools" 时，向前端发送“正在调用工具”的信号。
3. 本地开发排错：直接开启 stream_mode="debug"，像用放大镜一样看清 Agent 的每一步路由决策。

---

## Tools 状态传递与性能优化

Tools 之间的状态传递方式
在多工具 Agent 场景下，工具之间传递状态/数据主要有以下 3 种方式：
1. LLM 自动传递：依赖大模型自身的理解，自动把前一个工具的输出作为参数传给下一个工具。
2. 通过 Context 传递：使用 ToolRuntime[Context]。工具可以直接读取或修改全局的 Context（如 user_id），实现底层状态共享，无需 LLM 显式传参。
3. 直接合并 Tools（性能优化）：将逻辑强关联的小工具合并成一个大工具，减少 LLM “思考-调用-返回”的循环次数。

---

## 工具调用性能优化（实战：耗时 28s ➡️ 9s）

通过 3 步优化，将 Agent 整体执行耗时压缩了 3 倍以上。
优化 1：修改系统提示词 (28s ➡️ 12s) 【核心优化点】
思路：混合结构化与自然语言引导，明确条件分支，强制禁止输出思考过程，直接减轻模型推理负担。
优化后的 Prompt

```python
SYSTEM_PROMPT = """你是一个天气查询助手。
请遵循以下确定性操作：
如果没有城市名，请立即执行 get_user_location。
如果已有城市名，请立即执行 get_weather_for_location。
不要输出任何思考过程，直接返回工具调用。在得到最终天气结果后，直接回复天气信息。"""
```

优化 2：精简工具描述 (耗时维持 12s，提升准确率)
思路：缩短 Docstring 直击要点，并明确标注参数类型，减少模型构造参数时的犹豫。

```python
@tool
def get_user_location(runtime: ToolRuntime[Context]) -> str:
    """获取用户当前城市(内部ID定位)。""" # 👈 缩短描述

@tool
def get_weather_for_location(city: str) -> str:
    """查询指定城市天气。参数:city(str)。""" # 👈 明确参数类型
```

优化 3：限制 Max Tokens (12s ➡️ 9s)
思路：在初始化 LLM 时显式设置 max_tokens，截断不必要的冗长输出，提升响应速度。
```python
llm = ChatOpenAI(
    model="doubao-seed-1-8-251228",
    temperature=0,
    max_tokens=5000, # 👈 增加 max_tokens 限制

    # ...

)

```
---

## 调试技巧：透视 LLM 思考决策过程 (Stream 模式)

使用 .invoke() 只能看到最终结果，是个黑盒。
改用 .stream() 模式 可以按节点（agent 节点和 tools 节点）遍历，提取中间隐藏的思考和调用过程。

### 使用 stream 模式替代 invoke

```python
for chunk in agent.stream(
    {"messages": [{"role": "user", "content": "今天什么天气？"}]},
    config=config,
    context=Context(user_id="1")
):
    for node_name, output in chunk.items():

        # 1. 解析 Agent 节点的思考与决策

        if "messages" in output:
            last_msg = output["messages"][-1]

            # A. 提取隐藏推理过程 (Reasoning Tokens，如豆包1.8)

            reasoning = last_msg.additional_kwargs.get("reasoning_content", "")
            if reasoning:
                print(f"🧠 【内部推理】:\n{reasoning}")

            # B. 提取工具调用决策

            if hasattr(last_msg, 'tool_calls') and last_msg.tool_calls:
                for tc in last_msg.tool_calls:
                    print(f"🎯 【调用工具】: {tc['name']} | 参数: {tc['args']}")

            # C. 提取最终回复

            elif last_msg.content:
                print(f"💬 【最终回复】: {last_msg.content}")

        # 2. 解析 Tools 节点的物理返回结果

        elif node_name == "tools":
             print(f"✅ 【工具返回结果】: {output}")

```
总结：在开发和调试 Agent 时，必须使用 Stream 模式配合上述解析逻辑，才能清楚知道大模型到底在想什么、调用了什么工具、传入了什么参数，从而针对性地优化 Prompt 和 Tool 描述。

---

## LangChain InMemorySaver 与 Agent 状态 (State) ：给 Agent 装上“记忆”与查看“快照”

在创建 Agent 时，最核心的动作是给 Agent 加上了 checkpointer=InMemorySaver()。
你可以把 InMemorySaver 理解为给 Agent 插上了一根**“短期记忆内存条”**。
如果没有它，Agent 就像金鱼一样，每次对话完就失忆；
有了它，只要你提供同一个 thread_id（比如代表同一个用户），Agent 就能顺着之前的上下文继续聊。
代码实现：

```python
from langgraph.checkpoint.memory import InMemorySaver

# 创建 Agent (注册工具并挂载记忆)
agent = create_agent(
    model=llm,
    system_prompt=SYSTEM_PROMPT,
    tools=[get_weather_info],
    context_schema=Context,
    checkpointer=InMemorySaver(), # 核心：挂载短期记忆
)
```

使用 config 获取当前 thread_id 的最新状态（拍下全息快照）

```python
state = agent.get_state(config)
```

当你调用 state = agent.get_state(config) 时，你其实是按下了暂停键，
给 Agent 当前的大脑拍了一张**“全息快照”**。这个快照里包含了四个极其重要的部分：

1. state.values（核心业务数据）
这是你最需要关注的地方。它就像是 Agent 的**“工作台”**，里面摆放着
所有的对话历史（messages）、
模型思考的过程（thought_metadata）、
工具调用的记录以及 Token 的消耗量。
Agent 就是看着工作台上的这些数据，来决定下一句话该说什么的。
数据结构示例：

```json
{
  "checkpoint_info": {
    "thread_id": "1",
    "status": "completed"
  },
  "messages": [
    {
      "role": "user",
      "type": "HumanMessage",
      "content": "今天什么天气？"
    },
    {
      "role": "assistant",
      "type": "AIMessage",
      "thought_metadata": {
        "reasoning_tokens": 155,
        "thought_process": "模型判断需要获取位置并查询天气"
      },
      "tool_calls": [
        {
          "tool_name": "get_weather_info"
        }
      ]
    },
    {
      "role": "tool",
      "type": "ToolMessage",
      "content": "佛罗里达 总是阳光明媚！"
    },
    {
      "role": "assistant",
      "type": "AIMessage",
      "content": "佛罗里达今天阳光明媚呢！☀️",
      "thought_metadata": {
        "reasoning_tokens": 98
      }
    }
  ],
  "cumulative_usage": {
    "total_input_tokens": 971,
    "total_output_tokens": 292
  }
}
```

2. state.metadata（运行溯源信息）
这部分记录了状态是怎么来的。比如 step: 3 清楚地告诉你，从用户输入到现在，系统已经流转了 3 步（思考 -> 调用工具 -> 总结回复）。
source: "loop" 则说明这是框架自动循环执行产生的结果。这在排查死循环 Bug 时非常有用。
数据结构示例：
{"source": "loop","step": 3,"parents": {}}
3. state.next（流程控制指针）
它就像是 Agent 的**“下一步计划表”**。它会告诉你接下来准备执行哪个节点（比如是继续思考 ('agent',)，还是去执行工具 ('tools',)）。如果任务已经全部完成，它就会是一个空值 ()。在需要“人工审批”的场景中，这个属性是关键。
4. state.config（状态的身份证）
这里面包含了 thread_id 和唯一的 checkpoint_id。
有了这个唯一的 ID，你甚至可以实现**“时间旅行”**——让代码拿着旧的 ID，强行让 Agent 回滚到历史的某一步重新开始跑。

---

## Agent 的内置中间件 (Middleware)

中间件是在 Agent 接收输入前，或输出结果后，自动执行的一系列拦截和处理机制。
合理使用中间件，可以让 Agent 从“玩具”变成“企业级应用”。
内存与上下文管理（解决“Token太长/太贵”问题）
SummarizationMiddleware（上下文压缩）：写回忆录。调用小模型把用户的早期聊天记录总结成摘要，保留长期记忆的同时给大模型减负。
ContextEditingMiddleware（上下文清除）：进碎纸机。直接在代码底层物理删除冗长的、无用的工具执行日志，零成本瞬间释放上下文空间。

### SummarizationMiddleware（压缩对话上下文）

作用：给大模型“减负”与“省钱”。
大模型的上下文窗口有限且按 Token 收费。如果对话太长，不仅贵，还容易报错。
这个中间件会在 Token 达到阈值时，自动用一个便宜的小模型把早期的历史记录总结成摘要，同时保留最近的几条原始消息。
```python
from langchain.agents import create_agent
from langchain.agents.middleware import SummarizationMiddleware

agent = create_agent(
    model="gpt-4o",
    tools=[your_weather_tool, your_calculator_tool],
    middleware=[
        SummarizationMiddleware(
            model="gpt-4o-mini",       # 核心：用便宜的小模型来干总结的活
            trigger=("tokens", 4000),  # 触发红线：当这轮对话累积消耗达到 4000 Token 时触发
            keep=("messages", 20),     # 保留策略：最近的 20 条消息原封不动保留，更早的被压缩
        ),
    ],
)
```

ContextEditingMiddleware (清除工具上下文)

适用场景：长时间的多轮对话中，Agent 会积累大量的“工具调用日志”，极易撑爆 Token 限制。
核心逻辑：延迟清理策略（平时保留，快满了才删，且保留最近的记忆以防死循环）。

```python
from langchain.agents import create_agent
from langchain.agents.middleware import ContextEditingMiddleware, ClearToolUsesEdit

agent = create_agent(
    model="gpt-4o",
    tools=[],
    middleware=[
        ContextEditingMiddleware(
            edits=[
                ClearToolUsesEdit(
                    trigger=100000, # 触发阈值：历史记录达到 10万 Token 时才启动清理
                    keep=3,         # 保留数量：删掉旧日志，但保留最近 3 次工具调用，防止 Agent 忘记刚才干了啥
                ),
            ],
        ),
    ],
)
```

---

## 稳定性与高可用（解决“网络波动/服务宕机”问题）

ModelFallbackMiddleware（模型兜底）：主模型挂了备胎上。当主模型（如 gpt-4o）因为限流或宕机报错时，自动切换到备用模型，保障业务不中断。
ToolRetryMiddleware（工具自动重试）：防熄火系统。当调用外部第三方 API 失败或超时时，在底层自动重新发起请求，解决外部接口不稳定的问题。

### ModelFallbackMiddleware（模型错误兜底）

作用：保证系统的高可用性（High Availability）。
在实际生产中，主模型（如 gpt-4o）可能会因为网络波动、API 宕机或并发限流而调用失败。这个中间件允许你配置一系列备胎模型，主模型挂了，备胎立刻顶上，用户完全无感知。
```python
from langchain.agents import create_agent
from langchain.agents.middleware import ModelFallbackMiddleware

agent = create_agent(
    model="gpt-4o",
    tools=[],
    middleware=[
        ModelFallbackMiddleware(
            "gpt-4o-mini", # 第一备胎：同厂小模型
            "claude-3-5-sonnet-20241022", # 第二备胎：其他厂商模型
        ),
    ],
)
```

### RetryMiddleware（工具自动重试）

在调用外部 API（如数据库、天气接口）或大模型自身时，网络波动是常态。我们需要优雅的重试机制。
工具重试 (ToolRetryMiddleware)：处理工具执行失败。
模型重试 (ModelRetryMiddleware)：处理 LLM API 调用失败。
```python
from langchain.agents import create_agent
from langchain.agents.middleware import ToolRetryMiddleware, ModelRetryMiddleware

agent = create_agent(
    model="gpt-4o",
    tools=[search_tool, database_tool],
    middleware=[
        # 工具执行失败时的重试策略
        ToolRetryMiddleware(
            max_retries=3,      # 最大重试次数（防止无限重试卡死）
            initial_delay=1.0,  # 初始等待时间（秒），给目标服务喘息时间
            backoff_factor=2.0, # 退避倍数（指数级增长，如1s -> 2s -> 4s，有效应对并发拥堵）
        ),

        # 模型调用失败时的重试策略（参数同上）
        ModelRetryMiddleware(max_retries=3, backoff_factor=2.0, initial_delay=1.0),
    ],
)
```

---

## 安全与风控（解决“失控/泄密/破产”问题）

PIIMiddleware（隐私打码）：数据安检员。在数据发给大模型前，自动将邮箱、信用卡等敏感信息涂黑或掩码，防止企业机密和用户隐私泄露。
ModelCallLimit / ToolCallLimit（熔断限流）：防死循环与防破产。强制设定最大思考步数或工具调用次数，一旦超过立刻停止，防止大模型发疯把 API 余额刷光。

### PIIMiddleware（PII 检测与敏感信息屏蔽）

作用：保护用户隐私与数据安全。
PII (Personally Identifiable Information) 指个人身份信息。
企业绝对不能把用户的邮箱、信用卡号等明文发给外部的大模型 API。这个中间件充当**“安检员”**，在数据发给模型前，自动打码或替换敏感词。

```python
from langchain.agents import create_agent
from langchain.agents.middleware import PIIMiddleware

agent = create_agent(
    model="gpt-4o",
    tools=[],
    middleware=[
        # strategy="redact": 直接涂黑/删除（例如变成 [REDACTED]）
        PIIMiddleware("email", strategy="redact", apply_to_input=True),

        # strategy="mask": 掩码打码（例如变成 ****1234）
        PIIMiddleware("credit_card", strategy="mask", apply_to_input=True),
    ],
)
```

### ModelCallLimit & ToolCallLimit（调用限制）

作用：防止 Agent 陷入“死循环”。
有时候模型会“发疯”，不断地重复调用某个工具，或者在思考节点无限死循环。这两个中间件就是强制熔断机制，设定最大步数，超过就强制停止。
```python
from langchain.agents import create_agent
from langchain.agents.middleware import ModelCallLimitMiddleware, ToolCallLimitMiddleware
from langgraph.checkpoint.memory import InMemorySaver

agent = create_agent(
    model="gpt-4o",
    checkpointer=InMemorySaver(), # 必须挂载记忆才能限制整个 thread
    tools=[search_tool, database_tool],
    middleware=[
        # 1. 限制模型思考次数
        ModelCallLimitMiddleware(
            thread_limit=10,       # 整个对话历史最多调用 10 次模型
            run_limit=5,           # 单次运行最多调用 5 次模型
            exit_behavior="end",   # 超出限制后的行为：直接结束
        ),

        # 2. 限制工具调用次数
        ToolCallLimitMiddleware(thread_limit=20, run_limit=10), # 全局工具限制
        ToolCallLimitMiddleware(
            tool_name="search",    # 针对特定工具单独限制
            thread_limit=5,
            run_limit=3,
        ),
    ],
)
```

---

## 路由与性能优化（解决“工具太多/大模型看花眼”问题）

LLMToolSelectorMiddleware（工具初筛路由）：小模型当秘书。面对海量工具时，先用便宜的小模型挑出最相关的几个，再交给大模型处理，大幅提升大模型的专注度并降低成本。

LLMToolSelectorMiddleware（工具初筛路由)
适用场景：比如一个“个人 AI 助手”，用户需求跳跃，导致你必须给 Agent 挂载几十上百个工具。
核心痛点：把所有工具描述都塞给大模型（如 GPT-4o）会导致：
  1. 上下文污染（Attention Loss）：模型在海量工具中迷失，产生幻觉或选错工具。
  2. 推理延迟：Prompt 越长，首字响应时间（TTFT）越久。
  3. 成本高昂：大模型输入 Token 极贵，每轮带上百个工具说明极不划算。
架构理念：业内称为 "Router-Worker"（路由-执行） 或 "Small-to-Large"（小模型驱动大模型） 架构。
```python
from langchain.agents import create_agent
from langchain.agents.middleware import LLMToolSelectorMiddleware

agent = create_agent(
    model="gpt-4o",
    tools=[tool1, tool2, tool3, tool4, tool5, ...], # 假设这里有100个工具
    middleware=[
        LLMToolSelectorMiddleware(
            model="gpt-4o-mini", # 用廉价、极速的小模型做“初筛”
            max_tools=3,         # 精简上下文：最终只给大模型3个最相关的工具
            always_include=["search"], # 保底策略：防止小模型漏掉核心工具
        ),
    ],
)
```

---

## 工程化与高阶赋能（解决“开发测试/复杂任务”问题）

LLMToolEmulator（本地 Mock 模拟）：开发测试神器。在本地模拟工具的返回结果，让开发者在不消耗真实 API 额度的情况下进行调试。
ShellToolMiddleware / Filesystem 等（底层提权）：终极权限。直接赋予 Agent 操作系统级别的权限，让它能自己执行脚本、查阅本地文件，让 Agent 具备干复杂脏活的能力。

### LLMToolEmulator（本地 Mock 模拟）

适用场景：在开发和测试阶段，不想真实消耗第三方 API 的额度（比如发邮件、扣费接口），可以使用模拟器。
```python
from langchain.agents import create_agent
from langchain.agents.middleware import LLMToolEmulator

agent = create_agent(
    model="gpt-4o",
    tools=[get_weather, search_database, send_email],
    middleware=[
        LLMToolEmulator(),  # 拦截所有工具调用，由本地模拟返回结果
    ],
)
```

ShellToolMiddleware / Filesystem 等（底层操作权限赋予）
适用场景：当预设的 Tools 无法满足需求时，赋予 Agent “自己写代码、自己运行、自己查文件” 的最高权限。
核心预警：这就是类似 Manus 等高级 Agent 的底层逻辑。只要宿主机权限够，Agent 理论上可以在 Workspace 里做任何事。

```python
from langchain.agents import create_agent
from langchain.agents.middleware import ShellToolMiddleware, HostExecutionPolicy, FilesystemFileSearchMiddleware

agent = create_agent(
    model="gpt-4o",
    tools=[],
    middleware=[
        # 1. 脚本执行能力：允许 Agent 在指定工作区编写并执行 Python/Shell 脚本
        ShellToolMiddleware(
            workspace_root="/workspace",
            execution_policy=HostExecutionPolicy(),
        ),

        # 2. 本地文件搜索能力：允许 Agent 使用 ripgrep 极速检索工作区内的海量文档
        FilesystemFileSearchMiddleware(
            root_path="/workspace",
            use_ripgrep=True,
        ),
    ],
)
```

---

## 学习建议：

这五个中间件涵盖了 Agent 走向生产环境的必经之路：**性能（路由） -> 稳定性（重试） -> 成本（Mock/清理） -> 终极扩展（Shell）**。
建议在实际项目中，先从 ToolRetryMiddleware 用起，感受一下自动容错的魅力！

---

## Agent 生命周期 (Lifecycle) 钩子：给 Agent 装上“全景监控”与“流程护卫”

在构建复杂的 Agent 时，仅仅让它跑起来是不够的。我们需要在它运行的各个阶段进行监控、拦截或修改。
你可以把生命周期钩子（Hooks）理解为在 Agent 运行轨道上设置的**“安检站”与“监控探头”**。

如果没有它们，Agent 就像一个黑盒，你无法控制它的中间过程；
有了它们，你可以在 Agent 启动前拦截、在模型思考前塞入纸条、在工具执行时加上安全锁。

### 动态提示词注入 (Dynamic Prompt)：给 Agent 戴上“实时智能手表”

在进入具体的钩子前，最常用的一个技巧是 @dynamic_prompt。它允许我们在每次模型调用前，动态地向 System Prompt 中注入最新信息（比如当前时间、用户身份）。
代码实现：

```python
from langchain.agents import create_agent, dynamic_prompt
from datetime import datetime

# 定义动态提示词函数
@dynamic_prompt
def get_realtime_info(state: AgentState, runtime: ToolRuntime[Context]) -> str:
    # 每次模型调用前，都会执行这个函数，获取最新状态
    now = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    user_name = "VIP客户" if runtime.context.user_id == "1" else "普通用户"
    return f"当前的服务器时间是：{now}。当前对话的用户身份是：{user_name}。请根据这些背景回复。"

# 在创建 Agent 时使用
agent = create_agent(
    model=llm,
    system_prompt=[
        "你是一个全能助手。",
        get_realtime_info, # 核心：这里不再是死板的字符串，而是注入了动态函数
    ],
    tools=[...],
    context_schema=Context
)
```

---

### 1. before_agent (全局启动哨兵)

运行时机： 代理开始前（每次对话仅调用一次）
你可以把它理解为 Agent 的**“入场安检门”**。
- 黑名单拦截： 在请求发送给模型前，先检查 UserID 是否在禁言名单中。
- 额度预扣除： 检查用户的 Token 余额是否足够支付本次对话。
- 上下文注入： 自动从数据库提取用户的历史偏好（如“喜欢简短回答”），提前放入初始状态。

### 2. before_model (模型输入质检)

运行时机： 每次模型调用前（可能会触发多次）
你可以把它理解为给大模型递交文件前的**“打包员”**。
- 敏感词过滤： 检查用户问题是否包含违规词，如果有，直接通过 jump_to="end" 结束并报错。
- Prompt 注入： 实时获取系统负载，告诉模型：“当前系统繁忙，请简短回答”。
- 上下文裁减： 如果对话轮数过多，手动删掉最早的消息，防止超出模型最大输入长度（Context Window）。

### 3. after_model (模型输出审计)

运行时机： 每次模型响应后（可能会触发多次）
你可以把它理解为模型输出结果的**“内容审核员”**。
- 工具调用日志： 实时记录模型打算调用哪个工具及参数，用于后台分析决策逻辑。
- 响应脱敏： 检查模型是否意外泄露了系统密钥或密码，在显示给用户前打码。
- 强制格式化： 如果模型应该返回 JSON 但格式有微小错误，在这里进行尝试性的字符串修复。

### 4. wrap_model_call (模型调用全生命周期)

运行时机： 围绕每个模型调用（包裹整个请求过程）
你可以把它理解为套在模型 API 外层的**“防弹衣与计价器”**。
- 耗时统计： 记录模型从发送请求到返回响应的精确时间。
- 自动重试： 如果 API 报 500 错误或触发频率限制，在钩子内自动进行 3 次指数退避重试，对业务层完全透明。
- 成本计算： 获取响应中的 usage 字段，计算本次调用的精确 Token 费用并存入数据库。

### 5. wrap_tool_call (工具执行保护伞)

运行时机： 围绕每个工具调用（包裹工具执行过程）
你可以把它理解为危险工具的**“沙箱安全员”**。
- 二次确认： 如果模型尝试调用 delete_user 工具，拦截执行并返回：“由于操作敏感，请联系人工处理”。
- 超时控制： 给耗时较长的工具（如爬虫）设置硬超时，防止工具运行过久导致 Agent 挂起。
- 模拟/ Mock： 在测试环境下，不真正执行昂贵的 API，而是通过钩子直接返回预设的 Mock 数据。

### 6. after_agent (最终交付质检)

运行时机： 代理完成后（每次对话仅调用一次）
你可以把它理解为整个流程结束后的**“收尾大管家”**。
- 归档存储： 任务结束后，将完整的对话记录（包括所有中间思考过程）保存到向量数据库或日志系统中。
- 最终通知： 任务完成后，自动给开发者发送 Webhook 通知（如“订单处理 Agent 已完成操作”）。
- 交互优化： 检查 Agent 最终的回复，如果为空或由于异常中断，将其替换为更友好的兜底提示语：“对不起，我遇到了一点问题，请稍后再试”。
假设已经创建了 agent
```python
agent = create_agent(...)

```
### 1. 🟢 全局启动哨兵

```python
@agent.before_agent
def check_user_permission(state: AgentState):

    # 检查黑名单或扣除额度

    if state.user_id in BLACKLIST:
        raise PermissionError("用户被禁言")
    return state

```
### 2. 🟡 模型输入质检

```python
@agent.before_model
def inject_system_load(state: AgentState):

    # 每次模型调用前，动态修改上下文

    state.context.append("系统提示：当前负载较高，请尽量简短回答。")
    return state

```
### 3. 🟠 模型输出审计

```python
@agent.after_model
def audit_model_output(state: AgentState, response: ModelResponse):

    # 检查模型输出是否包含敏感密钥并打码

    response.content = response.content.replace("secret_key", "******")
    return response

```
### 4. 🔵 模型调用全生命周期 (Wrap 通常需要 yield 或特殊语法来包裹前后)

```python
@agent.wrap_model_call
def monitor_model_performance(state: AgentState, model_call_func):
    start_time = time.time()

    # 执行真正的模型调用

    response = model_call_func() 

    # 计算耗时

    print(f"模型调用耗时: {time.time() - start_time}秒")
    return response

```
### 5. 🟣 工具执行保护伞

```python
@agent.wrap_tool_call
def safe_tool_execution(state: AgentState, tool_name: str, tool_func):
    if tool_name == "delete_user":
        return "拦截：由于操作敏感，请联系人工处理"

    # 否则正常执行工具

    return tool_func()

```
### 6. 🔴 最终交付质检

```python
@agent.after_agent
def final_cleanup(state: AgentState, final_response: str):
    # 归档存储或发送 Webhook 通知
    send_webhook_notification("Agent 任务已完成")
    return final_response

```

---

## Langchain 使用 MCP 服务 (Model Context Protocol)：给 AI 打造“通用共享工具箱”

在传统的 Agent 开发中，工具（Tools）通常是和 Agent 绑死在一起的。
你可以把 MCP (Model Context Protocol) 理解为一种**“工具外包”或“共享工具箱”**机制。
它的核心思想是：把 Tools 换个地方单独定义并作为服务运行。 
这样一来，无论是哪个大模型（Claude、GPT、还是你自己写的 Agent），只要接入了这个 MCP 服务，就能直接调用里面的工具。

核心通信机制
MCP 底层使用的是标准的 JSON-RPC 2.0 协议进行对话。
它支持两种主流的通信方式：
1. STDIO (标准输入输出)：适用于本地调用（就像把工具箱放在自己电脑桌底下）。
2. HTTP / SSE (Server-Sent Events)：适用于远程调用（就像把工具箱放在云端服务器上）。

---

### 第一步：创建一个 MCP 服务端 (造一个工具箱)

使用 FastMCP 可以极快地把普通的 Python 函数包装成 MCP 服务。

调试小技巧：
安装 Node.js 后，可以通过 npx @modelcontextprotocol/inspector python "你的文件".py 启动可视化面板来测试你的工具。

```python
from mcp.server.fastmcp import FastMCP

# 1. 初始化 FastMCP (给工具箱起个名字)
mcp = FastMCP("MyWeatherService")

# 2. 定义工具 (把工具放进箱子)
@mcp.tool()
def query_weather(city: str) -> str:
    """根据城市名称查询天气。"""
    if "北京" in city:
        return "北京天气：晴，气温 25°C，微风。"
    elif "上海" in city:
        return "上海天气：阴，气温 22°C，局部有小雨。"
    else:
        return f"暂时查不到 {city} 的天气，可能那里是世外桃源吧。"

if __name__ == "__main__":
    # 默认启动为 STDIO 模式
    mcp.run()

    # 如果要启动为 HTTP/SSE 模式，改为：mcp.run(transport="sse")

```

---

### 第二步：客户端调用 MCP 服务 (两种场景)

场景 A：本地调用 (STDIO 模式)
适用场景： 工具脚本和 Agent 代码在同一台机器上运行。
核心优势： MultiServerMCPClient 可以一行代码替代所有繁琐的进程和 Session 管理。
在客户端代码中确实不需要声明 MyWeatherService 这个名字。是通过文件名来找到本地注册的mcp

```python
import os
import asyncio
from langchain.agents import create_agent
from langchain_openai import ChatOpenAI
from langchain_mcp_adapters.client import MultiServerMCPClient

async def main():
    # 1. 连接本地 MCP 服务 (通过命令行启动 Python 脚本)
    client = MultiServerMCPClient({
        "weather": {
            "transport": "stdio",
            "command": "python",
            "args": [os.path.abspath("你的文件.py")],
            "env": os.environ.copy()
        }
    })

    # 2. 核心：一键获取工具箱里的所有工具
    tools = await client.get_tools()

    llm = ChatOpenAI(...)

    # 3. 把拿到的 tools 直接喂给 Agent
    agent = create_agent(model=llm, tools=tools, system_prompt="...")

    response = await agent.ainvoke({"messages": [{"role": "user", "content": "北京天气怎么样？"}]})
    print(f"回答: {response['messages'][-1].content}")

if __name__ == "__main__":
    asyncio.run(main())

```

场景 B：远程调用 (HTTP / SSE 模式)
适用场景： 工具部署在独立的服务器上，Agent 通过网络去调用。
核心流程： 建立 SSE 连接 -> 初始化 Session -> 加载工具。

---

### 总结对比：STDIO vs SSE

---

## 父子 Agent 的调用机制与逻辑

在复杂的项目中，单一 Agent 往往无法处理所有任务，需要引入“父子 Agent（多智能体）”架构。
核心概念：在 LLM 底层，子 Agent 就是一个“工具”
在主 Agent（Main Agent）的视角里，根本不存在所谓的“子 Agent”，它只看到了一个普通的 Tool。
1. 主 Agent 的决策：当用户提出复杂需求时，主 Agent 的 LLM 会根据工具描述（subagent1_description），决定调用名为 subagent1_name 的工具。
2. 黑盒执行：主 Agent 将参数（如 query）传给这个工具。至于这个工具内部是跑了一段 Python 脚本，还是唤醒了另一个拥有完整思考能力的子 Agent，主 Agent 完全不关心。
3. 等待结果：主 Agent 停下来，等待这个“工具”返回执行结果。

---

### 代码解析：如何将子 Agent 包装成工具？

为了让主 Agent 能够调用子 Agent，
我们需要用 @tool 装饰器将子 Agent 的调用过程封装起来，并通过 Command 对象返回结果。

```python
from typing import Annotated
from langchain.tools import tool, InjectedToolCallId
from langchain_core.messages import ToolMessage
from langgraph.types import Command

# 1. 将子 Agent 包装成工具
@tool(
    "subagent1_name",
    description="当需要处理特定任务时调用此工具。参数: query(str)"
)
def call_subagent1(
    query: str,
    # 2. 自动注入工具调用 ID
    tool_call_id: Annotated[str, InjectedToolCallId], 
) -> Command:
    # 3. 真正触发子 Agent 的运行 (invoke)
    result = subagent1.invoke({
        "messages": [{"role": "user", "content": query}]
    })

    # 4. 将子 Agent 的结果打包返回给主 Agent
    return Command(update={
        # 更新主 Agent 的全局状态 (State)
        "example_state_key": result.get("example_state_key"),

        # 必须返回 ToolMessage，告诉主 Agent 工具执行完毕
        "messages": [
            ToolMessage(
                content=result["messages"][-1].content, # 提取子 Agent 的最终回复
                tool_call_id=tool_call_id               # 匹配主 Agent 的调用 ID
            )
        ]
    })
```

---

## 返回的信息给主 Agent

要让主 Agent 能够完美接收子 Agent 通过 Command(update={...}) 传回来的数据，主 Agent 需要做三件事：定义包含该变量的 State、绑定工具、构建执行图。
```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langgraph.prebuilt import ToolNode, tools_condition
from langchain_openai import ChatOpenAI

# 1. 定义主 Agent 的全局状态 (State)
# ==========================================
class MainAgentState(TypedDict):
    # 接收聊天记录（包括子 Agent 传回来的 ToolMessage）
    messages: Annotated[list, add_messages] 

    # 👈 核心对应点：这里必须预留字段，才能接收子 Agent 更新的结构化数据
    example_state_key: str 

# 2. 初始化大模型并绑定子 Agent 工具
# ==========================================
llm = ChatOpenAI(model="gpt-4", temperature=0)

# 将之前封装好的子 Agent 工具 (call_subagent1) 绑定给主 Agent 的 LLM
# 这样主 Agent 的大模型才知道自己有这个“工具”可用
llm_with_tools = llm.bind_tools([call_subagent1]) 

# 3. 定义主 Agent 的核心推理节点
# ==========================================
def main_agent_node(state: MainAgentState):
    # 主 Agent 在这里进行思考。
    # 如果上一步子 Agent 刚执行完，这里的 state["messages"] 里就会包含那个 ToolMessage。
    # LLM 看到 ToolMessage 后，就会根据子 Agent 的汇报生成最终回复。
    response = llm_with_tools.invoke(state["messages"])
    return {"messages": [response]}

# 4. 构建主 Agent 的工作流 (Graph)
# ==========================================
workflow = StateGraph(MainAgentState)

# 添加推理节点
workflow.add_node("agent", main_agent_node)

# 添加工具执行节点 (LangGraph 内置的 ToolNode 会自动处理工具的执行)
# 当 LLM 决定调用 call_subagent1 时，流程会流转到这个节点
tool_node = ToolNode(tools=[call_subagent1])
workflow.add_node("tools", tool_node)

# 设置边和路由逻辑
workflow.add_edge(START, "agent")

# tools_condition 会自动判断：如果 LLM 要求调用工具，就去 "tools" 节点；否则去 END
workflow.add_conditional_edges("agent", tools_condition)
workflow.add_edge("tools", "agent") # 工具执行完（子 Agent 返回后），回到主 Agent 继续思考

# 编译主 Agent
main_agent = workflow.compile()
```

## 常见问题

如何提升系统回复的准确率？
核心方法：把参考资料放在系统提示词（System Prompt）中。
优势与原因：
  1. 系统提示词权重更高，大模型会更严格地遵循。
  2. 当上下文太长时，系统提示词不会被压缩或遗忘。
  3. 利于前缀缓存（Prompt Caching），能加快响应速度并降低成本。
适用场景：你的整轮对话，强依赖这段“参考文本”。
批量提问的作用是什么？
核心作用：降低 Token 消耗，并且能够提升系统整体性能。
如何进一步优化回复质量？
核心方法：利用 logprobs（对数概率）来观察大模型生成内容的把握程度（置信度），
根据这些底层数据来针对性地优化 Prompt 或者补充上下文。
有什么具体的性能提升方案（针对多工具场景）？
核心方案：中间件利用小模型进行 Tools（工具）的初步筛选。例如，原本有 50 个工具，先用速度快、成本低的小模型筛选出最相关的 3 个，再交给大模型处理（即 50 => 3）。

```python
from langchain.agents import create_agent
from langchain.agents.middleware import LLMToolSelectorMiddleware

agent = create_agent(
    model="gpt-4o", # 主力大模型
    tools=[tool1, tool2, tool3, tool4, tool5, ...], # 这里原本可能有几十个工具
    middleware=[
        LLMToolSelectorMiddleware(
            model="gpt-4o-mini", # 使用小模型进行初步筛选
            max_tools=3,         # 限制最多选出3个最相关的工具
            always_include=["search"], # 设定必须包含的默认工具
        ),
    ],
)
```
