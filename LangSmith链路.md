# LangSmith 链路笔记

## 官网地址

官网地址：

```text
smith.langchain.com
```

登录后查看项目链路。

> [图片]

---

## `.env` 文件配置

在 `.env` 文件中写入：

```env
# 这是注释：不要把这个文件传给别人！
ARK_API_KEY="xxxxx"

# LangSmith 配置
LANGCHAIN_TRACING_V2=true
LANGCHAIN_ENDPOINT=https://api.smith.langchain.com
LANGCHAIN_API_KEY=xxxxxx
LANGCHAIN_PROJECT=Volc_Vision_Memory_Store
```

---

## 这段代码我们到底改动了一个啥？

这份 `VolcVisionEmbeddings` 只负责一件事：

```text
把文本 / 输入转成向量
```

真正存和搜的，是外面的向量数据库。

```python
class VolcVisionEmbeddings(Embeddings):
    def __init__(self):
        self.url = "https://ark.cn-beijing.volces.com/api/v3/embeddings/multimodal"
        self.api_key = os.getenv("ARK_API_KEY")
        self.endpoint_id = "ep-m-20251218162405-n5fw2"

    def _request(self, input_data: List) -> List[List[float]]:
        headers = {
            "Content-Type": "application/json",
            "Authorization": f"Bearer {self.api_key}"
        }
        payload = {
            "model": self.endpoint_id,
            "input": input_data
        }

        res = requests.post(self.url, json=payload, headers=headers)
        if res.status_code != 200:
            print(f"❌ 原始报错: {res.text}")
            res.raise_for_status()

        full_json = res.json()
        data_field = full_json.get("data")

        # 兼容字典结构
        if isinstance(data_field, dict) and "embedding" in data_field:
            return [data_field["embedding"]]

        # 兼容列表结构
        if isinstance(data_field, list):
            return [item["embedding"] if isinstance(item, dict) else item for item in data_field]

        raise ValueError(f"无法解析的返回结构: {full_json}")

    def _parse_input(self, val: Any) -> List:
        """内部工具：将传入的字符串或结构还原为火山需要的 List[Dict]"""
        try:
            if isinstance(val, str):
                # 尝试解析 JSON 字符串
                parsed = json.loads(val)
                return parsed if isinstance(parsed, list) else [parsed]
        except:
            pass

        # 如果是普通字符串或已经解析好的结构
        if isinstance(val, list):
            return val

        return [{"type": "text", "text": str(val)}]

    def embed_documents(self, texts: List[Any]) -> List[List[float]]:
        # LangGraph 批量处理
        final_embeddings = []
        for text_item in texts:
            actual_input = self._parse_input(text_item)
            vecs = self._request(actual_input)
            final_embeddings.append(vecs[0])
        return final_embeddings

    def embed_query(self, text: Any) -> List[float]:
        # 搜索处理
        actual_input = self._parse_input(text)
        return self._request(actual_input)[0]
```

---

## 向量存储链路

### 阶段 A：数据存入（向量化并存储）

当你执行 `store.put()` 往数据库里存数据时：

1. 触发 `embed_documents`：数据库发现你存入的是文本，它需要把文本转成数字。
2. 循环调用 `_request`：如果你一次存 10 条记忆，`embed_documents` 会循环调用 `_request` 10 次（或批量调用）。
3. 结果：文本变成了向量，存入内存或磁盘。

### 阶段 B：数据检索（搜索最相似的内容）

当你执行 `store.search()` 进行查询时：

1. 触发 `embed_query`：数据库首先需要把你的“搜索词”（如：“大海颜色”）也变成向量，才能去库里对比。
2. 调用 `_request`：将搜索词发送给火山引擎，拿到它的向量。
3. 数学比对：数据库拿着这个“搜索词向量”，去和库里已有的“记忆向量”算距离（余弦相似度等）。

> [图片]

---

## ChromaDB 小 Demo：从向量生成到存储检索

可以补一个新手能直接看懂的 ChromaDB 小 demo。

你现在这份 `VolcVisionEmbeddings` 只负责一件事：

```text
把文本 / 输入转成向量
```

真正存和搜的，是外面的 ChromaDB。

### 1. 先准备你的 embedding 适配器

你前面那份类保持不变，假设已经有了：

```python
class VolcVisionEmbeddings(Embeddings):
    ...
```

### 2. 外部接一个 ChromaDB 来存和查

下面这个 demo 就是完整链路：

