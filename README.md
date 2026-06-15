# 财经新闻 RAG 问答系统

本项目基于 **东方财富财经早餐** 新闻数据，构建了一个完整的 **RAG（检索增强生成）** 问答系统。系统能够根据用户提出的问题，从新闻库中检索相关上下文，并利用大语言模型生成准确、可溯源的回答。

## 📌 项目特点

-  **数据来源**：使用 `akshare` 自动获取东方财富网“财经早餐”栏目新闻（包含标题、摘要、发布时间、原文链接）。
-  **检索增强生成**：通过向量检索召回最相关的新闻片段，再交由 LLM 生成答案，有效缓解模型幻觉问题。
-  **高效向量检索**：采用 `FAISS` 作为向量数据库，结合 `intfloat/multilingual-e5-large` 多语言嵌入模型，支持中英文混合检索。
-  **友好交互界面**：基于 `Gradio` 搭建 Web 界面，提供可视化问答体验，并可直接生成公网分享链接。
-  **一键初始化与缓存**：首次运行自动下载数据、构建索引并缓存，后续启动直接加载，无需重复计算。

##  技术栈

| 组件               | 技术选型                                                                 |
| ------------------ | ------------------------------------------------------------------------ |
| **向量数据库**     | FAISS (CPU 版)                                                           |
| **嵌入模型**       | `intfloat/multilingual-e5-large` (通过 HuggingFace Endpoint)             |
| **大语言模型**     | `deepseek-ai/DeepSeek-V4-Pro` (可通过 HuggingFace Endpoint 替换)         |
| **RAG 框架**       | LangChain (`0.3.23`) + LangChain Community (`0.3.21`)                    |
| **前端交互**       | Gradio                                                                   |
| **数据获取**       | akshare                                                                  |
| **运行环境**       | Google Colab / 本地 Jupyter Notebook                                     |

##  文件说明

| 文件名                                    | 描述                                                                 |
| ----------------------------------------- | -------------------------------------------------------------------- |
| `Basic_RAG_EcoNews.ipynb`                | RAG 核心实现，包括数据加载、文本分块、向量索引构建、检索与提示增强   |
| `Basic_RAG_EcoNews_Website.ipynb`        | 基于 Gradio 的 Web 界面封装，支持缓存复用，可一键启动交互式问答应用  |

##  快速开始

### 1. 环境准备

- 推荐使用 **Google Colab** 运行两个 notebook，或本地安装 Python 3.10+ 及 Jupyter。
- 获取 **HuggingFace API Token**（访问 [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens) 创建）。
- 确保 Colab 中已挂载 Google Drive（用于持久化缓存和 API Key）。

### 2. 运行核心 RAG 系统 (`Basic_RAG_EcoNews.ipynb`)

1. 打开 notebook，依次执行各单元格。
2. 在第二个单元格中修改 `pth` 路径指向你的工作目录，并将 `os.environ["HUGGINGFACEHUB_API_TOKEN"]` 设为你的 API Key。
3. 运行后会自动下载数据、构建 FAISS 索引（首次运行需几分钟），并演示一个检索 + 问答示例。

### 3. 启动 Web 交互界面 (`Basic_RAG_EcoNews_Website.ipynb`)

1. 同样配置 HuggingFace API Key。
2. 运行所有单元格，系统会自动检测并复用已有的 FAISS 索引和文本块缓存（若无则重新构建）。
3. 成功运行后控制台会输出一个公网链接（例如 `https://xxxx.gradio.live`），点击即可使用。
4. 在文本框中输入问题（如 “六月份有什么重要的财经新闻？”），系统将返回回答并展示检索到的相关新闻片段。

> 💡 首次运行 Web 版时，如果本地没有缓存，会先下载数据并构建索引（耗时数分钟）。之后的启动速度会非常快。

## 📋 示例问答

| 用户问题                                           | 系统回答片段（示例）                                                                 |
| -------------------------------------------------- | ----------------------------------------------------------------------------------- |
| 六月份有什么重要的财经新闻？                       | 6 月 10 日：特朗普称伊朗击落美军直升机，美军报复打击；美方将阿里巴巴、比亚迪列入“涉军”清单等。 |
| 关于人工智能的政策有哪些？                        | 工信部开展“人工智能+软件”专项行动；国务院印发《关于深入实施“人工智能+”行动的意见》等。     |
| 美伊冲突对市场有什么影响？                        | 霍尔木兹海峡关闭风险推升油价；避险情绪升温导致黄金价格上涨；A股相关板块波动加剧等。       |
| 哪些公司发布了 IPO 相关消息？                     | 宇树科技 IPO 过会；长鑫科技科创板提交注册；摩尔线程 IPO 获受理；沐曦股份上市等。           |

##  核心流程

```
用户提问
   │
   ▼
文本向量化 (multilingual-e5-large)
   │
   ▼
FAISS 相似度检索 (Top‑k)
   │
   ▼
拼接检索到的新闻片段 → 构建增强提示
   │
   ▼
LLM (DeepSeek-V4-Pro) 生成回答
   │
   ▼
返回答案 + 展示检索来源
```

## ⚙️ 自定义修改

- **更换嵌入模型**：修改 `HuggingFaceEndpointEmbeddings` 中的 `model` 参数。
- **更换 LLM**：修改 `repo_id` 为其他 HuggingFace 支持的对话模型（如 `meta-llama/Llama-3-8B-Instruct`）。
- **调整检索数量**：修改 `retrieval()` 函数中的 `k` 值（默认为 3）。
- **修改分块大小**：调整 `RecursiveCharacterTextSplitter` 的 `chunk_size` 和 `chunk_overlap`。

##  许可证

本项目仅供学习和研究使用，数据来源于东方财富网，请遵守相关网站的使用条款。

##  致谢

- [东方财富网](http://www.eastmoney.com) 提供的财经早餐数据
- [akshare](https://github.com/akfamily/akshare) 金融数据接口库
- [LangChain](https://www.langchain.com) 和 [HuggingFace](https://huggingface.co) 社区

