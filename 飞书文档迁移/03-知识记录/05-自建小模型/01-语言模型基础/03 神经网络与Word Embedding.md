---
created: '2025-03-20 09:56:28'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/SWrddTAqeoWDstx4F1lcQc0lnrc
modified: '2025-03-28 14:45:11'
source: feishu
title: 03 神经网络与Word Embedding
---

03 神经网络与Word Embedding

Q：什么是神经网络学习，同时为什么词转化为向量后，可以给神经网络学习，神经网络与大模型、深度学习之间的关系

A：这是个非常关键的问题！😊 让我们从基础开始，逐步深入，解释为什么词向量（Word Embedding）是神经网络可以学习的输入，以及它们之间的关系。

🌟 1. 什么是神经网络？
神经网络（Neural Network, NN） 是一种受人类大脑结构启发而建立的数学模型，用于模拟人类的学习过程。
🧠 (1) 大脑的类比：
人的大脑由神经元（Neuron）组成
神经元之间通过突触（Synapse）相连，形成复杂的网络
当人接触到新事物时，神经元会根据输入信号的强度来调整突触的连接权重，形成“记忆”
👉 神经网络就是通过这种“模拟神经元连接和激活”的方式，来实现学习和推理。

🏗️ (2) 结构示例
一个典型的神经网络包括以下部分：
输入层：接收输入特征
隐藏层：对输入数据进行非线性变换
输出层：输出结果（分类、生成等）
➡️ 示例：一个简单的 3 层神经网络

输入层 → 隐藏层 → 输出层
➡️ 输入是一个词向量（如 [0.45, -0.23, 0.78]），通过网络学习之后输出为某个分类（如 "正面情感"）。

🏆 (3) 神经网络的基本公式
一个神经元的输出可以表示为：
 
其中：
  = 输入特征（如词向量）
  = 权重（需要学习的参数）
  = 偏置项（需要学习的参数）
 = 激活函数（引入非线性，如 ReLU, Sigmoid）

🌈 (4) 示例（2 层神经网络）
输入：[0.45, -0.23, 0.78]
 第一层：
 
第二层：
 
输出层：
 

🚀 2. 为什么词向量（Word Embedding）可以被神经网络学习？
🏆 (1) 词向量 = 将离散的文本映射为连续空间的表示
原始文本是离散的，计算机无法直接理解它。
👉 通过 词向量（Word Embedding），我们可以将文本转换成连续的、低维的数值向量，神经网络就可以像处理图片或音频一样处理文本。

🎯 (2) 示例：
假设输入文本：

"I love programming"
➡️ Tokenization 之后：

["I", "love", "programming"]
➡️ 映射成词向量：

"I" → [0.45, -0.23, 0.78]  
"love" → [0.12, 0.56, -0.34]  
"programming" → [-0.77, 0.45, 0.31]  
➡️ 传入神经网络后：
神经网络可以通过调整权重（W）来学习词之间的联系
学习到哪些词是正面情感（如 love），哪些词是负面情感（如 hate） 
通过反向传播（Back Propagation）调整 W 和 b，来优化输出 04 情感分类与反向传播05 前向传播 + 反向传播 + 梯度下降

🎯 (3) 词向量的核心特性
相似词向量更接近（余弦相似度）
语义结构保留（如“国王” - “男人” + “女人” ≈ “女王”）
嵌入空间中存在聚类现象（如“水果”词汇集中在某个区域）

🏆 (4) 为什么神经网络更容易学习向量？
词向量是一个数值矩阵（连续值），而神经网络正是为处理这种结构而设计的。
神经网络的基本运算（加权求和、矩阵乘法、非线性变换）可以直接作用于数值向量。

🌍 3. 深度学习、神经网络与大模型的关系
✅ （1）神经网络 → 深度学习 → 大模型
神经网络（NN）：基础结构
深度学习（Deep Learning）：包含多层神经网络，网络更深、特征提取能力更强
大模型（LLM）：通过深度学习 + Transformer 结构，训练超大规模语料，学习更复杂的语义和任务

🚀 （2）关系总结


🧠 （3）示例：GPT 结构
输入：文本（Token）
分词：Tokenization
词向量：Word Embedding（如 768 维）
模型：多层 Transformer（基于 Attention）
输出：生成或分类

🏅 4. 通过神经网络学习词向量的完整流程
输入层：文本 → Tokenization
嵌入层：Token → Word Embedding
隐藏层：神经网络学习上下文特征（通过 Attention 或卷积）
输出层：生成/分类
训练：通过反向传播（Back Propagation）调整参数

🔥 5. 示例：用神经网络训练词向量（Word2Vec）

from gensim.models import Word2Vec
#输入语料
sentences = [["I", "love", "programming"], ["I", "hate", "bugs"], ["Python", "is", "awesome"]]
#构建 Word2Vec 模型
model = Word2Vec(sentences, vector_size=3, window=2, min_count=1, workers=4)
#获取 "love" 的词向量
vector = model.wv['love']
print(f"Vector for 'love': {vector}")
#输出相似词
similar_words = model.wv.most_similar('love')
print(f"Similar words to 'love': {similar_words}")

✅ 6. 为什么这很重要？
神经网络通过处理连续数值（如词向量）来提取特征
Word Embedding 保持了语义信息
N-Gram 捕获了上下文结构

🎯 7. 总结


💡 👉 到这里，神经网络 + Word Embedding + Tokenization 的完整路径就清晰了！ 😎
