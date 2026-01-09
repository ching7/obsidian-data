---
created: '2025-03-19 17:24:10'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/CvMEdLcBeoJWRUx5cWecvIpXnlg
modified: '2025-03-20 16:55:42'
source: feishu
title: ' 01 N-Gram模型'
---

 01 N-Gram模型
N-Gram 模型是一种用于自然语言处理（NLP）的统计语言模型，它通过分析文本中连续出现的 N 个词（或字符） 的组合，来预测下一个可能出现的词或字符。
🌟 1. N-Gram 的基本概念
N 指的是词或字符的数量。
N-Gram 模型的目标是基于前面 N-1 个词（或字符）来预测下一个词（或字符）。
基本公式是通过计算词序列的联合概率，得到下一个词出现的可能性。
🎯 公式
给定一个词序列  ，N-Gram 模型的目标是估计下一个词 的概率，即：
 
使用 N-Gram 简化为：
 
也就是说，只考虑前面 N-1 个词 来预测下一个词。

🌈 2. 常见的 N-Gram 类型

🏆 例子：
假设有一个句子：
"I love this product very much."
Unigram：I、love、this、product、very、much
Bigram：I love、love this、this product、product very、very much
Trigram：I love this、love this product、this product very、product very much
4-gram：I love this product、love this product very、this product very much

🚀 3. N-Gram 的实现原理
👉 通过统计大量文本中 N-Gram 组合出现的频率，来估计联合概率。
🎯 Bigram 概率估计公式
在 Bigram 模型中，单词序列的概率可以写成：
 
即：
  表示在词   之后出现   的概率
通过统计训练集中 Bigram 出现的次数来估计这个概率：
 
例子：
 假设训练集中出现了以下语句：
"I love this" 出现了 10 次
"I love" 出现了 50 次
则：
 

🏗️ 4. N-Gram 的 Python 示例代码
示例：用 Bigram 生成文本预测
使用 Python 统计 Bigram 组合出现的频率并进行预测：

from collections import defaultdict
import random
#训练文本
text = "I love this product very much. I love this shop. I hate bad products."
#分词
words = text.split()
#构建 Bigram 模型
bigram_model = defaultdict(list)
for i in range(len(words) - 1):
    bigram_model[words[i]].append(words[i + 1])
#生成文本
current_word = "I"
result = [current_word]
for _ in range(10):
    next_word = random.choice(bigram_model[current_word])
    result.append(next_word)
    current_word = next_word
print("Generated Sentence: ", ' '.join(result))
示例输出：

Generated Sentence: I love this product very much. I love this shop.
解释：
将输入文本进行分词。
构建 Bigram 词典，统计每个词的下一个可能词。
通过随机选择下一个词，生成新的句子。

🎨 5. 平滑（Smoothing）处理 —— 解决未见组合的问题
问题： 如果模型中没有见过某个组合（即  ），就会导致生成失败。
解决方案：
拉普拉斯平滑（加 1 平滑）：
  
其中 是词汇表大小。
Python 示例：

#添加拉普拉斯平滑
V = len(set(words))  # 词汇表大小
def bigram_probability(w1, w2):
    count_w1_w2 = bigram_model[w1].count(w2)
    count_w1 = len(bigram_model[w1])
    return (count_w1_w2 + 1) / (count_w1 + V)
#测试
print(bigram_probability("I", "love"))

🎯 6. N-Gram 的优势与劣势


🎯 7. N-Gram 的改进方案
✅ 引入神经网络：RNN、LSTM、Transformer 等深度学习模型可以捕获更长距离的上下文关系。
✅ 引入注意力机制（Attention）：突破固定长度限制，动态关注上下文。
✅ 使用子词级别的 N-Gram：在形态丰富的语言中，使用子词（如 BPE）可以提高效果。

🏆 8. 总结
✔️ N-Gram 是一种基于统计的方法，通过统计连续 N 个词的组合概率来建模文本。
 ✔️ 适用于语音识别、机器翻译等任务。
 ✔️ 常见平滑方法包括拉普拉斯平滑、回退平滑（Backoff）。
 ✔️ 现代 NLP 中，N-Gram 已逐渐被 Transformer、RNN 等神经网络方法替代，但在小数据集或低资源场景下，N-Gram 依然非常有效。

👉 如果你想深入了解 RNN、LSTM 或 Transformer 这些更高级的模型，可以告诉我！ 😎
