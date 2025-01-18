---
layout:     post
title:      "ChatGLM+LangChain"
subtitle:   " \"Open-sourced QA system\""
date:       2023-12-27 21:00:00
author:     "xxc"
header-img: "img/black_bg.jpg"
catalog: true
tags:
    - 知识分享
    - 日常学习

---
<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>


[bilibili](https://player.bilibili.com/player.html?bvid=BV13M4y1e7cN&autoplay=0)

[基于LangChain+LLM的本地知识库问答：从企业单文档问答到批量文档问答_langchain 本地知识库-CSDN博客](https://blog.csdn.net/v_JULY_v/article/details/131552592)

#### 参考文献
[https://github.com/datawhalechina/hugging-llm](https://github.com/datawhalechina/hugging-llm)
[https://github.com/datawhalechina/llm-universe](https://github.com/datawhalechina/llm-universe)
[https://github.com/milvus-io/milvus](https://github.com/milvus-io/milvus)
[https://github.com/chatchat-space/Langchain-Chatchat](https://github.com/chatchat-space/Langchain-Chatchat)
[https://www.milvus-io.com/integrate_with_langchain](https://www.milvus-io.com/integrate_with_langchain)
[https://blog.csdn.net/liluo_2951121599/article/details/134229683](https://blog.csdn.net/liluo_2951121599/article/details/134229683)
[https://cloud.tencent.com/developer/article/2302432](https://cloud.tencent.com/developer/article/2302432)
[https://www.aliyun.com/activity/bigdata/opensearch/llmsearch](https://www.aliyun.com/activity/bigdata/opensearch/llmsearch)
[https://zhuanlan.zhihu.com/p/641132245](https://zhuanlan.zhihu.com/p/641132245)
[https://zhuanlan.zhihu.com/p/364923722](https://zhuanlan.zhihu.com/p/364923722)
[https://zhuanlan.zhihu.com/p/644603736](https://zhuanlan.zhihu.com/p/644603736)

#### 现成产品
[OpenSearch LLM智能问答版](https://www.aliyun.com/activity/bigdata/opensearch/llmsearch)

#### 一些坑
1. 直接上传数据库就算能成功效果也很差，目前最佳实践还是人工来进行预处理  
分成多个很小的markdown格式化文件，效果最好

#### 实现原理
1. 加载本地文档
    1. OCR & Image
        1. 目前：[https://github.com/RapidAI/RapidOCR](https://github.com/RapidAI/RapidOCR)
        2. [https://github.com/breezedeus/CnOCR](https://github.com/breezedeus/CnOCR)
        3. [https://github.com/hiroi-sora/Umi-OCR](https://github.com/hiroi-sora/Umi-OCR)
        4. 太吃算力：[https://github.com/facebookresearch/nougat](https://github.com/facebookresearch/nougat)
2. 文本拆分
3. Embedding
    1. ernie
    2. text2vec
    3. m3e
    4. bge
    5. piccolo
4. 根据提问匹配文本（向量数据库）
    1. 字符匹配（类关键词）
        1. [使用 Milvus 和 LangChain 进行文档问答 – Milvus向量库中文文档](https://www.milvus-io.com/integrate_with_langchain)
        2. [Faiss](https://github.com/facebookresearch/faiss)
    2. 语义检索
5. 构建Prompt
    1. Prompt engineering
6. LLM生成回答

![](/img/chatglm-langchain/1.png)

#### 微调和提示词工程
![](/img/chatglm-langchain/2.png)

#### LangChain介绍
![](/img/chatglm-langchain/3.png)

#### LangChain 应用场景
![](/img/chatglm-langchain/4.png)

#### 如何实现基于本地知识的问答
![](/img/chatglm-langchain/5.png)

#### 效果优化方向
![](/img/chatglm-langchain/6.png)

#### 改进思路
[《基于智能搜索和大模型打造企业下一代知识库》之《典型实用场景及核心组件介绍》 | Amazon Web Services](https://aws.amazon.com/cn/blogs/china/intelligent-search-based-enhancement-solutions-for-llm-part-one/)

[LLM+Embedding构建问答系统的局限性及优化方案](https://zhuanlan.zhihu.com/p/641132245)

[如何用大语言模型构建一个知识问答系统-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/2302432)

[【大模型应用开发教程】04_大模型开发整体流程 & 基于个人知识库的问答助手 项目流程架构解析-CSDN博客](https://blog.csdn.net/liluo_2951121599/article/details/134229683)

[RAG探索之路的血泪史及曙光](https://zhuanlan.zhihu.com/p/664921095)

[大模型RAG检索增强问答如何评估：噪声、拒答、反事实、信息整合四大能力评测任务探索 - 智源社区](https://hub.baai.ac.cn/view/31683)

+ 需要结合传统NLP，做意图理解（NLU），降级处理（异常处理），否则无法保证鲁棒性
+ 数据预处理成html，做分级，检索时根据需要取高一级的内容
+ nougat-ocr，ocr需要优化

