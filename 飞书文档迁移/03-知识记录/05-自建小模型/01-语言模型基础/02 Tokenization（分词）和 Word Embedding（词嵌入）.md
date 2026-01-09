---
created: '2025-03-20 09:52:18'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/IYWzdZRgAoF5vjxjsxgclRZunDh
modified: '2025-03-28 16:56:53'
source: feishu
title: 02 Tokenization（分词）和 Word Embedding（词嵌入）
---

02 Tokenization（分词）和 Word Embedding（词嵌入）
Tokenization（分词）
将文本分解成更小的基本单元（通常是词或子词），便于模型处理。
常见的 Tokenization 方式： 
词级别分词（Word Tokenization）：按空格、标点符号分词。
子词级别分词（Subword Tokenization）：使用 BPE（Byte Pair Encoding）或 WordPiece，将词进一步拆分成更小的单元。
字符级分词（Character Tokenization）：直接按字符拆分。
例子：
 文本："I love programming"
词级别（Word Tokenization）：["I", "love", "programming"]
子词级别（Subword Tokenization）：["I", "love", "pro", "gram", "ming"]
字符级别（Character Tokenization）：["I", " ", "l", "o", "v", "e", " ", "p", "r", "o", "g", "r", "a", "m", "m", "i", "n", "g"]

Word Embedding（词向量）
将词（或子词）转换成固定长度的向量（稠密向量），以便神经网络等模型能够直接处理。
常用方法：  02-1 词向量（Word Embedding）
One-hot 编码 → 高维度且稀疏（例如 [0, 0, 1, 0, 0]）
Word2Vec → 低维稠密向量（例如 [0.45, -0.23, 0.78]）
GloVe → 全局语境表示
FastText → 使用子词信息
例子：
 假设我们用 Word2Vec 训练了一个模型，将词“love”表示成一个 3 维向量：
arduino
复制代码
"love" → [0.45, -0.23, 0.78]
