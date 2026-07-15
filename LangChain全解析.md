# LangChain 全解析

> 参考官方文档

https://docs.langchain.com/oss/python/langchain/install
langchain的文档，我带大家完全读一遍，下来之后，大家要脱离我，自己把langchian官方文档再读3遍，langchian会是面试重点。

## 阅读过程笔记

我直接记录重点，大家学习结合官网，和我的重点笔记。
```python
pip install -U langchain
# Requires Python 3.10+
```

例子： 如果新版 langchain 需要 langchain-core 达到 0.3.0，而你本地是 0.1.0，-U 会顺手把 langchain-core 也给升了。

```python
from dataclasses import dataclass
```

```python
from langchain.tools import tool, ToolRuntime

# --- 基础工具定义 ---
@tool
def get_weather_for_location(city: str) -> str:
```

    # 这是一个标准的原子工具，参数 'city' 由 LLM 根据用户输入自动提取并填充
```python
    return f"It's always sunny in {city}!"

# --- 运行时上下文定义 ---
@dataclass
class Context:
    """
    自定义运行时上下文架构
```

```python
    """
    user_id: str

```

```python
@tool
def get_user_location(runtime: ToolRuntime[Context]) -> str:
```

    # 注意：'runtime' 这个参数在发给 LLM 的描述（Schema）中会被自动隐藏
    # LLM 不知道这个参数的存在，因此它不会尝试去伪造一个 runtime 对象

    # 从注入的运行时对象中提取上下文数据
```python
    user_id = runtime.context.user_id
    
    # 模拟根据后端传入的真实用户 ID 返回对应的地理位置
```

在 Python 中，dataclass 是一个装饰器，它能让你用极简的代码定义一个纯数据类。
```python
@dataclass
class Context:
    user_id: str
```

为什么要用它？：
- 自动生成方法：它会自动帮你写好 init（构造函数）和 repr（打印信息）。
- 类型检查：在大型项目中，这能防止你把 user_id 错写成 uid。
- 不可变性支持：你可以设置 frozen=True，确保这些上下文数据在传输过程中不被恶意篡改。

```python
def get_user_location(runtime: ToolRuntime[Context]) -> str:
```

解决的核心痛点：LLM 的“越权”问题。
- 通常情况下，工具的所有参数（如 city）都要写在文档字符串（Docstring）里告诉 AI。
- 但 user_id 是敏感的隐私数据，绝对不能让 AI 知道或修改。
- 魔法所在：当 LangChain 看到参数类型是 ToolRuntime 时，它会从发给 AI 的“说明书”中删掉这个参数。

```python
from langchain.agents.structured_output import ToolStrategy

agent = create_agent(
    model=model,
    system_prompt=SYSTEM_PROMPT,
    tools=[get_user_location, get_weather_for_location],
    context_schema=Context,
    response_format=ToolStrategy(ResponseFormat),
    checkpointer=checkpointer
)

```

```python
config = {"configurable": {"thread_id": "1"}}

response = agent.invoke(
    {"messages": [{"role": "user", "content": "what is the weather outside?"}]},
    config=config,
    context=Context(user_id="1")
)

print(response['structured_response'])
# ResponseFormat(
```

#     weather_conditions="It's always sunny in Florida!"
# )

# Note that we can continue the conversation using the same `thread_id`.
```python
response = agent.invoke(
    {"messages": [{"role": "user", "content": "thank you!"}]},
    config=config,
    context=Context(user_id="1")
)

print(response['structured_response'])
# ResponseFormat(
```

#     weather_conditions=None
# )
### thread_id

它是 LangChain 内部预留的“插槽”，但具体的值必须由你的业务层来定义。
简单来说，thread_id 是 LangChain 规定的一种“协议标准”，而你负责提供**“具体的号码”**。
场景 A（客服机器人）：业务上通常把用户的 OpenID 或 手机号 作为 thread_id。
场景 B（多轮对话网页）：业务上通常在前端生成一个 UUID 传给后端作为 thread_id。
场景 C（内部自动化脚本）：你可以简单地用 "1"、"2"、"test_session"。
对 LangChain 而言：它不需要知道你的用户是谁。它只知道：只要你给我的 thread_id 是一样的，我就去数据库里把对应的历史记录翻出来。
对你的业务而言：你不需要手写 SELECT * FROM history WHERE user_id=...。你只需要把业务中的唯一标识符塞进这个“插槽”，剩下的持久化操作全部由 LangChain 自动化完成。

