# LLM 与 RAG 架构

我们需要对项目进行改造，核心矛盾点在于：当前项目中很难自定义 LLM 的一些能力，只能通过 `json` 里面的 `Config` 配置，然后交给火山引擎去引用大模型。

这相当于对我们来说，LLM 调用链路是一个黑盒：

```text
前端/后端只负责提交 VoiceChat 配置
  ↓
火山引擎内部完成 ASR、LLM、TTS 编排
  ↓
我们很难深度定制 LLM 的调用逻辑
```

因此，我们要把 LLM 的能力抽离出来，并且对它进行定制化。

本章即为相关架构介绍。

---

## AI 实时对话-智能客服-项目架构设计

### 1. Orchestration 服务接管架构

这一层负责把大模型、提示词、知识库等能力组织起来。

主要包括：

1. 豆包 1.6（ARK）
2. 方舟 Prompt 提示词编排
3. RAG 知识库

### 2. RTC 通信层

这一层负责实时语音通信，也就是用户和火山引擎提供的 Agent 在房间里的声音交互。

主要包括：

1. ASR 语音转文字
2. TTS 语音合成
3. 声音模拟

### 3. 后端服务

这一层负责业务接口、数据库、会话管理，以及未来接管 LLM 调用链路。

主要包括：

1. Python FastAPI
2. veDB 业务数据库
3. veDB AI 会话管理

### 4. 部署

这一层负责项目上线、运行环境和扩容能力。

主要包括：

1. Lighthouse
2. 容器化、弹性扩容

### 5. 内容安全管控 Censor 架构

这一层负责对用户输入和 AI 输出进行安全检查。

主要包括：

```text
用户输入审核
AI 回复审核
敏感内容拦截
违规内容记录
```

### 6. API 网关 / 安全鉴权

这一层负责统一管理接口入口和安全访问。

主要包括：

```text
接口鉴权
Token 校验
访问频率限制
服务转发
日志记录
```

### 7. 前端界面

这一层负责用户看到和操作的页面。

主要包括：

1. React
2. WebRTC / RTC SDK
3. Redux Toolkit

---

## 火山引擎 Agent 服务改造

接下来我们将加入 RAG、知识库和自己的 LLM Prompt。

目前火山引擎提供的 Demo 是一套比较完整的实时语音对话示例，但是它对我们自己的业务需求来说还不够。

核心原因是：

```text
当前 LLM 调用链路主要由火山引擎 RTC 云端内部完成
我们只能在 json 里配置 LLMConfig
很难在中间插入自己的 RAG、知识库、Prompt、内容安全、会话管理等逻辑
```

所以我们需要对火山引擎提供的 Agent 调用链路进行改造。

### 当前核心问题

当前配置里，LLM 是这样交给火山引擎处理的：

```json
"LLMConfig": {
  "Mode": "ArkV3",
  "EndPointId": "ep-20260720222126-fd4nj",
  "SystemMessages": [
    "你是小宁，性格幽默又善解人意。你在表达时需简明扼要，有自己的观点。"
  ],
  "VisionConfig": {
    "Enable": false
  }
}
```

这段配置的意思是：

```text
RTC 云端收到用户语音
  ↓
ASR 转成文字
  ↓
RTC 云端根据 LLMConfig 直接调用 ArkV3 / Doubao 模型
  ↓
模型生成回复
  ↓
RTC 云端继续 TTS 合成语音
```

问题在于：

```text
我们只能配置 EndPointId 和 SystemMessages
但是很难自己控制 LLM 调用前后的业务逻辑
```

比如我们想做这些事情：

```text
从数据库读取不同用户对应的 Prompt
根据用户问题去知识库检索资料
把检索结果拼进 Prompt
调用自己的内容安全审核
记录完整会话
根据业务规则改写用户问题
根据用户身份决定模型参数
```

如果 LLM 调用完全被火山 RTC 云端接管，这些定制逻辑就很难插进去。

### 改造核心

核心操作是：

```text
拦截 RTC 触发的 LLM 调用
```

