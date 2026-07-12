# Langchain2.0

## 注意：langchain部分

优先看红色部分，langchain整体难度偏难，红色部分建议看3遍再说。
看3遍，听懂了，绿色部分是旧版，可以不看。
如果3遍，感觉还有点欠缺，再把旧版刷一遍。
[图片]

## 代码仓库

```
https://gitee.com/mood6666/langchian2.0
```

官方Ai问答 langchain

```
https://chat.langchain.com/?threadId=d8b35b42-0217-439b-a006-de1fed7cd28f
```

## 安装环境

```
conda create -n langchain python=3.13
```

```
conda activate langchain 
```

```
conda install langchain 
conda install langchain-openai
```

```
# 导出依赖文件
conda env export > environment.yml
```

```
# 同步conda依赖
conda env update -f environment.yml --prune
```

将编辑器的python环境切换到对应conda环境

```
Ctrl + shift + p
```

[图片]

## 入门案例 Main.py

```python
import os
from dataclasses import dataclass
from langchain.agents import create_agent
from langchain.tools import tool, ToolRuntime
from langgraph.checkpoint.memory import InMemorySaver
from langchain.agents.structured_output import ToolStrategy
from langchain_openai import ChatOpenAI
from dotenv import load_dotenv

load_dotenv()
api_key = os.getenv('ARK_API_KEY')

# Define system prompt
SYSTEM_PROMPT = """You are an expert weather forecaster, who speaks in puns.

You have access to two tools:

- get_weather_for_location: use this to get the weather for a specific location
- get_user_location: use this to get the user's location

If a user asks you for the weather, make sure you know the location. If you can tell from the question that they mean wherever they are, use the get_user_location tool to find their location."""

# Define context schema
@dataclass
class Context:
    """Custom runtime context schema."""
    user_id: str

# Define tools
@tool
def get_weather_for_location(city: str) -> str:
    """Get weather for a given city."""
    return f"It's always sunny in {city}!"

@tool
def get_user_location(runtime: ToolRuntime[Context]) -> str:
    """Retrieve user information based on user ID."""
    user_id = runtime.context.user_id
    return "Florida" if user_id == "1" else "SF"

api_key = os.environ["ARK_API_KEY"]
# 直接传入参数，无需os.environ
llm = ChatOpenAI(
    model="doubao-seed-1-6-251015",
    temperature=0,
    api_key=api_key,                    # 👈 直接传入API Key
    base_url="https://ark.cn-beijing.volces.com/api/v3",  # 👈 直接传入Base URL
)

# Define response format
@dataclass
class ResponseFormat:
    """Response schema for the agent."""
    # A punny response (always required)
    punny_response: str
    # Any interesting information about the weather if available
    weather_conditions: str | None = None

# Set up memory
checkpointer = InMemorySaver()

# Create agent
agent = create_agent(
    model=llm,
    system_prompt=SYSTEM_PROMPT,
    tools=[get_user_location, get_weather_for_location],
    context_schema=Context,
    response_format=ToolStrategy(ResponseFormat),
    checkpointer=checkpointer
)

# Run agent
# `thread_id` is a unique identifier for a given conversation.
config = {"configurable": {"thread_id": "1"}}

response = agent.invoke(
    {"messages": [{"role": "user", "content": "what is the weather outside?"}]},
    config=config,
    context=Context(user_id="1")
)

print(response['structured_response'])
# ResponseFormat(
#     punny_response="Florida is still having a 'sun-derful' day! The sunshine is playing 'ray-dio' hits all day long! I'd say it's the perfect weather for some 'solar-bration'! If you were hoping for rain, I'm afraid that idea is all 'washed up' - the forecast remains 'clear-ly' brilliant!",
#     weather_conditions="It's always sunny in Florida!"
# )

# Note that we can continue the conversation using the same `thread_id`.
response = agent.invoke(
    {"messages": [{"role": "user", "content": "thank you!"}]},
    config=config,
    context=Context(user_id="1")
)

print(response['structured_response'])
# ResponseFormat(
#     punny_response="You're 'thund-erfully' welcome! It's always a 'breeze' to help you stay 'current' with the weather. I'm just 'cloud'-ing around waiting to 'shower' you with more forecasts whenever you need them. Have a 'sun-sational' day in the Florida sunshine!",
#     weather_conditions=None
# )

agent = create_agent(
    model=llm,
    system_prompt=SYSTEM_PROMPT,
    tools=[get_user_location, get_weather_for_location],
    context_schema=Context,
    response_format=ToolStrategy(ResponseFormat),
    checkpointer=checkpointer
)
```

Checkpointer 存储的是整个 State（状态） 的快照。这个“状态”通常包含：
1. 聊天记录 (Messages)：你和 AI 说的每一句话。
2. 上下文变量 (Context)：这就是由 Context Schema 定义的部分。
3. 内部节点位置：Agent 现在跑到了流程图的哪一步。

## langChian的核心组件

1. Agent
2. Llm
3. Message
4. Tools
5. 记忆
6. Stream流式输出
7. 结构化输出

## 什么是langchain？

1. 国外的
2. 是目前搞Agent最火的框架
3. Langchain 2025年10月份，升级了一个大版本 1.0
  1. 1.0的版本和以前的老版本 0.x完全是两个东西。

## Agent部分

### 01.py Hello world

Agent的返回消息格式。

```
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
{
    "messages": [
        "content='你是谁？' additional_kwargs={} response_metadata={} id='1952a4d9-6f93-4ba7-bd7d-46fe1d42393b'",
        "content='我是智能助手小宁，很高兴能为你提供帮助~' additional_kwargs={'refusal': None} response_metadata={'token_usage': {'completion_tokens': 85, 'prompt_tokens': 46, 'total_tokens': 131, 'completion_tokens_details': {'accepted_prediction_tokens': None, 'audio_tokens': None, 'reasoning_tokens': 71, 'rejected_prediction_tokens': None}, 'prompt_tokens_details': {'audio_tokens': None, 'cached_tokens': 0}}, 'model_provider': 'openai', 'model_name': 'doubao-seed-1-6-251015', 'system_fingerprint': None, 'id': '0217684493419497d8ec4d988cc7ba637a273ca7a40c11bd8c5d4', 'service_tier': 'default', 'finish_reason': 'stop', 'logprobs': None} id='lc_run--019bbfcb-77f6-72f3-94e1-840b598f04dd-0' tool_calls=[] invalid_tool_calls=[] usage_metadata={'input_tokens': 46, 'output_tokens': 85, 'total_tokens': 131, 'input_token_details': {'cache_read': 0}, 'output_token_details': {'reasoning': 71}}",
    ]
}
```

### 1. 基础识别属性