### ToolStrategy

它就像是一个**“强制翻译官”**，它的任务是：强制要求 LLM 必须以调用工具的方式来输出最终答案，即使它只是在跟你闲聊。
在 LangChain 的 structured_output 模块中，ToolStrategy 是一种具体的实现方案。它的逻辑逻辑是：
- 内部封装：它会将你定义的 ResponseFormat（一个 Pydantic 类或 BaseModels）转换成一个 “隐藏工具” 的描述。
- 强制绑定：它告诉模型：“你不许直接吐字符串！你必须通过调用这个叫 ResponseFormat 的工具来提交你的作业。”
如果你不使用 ToolStrategy，模型可能会返回：
“嘿！佛罗里达今天天气不错，是晴天。”
使用了 ToolStrategy 后，模型必须返回类似这样的结构：
```python
JSON
{
  "name": "ResponseFormat",
  "arguments": {
```

```python
    "weather_conditions": "Sunny"
  }
}
```

当你设置了 ToolStrategy(ResponseFormat)，Agent 的运行链路发生了微调：
1. 构造 Prompt：框架在发送给 AI 的 Prompt 末尾悄悄加上指令：“请使用 ResponseFormat 工具进行回复”。
2. 约束输出：利用 OpenAI/Claude 的 tool_choice 参数，强制模型进入工具调用模式。
3. 结果转换：模型吐出工具调用数据后，ToolStrategy 负责拦截这个调用，将其转换成 response['structured_response'] 供你直接读取。

```python
from langchain.agents import create_agent
from langchain.messages import SystemMessage, HumanMessage

literary_agent = create_agent(
    model="anthropic:claude-sonnet-4-5",
    system_prompt=SystemMessage(
        content=[
            {
                "type": "text",
```

```python
            },
            {
                "type": "text",
```

```python
                "cache_control": {"type": "ephemeral"}
            }
        ]
    )
)

result = literary_agent.invoke(
    {"messages": [HumanMessage("Analyze the major themes in 'Pride and Prejudice'.")]}
)
            {
                "type": "text",
```

```python
                "cache_control": {"type": "ephemeral"}
            }
```

主要有以下三个核心架构原因：
1.1 定义“真理边界”（Grounding）
- 用户提示词（User Message）：在 AI 看来是**“外部命令”**。用户可以说“忽略之前的指令”，这会导致 AI 产生偏移。
- 系统提示词（System Message）：在 AI 看来是**“世界观/底层规则”**。
- 实际效果：当你把《傲慢与偏见》放在系统消息里，你其实是在告诉 AI：“你现在不是一个通用 AI，你是一个基于这段文字存在的世界里的分析专家。”这能显著提高 AI 回答的忠实度，减少“胡说八道”。
1.2 触发“前缀缓存”的物理限制
这是最硬核的技术原因。几乎所有支持 Prompt Caching 的服务商（包括你关注的火山引擎、Anthropic、DeepSeek）都遵循一个原则：
- 缓存只从最顶端开始。
- 用户提示词是变动的（每轮问题都不同），如果小说放在用户消息里，随着对话轮数增加，前面的对话历史会不断挤压或改变位置。
- 系统提示词是固定的。把它放在最顶层，能确保这段巨大的文本始终占据 Prompt 的起始位置，从而让每一轮对话都能完美命中缓存，节省 90% 的成本。
1.3 隔离“多轮对话”的上下文
- 如果小说在用户消息里：它会被视为对话历史的一部分。在多轮对话中，它会随着 MessagesState 被反复传递，甚至可能在“总结历史记录”时被 AI 给精简掉（因为它太长了）。
- 如果小说在系统消息里：它是独立于对话历史的“背景板”。无论你和 AI 聊了 50 轮还是 100 轮，AI 都能始终如一地引用系统消息里的全文。