更准确地说，不是真的去“黑客式拦截”，而是通过 RTC AIGC 的第三方大模型配置，把原本指向 Ark 模型的 LLM 调用，改成指向我们自己的 Python Server API。

参考文档：

```text
callback 接入豆包 1.6 第三方 LLM：
https://www.volcengine.com/docs/6348/1399966?lang=zh
```

也就是在 `LLMConfig` 里额外配置一个 URL：

```text
LLMConfig
  ↓
不再只指向 ArkV3 EndPointId
  ↓
而是指向我们的 Python Server API
  ↓
Python Server 负责 RAG、Prompt、LLM 调用、管控
```

可以理解为：

```text
原来：RTC 云端直接问豆包
现在：RTC 云端先问我们的 Python Server，再由 Python Server 问豆包
```

### 新流程：加入 RAG 与管控

改造后的新流程是：

```text
用户说话
  ↓
火山引擎 RTC 云端
  ↓
ASR 将用户语音转成文本
  ↓
RTC 云端把文本发送给我们的 Python Server
  ↓
Python Server 执行 RAG 检索
  ↓
Python Server 从数据库读取定制 Prompt
  ↓
Python Server 组装最终 Prompt
  ↓
Python Server 调用火山引擎 Ark / Doubao 生成回复
  ↓
Python Server 将回复返回给 RTC 云端
  ↓
RTC 云端执行 TTS 语音合成
  ↓
火山引擎提供的 Agent 在 RTC 房间里播放语音
```

拆开看：

1. **ASR**：火山引擎 RTC 负责将用户语音转为文本
2. **中转**：RTC 云端将文本发送给我们的 Python Server
3. **RAG 检索**：Python Server 调用火山引擎知识库，检索相关文档片段
4. **Prompt 构建**：Python Server 从数据库读取定制 Prompt，结合检索到的知识，组装最终系统提示词
5. **生成**：Python Server 调用火山引擎 Ark / Doubao 生成回复
6. **返回**：Python Server 将回复返回给 RTC 云端
7. **TTS**：RTC 云端将回复转为语音并播放

### 新旧架构对比

```text
旧架构：

用户语音
  ↓
RTC 云端 ASR
  ↓
RTC 云端直接调用 Ark / Doubao
  ↓
RTC 云端 TTS
  ↓
AI 语音回复
```

```text
新架构：

用户语音
  ↓
RTC 云端 ASR
  ↓
我们的 Python Server
  ↓
RAG 知识库检索
  ↓
数据库读取 Prompt / 会话 / 用户信息
  ↓
调用 Ark / Doubao
  ↓
返回回复给 RTC 云端
  ↓
RTC 云端 TTS
  ↓
AI 语音回复
```

### Python Server 未来负责什么

改造后，Python Server 不再只是一个简单代理。

它会变成 LLM 编排中心：

```text
接收 RTC 云端传来的用户文本
调用知识库检索
读取业务数据库
构建 Prompt
调用大模型
执行内容安全审核
记录会话上下文
返回最终回复
```

也就是说：

```text
RTC 继续负责实时音视频
Python Server 接管 LLM 和 RAG 编排
```

### 最后记忆版

如果只记一件事：

```text
我们不是要替换 RTC
我们是要接管 RTC 后面的 LLM 调用链路
```

再短一点：

```text
RTC 管声音实时传输
Python Server 管大脑怎么思考
RAG 管回答有没有业务知识
TTS 管把回复变成声音
```

---

## 老师架构图解读

老师给的改造图可以理解成：

```text
RTC 仍然负责实时语音房间
火山引擎提供的 Agent 仍然负责 ASR 和 TTS
但是 LLM 调用不再直接黑盒走火山内部配置
而是通过 callback 转到我们的 Python Server
```

### 图里的几个角色

图里主要有这些角色：

```text
前端 Web 用户
RTC
火山引擎提供的 Agent
Callback API
LLM
RAG / 知识库 / embedding
```

分别是什么意思：