这部分决定了这条消息“是谁发的”以及“怎么找到它”。
- content: 核心内容。这就是模型真正说出来的话。
- id: 唯一标识符。系统为每一条消息生成的“身份证号”，用于在复杂的图流程（LangGraph）中精准回溯。
- type: 消息类型。在你的输出中通过类名体现（HumanMessage 代表用户，AIMessage 代表 AI）。
- additional_kwargs: 额外参数。通常存放一些特定供应商（如 OpenAI 或 字节）特有、但在 LangChain 标准字段中没地方放的数据。

---

### 2. usage_metadata (消耗统计属性)

这部分直接关系到你的账单，记录了这次对话消耗的资源。
- input_tokens: 输入 Token 数。你问的问题转化成模型能理解的单位后的数量。
- output_tokens: 输出 Token 数。AI 回答生成的单位数量。
- total_tokens: 总消耗。输入 + 输出的总和。
- input_token_details: 输入明细。
  - cache_read: 是否读取了之前的缓存（如果有缓存，通常会便宜很多）。
- output_token_details: 输出明细。
  - reasoning: 推理 Token。这是像“豆包”或“DeepSeek”这类带有思考过程的模型特有的。它记录了模型在给出最终答案前，在后台“思考”了多久。这部分虽然不一定显示在最终正文里，但通常也是计费的。

---

### 3. response_metadata (响应详情属性)

这是由大模型供应商返回的原始信息“快照”。
- model_provider: 供应商。显示 openai，虽然你用的是豆包，但因为你用了 langchain-openai 的适配器，它会按这个格式上报。
- model_name: 具体的模型型号。这里显示你调用的具体版本（doubao-seed-1-6-251015）。
- service_tier: 服务层级。default 代表标准服务速度。
- finish_reason: 结束原因。stop 是最理想的状态，代表模型完整地表达完了它的意思，而不是因为字数限制被掐断了。
- id: 底层 ID。供应商侧生成的请求 ID，方便联系客服或查后台日志。

---

### 4. 工具与状态属性（用于 Agent 逻辑）

当你的 Agent 变得复杂时，这几个字段会变得非常重要。
- tool_calls: 工具调用指令。如果 Agent 觉得需要查天气，这里会存放 {"name": "get_weather", "args": {...}}。
- invalid_tool_calls: 非法工具调用。如果模型想用工具但格式写错了，这里会记录错误。
- refusal: 拒绝回复。如果模型认为你的问题违反了安全策略，它可能会在这里写下拒绝的原因。
- logprobs: 对数概率。记录模型生成每个字时的“信心指数”，一般用于学术研究或极高精度的控制，默认通常为 None。

### 02.py 动态选择LLM

```
@wrap_model_call
def dynamic_model_selection(request: ModelRequest, handler) -> ModelResponse:
    """动态模型选择"""
    # ✅ 修复1: 用 request.messages，不是 request.state.get()
    messages = request.messages
    message_count = len(messages)
```

```
    print(f"\n[中间件] 消息数: {message_count}")
```

```
    if message_count > 6:
        print("🚀 高级模型")
        model = advanced_model
    else:
        print("☁️ 基础模型")
        model = basic_model
```

```
    # ✅ 修复2: request.override(model=model)
    return handler(request.override(model=model))
```

### request包含哪些内容？

1. request.messages (最常用)
这是最重要的属性，包含当前对话的完整消息列表。
- 内容：一个包含 HumanMessage、AIMessage 等对象的列表。
- 用途：你代码中判断 len(messages) 就是在统计这个列表的长度，以此决定对话的复杂度。
2. request.model
这是原始打算使用的模型对象。
- 内容：你在 create_agent(model=llm, ...) 中传入的那个 llm 对象。
- 用途：中间件可以读取它，也可以通过 request.override(model=new_model) 来替换它。
3. request.tools
这是 Agent 此时可以调用的工具列表。
- 内容：你在创建 Agent 时定义的 tools 数组。
- 用途：你可以根据请求的内容动态决定是否禁用某些工具，或者在高级模型下开启更多复杂工具。
4. request.state
这是当前的状态字典。
- 内容：除了消息之外的其他状态数据（例如：用户 ID、当前处理步骤、临时变量等）。
- 用途：如果你在 invoke 时传入了除了 messages 以外的其他键值对，它们通常会出现在这里。
5.其他控制参数 (如 request.config)
包含运行时的配置信息。
- 内容：例如 recursion_limit（递归深度限制）、configurable（可配置项）等。

### 03.py 动态选择提示词

```
@dynamic_prompt
def user_role_prompt(request: ModelRequest) -> str:
```

```
    request_dict = vars(request)
    print(json.dumps(request_dict, indent=2, ensure_ascii=False, default=lambda x: str(x)))
    print("----------------------------------\n")
```

```
    """根据用户角色生成动态系统提示词"""
    # 💡 关键：从 request.runtime.context 获取 invoke 时传入的参数
    # 注意：在某些版本中，路径可能是 request.context 或 request.state["context"]
    # 按照你提供的参考片段，使用 request.runtime.context
    context = getattr(request.runtime, "context", {})
    user_role = context.get("user_role", "common")
```

```
    base_prompt = "你是智能助手小宁。"
```

```
    if user_role == "expert":
        print("--- [动态提示词] 当前角色：专家模式 (提供深度技术回答) ---")
        return f"{base_prompt} 你现在是资深技术专家，请提供详细、深度的专业回答。"
    elif user_role == "beginner":
        print("--- [动态提示词] 当前角色：小白模式 (解释通俗易懂) ---")
        return f"{base_prompt} 你现在是启蒙老师，请用最通俗易懂、大白话的方式解释概念。"
```

```
    return base_prompt
```

### 04.py 结构化输出

[图片]
用提示词输出json： 
类似： 你给一个大的题目，让Ai写作文。

用ToolStrategy
类似： 完形填空，让Ai填空。

### 05.py 流式输出

[图片]
chunk长啥样？

```
{
  "messages": [
    {
      "role": "user",
      "content": "罗永浩帅吗？",
      "id": "17ddd5bc-c376-49a6-ba80-0e23099a8b64",
      "additional_kwargs": {}
    },
    {
      "role": "assistant",
      "content": "外貌评价是比较主观的事情，不同人可能有不同看法~ \n\n罗永浩作为公众人物，更多被大家关注的是他的创业经历、演讲风格以及对产品的思考。他身上那种敢想敢做的冲劲、表达时的坦诚与幽默，还有在挫折中坚持的韧性，或许是更能打动很多人的“魅力点”。如果单论外貌，有人会觉得他有种独特的“直男式真诚感”，也有人可能不那么关注这方面，毕竟比起颜值，他的故事和观点往往更有讨论度呀~",
      "id": "lc_run--019bc4ab-94b4-7812-be58-b6861351a473-0",
      "model_info": {
        "model_name": "doubao-seed-1-6-251015",
        "provider": "openai",
        "finish_reason": "stop"
      },
      "usage": {
        "prompt_tokens": 48,
        "completion_tokens": 388,
        "total_tokens": 436,
        "reasoning_tokens": 265
      },
      "tool_calls": [],
      "invalid_tool_calls": []
    }
  ]
}
```