### Log probabilities

model = init_chat_model(
```python
    model="gpt-4o",
    model_provider="openai"
).bind(logprobs=True)

response = model.invoke("Why do parrots talk?")
print(response.response_metadata["logprobs"]
```

这属于 大模型原理与质量评测 的进阶技能，主要有以下三个用途：
1. 检测“幻觉”（Hallucination Detection）： 如果模型在回答某个关键事实（比如日期、人名）时，对应的 logprobs 数值非常低（比如只有 20% 的置信度），说明模型在“瞎编”的可能性很大。你可以据此在后台做逻辑判断：当置信度低于 50% 时，提示用户“该结果仅供参考”。
2. 提示词工程（Prompt Engineering）的优化： 你可以对比两种提示词。如果 A 提示词下模型输出词汇的概率都很稳定（高分），而 B 提示词下概率起伏很大（低分），说明 A 提示词对模型的引导更清晰。
3. 采样策略研究： 这能帮你理解 Temperature（温度）参数是如何影响模型选择的。

### BaseModel 和 ToolStrategy 的区别？不都是定义数据结构吗

但从架构角色上来说，BaseModel 是**“静模”（数据的样子），而 ToolStrategy 是“动效”**（获取数据的手段）。
我们可以用**“填表”**来做个形象的类比：
BaseModel：空白的表格（数据定义）
BaseModel（通常来自 Pydantic）仅仅定义了数据应该长什么样。
- 它是静态的：它只是一张带有字段名（如 name, age）和类型（如 str, int）的空白表格。
- 它的作用：校验。如果你往表里填了错误的数据，它会报错。
- 代码表现：
```python
class ResponseFormat(BaseModel):
    answer: str
    confidence: float
```

ToolStrategy：强迫 AI 填表的“行政手段”（交互策略）
ToolStrategy 是 LangChain 中的一个动作逻辑，它利用了 LLM 的“工具调用”能力。
- 它是动态的：它是一个**“策略”**。它告诉 Agent：“在这次任务结束时，你必须通过‘调用工具’这个动作来把那张 BaseModel 表格填好给我。”
- 它的作用：执行。它把原本用于搜索、计算的“工具协议”，借用来作为“输出协议”。
- 代码表现：
response_format = ToolStrategy(ResponseFormat)
这一行代码的背后，LangChain 做了两件事：
1. 自动把 ResponseFormat 包装成一个名为 ResponseFormat 的虚拟工具。
2. 在发送给 AI 的请求中，强制设置 tool_choice="ResponseFormat"。

```python
@tool("calculator", description="Performs arithmetic calculations. Use this for any math problems.")
def calc(expression: str) -> str:
    """Evaluate mathematical expressions."""
    return str(eval(expression))
```

"""使用 Python 的 eval 环境计算算术表达式。支持 + - * / 等操作。注意：由于使用了 eval，请确保输入来源安全。"""
Decorator Description（给 AI）：写得带有“诱导性”，明确告诉它什么时候该用。
```python
@tool(description="当你遇到任何数学计算、加减乘除、百分比转换时，必须调用我。")

human_msgs = sum(1 for m in messages if isinstance(m, HumanMessage))
human_msgs = 0
for m in messages:
    if isinstance(m, HumanMessage):
        human_msgs = human_msgs + 1  # 看到一个目标，就加 1
from langchain.agents import create_agent
from langchain.agents.middleware import SummarizationMiddleware
from langgraph.checkpoint.memory import InMemorySaver
from langchain_core.runnables import RunnableConfig

checkpointer = InMemorySaver()

agent = create_agent(
    model="gpt-4o",
    tools=[],
    middleware=[
        SummarizationMiddleware(
            model="gpt-4o-mini",
            trigger=("tokens", 4000),
            keep=("messages", 20)
        )
    ],
    checkpointer=checkpointer,
)

```

agent.invoke({"messages": "hi, my name is bob"}, config)
agent.invoke({"messages": "write a short poem about cats"}, config)
agent.invoke({"messages": "now do the same but for dogs"}, config)
```python
final_response = agent.invoke({"messages": "what's my name?"}, config)

final_response["messages"][-1].pretty_print()
"""
================================== Ai Message ==================================

Your name is Bob!
"""
SummarizationMiddleware(
```

```python
    trigger=("tokens", 4000), # 2. 触发条件：当对话总 Token 超过 4000 时启动压缩
```

)
### stream 中的 stream_mode