| 角色 | 作用 |
| --- | --- |
| 前端 Web 用户 | 用户打开网页、说话、听 AI 回复 |
| RTC | 实时语音房间，负责传输用户和 Agent 的音频流 |
| 火山引擎提供的 Agent | 火山引擎提供的实时对话 Agent，负责 ASR、TTS 和对话编排 |
| Callback API | 我们自己的接口，例如 `POST /api/callback` |
| LLM | 真正生成文字回复的大模型，例如豆包 1.6 |
| RAG / 知识库 / embedding | 给 LLM 提供业务知识，让回答更准确 |

### 红色链路：语音流

图里的红色箭头主要表示语音流。

用户说话时：

```text
前端 Web 用户
  ↓ 说话，也就是用户语音
火山引擎提供的 Agent
```

同时用户的语音也在 RTC 房间里传输：

```text
前端 Web 用户
  ↓ 语音流
RTC
```

AI 回复时：

```text
火山引擎提供的 Agent
  ↓ TTS 文字转语音，推流
RTC
  ↓ 语音流
前端 Web 用户
```

也就是说：

```text
用户声音和 AI 声音都走 RTC 音频流
```

### 蓝色链路：文字与 LLM 调用

图里的蓝色箭头主要表示文字链路。

用户说完话之后：

```text
火山引擎提供的 Agent
  ↓ ASR 将语音转成文字
Callback API：POST /api/callback
```

这里的意思是：

```text
火山引擎提供的 Agent 不再直接调用固定的 Ark LLMConfig
而是把 ASR 后的用户文字发给我们的 callback 接口
```

然后：

```text
Callback API
  ↓
LLM，例如豆包 1.6
```

LLM 返回时：

```text
LLM
  ↓ 文字回复，SSE stream
火山引擎提供的 Agent
```

最后：

```text
火山引擎提供的 Agent
  ↓ TTS
RTC
  ↓
前端 Web 用户听到 AI 语音
```

### RAG / 知识库 / embedding 在哪里

图左上角的：

```text
RAG
知识库
embedding
```

表示我们自己的 LLM 调用前置能力。

它们通常在 Python Server 里面完成：

```text
Callback API 收到用户问题
  ↓
用 embedding 把问题向量化
  ↓
去知识库检索相关资料
  ↓
把检索结果拼进 Prompt
  ↓
调用 LLM
  ↓
把 LLM 回复返回给火山引擎提供的 Agent
```

所以 RAG 不是直接替代 RTC。

RAG 的作用是：

```text
给 LLM 提供业务知识
让 AI 不只是凭模型记忆回答
而是结合我们的知识库回答
```

### 整体新链路

完整流程可以这样记：

```text
用户在前端 Web 说话
  ↓
用户语音进入 RTC 房间
  ↓
火山引擎提供的 Agent 听到用户语音
  ↓
Agent 用 ASR 把语音转成文字
  ↓
Agent 调用我们的 Callback API
  ↓
Python Server 做 RAG 检索
  ↓
Python Server 组装 Prompt
  ↓
Python Server 调用 LLM
  ↓
LLM 通过 SSE stream 返回文字回复
  ↓
Python Server 把文字回复返回给火山引擎提供的 Agent
  ↓
Agent 用 TTS 把文字转成语音
  ↓
Agent 把语音推回 RTC 房间
  ↓
前端 Web 播放 AI 语音
```

### 这张图最重要的改造点

最重要的点是：

```text
火山引擎提供的 Agent -> Callback API -> 我们自己的 LLM/RAG 服务
```

也就是说：

```text
原来：火山引擎提供的 Agent 直接调 Ark / Doubao
现在：火山引擎提供的 Agent 调我们的 Python Server
```

我们的 Python Server 再决定：

```text
查不查知识库
拼什么 Prompt
用哪个模型
做不做内容安全
要不要记录会话
如何返回给 Agent
```

### 最后记忆版

```text
RTC = 语音房间
火山引擎提供的 Agent = ASR + TTS + 对话入口
Callback API = 我们接管 LLM 的入口
Python Server = RAG / Prompt / LLM 编排中心
LLM = 生成文字回答
TTS = 把文字回答变成语音
```

最核心一句：

```text
这张图不是要替换火山引擎提供的 Agent，而是让这个 Agent 调用我们的 callback，从而把 LLM 和 RAG 能力接到我们自己手里。
```