```python
import os
import json
import requests
from typing import Any, List

from langchain_core.embeddings import Embeddings
from langchain_chroma import Chroma
from dotenv import load_dotenv


# =========================
# 1. 你的自定义 Embeddings
# =========================
class VolcVisionEmbeddings(Embeddings):
    def __init__(self):
        self.url = "https://ark.cn-beijing.volces.com/api/v3/embeddings/multimodal"
        self.api_key = os.getenv("ARK_API_KEY")
        self.endpoint_id = "ep-m-20251218162405-n5fw2"

    def _request(self, input_data: List) -> List[List[float]]:
        headers = {
            "Content-Type": "application/json",
            "Authorization": f"Bearer {self.api_key}"
        }
        payload = {
            "model": self.endpoint_id,
            "input": input_data
        }

        res = requests.post(self.url, json=payload, headers=headers)
        if res.status_code != 200:
            print(f"❌ 原始报错: {res.text}")
            res.raise_for_status()

        full_json = res.json()
        data_field = full_json.get("data")

        # 兼容字典结构
        if isinstance(data_field, dict) and "embedding" in data_field:
            return [data_field["embedding"]]

        # 兼容列表结构
        if isinstance(data_field, list):
            return [item["embedding"] if isinstance(item, dict) else item for item in data_field]

        raise ValueError(f"无法解析的返回结构: {full_json}")

    def _parse_input(self, val: Any) -> List:
        """把传入的字符串或结构统一成火山接口需要的 List 格式"""
        try:
            if isinstance(val, str):
                parsed = json.loads(val)
                return parsed if isinstance(parsed, list) else [parsed]
        except:
            pass

        if isinstance(val, list):
            return val

        return [{"type": "text", "text": str(val)}]

    def embed_documents(self, texts: List[Any]) -> List[List[float]]:
        """Chroma 在存文档时会调用它"""
        final_embeddings = []
        for text_item in texts:
            actual_input = self._parse_input(text_item)
            vecs = self._request(actual_input)
            final_embeddings.append(vecs[0])
        return final_embeddings

    def embed_query(self, text: Any) -> List[float]:
        """Chroma 在搜索时会调用它"""
        actual_input = self._parse_input(text)
        return self._request(actual_input)[0]


# =========================
# 2. 主程序：接入 ChromaDB
# =========================
def main():
    load_dotenv()

    # 你的 embedding 适配器
    embeddings = VolcVisionEmbeddings()

    # ChromaDB 向量库
    vectorstore = Chroma(
        collection_name="demo_memory",
        embedding_function=embeddings,
        persist_directory="./chroma_db"   # 本地持久化目录
    )

    # =========================
    # 3. 存数据：Chroma 会自动调用 embed_documents()
    # =========================
    docs = [
        "北京是中国的首都",
        "上海是一个国际化大都市",
        "大海的颜色通常看起来是蓝色",
        "苹果是一种水果"
    ]

    vectorstore.add_texts(
        texts=docs,
        metadatas=[
            {"source": "note1"},
            {"source": "note2"},
            {"source": "note3"},
            {"source": "note4"},
        ]
    )

    print("✅ 文档已写入 ChromaDB")

    # =========================
    # 4. 查数据：Chroma 会自动调用 embed_query()
    # =========================
    query = "海是什么颜色"
    results = vectorstore.similarity_search(query, k=2)

    print(f"\n🔍 查询：{query}")
    for i, doc in enumerate(results):
        print(f"\n--- 结果 {i + 1} ---")
        print("内容:", doc.page_content)
        print("元数据:", doc.metadata)


if __name__ == "__main__":
    main()
```

### 3. 这段 demo 到底发生了什么

你最关心的其实是这个流程：

#### 存的时候

```python
vectorstore.add_texts(["北京是中国的首都"])
```

Chroma 会自动做：

```text
收到文本
  ↓
调你的 embedding_function
  ↓
调 embed_documents(...)
  ↓
拿到向量
  ↓
存进 Chroma
```

#### 查的时候

```python
vectorstore.similarity_search("海是什么颜色")
```

Chroma 会自动做：

```text
收到查询词
  ↓
调你的 embedding_function
  ↓
调 embed_query(...)
  ↓
拿到查询向量
  ↓
跟库里向量算相似度
  ↓
返回最像的文档
```

---

## `traceable` 的 `run_type` 支持的几种类型

### 1. 核心逻辑类型

这是最常用的几种类型，定义了链（Chain）的基本骨架：

- `chain`（默认值）：最通用的类型。适用于任何逻辑组合、工作流或 Python 函数。
- `llm`：专门用于大语言模型的调用。标记为此类型后，LangSmith 会尝试解析 Prompt 和 Completion。
- `tool`：用于工具/函数调用（Function Calling）。在 Agent 架构中，这能清晰地展示 Agent 选择了哪个工具。

---

### 2. RAG 与检索相关

标记这些类型后，LangSmith 的 UI 会显示“放大镜”等图标，方便调试知识库检索：

- `retriever`：专门用于检索器（如向量数据库查询）。
- `embedding`：用于嵌入模型的计算过程。
- `parser`：用于输出解析器（Output Parser），将 LLM 的字符串转换为结构化数据（JSON/Pydantic）。

---

### 3. Agent 与决策相关

用于更复杂的交互流程：

- `agent`：用于标记 Agent 决策层。
- `action`：记录 Agent 执行的具体动作。

---

### 4. 数据与评估相关

通常在自动化测试或生产监控中使用：

- `prompt`：专门用于模板渲染步骤。
- `evaluator`：用于自动评估器。如果你写了一个“裁判 LLM”来给其他模型打分，建议标记为这个类型。