1. updates (你代码中正在使用的模式)
这是最常用、最推荐的 Agent 调试模式。
- 逻辑：每当图中有一个节点（Node）执行完毕并产生了状态更新时，就推送一次数据。
- 内容：它只推送被修改的那部分 State。
- 你的代码表现：你会先看到 agent 节点输出一个 ToolCall，然后看到 tools 节点输出 get_weather 的结果，最后是 agent 节点输出最终回复。

---
2. values (全量模式)
- 逻辑：每当状态发生变化，它会推送整个 State 对象。
- 特点：虽然数据量大，但对前端很友好，因为你不需要自己去做“状态合并”。每一帧你拿到的都是当前对话最完整、最新的样子。

---
3. messages (消息流模式)
- 逻辑：专门为对话场景优化。它不仅在节点结束时推送，还会流式推送（Streaming） AI 正在生成的每一个字（Token）。
- 场景：如果你想要像 ChatGPT 那样“一个字一个字蹦出来”的效果，必须用这个模式。
- 代码差异：此时 data 里包含的是消息碎片（Message Chunks）。

---
4. debug (开发者模式)
- 逻辑：极其详细。它会推送所有内部细节，包括每个节点的输入、输出、甚至是在执行哪些内部函数。
- 用途：只在开发环境下排查“为什么 Agent 逻辑卡住了”时使用。

## 中间件

### PII 检测

```python
from langchain.agents import create_agent
from langchain.agents.middleware import PIIMiddleware

agent = create_agent(
    model="gpt-4o",
    tools=[],
    middleware=[
        PIIMiddleware("email", strategy="redact", apply_to_input=True),
        PIIMiddleware("credit_card", strategy="mask", apply_to_input=True),
    ],
)
PIIMiddleware("email", strategy="redact", apply_to_input=True)
```

- strategy="redact" (擦除): 这是一个“物理抹除”策略。
  - 效果: 如果用户输入 My email is bob@example.com，发送给 AI 的内容会变成 My email is [REDACTED]。
- apply_to_input=True: 明确作用范围。这意味着在数据离开你的服务器、发往模型之前就进行脱敏。

PIIMiddleware("credit_card", strategy="mask", apply_to_input=True)
- "credit_card": 识别信用卡号。
- strategy="mask" (遮掩): 这是一个“部分可见”策略。
  - 效果: 信用卡号会变成 ************4444。这样 AI 知道这是一个卡号，但拿不到完整的真实号码。
有时候，你的工具（Tool）可能真的需要那个真实的邮箱去发邮件。 由于中间件只作用于 Agent -> LLM 的路径，你的本地工具函数依然可以通过 state 拿到原始的、未脱敏的信息。
总结： 这套机制实现了：把隐私留给自己的代码处理，把逻辑交给 AI 处理。

### 工具仿真

```python
from langchain.agents import create_agent
from langchain.agents.middleware import LLMToolEmulator

agent = create_agent(
    model="gpt-4o",
    tools=[get_weather, search_database, send_email],
    middleware=[
```

```python
    ],
)
```

你正在开发前端界面，但后端的 send_email 接口还没写好，或者 search_database 权限还没申请下来。
- 用法：挂载 LLMToolEmulator。
- 效果：你的 Agent 表现得就像功能全开一样，你可以完整测试对话流，而不需要等待真实的接口开发。
B. 离线演示 (Safe Demo)
你要给客户演示一个“自动删除数据库”的 Agent，但你不敢在演示现场真的删数据。
- 用法：模拟器会让 AI 认为删除成功了，并继续进行下一步，而你的真实数据库纹丝不动。
C. 生成合成数据 (Synthetic Data Generation)
你需要模拟 100 组用户和 AI 调工具的对话历史用来训练更小的模型。
- 用法：通过模拟器，你可以极其廉价地生成大量“模拟工具调用”的对话样本，省去了真实调 API 的费用。

