---
sidebar_position: 2
title: 2. Word Embedding & NLP
---

学习目标

- 了解词嵌入的优势
- 掌握词嵌入的训练gensim库的使用

### 2.1 one hot

#### 在RNN中词使用one_hot表示的问题

![](https://imgur.com/QLvRxGd.png)

- 假设有10000个词，每个词的向量长度都为10000，整体大小太大

- 没能表示出词与词之间的关系

  例如Apple与Orange会更近一些，Man与Woman会近一些，取任意两个向量计算内积都为0

### 2.2 词嵌入

定义：指把一个维数为所有词的数量的高维空间**嵌入到一个维数低得多的连续向量空间中**，每个单词或词组被映射为实数域上的向量。

> 注：这个维数通常不定，不同实现算法指定维度都不一样，通常在30~500之间。

如下图所示：

![](https://imgur.com/NVZHSwI.png)

#### 2.2.1 特点

- 能够体现出词与词之间的关系，比如说我们用`Man - Woman`,或者`Apple - Orange`，都能得到一个向量
- 能够得到相似词，例如`Man - Woman = King - ?` -> `? = Queen`

  ![](https://imgur.com/gFjh2Uq.png)

#### 2.2.3 算法类别

Bengio等人在一系列论文中使用了神经概率语言模型使机器“习得词语的分布式表示。

2013年，谷歌托马斯·米科洛维（Tomas Mikolov）领导的团队发明了一套工具[word2vec](https://www.tensorflow.org/tutorials/representation/word2vec)来进行词嵌入。

- skip-gram
- CBow

下载gensim库

```bash
pip install gensim
```

### 2.3 Word2Vec案例

#### 2.3.1 训练语料

由于语料比较大，就提供了一个[下载地址](http://www.sogou.com/labs/resource/cs.php)

- 搜狗新闻中文语料(2.7G)
- 做中文分词处理之后的结果

#### 2.3.2 步骤

1. 训练模型
2. 测试模型结果
  
#### 2.3.3 代码

- 训练模型API

  ```python
  from gensim import Word2Vec
  Word2Vec(LineSentence(inp), size=400, window=5, min_count=5)
  ```

  - `LineSentence(inp)`：把word2vec训练模型的磁盘存储文件
  - 转换成所需要的格式,如：\[\[“sentence1”\],\[”sentence1”\]\]
  - `size`：是每个词的向量维度
  - `window`：是词向量训练时的上下文扫描窗口大小，窗口为5就是考虑前5个词和后5个词
  - `min-count`：设置最低频率，默认是5，如果一个词语在文档中出现的次数小于5，那么就会丢弃

- 训练的代码如下

  ```python
  import sys
  import multiprocessing

  from gensim.models import Word2Vec
  from gensim.models.word2vec import LineSentence

  if __name__ == '__main__':
    if len(sys.argv) < 3:
      sys.exit(1)

    # inp表示语料库(分词)，outp：模型
    inp, outp = sys.argv[1:3]

    model = Word2Vec(LineSentence(inp), size=400, window=5, min_count=5, workers=multiprocessing.cpu_count())

    model.save(outp)
  ```

  运行命令

  ```python
  python trainword2vec.py ./corpus_seg.txt ./model/*
  ```

- 指定好分词的文件以及，保存模型的文件，加载模型测试代码

  ```python
    improt gensim
    gensim.models.Word2Vec.load("./model/corpus.model")
    model.most_similar("警察")

    Out:
    [('警员', 0.6961891651153564),
    ('保安人员', 0.6414757370948792),
    ('警官', 0.6149201989173889),
    ('消防员', 0.6082159876823425),
    ('宪兵', 0.6013336181640625),
    ('保安', 0.5982533693313599),
    ('武警战士', 0.5962344408035278),
    ('公安人员', 0.5880240201950073),
    ('民警', 0.5878666639328003),
    ('刑警', 0.5800305604934692)]

    model.similarity('男人','女人')
    Out: 0.8909852730435042

    model.most_similar(positive=['女人', '丈夫'], negative=['男人'], topn=1)
    Out: [('妻子', 0.7788498997688293)]
  ```

### 2.4 总结

掌握gensim库的词向量训练和使用
