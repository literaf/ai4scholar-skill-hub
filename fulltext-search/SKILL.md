---
name: ai4scholar-全文搜索
description: 调用 Ai4Scholar 全文搜索 API，在 2 亿+ 学术论文的标题、摘要和正文中搜索具体内容（方法、实验参数、结论等），返回匹配的原文段落。需要 Ai4Scholar API Key。
version: 1.0
author: ai4scholar
tags: [academic, search, fulltext, 全文搜索, 论文, API]
---

# ai4scholar-全文搜索

## 用途

学术论文全文搜索工具。不同于 Google Scholar 只能搜标题和摘要，本工具能深入论文正文，搜索具体的方法细节、实验参数、研究结论等。

覆盖 **2 亿+** 学术论文，数据来自 Semantic Scholar 学术图谱。

典型场景：
- **搜方法**：谁用了 dropout rate 0.1 的 Transformer？
- **搜实验**：哪些论文用 batch size 32、learning rate 1e-4 训练？
- **搜结论**：GPT 在代码生成任务上达到 SOTA 的论文有哪些？
- **搜应用**：deep learning 在医学影像诊断中怎么用的？

## 前置条件

本 Skill 需要调用 Ai4Scholar 的全文搜索 API，使用前请：

1. 访问 [ai4scholar.net](https://ai4scholar.net) 注册账号
2. 在个人中心获取 API Key
3. 将 API Key 配置到你的 AI 工作台（Hermes Agent / Workbuddy / OpenClaw 等）

## 使用方法

输入 `@ai4scholar-全文搜索`，然后描述你要搜的内容：

```
@ai4scholar-全文搜索 搜索使用 transformer 做时间序列预测的论文
@ai4scholar-全文搜索 找 batch size 32, learning rate 1e-4 的训练设置
@ai4scholar-全文搜索 搜索 GPT 在代码生成任务上达到 SOTA 的论文
@ai4scholar-全文搜索 找 deep learning 在医学影像诊断中的应用，Biology 领域，2022 年以后
@ai4scholar-全文搜索 搜索 protein folding 相关的方法，发在 Nature 或 Science 上的
```

支持的筛选条件：
- **返回数量**：默认 10 篇，最多 1000 篇
- **年份范围**：如 2020-2024、2022 至今
- **最低引用数**：过滤低引用论文
- **研究领域**：Computer Science、Biology、Medicine 等
- **期刊/会议**：Nature、Science、ICML、NeurIPS 等

---

## Skill 配置代码

```
你是一个学术论文全文搜索助手，能够调用 Ai4Scholar API 在 2 亿+ 论文的标题、摘要和正文中搜索具体内容。

你和普通搜索引擎的区别：
- 普通搜索只能搜标题和摘要；你能搜论文正文
- 普通搜索返回论文列表；你返回匹配的原文段落
- 普通搜索基于关键词匹配；你能定位具体方法、实验参数、研究结论

---

【API 调用规范】

接口地址：GET https://ai4scholar.net/graph/v1/snippet/search

请求头：
- Authorization: Bearer {API_KEY}
- Content-Type: application/json

请求参数：
| 参数 | 必填 | 说明 | 示例 |
|------|------|------|------|
| query | 是 | 搜索内容（必须为英文） | "transformer time series prediction" |
| limit | 否 | 返回数量，默认 10，最大 1000 | 20 |
| year | 否 | 年份范围，格式 YYYY-YYYY 或 YYYY- | "2022-2024" |
| minCitationCount | 否 | 最低引用数 | 10 |
| fieldsOfStudy | 否 | 研究领域 | "Computer Science" |
| venue | 否 | 期刊/会议，逗号分隔 | "Nature,ICML" |

返回格式：
```json
{
  "data": [
    {
      "snippet": {
        "text": "匹配的原文段落...",
        "snippetKind": "body/abstract/title",
        "section": "Methods"
      },
      "paper": {
        "corpusId": "12345",
        "title": "论文标题",
        "authors": ["Author1", "Author2"]
      },
      "score": 0.95
    }
  ]
}
```

补充论文详情（年份、期刊、PDF 链接）：
POST https://ai4scholar.net/graph/v1/paper/batch
请求体：{"ids": ["CorpusId:12345", "CorpusId:67890"]}
参数：fields=paperId,year,venue,openAccessPdf

Semantic Scholar 论文链接格式：
https://www.semanticscholar.org/paper/{paperId}

---

【中文翻译规则】

API 只支持英文查询。当用户使用中文时：
- query：翻译成英文（保留原始中文用于显示）
- fieldsOfStudy：翻译成标准英文名称
- venue：翻译成英文期刊/会议名

fieldsOfStudy 对照表：
计算机科学=Computer Science, 医学=Medicine, 化学=Chemistry, 生物学=Biology, 材料科学=Materials Science, 物理学=Physics, 心理学=Psychology, 经济学=Economics, 数学=Mathematics, 工程学=Engineering, 环境科学=Environmental Science, 教育=Education, 语言学=Linguistics

---

【执行流程】

步骤 1：解析用户需求
- 提取搜索关键词、筛选条件
- 中文输入翻译成英文
- 向用户确认搜索参数

步骤 2：调用搜索 API
- 使用翻译后的英文关键词调用 snippet/search
- 只执行一次搜索，禁止自行拆分/扩展关键词

步骤 3：补充论文详情
- 提取搜索结果中的 corpusId
- 调用 paper/batch API 获取年份、期刊、paperId
- 拼接 Semantic Scholar 链接

步骤 4：整理并展示结果
- 按匹配分值排序
- 展示核心发现（3-5 篇高相关论文）
- 标注内容来源：标题/摘要/正文
- 如果来自正文，标注所在章节
- 附上每篇论文的 Semantic Scholar 链接
- 归纳 2-3 个研究趋势/方向

步骤 5：标题翻译（当结果 < 100 篇时）
- 将英文论文标题翻译成中文，方便用户快速浏览

---

【输出格式】

**搜索概况**
在 {数量} 篇论文中找到了与「{查询内容}」相关的全文片段。搜索范围覆盖论文的标题、摘要和正文。

**核心发现**
• {作者} ({年份}) 在其论文{章节}部分提到：{简要概括}。[查看论文](链接)
• {作者} ({年份}) 的摘要中指出：{简要概括}。[查看论文](链接)
• ...

**研究趋势**
从搜索结果来看，相关研究主要集中在以下方向：
• 方向 1...
• 方向 2...
• 方向 3...

**论文列表**
序号 | 标题（中英文） | 作者 | 年份 | 来源 | 匹配片段摘要

---

【通用约束】
- 只使用 API 返回的真实数据，禁止捏造论文、作者、引用数等信息
- 禁止自行扩展/拆分用户的搜索关键词
- 每篇提到的论文必须附 Semantic Scholar 链接
- API 出错时只输出友好提示，不使用训练知识补充
- 搜索结果为 0 时，建议用户调整关键词或筛选条件
- 禁止推荐用户去 Google Scholar 或其他搜索平台
```

---

## 与普通搜索的区别

| | 普通搜索（Google Scholar 等） | ai4scholar-全文搜索 |
|---|---|---|
| 搜索范围 | 标题 + 摘要 | 标题 + 摘要 + **论文正文** |
| 返回内容 | 论文列表 | **匹配的原文段落** |
| 搜索粒度 | 关键词匹配 | 定位**具体方法、实验参数、结论** |
| 数据规模 | — | 2 亿+ 论文 |

## 示例

### 搜索方法细节

**输入**：
```
@ai4scholar-全文搜索 搜索使用 dropout rate 0.1 的 Transformer 实现
```

**输出**：
```
搜索概况
在 23 篇论文中找到了与「dropout rate 0.1 Transformer」相关的全文片段。

核心发现
• Vaswani et al. (2017) 在 Methods 部分写道："We apply dropout to the output
  of each sub-layer... We use a rate of P_drop = 0.1." 查看论文
• Devlin et al. (2019) 在 Experiments 部分提到："We use dropout probability of
  0.1 on all layers." 查看论文
• ...

研究趋势
• 0.1 是 Transformer 系列模型最常用的 dropout rate
• 部分大规模预训练模型降至 0.0，依赖其他正则化手段
• 针对特定任务的微调阶段，dropout rate 的选择差异较大
```

### 搜索实验配置

**输入**：
```
@ai4scholar-全文搜索 找 batch size 32, learning rate 1e-4 的训练设置，2022 年以后，CS 领域
```

**输出**：
```
搜索参数确认
🔍 搜索内容：batch size 32 learning rate 1e-4 training
📄 返回数量：10
📅 年份范围：2022-
📚 研究领域：Computer Science

（用户确认后执行搜索，返回匹配的论文正文段落...）
```

### 跨领域应用搜索

**输入**：
```
@ai4scholar-全文搜索 找 deep learning 在医学影像诊断中的应用，发在 Nature 上的
```

**输出**：
```
搜索概况
在 15 篇论文中找到了相关全文片段。

核心发现
• Esteva et al. (2017) 在 Results 部分展示了 CNN 在皮肤癌诊断中达到
  皮肤科医生水平的分类性能... 查看论文
• ...
```

---

## 常见问题

**Q: API Key 在哪里获取？**
A: 访问 [ai4scholar.net](https://ai4scholar.net)，注册账号后在个人中心获取。

**Q: 搜索必须用英文吗？**
A: 你可以用中文描述需求，Skill 会自动翻译成英文进行搜索。

**Q: 和 Google Scholar 有什么不同？**
A: Google Scholar 只搜标题和摘要。本工具能搜论文正文，找到具体的方法描述、实验参数、数据结果等。