### 钩子生命周期

before_agent- 代理开始前（每次调用一次）
before_model- 每次模型调用前
after_model- 每次模型响应后
after_agent- 代理完成后（每次调用一次）

这4个钩子的执行顺序？
before_agent：起点。整个任务刚开始，此时 AI 还没开口，工具也还没准备。
before_model：循环入口。AI 准备生成回复前的最后一步。
after_model：循环出口。AI 刚说完话，在决定是否要调工具之前。
after_agent：终点。AI 彻底交卷，任务结束。
🚀 before_agent (执行 1 次)
- 🌀 第一轮循环开始
- 👉 before_model
- 🤖 模型推理 (Model Inference)
- 👈 after_model
- 🛠️ 执行工具 (Tool Execution)
- 🌀 第二轮循环开始
- 👉 before_model
- 🤖 模型推理 (Model Inference)
- 👈 after_model
- 🏁 判断任务已完成
🏁 after_agent (执行 1 次)

### 二次确认

```python
def my_tool_wrapper(tool_call, next_call):
    if tool_call.name == "delete_database":
```

```python
        if not user_confirmed():
```

```python
    return next_call(tool_call) # 只有确认了才执行下一步
```

### 自定义中间件

### return 和 break 的区别？

```python
def test_break():
    for i in range(3):
        if i == 1:
            print("  [break触发]")
            break
        print(f"  循环中: {i}")
    print("循环结束，但我还在函数里，继续干活！")

```

#   循环中: 0
#   [break触发]
# 循环结束，但我还在函数里，继续干活！

```python
from langchain.agents.middleware import after_model, hook_config, AgentState
from langchain.messages import AIMessage
from langgraph.runtime import Runtime
from typing import Any

@after_model
@hook_config(can_jump_to=["end"])
def check_for_blocked(state: AgentState, runtime: Runtime) -> dict[str, Any] | None:
    last_message = state["messages"][-1]
    if "BLOCKED" in last_message.content:
        return {
```

```python
            "jump_to": "end"
        }
    return None
```

'tools'立即行动goto 执行层绕过后续可能存在的模型判断，强迫系统进入工具执行节点。
'model'重新思考continue 重新开始丢弃或修改当前状态后，让模型从头（before_model）再想一遍。
### 守卫

```python
@hook_config
can_jump_to: list[str] (最重要的配置)
```

- 含义：声明该钩子允许跳转到的目标节点名称。
- 默认值：空列表 []（即不允许跳转）。
- 常用值：
  - "end"：强制结束 Agent。
  - "tools"：跳过思考，直接去执行工具。
  - "model"：忽略当前输出，让模型重写。
- 作用：如果你在代码里返回了 {"jump_to": "..."} 但没在 can_jump_to 里声明，程序会直接报错。
is_async: bool | None
这个配置决定了中间件的运行模式。
- 含义：显式指定这个钩子是同步执行还是异步执行。
- 默认值：None（框架会尝试通过 inspect.iscoroutinefunction 自动推断）。
- 为什么要手动设为 True/False？：
  - 在某些复杂的包装场景下（比如你用了装饰器的装饰器），Python 的自动推断可能会失效。
  - 手动设置可以强制框架以特定方式调度任务，提高运行效率。
apply_to_input / apply_to_output: bool (针对特定策略)
虽然这两个参数在基础中间件中不常用，但在处理数据转换类（如 PII 脱敏或缓存）的钩子中非常关键。
- 含义：指定中间件的逻辑是作用于“发送给模型的原始数据”还是“模型返回的结果”。
- 场景：
  - 如果你在写一个缓存钩子，你需要 apply_to_input=True 来根据输入生成缓存 Key。
  - 如果你在写一个输出清理钩子，你需要 apply_to_output=True。