### 06.py stream控制台打字机效果

```
import json
import os
from langchain_openai import ChatOpenAI
from dotenv import load_dotenv
```

```
load_dotenv()
api_key = os.getenv("ARK_API_KEY")
```

```
SYSTEM_PROMPT = "你是智能助手小宁"
```

```
llm = ChatOpenAI(
    model="doubao-seed-1-6-251015",
    temperature=0,
    api_key=api_key,
    base_url="https://ark.cn-beijing.volces.com/api/v3",
)
```

```
print("LLM streaming (typing effect):")
# Direct LLM stream for token-by-token
messages = [
    {"role": "system", "content": SYSTEM_PROMPT},
    {"role": "user", "content": "罗永浩帅吗？"}
]
for chunk in llm.stream(messages):
```

```
    # print("----------------------------------\n")
    # print(json.dumps(chunk, indent=2, ensure_ascii=False, default=lambda x: str(x)))
```

```
    if chunk.content:
        print(chunk.content, end='', flush=True)
```

```
print("\n\nAgent full response (non-streaming):")
```

### 核心是这里生效的

```
    if chunk.content:
        print(chunk.content, end='', flush=True)
end=''：默认情况下，Python 的 print() 会在末尾加上回车 \n。设置为空字符串后，下一次打印的内容会紧接着上一次的内容，而不会跳到下一行。
flush=True：通常为了性能，Python 会把要打印的内容先存入缓存。设置为 True 会强制程序立即把这个字显示在屏幕上，而不是等缓存满了才一起显示。
```

### 07.py 工具调用

### 调用过程

```
{
  "messages": [
    {
      "role": "user",
      "content": "今天什么天气？",
      "id": "f5326f57-dd8e-4761-891c-2748077222cb"
    },
    {
      "role": "assistant",
      "content": "",
      "tool_calls": [
        {
          "id": "call_541xwtzjbr8l1vxfoxacyp3b",
          "name": "get_user_location",
          "args": {}
        }
      ],
      "metadata": {
        "reasoning_tokens": 102,
        "model": "doubao-seed-1-6-251015",
        "finish_reason": "tool_calls"
      }
    },
    {
      "role": "tool",
      "name": "get_user_location",
      "tool_call_id": "call_541xwtzjbr8l1vxfoxacyp3b",
      "content": "Florida"
    },
    {
      "role": "assistant",
      "content": "",
      "tool_calls": [
        {
          "id": "call_l2z7dhwfqz3py2n2d7hn1pnn",
          "name": "get_weather_for_location",
          "args": {
            "city": "Florida"
          }
        }
      ],
      "metadata": {
        "reasoning_tokens": 103,
        "model": "doubao-seed-1-6-251015",
        "finish_reason": "tool_calls"
      }
    },
    {
      "role": "tool",
      "name": "get_weather_for_location",
      "tool_call_id": "call_l2z7dhwfqz3py2n2d7hn1pnn",
      "content": "It's always sunny in Florida!"
    },
    {
      "role": "assistant",
      "content": "Looks like Florida’s serving up a side of sunshine today—no need to “cloud” your plans with rain worries! The sky’s as clear as can be, so go out and soak up those rays (but don’t get too “ray-diant” without slathering on sunscreen first!). It’s always sunny there, so you can bet your umbrella will be taking a well-deserved day off! ☀️😉",
      "metadata": {
        "reasoning_tokens": 330,
        "total_tokens": 1040,
        "finish_reason": "stop"
      }
    }
  ]
}
```

### 08.py 流式输出工具调用过程

```python
import os
from dataclasses import dataclass
from langchain.agents import create_agent
from langchain.tools import tool, ToolRuntime
from langchain_openai import ChatOpenAI
from dotenv import load_dotenv
import json

load_dotenv()
api_key = os.getenv('ARK_API_KEY')

SYSTEM_PROMPT = """你是一位精通天气预报的专家，说话喜欢用谐音梗或幽默的口吻。

你可以使用以下两个工具：
- get_weather_for_location: 用于查询特定城市的天气。
- get_user_location: 用于获取当前用户所在的地理位置。

如果用户询问天气，请确保你知道具体地点。如果从问题中可以看出用户指的是他们所在地，请先使用 get_user_location 工具获取位置。"""

@dataclass
class Context:
    """自定义运行时上下文架构。"""
    user_id: str

@tool
def get_weather_for_location(city: str) -> str:
    """获取给定城市的天气情况。"""
    return f"{city} 天气晴朗，阳光明媚，非常适合出门玩耍！"

@tool
def get_user_location(runtime: ToolRuntime[Context]) -> str:
    """根据用户 ID 检索用户信息。"""
    user_id = runtime.context.user_id
    return "佛罗里达" if user_id == "1" else "旧金山"

llm = ChatOpenAI(
    model="doubao-seed-1-6-251015",
    temperature=0,
    api_key=api_key,
    base_url="https://ark.cn-beijing.volces.com/api/v3",
)

agent = create_agent(
    model=llm,
    system_prompt=SYSTEM_PROMPT,
    tools=[get_user_location, get_weather_for_location],
    context_schema=Context,
)

config = {"configurable": {"thread_id": "1"}}
ctx = Context(user_id="1")

print("Streaming agent execution:\n")
for chunk in agent.stream(
    {"messages": [{"role": "user", "content": "今天什么天气？"}]},
    config=config,
    context=ctx,
    stream_mode="values"
):
    if "messages" in chunk:
        latest_msg = chunk["messages"][-1]
        msg_type = latest_msg.__class__.__name__

        if latest_msg.content:
            print(f"[{msg_type}] {latest_msg.content}")
        elif hasattr(latest_msg, 'tool_calls') and latest_msg.tool_calls:
            # Fix: tool_calls is list of dicts, use tc["name"]
            tools = [tc["name"] for tc in latest_msg.tool_calls]
            print(f"[{msg_type}] Calling tools: {tools}")
        else:
            print(f"[{msg_type}]")
```

### 09 10.py Message消息格式
做复杂的Langchain的项目，推荐用以下方式：

```
system_msg = SystemMessage("You are a helpful assistant.")
human_msg = HumanMessage("Hello, how are you?")
```

```
from langchain.chat_models import init_chat_model
from langchain.messages import HumanMessage, AIMessage, SystemMessage
```

```
model = init_chat_model("gpt-5-nano")
```

```
system_msg = SystemMessage("You are a helpful assistant.")
human_msg = HumanMessage("Hello, how are you?")
```

```
# Use with chat models
messages = [system_msg, human_msg]
response = model.invoke(messages)  # Returns AIMessage
```

你也可以直接用OpenAI聊天完成格式指定消息。