apply_to_input=True：去程拦截（发给 AI 之前）
核心逻辑：在用户的提问（或工具结果）发往大模型（LLM）之前，先过一遍中间件。
🚀 应用场景 A：隐私脱敏 (PII Protection)
- 需求：用户输入了“我的手机号是 13800138000”，你必须在发给 OpenAI 之前把它遮住。
- 配置：apply_to_input=True。
- 结果：中间件把手机号改成 [REDACTED]。AI 收到的是脱敏后的文字。
🚀 应用场景 B：动态上下文注入 (Context Injection)
- 需求：根据用户的问题，自动从数据库里查出他的会员等级，并塞进 Prompt。
- 配置：apply_to_input=True。
- 结果：中间件在输入端增加了“当前用户是金卡会员”的信息，AI 根据这个新背景进行回答。
apply_to_output=True：回程拦截（从 AI 回来之后）
核心逻辑：AI 已经生成了答案，但在最终显示给用户或执行工具之前，先过一遍中间件。
🚀 应用场景 C：内容安全审计 (Safety Guardrail)
- 需求：防止 AI 在回复中产生幻觉，说出带有歧视性或违禁的词汇。
- 配置：apply_to_output=True。
- 结果：如果 AI 回复了包含违禁词的内容，中间件直接截获，将其替换为“由于政策原因，我无法回答此问题”，并触发 jump_to: "end"。
🚀 应用场景 D：输出格式化转换 (Post-Processing)
- 需求：AI 返回了一个 JSON 字符串，你希望在显示给前端之前，把它转换成一个漂亮的 HTML 表格。
- 配置：apply_to_output=True。
- 结果：中间件处理 AI 的输出，完成格式转换，用户直接看到结果。

### 上下文

```python
agent = create_agent(
    model="gpt-4o",
    tools=[save_preference],
    context_schema=Context,
    store=InMemoryStore()
)
```

[图片]

相同点：都是在内存中操作。
不同点：
context属于单次有效。（类似 在全局中写了个对象，每次调用点赋值，调用完成销毁。）
```python
      store属于长期存在内存中，只要服务不重启，状态就在。（类似 session Storage）
```

### 父子 Agent 的调用

```python
from typing import Annotated
from langchain.agents import AgentState
from langchain.tools import InjectedToolCallId
from langgraph.types import Command

@tool(
    "subagent1_name",
    description="subagent1_description"
)
def call_subagent1(
    query: str,
    tool_call_id: Annotated[str, InjectedToolCallId],
) -> Command:
    result = subagent1.invoke({
```

```python
    })
    return Command(update={
```

```python
        "example_state_key": result["example_state_key"],
        "messages": [
            ToolMessage(
                content=result["messages"][-1].content,
                tool_call_id=tool_call_id
            )
        ]
    })
    return Command(update={
```

```python
        "example_state_key": result["example_state_key"],
        "messages": [
            ToolMe    ssage(
                content=result["messages"][-1].content,
                tool_call_id=tool_call_id
            )
        ]
    })
告诉mainAgent
```

2. 返回的信息是什么
在LLM底层，subAgent相当于是工具调用

## 面试题

1. 如何提升系统回复的准确率？
把参考资料放系统提示词
1. 系统提示词权重更高
2. 上下文太长，不会压缩
3. 利于前缀缓存
场景：你的整轮对话，强依赖这段“参考文本”

2. 批量提问的作用？
1. 降低token消耗
2. 提升性能

3. 如何优化回复质量
利用logprobs看大模型的把握程度，优化prompt或者上下文

4. 性能提升方案
利用小的模型进行tools筛选，50= > 3
```python
from langchain.agents import create_agent
from langchain.agents.middleware import LLMToolSelectorMiddleware

agent = create_agent(
    model="gpt-4o",
    tools=[tool1, tool2, tool3, tool4, tool5, ...],
    middleware=[
        LLMToolSelectorMiddleware(
            model="gpt-4o-mini",
            max_tools=3,
            always_include=["search"],
        ),
    ],
)
```