```
messages = [
    {"role": "system", "content": "You are a poetry expert"},
    {"role": "user", "content": "Write a haiku about spring"},
    {"role": "assistant", "content": "Cherry blossoms bloom..."}
]
response = model.invoke(messages)
```

多模态与结构化扩展
简单的 dict 难以表达复杂的结构。例如，AIMessage 可能包含 tool_calls（工具调用）、usage_metadata（Token 消耗统计）或多模态数据（图片/音频）。使用类对象可以更好地封装这些逻辑。
一致性的接口（Interface Consistency）
无论底层模型需要的是 {"role": "user"} 还是 {"author": "human"}，在 LangChain 层面你只需要关心 HumanMessage。LangChain 内部会有“适配器”将其转换为对应 API 的格式。
状态追踪与持久化
这些对象自带 id 和 metadata，这在构建复杂的 LangGraph 工作流或将对话保存到数据库（如 Redis, MongoDB）时非常关键。

LLM invoke个Agent invoke的区别？

```
llm.invoke(messages) → 单次 LLM 生成响应
```

- 输入： [BaseMessage列表] 或 str/prompt
- 输出： AIMessage (单条响应)
- 功能：
  - 简单对话/生成
  - 支持消息历史
  - 无工具调用、无循环、无状态管理

```
agent.invoke() – 智能代理执行
agent.invoke({"messages": [...]}) → 多步工具调用 + 最终回答
```

- 输入： {"messages": [dict(role, content)]}
- 输出： {"messages": [完整对话历史]}
- 功能：
  - 工具调用 + ReAct 循环 (思考→工具→观察→重复)
  - 状态管理 (LangGraph 底层)
  - 持久化/流式/HITL 支持
  - 需要 tools=[]

### 11.py 工具之间 的状态传递
tools状态传递有三种方式
1. llm自动传递tools的参数
2. 通过context去改状态，tools获取
3. 直接将tools合并（性能优化）

tools调用优化手段（大概有3-5倍性能提升。）
1. 提示词
2. tools描述
3. Max token
4. tools合并

关于工具调用的性能优化
初始版本   28s

```
import os
import time
import json
from dataclasses import dataclass
from dotenv import load_dotenv
```

```
from langchain.agents import create_agent
from langchain.tools import tool, ToolRuntime
from langchain_openai import ChatOpenAI
```

```
load_dotenv()
```

```
# --- 时间追踪器：记录相对于启动时刻的偏移 ---
class AgentTimer:
    def __init__(self):
        self.start_time = time.perf_counter()
        self.last_mark = self.start_time
```

```
    def mark(self, event_name):
        now = time.perf_counter()
        elapsed_total = now - self.start_time
        since_last = now - self.last_mark
        print(f"⏱️ [{elapsed_total:.3f}s] {event_name} (距离上一阶段: {since_last:.3f}s)")
        self.last_mark = now
```

```
timer = AgentTimer()
```

```
# --- 1. 系统提示词 ---
SYSTEM_PROMPT = "你是一位专业的专业气象预报员。询问天气时必须先用 get_user_location 获取位置，再查天气。"
```

```
@dataclass
class Context:
    user_id: str
```

```
# --- 2. 工具定义：在入口处记录时间 ---
@tool
def get_weather_for_location(city: str) -> str:
    """获取指定城市的当前天气。"""
    timer.mark(f"➡️ 工具开始执行: get_weather_for_location (参数: {city})")
    return f"{city} 总是阳光明媚！"
```

```
@tool
def get_user_location(runtime: ToolRuntime[Context]) -> str:
    """根据用户 ID 获取用户的地理位置。"""
    timer.mark("➡️ 工具开始执行: get_user_location")
    user_id = runtime.context.user_id
    return "佛罗里达" if user_id == "1" else "旧金山"
```

```
# --- 3. 初始化 LLM ---
llm = ChatOpenAI(
    model="doubao-seed-1-8-251228",
    temperature=0,
    api_key=os.environ.get("ARK_API_KEY"),
    base_url="https://ark.cn-beijing.volces.com/api/v3",
)
```

```
# --- 4. 创建 Agent ---
agent = create_agent(
    model=llm,
    system_prompt=SYSTEM_PROMPT,
    tools=[get_user_location, get_weather_for_location],
    context_schema=Context,
)
```

```
# --- 5. 运行并分析流程耗时 ---
print("🚀 [0.000s] Agent 调用发起")
config = {"configurable": {"thread_id": "1"}}
```

```
response = agent.invoke(
    {"messages": [{"role": "user", "content": "今天什么天气？"}]},
    config=config,
    context=Context(user_id="1")
)
```

```
timer.mark("🏁 Agent 流程结束，返回最终答案")
```

修改提示词  28s=> 12s

```
# 混合了结构化与自然语言引导，减少模型推理负担
SYSTEM_PROMPT = """你是一个天气查询助手。
请遵循以下确定性操作：
- 如果没有城市名，请立即执行 get_user_location。
- 如果已有城市名，请立即执行 get_weather_for_location。
```

不要输出任何思考过程，直接返回工具调用。在得到最终天气结果后，直接回复天气信息。"""

tools描述修改   还是12s左右
性能提升不明显

```
@tool
def get_user_location(runtime: ToolRuntime[Context]) -> str:
    """获取用户当前城市(内部ID定位)。""" # 👈 缩短描述，直击要点
    timer.mark("➡️ 工具开始执行: get_user_location")
    return "佛罗里达" if runtime.context.user_id == "1" else "旧金山"
```

```
@tool
def get_weather_for_location(city: str) -> str:
    """查询指定城市天气。参数:city(str)。""" # 👈 明确参数类型，减少模型犹豫
    timer.mark(f"➡️ 工具开始执行: get_weather_for_location (参数: {city})")
    return f"{city} 总是阳光明媚！"
```

优化max——token  12s => 9s

```
llm = ChatOpenAI(
    model="doubao-seed-1-8-251228",
    temperature=0,
    max_tokens=5000,
    api_key=os.environ.get("ARK_API_KEY"),
    base_url="https://ark.cn-beijing.volces.com/api/v3",
)
```

15.py 看LLM思考决策过程
  stream

```
# 使用 stream 模式获取中间环节
for chunk in agent.stream(
    {"messages": [{"role": "user", "content": "今天什么天气？"}]},
    config=config,
    context=Context(user_id="1")
```

):

```
    # 遍历每个节点（agent 节点负责思考和决策，tools 节点负责执行）
    for node_name, output in chunk.items():
        if "messages" in output:
            last_msg = output["messages"][-1]
```

```
            # --- A. 打印豆包的“隐藏思考” (Reasoning Tokens) ---
            # 豆包 1.8 的推理内容通常在这个字段
            reasoning = last_msg.additional_kwargs.get("reasoning_content", "")
            if reasoning:
                print(f"\n🧠 【LLM 内部推理/Reasoning】:\n{reasoning}")
```

```
            # --- B. 打印工具调用决策 (Tool Calls) ---
            if hasattr(last_msg, 'tool_calls') and last_msg.tool_calls:
                for tc in last_msg.tool_calls:
                    print(f"\n🎯 【决策：调用工具】: {tc['name']} | 参数: {tc['args']}")
```

```
            # --- C. 打印最终回复内容 ---
            elif last_msg.content:
                print(f"\n💬 【最终回复】: {last_msg.content}")
```

```
        # --- D. 打印工具节点返回的原始结果 ---
        elif node_name == "tools":
             print(f"✅ 【工具返回结果】: {output}")
```

16.py InMemorySaver 短期存储

```
# --- 4. 创建 Agent (只注册一个工具) ---
agent = create_agent(
    model=llm,
    system_prompt=SYSTEM_PROMPT,
    tools=[get_weather_info],
    context_schema=Context,
    checkpointer=InMemorySaver(), 
)
# 使用 config 获取当前 thread_id 的最新状态
state = agent.get_state(config)
```

id的作用就可以帮助我们恢复被中断的调用
--- 存储的消息历史 (Messages) ---

```
[0] HumanMessage (ID: a9c46482-46a2-425e-986c-ccec02c221b7): 今天什么天气？
[1] AIMessage (ID: lc_run--019bd601-58de-7dd2-98f1-e94cecf9c397-0): 
    └─ 🛠️ 包含工具指令: get_weather_info
[2] ToolMessage (ID: 330d8266-bf86-4fe5-9798-7efc01674d77): null 总是阳光明媚！
[3] AIMessage (ID: lc_run--019bd601-6e43-7103-ad41-fab19b3c8d09-0): 今天天气总是阳光明媚哦！
```

--- 原始快照元数据 (Metadata) ---  state.metadata

```
{
  "source": "loop",
  "step": 3,
  "parents": {}
}
```

1. source: 状态来源
这个字段告诉你当前的快照（Snapshot）是怎么产生的。
- loop (你目前看到的值)：表示这是由 LangGraph 的控制循环自动生成的。当一个节点（Node，如 agent 或 tools）运行结束，状态发生改变并自动存盘时，来源就是 loop。
- input：表示这是由用户初次输入产生的初始状态。
- update：表示开发者手动调用了 agent.update_state() 来人工修改数据，这会产生一个新的快照，来源标记为 update。

---

2. step: 运行步数
这是一个计数器，记录了从对话开始到现在，状态机（Graph）总共经历了多少次节点转换。
在你的这个“查询天气”任务中，step: 3 通常代表了以下完整的生命周期：
1. Step 0: 处理初始输入。
2. Step 1: 运行 agent 节点（模型思考，输出 tool_calls）。
3. Step 2: 运行 tools 节点（执行 Python 函数 get_weather_info）。
4. Step 3: 再次运行 agent 节点（模型拿到工具结果，组织最终语言）。

---

3. parents: 父节点映射
这个字段用于记录“当前状态是从哪个状态演变而来的”。
- 内容 {}：在简单的线性流程（像你现在的代码）中，它通常显示为空或指向前一个 checkpoint_id。
- 作用：在复杂的**并发（Parallel）或分支（Branching）**流程中，这个字段至关重要。例如，如果你同时启动了两个工具（查天气和查股价），这两个分支在合并时，parents 会记录这两个并行步骤的 ID，确保状态合并时不会冲突。

```
pprint(state.values)  输出
{
  "checkpoint_info": {
    "thread_id": "1",
    "model_name": "doubao-seed-1-8-251228",
    "status": "completed"
```

  },

```
  "messages": [
    {
      "role": "user",
      "type": "HumanMessage",
      "content": "今天什么天气？",
      "message_id": "92464669-b1a0-4e0d-b826-4760636f28aa"
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
          "call_id": "call_7864hu1q4csu8mtskjvdt08h",
          "tool_name": "get_weather_info",
          "arguments": {}
        }
      ],
      "token_usage": {
        "prompt_tokens": 462,
        "completion_tokens": 184,
        "total_tokens": 646
      },
      "finish_reason": "tool_calls"
    },
    {
      "role": "tool",
      "type": "ToolMessage",
      "tool_name": "get_weather_info",
      "tool_call_id": "call_7864hu1q4csu8mtskjvdt08h",
      "content": "佛罗里达 总是阳光明媚！"
    },
    {
      "role": "assistant",
      "type": "AIMessage",
      "content": "佛罗里达今天阳光明媚呢！☀️",
      "thought_metadata": {
        "reasoning_tokens": 98
      },
      "token_usage": {
        "prompt_tokens": 509,
        "completion_tokens": 108,
        "total_tokens": 617
      },
      "finish_reason": "stop"
    }
```

  ],

```
  "cumulative_usage": {
    "total_input_tokens": 971,
    "total_output_tokens": 292,
    "total_reasoning_tokens": 253,
    "grand_total_tokens": 1263
  }
}
state = agent.get_state(config)
```

1. state.values (核心数据)
这是 snapshot 中最重要的部分，它存储了你定义的 State 结构中的实际内容。
- 包含什么：在你的代码里，它主要包含 messages 列表（对话历史）。
- 作用：
  - 上下文记忆：让 AI 知道用户之前说了什么，以及工具返回了什么结果。
  - 数据传递：如果你在 State 中定义了其他变量（如 user_info 或 query_count），它们也存在这里。
  - 业务逻辑：你的程序通过读取 values 来决定最终给用户展示什么。
2. state.next (流程控制)
这是一个元组（Tuple），记录了图（Graph）中接下来准备执行的节点名称。
- 包含什么：例如 ('agent',) 或 ('tools',)。如果流程已经结束，它是一个空元组 ()。
- 作用：
  - 断点续传：如果你的 Agent 需要“人工审批”，程序会停下来。此时 next 会指向审批节点，直到你允许它继续。
  - 状态监控：通过它你可以判断：AI 现在是在“思考”（agent 节点）还是在“干活”（tools 节点）。
3. state.metadata (运行溯源)
这是关于“状态本身”的数据。
- 包含什么：
  - source: 来源（如 loop 代表自动运行，input 代表初始输入）。
  - step: 步数（记录这是第几次节点转换）。
  - parents: 父级状态的映射。
- 作用：
  - 性能分析：通过 step 判断模型是否陷入了死循环。
  - 版本管理：在复杂的分支流程中，通过 parents 追踪逻辑是如何合并的。
4. state.config (索引标签)
这是用于再次定位到这个特定状态的“钥匙”。
- 包含什么：包含 thread_id 和唯一的 checkpoint_id（通常是一串哈希值）。
- 作用：
  - 时间旅行 (Time Travel)：如果你存了 10 个步骤的快照，你可以拿着某个旧的 checkpoint_id 调用 agent.invoke，让 AI 从那个特定的历史瞬间“重活一次”。
  - 多线程隔离：确保不同用户的对话不会混淆。

内置中间件

SummarizationMiddleware 自动压缩上下文

```
from langchain.agents import create_agent
from langchain.agents.middleware import SummarizationMiddleware
```

```
agent = create_agent(
    model="gpt-4o",
    tools=[your_weather_tool, your_calculator_tool],
    middleware=[
        SummarizationMiddleware(
            model="gpt-4o-mini",
            trigger=("tokens", 4000),
            keep=("messages", 20),
        ),
    ],
)
```

模型调用限制

```
from langchain.agents import create_agent
from langchain.agents.middleware import ModelCallLimitMiddleware
from langgraph.checkpoint.memory import InMemorySaver
```

```
agent = create_agent(
    model="gpt-4o",
    checkpointer=InMemorySaver(),  # Required for thread limiting
    tools=[],
    middleware=[
        ModelCallLimitMiddleware(
            thread_limit=10,
            run_limit=5,
            exit_behavior="end",
        ),
    ],
)
```

工具调用限制

```
from langchain.agents import create_agent
from langchain.agents.middleware import ToolCallLimitMiddleware
```

```
agent = create_agent(
    model="gpt-4o",
    tools=[search_tool, database_tool],
    middleware=[
        # Global limit
        ToolCallLimitMiddleware(thread_limit=20, run_limit=10),
        # Tool-specific limit
        ToolCallLimitMiddleware(
            tool_name="search",
            thread_limit=5,
            run_limit=3,
        ),
    ],
)
```

模型错误兜底

```
from langchain.agents import create_agent
from langchain.agents.middleware import ModelFallbackMiddleware
```

```
agent = create_agent(
    model="gpt-4o",
    tools=[],
    middleware=[
        ModelFallbackMiddleware(
            "gpt-4o-mini",
            "claude-3-5-sonnet-20241022",
        ),
    ],
)
```

PII检测，敏感信息屏蔽

```
from langchain.agents import create_agent
from langchain.agents.middleware import PIIMiddleware
```

```
agent = create_agent(
    model="gpt-4o",
    tools=[],
    middleware=[
        PIIMiddleware("email", strategy="redact", apply_to_input=True),
        PIIMiddleware("credit_card", strategy="mask", apply_to_input=True),
    ],
)
```

LLM 工具选择器、筛选器
什么场景有用？
比如一个“个人 AI 助手”，用户可能下一秒问天气，再下一秒让你查数据库。你无法预知用户需求的顺序。

这种模式在业内通常被称为 "Router-Worker"（路由-执行）架构 或 "Small-to-Large"（小模型驱动大模型）架构。

```
from langchain.agents import create_agent
from langchain.agents.middleware import LLMToolSelectorMiddleware
```

```
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

当你的 tools 列表变得很长时，直接将所有工具描述塞给 GPT-4o 会面临三个问题：
1. 上下文污染（Attention Loss）：模型可能会在大量的工具描述中“迷失”，导致选错工具或产生幻觉。
2. 推理延迟：Prompt 越长，首字响应时间（TTFT）越久。
3. 成本高昂：GPT-4o 的输入 Token 价格昂贵，每一轮对话都要带上几十个工具的说明极其不划算。

使用llm tools筛选的作用

```
model="gpt-4o-mini"：利用廉价、极速的小模型来做“初筛”。判断“这个任务需要哪几个工具”并不需要极强的逻辑推理，小模型完全胜任。
max_tools=3：精简上下文。最终交给 GPT-4o 的只有 3 个最相关的工具描述，而不是原始的 100 个。
always_include=["search"]：保底策略。确保核心工具永远在线，防止小模型筛选时漏掉关键选项。
```

工具重试

```
from langchain.agents import create_agent
from langchain.agents.middleware import ToolRetryMiddleware
```

```
agent = create_agent(
    model="gpt-4o",
    tools=[search_tool, database_tool],
    middleware=[
        ToolRetryMiddleware(
            max_retries=3,
            backoff_factor=2.0,
            initial_delay=1.0,
        ),
    ],
)
```

1. max_retries=3 (最大重试次数)
- 含义：当工具执行失败时，系统最多允许重新尝试的次数。
- 作用：防止无限重试浪费资源。如果重试 3 次仍然失败，中间件将停止尝试并向 Agent 报错。
- 注意：这里的“3 次”通常指额外重试的次数。
2. initial_delay=1.0 (初始等待时间)
- 单位：秒。
- 含义：第一次失败后，在进行下一次尝试之前，程序需要等待的时间。
- 作用：给目标服务（比如一个数据库或第三方 API）一点“喘息”时间，避免立即重试导致连续失败。
3. backoff_factor=2.0 (退避因子/倍数)
- 含义：每次重试之间等待时间的增长倍数。
- 作用：实现指数级增长的等待时间。这在处理高并发拥堵时非常有效，通过不断拉长等待时间，错开请求高峰。

模型重试

```
from langchain.agents import create_agent
from langchain.agents.middleware import ModelRetryMiddleware
```

```
agent = create_agent(
    model="gpt-4o",
    tools=[search_tool, database_tool],
    middleware=[
        ModelRetryMiddleware(
            max_retries=3,
            backoff_factor=2.0,
            initial_delay=1.0,
        ),
    ],
)
```

LLM数据mock

```
from langchain.agents import create_agent
from langchain.agents.middleware import LLMToolEmulator
```

```
agent = create_agent(
    model="gpt-4o",
    tools=[get_weather, search_database, send_email],
    middleware=[
        LLMToolEmulator(),  # Emulate all tools
    ],
)
```

清除tools上下文

```
from langchain.agents import create_agent
from langchain.agents.middleware import ContextEditingMiddleware, ClearToolUsesEdit
```

```
agent = create_agent(
    model="gpt-4o",
    tools=[],
    middleware=[
        ContextEditingMiddleware(
            edits=[
                ClearToolUsesEdit(
                    trigger=100000,
                    keep=3,
                ),
            ],
        ),
    ],
)
```

1. trigger=100000 (触发阈值)
- 含义：这是清理行为的启动开关。
- 单位：通常指 Tokens（或者是字符数，取决于底层框架具体实现，LangChain 中多指 Token）。
- 解读：当整个对话的历史记录达到 100,000 Tokens 时，中间件才会开始干活。
- 逻辑：这是一种“延迟清理”策略。只要内存够用，就保留完整记录；只有快慢了，才开始删减。
2. keep=3 (保留数量)
- 含义：触发清理后，保留最近几次工具调用的完整记录。
- 解读：在清理过程中，它会把最久远的工具调用日志删掉，但会留下最后 3 次。
- 逻辑：为什么要留 3 次？因为 Agent 往往需要根据前一两步的结果来决定下一步。如果全删了，Agent 可能会忘记自己刚才已经执行过什么，导致进入死循环。

把tools变成脚本

场景： 当我的agent tools能力不够的时候，它自己写一个python脚本，脚本写在workspace中，理论上只要权限够，它爱干啥干啥。

如果自己写脚本干了？
manus？？？

```
from langchain.agents import create_agent
from langchain.agents.middleware import (
    ShellToolMiddleware,
    HostExecutionPolicy,
)
```

```
agent = create_agent(
    model="gpt-4o",
    tools=[search_tool],
    middleware=[
        ShellToolMiddleware(
            workspace_root="/workspace",
            execution_policy=HostExecutionPolicy(),
        ),
    ],
)
```

文件搜索
场景：
我在workspace下面，放了很多的文档，可以让agent在里面进行查询、搜索

```
from langchain.agents import create_agent
from langchain.agents.middleware import FilesystemFileSearchMiddleware
```

```
agent = create_agent(
    model="gpt-4o",
    tools=[],
    middleware=[
        FilesystemFileSearchMiddleware(
            root_path="/workspace",
            use_ripgrep=True,
        ),
    ],
)
```

18 19.py 生命周期
- before_agent- 代理开始前（每次调用一次）
- before_model- 每次模型调用前
- after_model- 每次模型响应后
- after_agent- 代理完成后（每次调用一次）
- wrap_model_call- 围绕每个模型调用
- wrap_tool_call- 围绕每个工具调用
- @dynamic_prompt

```
from langchain.agents import create_agent, dynamic_prompt
from datetime import datetime
```

```
# 定义动态提示词函数
@dynamic_prompt
def get_realtime_info(state: AgentState, runtime: ToolRuntime[Context]) -> str:
    # 每次模型调用前，都会执行这个函数
    now = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    user_name = "VIP客户" if runtime.context.user_id == "1" else "普通用户"
```

```
    return f"当前的服务器时间是：{now}。当前对话的用户身份是：{user_name}。请根据这些背景回复。"
```

```
# 在创建 Agent 时使用
agent = create_agent(
    model=llm,
    # 这里不再只传字符串，而是传动态函数
    system_prompt=[
        "你是一个全能助手。",
        get_realtime_info  # 注入动态部分
    ],
    tools=[...],
    context_schema=Context
)
```

1. before_agent (全局启动哨兵)
场景： 环境检查、用户权限校验、全局初始化。
- 黑名单拦截： 在请求发送给模型前，先检查该 UserID 是否在禁言名单中。
- 额度预扣除： 检查用户的 Token 余额是否足够支付本次潜在的对话消耗。
- 上下文注入： 自动从数据库提取用户的历史偏好（如“喜欢简短回答”），放入初始状态。
2. before_model (模型输入质检)
场景： 动态 Prompt 修改、上下文窗口管理。
- 敏感词过滤： 检查用户的问题是否包含政治或色情敏感词，如果有，直接通过 jump_to="end" 结束并报错。
- Prompt 注入： 每次模型调用前，实时获取当前时间或系统负载，告诉模型：“当前系统繁忙，请简短回答”。
- 上下文裁减： 如果对话轮数过多，在模型调用前手动删掉最早的几条消息，防止超出模型最大输入长度（Context Window）。
3. after_model (模型输出审计)
场景： 工具调用分析、初步响应格式化。
- 工具调用日志： 实时记录模型打算调用哪个工具及其参数，用于后台分析模型的决策逻辑。
- 响应脱敏： 模型生成结果后，检查其中是否包含了模型意外泄露的系统密钥或密码，并在显示给用户前打码。
- 强制格式化： 如果模型应该返回 JSON 但格式微调错误，在这里进行尝试性的字符串修复。

---

4. wrap_model_call (模型调用全生命周期)
场景： 性能分析、容错处理、成本计算。
- 耗时统计： 记录模型从发送请求到返回响应的精确时间。
- 自动重试： 如果模型 API 报 500 错误或频率限制，在钩子内自动进行 3 次指数退避重试，对业务层透明。
- 成本计算： 获取响应中的 usage 字段，计算本次调用的精确费用并存入数据库。
5. wrap_tool_call (工具执行保护伞)
场景： 运行保护、结果重塑、安全沙箱。
- 二次确认： 如果模型尝试调用 delete_user 工具，拦截执行并返回：“由于操作敏感，请联系人工处理”。
- 超时控制： 给耗时较长的工具（如爬虫）设置硬超时，防止工具运行过久导致 Agent 挂起。
- 模拟/ Mock： 在测试环境下，不真正执行昂贵的 API 工具，而是通过钩子返回预设的 Mock 数据。
6. after_agent (最终交付质检)
场景： 总结记录、最终清理、数据导出。
- 归档存储： 任务结束后，将完整的对话记录（包括所有中间思考过程）保存到向量数据库或日志系统中。
- 最终通知： 任务完成后，自动给开发者发送一条 Webhook 通知，告知“订单处理 Agent 已完成操作”。
- 交互优化： 检查 Agent 最终的回复，如果为空或由于异常中断，将其替换为更友好的提示语：“对不起，我遇到了一点问题，请稍后再试”。
[图片]

20.py MCP服务
绝大部分应用场景 定义tools调用

```
pip install mcp
```

前提  装个node  我的版本 v24
npx @modelcontextprotocol/inspector python 20.py

```
from mcp.server.fastmcp import FastMCP
```

```
# 1. 初始化 FastMCP
mcp = FastMCP("MyWeatherService")
```

```
# 2. 定义查天气工具
@mcp.tool()
def query_weather(city: str) -> str:
    """根据城市名称查询天气。"""
    # 模拟数据：你可以根据输入返回不同的结果
    if "北京" in city:
        return "北京天气：晴，气温 25°C，微风。"
    elif "上海" in city:
        return "上海天气：阴，气温 22°C，局部有小雨。"
    else:
        return f"暂时查不到 {city} 的天气，可能那里是世外桃源吧。"
```

```
if __name__ == "__main__":
    mcp.run()
```

什么是MCP？
MCP我就是把tools换了个地方单独定义，很多ai都能直接调用
可以通过两种方式通信
1. STDIO (标准输入输出)   本地
2. HTTP / SSE (Server-Sent Events)  （远程）
通信内容  JSON-RPC 2.0

```
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "query_weather",
    "arguments": {
      "city": "北京"
    }
```

  },

```
  "id": 1
}
{
  "jsonrpc": "2.0",
  "result": {
    "content": [
      {
        "type": "text",
        "text": "北京天气：晴，25°C..."
      }
    ]
```

  },

```
  "id": 1
}
```

调用MCP服务

```
pip install langchain-mcp-adapters 
import os
import asyncio
from langchain.agents import create_agent
from langchain_openai import ChatOpenAI
from langchain_mcp_adapters.client import MultiServerMCPClient
from dotenv import load_dotenv
```

```
load_dotenv()
```

```
async def main():
    # 一行代码替代所有的 stdio/ClientSession 管理
    client = MultiServerMCPClient({
        "weather": {
            "transport": "stdio",
            "command": "python",
            "args": [os.path.abspath("20.py")],
            "env": os.environ.copy()
        }
    })
```

```
    # 获取工具
    tools = await client.get_tools()
```

```
    # 初始化模型
    llm = ChatOpenAI(
        model="doubao-seed-1-6-251015",
        temperature=0,
        api_key=os.getenv("ARK_API_KEY"),
        base_url="https://ark.cn-beijing.volces.com/api/v3",
    )
```

```
    # 创建 Agent（一句话）
    agent = create_agent(
        model=llm,
        tools=tools,
        system_prompt="你是智能助手小宁。请使用 query_weather 查询北京天气。",
    )
```

```
    # 执行
    response = await agent.ainvoke({
        "messages": [{"role": "user", "content": "北京天气怎么样？"}]
    })
```

```
    print(f"回答: {response['messages'][-1].content}")
```

```
if __name__ == "__main__":
    asyncio.run(main())
```

Http Mcp服务

```
from mcp.server.fastmcp import FastMCP
```

```
# 1. 初始化 FastMCP
mcp = FastMCP("MyWeatherService")
```

```
# 2. 定义查天气工具
@mcp.tool()
def query_weather(city: str) -> str:
    """根据城市名称查询天气。"""
    if "北京" in city:
        return "北京天气：晴，气温 25°C，微风。"
    elif "上海" in city:
        return "上海天气：阴，气温 22°C，局部有小雨。"
    else:
        return f"暂时查不到 {city} 的天气，可能那里是世外桃源吧。"
```

```
if __name__ == "__main__":
    # 3. 关键修改：指定传输协议为 sse
    # 这会启动一个基于 Starlette/FastAPI 的 HTTP 服务
    mcp.run(transport="sse")
import asyncio
import os
from langchain.agents import create_agent
from langchain_openai import ChatOpenAI
from dotenv import load_dotenv
```

```
# 引入核心组件
from langchain_mcp_adapters.tools import load_mcp_tools
from mcp.client.sse import sse_client
from mcp.client.session import ClientSession # 必须引入这个
```

```
load_dotenv()
```

```
async def main():
    # 1. 建立 SSE 连接
    async with sse_client("http://localhost:8000/sse") as (read, write):
        # 2. 核心修正：手动创建并初始化 Session
        async with ClientSession(read, write) as session:
            await session.initialize()
```

```
            # 3. 此时 load_mcp_tools 只传这一个 session 对象
            mcp_tools = await load_mcp_tools(session)
```

```
            # 4. 初始化模型
            llm = ChatOpenAI(
                model="doubao-seed-1-6-251015",
                temperature=0,
                api_key=os.getenv("ARK_API_KEY"),
                base_url="https://ark.cn-beijing.volces.com/api/v3",
            )
```

```
            # 5. 创建 Agent
            agent = create_agent(
                model=llm,
                system_prompt="你是智能助手小宁。请使用 query_weather 查询北京天气。",
                tools=mcp_tools,
            )
```

```
            print("--- 正在通过 HTTP (SSE) Session 查询 ---")
            response = await agent.ainvoke({
                "messages": [{"role": "user", "content": "北京天气怎么样？"}]
            })
```

```
            print(f"回答: {response['messages'][-1].content}")
```

```
if __name__ == "__main__":
    asyncio.run(main())
```

## 真实场景中 中断代码与业务流程示意

1. Ai SDK
2. 后端
3. 前端
=Ai应用

### Ai逻辑

```
# ai_engine.py
from langgraph.types import interrupt
from langgraph.graph import StateGraph, START, END
```

```
def approval_node(state):
    # 【核心点】程序运行到这里会直接中断，并把字符串存在 Checkpoint 里
    # 等待被 Command(resume=...) 唤醒
    user_feedback = interrupt("请审核该操作：是否允许转账 100 元？")
```

```
    # 唤醒后，user_feedback 会拿到前端传回的值
    return {"approved": user_feedback["action"] == "confirm", "comment": user_feedback.get("reason")}
```

```
# 构建图... 记得 compile 时挂载 checkpointer（如 PostgresSaver）
```

### 后台Api

```
# server.py
from fastapi import FastAPI
from langgraph.types import Command
from ai_engine import graph
```

```
app = FastAPI()
```

```
@app.post("/run-task")
async def run_task(thread_id: str):
    config = {"configurable": {"thread_id": thread_id}}
    # 启动或继续运行
    result = graph.invoke({"input": "init_data"}, config)
```

```
    # 检查是否中断了
    if "__interrupt__" in result:
        return {
            "status": "waiting_human",
            "prompt": result["__interrupt__"][0].value, # "请审核..."
            "thread_id": thread_id
        }
    return {"status": "success", "data": result}
```

```
@app.post("/resume-task")
async def resume_task(thread_id: str, decision: dict):
    # decision 格式: {"action": "confirm", "reason": "OK"}
    config = {"configurable": {"thread_id": thread_id}}
```

```
    # 【唤醒关键】使用 Command 恢复，把前端的 decision 传给 AI 里的 interrupt()
    result = graph.invoke(Command(resume=decision), config)
    return {"status": "resumed", "final_result": result}
```

### 前端业务逻辑

<template>
  <div class="p-4">

```
    <button @click="startAI" class="bg-blue-500 text-white p-2">启动 AI 任务</button>
```

```
    <div v-if="showModal" class="modal-overlay">
      <div class="modal-content">
        <h3>AI 审批请求</h3>
        <p>{{ interruptPrompt }}</p>
        <input v-model="reason" placeholder="输入审批理由" />
        <button @click="submitDecision('confirm')">批准</button>
        <button @click="submitDecision('reject')">拒绝</button>
      </div>
    </div>
```

  </div>
</template>

<script setup>

```
import { ref } from 'vue';
import axios from 'axios';
```

```
const showModal = ref(false);
const interruptPrompt = ref('');
const threadId = "user_123_thread";
const reason = ref('');
```

```
const startAI = async () => {
  const res = await axios.post('/run-task', { thread_id: threadId });
  if (res.data.status === 'waiting_human') {
    interruptPrompt.value = res.data.prompt;
    showModal.value = true; // 唤起弹窗
  }
```

};

```
const submitDecision = async (action) => {
  await axios.post('/resume-task', {
    thread_id: threadId,
    decision: { action, reason: reason.value }
```

  });

```
  showModal.value = false;
  alert("任务已继续执行");
```

};
</script>
1. 前端的界面中有一个开始运行大模型的按钮
2. 前端发起post 请求，run-task
3. 后端收到请求，就开始graph.invoke，开始运行
4. graph检测到有一个中断的信号
5. 返回给前端的res中，携带一个status “waiting_human”
6. 前端就展示一个弹出  同意、拒绝
7. 用户点击同意
8. 前端发起post请求 resume-task
9. 后台就响应resume-task对应的逻辑
10. graph.invoke 大模型恢复运行
