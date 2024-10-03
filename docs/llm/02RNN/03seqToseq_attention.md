---
sidebar_position: 3
title: 3. seq2seq & Attention
---

目标

- 掌握seq2seq模型特点
- 掌握集束搜索方式
- 掌握BLEU评估方法
- 掌握Attention机制
- 应用Keras实现`seq2seq`对日期格式的翻译

### 3.1 seq2seq

`seq2seq`模型是在2014年，是由Google Brain团队和Yoshua Bengio 两个团队各自独立的提出来。

#### 3.1.1 定义

seq2seq是一个**Encoder–Decoder 结构**的网络，它的输入是一个序列，输出也是一个序列， **Encoder 中将一个可变长度的信号序列变为固定长度的向量表达**，**Decoder 将这个固定长度的向量变成可变长度的目标的信号序列。**

![](https://imgur.com/TAVMqGo.png)

- 注: Cell可以用 RNN ，GRU，LSTM 等结构。


#### 3.1.2 条件语言模型理解

- 编解码器作用

  编码器的作用是把一个不定长的输入序列$x_{1},\ldots,x_{t},$输出到一个编码状态$C$

  解码器输出$y^{t}$的条件概率将基于之前的输出序列$y^{1}, y^{t-1}$和编码状态$C$

  $argmax {P}(y_1, \ldots, y_{T'} \mid x_1, \ldots, x_T)$，**给定输入的序列，使得输出序列的概率值最大。**

- 根据maximum likelihood estimation，最大化输出序列的概率

  ${P}(y\_1, \ldots, y_{T'} \mid x_1, \ldots, x_T) = \prod_{t'=1}^{T'} {P}(y_{t'} \mid y_1, \ldots, y_{t'-1}, x\_1, \ldots, x\_T) = \prod_{t'=1}^{T'} {P}(y_{t'} \mid y_1, \ldots, y_{t'-1}, {C})$

  由于这个公式需要求出：$P(y^{1}|C) * P(y^{2}|y^{1},C)*P(y^{3}|y^{2},y^{2},y^{1},C)...$这个概率连乘会非常非常小不利于计算存储，所以需要对公式取$log$计算：

  $log{P}(y_1, \ldots, y_{T'} \mid x_1, \ldots, x_T) = \sum_{t'=1}^{T'} \log{P}(y_{t'} \mid y_1, \ldots, y_{t'-1}, {C})$

  所以这样就变成了$P(y^{1}|C) + P(y^{2}|y^{1},C) + P(y^{3}|y^{2},y^{2},y^{1},C)...$概率相加。这样也可以看成输出结果通过**softmax**就变成了**概率最大，而损失最小的问题，输出序列损失最小化。**

#### 3.1.3 应用场景

- 神经机器翻译(NMT)

  ![](https://imgur.com/Nx205x8.png)

- 聊天机器人

### 3.2 Attention机制

#### 3.2.1 长句子问题

![](https://imgur.com/fvRXg1b.png)

对于更长的句子，`seq2seq`就显得力不从心了，无法做到准确的翻译，一下是通常BLEU的分数随着句子的长度变化，可以看到句子非常长的时候，分数就很低。

![](https://imgur.com/8YTFUNn.png)

本质原因：在Encoder-Decoder结构中，Encoder把所有的输入序列都编码成一个统一的语义特征$C$再解码，**因此， $C$中必须包含原始序列中的所有信息，它的长度就成了限制模型性能的瓶颈。**当要翻译的句子较长时，一个$C$可能存不下那么多信息，就会造成翻译精度的下降。

#### 3.2.2 定义

- Attention机制通过计算输入序列中每个位置的权重，帮助解码器在生成输出时关注编码器的隐层状态输出中与当前时刻最相关的信息。具体来说，它动态地选择输入序列中对当前输出时刻最重要的上下文信息，而忽略或减弱不重要的信息。
  
  目的：Attention机制的主要目的是加强编码器信息输入到解码器时，关注解码器当前时刻所需的上下文信息。通过对不同输入时刻赋予不同的权重，Attention机制能够增强相同时刻的联系，并减弱其他时刻的干扰，解决了传统Seq2Seq模型中长序列信息压缩到固定长度向量时信息丢失的问题。
  
  ![](https://imgur.com/BXfvudw.png)

#### 3.2.3 公式

注意上述的几个细节，颜色的连接深浅不一样，假设Encoder的时刻记为t,而Decoder的时刻记为$t^{'}$​​​​。

- ${c}_{t'} = \sum_{t=1}^T \alpha_{t't}{h}_t$

  $a_{t^{'}t}$为参数，在网络中训练得到

  理解：蓝色的解码器中的cell举例子

  $\alpha_{41}h_{1}+\alpha_{42}h_{2} + \alpha_{43}h_{3} + \alpha_{44}h_{4} = c_{4}$

- $a_{t^{'}t}$的N个权重系数由来？

  权重系数通过softmax计算：$a_{t' t} = \frac{\exp(e_{t' t})}{ \sum_{k=1}^T \exp(e_{t' k}) },\quad t=1,\ldots,T$

  $e_{t' t} = g({s}_{t' - 1}, {h}_t)= {v}^\top \tanh({W}_s {s} + {W}_h {h})$

  - $e_{t' t}$是由t时刻的编码器隐层状态输出和解码器$t^{'}-1$时刻的隐层状态输出计算出来的

  - s为解码器隐层状态输出，h为编码器隐层状态输出

  - $v,W_{s},W_{h}$​都是网络学习的参数

​	![](https://imgur.com/FFbM6Vi.png)

### 3.3 机器翻译案例

#### 3.3.1 问题描述

在这个案例中，我们使用一个简单的**日期转换任务**来模拟机器翻译任务。模型的目标是将不同格式的日期（如“1958年8月29日”、“03/30/1968”）转换成标准化的机器可读格式 `YYYY-MM-DD`（如“1958-08-29”、“1968-03-30”）。我们使用 `seq2seq` 网络来完成这一任务。

#### 3.3.2 环境设置

在开始之前，我们需要安装一些依赖库：

```bash
pip install faker tqdm babel keras==2.2.4
```

- `faker`: 用于生成随机数据。
- `tqdm`: 用于显示进度条。
- `babel`: 处理国际化和本地化（如日期格式转换）。
- `keras`: 简单易用的深度学习库。

#### 3.3.3 数据加载与预处理

数据预处理的代码逻辑包含在 `nmt_utils` 中，用于加载和处理训练数据。具体包括：
- 加载不同格式的日期样本。
- 将日期转换为数字，并进行 `one-hot` 编码。

训练数据示例：
```python
dataset = [('9 may 1998', '1998-05-09'), ('10.09.70', '1970-09-10')]
x_vocab = {' ': 0, '.': 1, '/': 2, '0': 3, '1': 4, '2': 5, ...}
y_vocab = {'-': 0, '0': 1, '1': 2, '2': 3, '3': 4, ...}
```

#### 3.3.4 代码分析

1. **Seq2seq类主要方法**：
   - `load_data(self, m)`: 加载数据。
   - `init_seq2seq(self)`: 初始化模型，包含编码器、解码器和Attention机制的定义。
   - `train(self, X_onehot, Y_onehot)`: 训练模型。
   - `test(self)`: 测试模型。

2. **训练逻辑**：
   训练过程使用了10000个样本，特征值为 `one-hot` 编码的日期输入序列，目标值为 `one-hot` 编码的标准化日期输出序列。

   训练代码示例如下：

   ```python
   if __name__ == '__main__':
       s2s = Seq2seq()
       X_onehot, Y_onehot = s2s.load_data(10000)
       s2s.init_seq2seq()
       s2s.train(X_onehot, Y_onehot)
   ```

   训练过程中模型的输出：

   ```
   Epoch 1/1
   100/10000 [......] - ETA: 10:52 - loss: 23.9884 - dense_1_acc: 0.3200
   ```

#### 3.3.5 模型定义

1. **参数定义**：

   模型的输入输出参数如下：

   ```python
   def __init__(self, Tx=30, Ty=10, n_x=32, n_y=64):
       self.model_param = {
           "Tx": Tx,  # 编码器输入序列的最大长度
           "Ty": Ty,  # 解码器输出序列的最大长度
           "n_x": n_x,  # 编码器隐层维度
           "n_y": n_y,  # 解码器隐层维度
       }
   ```

2. **网络结构初始化**：

   通过 `init_seq2seq()` 初始化编码器、解码器、Attention机制和输出层。

   ```python
   def init_seq2seq(self):
       self.get_encoder()
       self.get_decoder()
       self.get_attention()
       self.get_output_layer()
   ```

3. **Attention机制的定义**：

   Attention机制的具体实现如下，用于计算上下文向量：

   ```python
   def computer_one_attention(self, a, s_prev):
       s_prev = self.attention["repeator"](s_prev)
       concat = self.attention["concatenator"]([a, s_prev])
       e = self.attention["densor1"](concat)
       energies = self.attention["densor2"](e)
       alphas = self.attention["activator"](energies)
       context = self.attention["dotor"]([alphas, a])
       return context
   ```

4. **模型整体结构**：

   **编码器**：双向LSTM处理输入序列并生成隐层状态。

   **解码器**：LSTM使用Attention机制生成输出。

   **输出层**：全连接层使用 `softmax` 输出概率分布。

   模型定义代码示例如下：

   ```python
   def model(self):
       X = Input(shape=(self.model_param["Tx"], self.model_param["x_vocab_size"]), name='X')
       s0 = Input(shape=(self.model_param["n_y"],), name='s0')
       c0 = Input(shape=(self.model_param["n_y"],), name='c0')
       s = s0
       c = c0
       outputs = []
       a = self.encoder(X)
       for t in range(self.model_param["Ty"]):
           context = self.computer_one_attention(a, s)
           s, _, c = self.decoder(context, initial_state=[s, c])
           out = self.output_layer(s)
           outputs.append(out)
       model = Model(inputs=(X, s0, c0), outputs=outputs)
       return model
   ```

#### 3.3.6 测试模型

通过加载训练好的模型权重进行测试：
```python
def test(self):
    model = self.model()
    model.load_weights("./models/model.h5")
    example = '1 March 2001'
    source = string_to_int(example, self.model_param["Tx"], self.model_param["x_vocab"])
    source = np.expand_dims(np.array(list(map(lambda x: to_categorical(x, num_classes=self.model_param["x_vocab_size"]), source))), axis=0)
    s0 = np.zeros((10000, self.model_param["n_y"]))
    c0 = np.zeros((10000, self.model_param["n_y"]))
    prediction = model.predict([source, s0, c0])
    prediction = np.argmax(prediction, axis=-1)
    output = [dict(zip(self.model_param["y_vocab"].values(), self.model_param["y_vocab"].keys()))[int(i)] for i in prediction]
    print("source:", example)
    print("output:", ''.join(output))
```

测试结果：
```text
source: 1 March 2001
output: 2001-03-01
```

### 3.4 集束搜索（Beam Search）

#### 3.4.1 问题引入

在机器翻译中，直观的方式是通过每一步选择最大概率的词进行翻译，但这可能并不能得到最优结果。比如，考虑以下法语句子翻译：

```python
法语句子："Jane visite l'Afrique en septembre."
翻译1: "Jane is visiting Africa in September."
翻译2: "Jane is going to be visiting Africa in September."
```

虽然翻译2是语法正确的，但翻译1更简洁、自然。基于这种现象，选择每一步最大概率的词并不总是得到最优的整体翻译。因此，需要一次性考虑整个单词序列，使得整个句子的条件概率最大。

#### 3.4.2 集束搜索流程

**定义**：集束搜索（Beam Search）是一种在每一步选择多个候选的搜索算法，通过设定一个参数 `B`（集束宽度），在每一步保留概率最高的 `B` 个候选词，然后继续下一步选择，直到生成完整句子。

**流程**：

1. 第一次：选择概率最大的 `B` 个词。
2. 第二次：为每个选出的词生成下一个候选词，并选出整体概率最高的 `B` 个序列。
3. 重复此过程，直到生成目标长度的序列。

例如，设 `B = 3`，假设选择两步之后可能得到的序列结果：
- "is fine"
- "in alright"
- "at alright"

这些就是最终筛选出的结果。

### 3.5 BLEU：机器翻译的自动评估方法

在机器翻译中，除了人工评估，我们还需要一种自动化的评估标准来衡量翻译质量。BLEU（Bilingual Evaluation Understudy）是一种用于评估机器翻译结果质量的工具，依据机器翻译与参考翻译的相似程度来打分。

#### 3.5.1 BLEU的定义

BLEU 评估机器翻译的质量，主要是通过判断候选翻译与参考翻译的相似度，主要计算基于 N-gram 的词组精度。

#### 3.5.2 N-gram 精度得分（N-gram Precision）

- **定义**：N-gram 精度用于计算候选翻译中的词组与参考翻译中的词组匹配程度。

- **N 的含义**：N 表示词组的长度，例如：

  候选翻译：`the the the the`

  参考翻译：`the cat is on the mat`

  如果 `N=1`，我们只关注单词的匹配。虽然候选翻译中所有单词“the”都在参考翻译中出现，得分可能很高，但这显然不是合适的翻译。因此，单纯的 1-gram 精度不足以评估翻译质量。

  为此，改进的方法是使用更高的 N 值（如 2-gram, 3-gram），例如 `{"the cat", "cat is", "is on", "on the", "the mat"}`。

- 改进的 N-gram 精度

  为了解决常见词干扰问题，使用 **modified N-gram precision** 进行改进。具体操作如下：

  - 过滤常见词（如“the”）。

  - 计算词组的匹配次数：
    
    $Count_{w_i,j}^{clip} = min(Count_{w_i},Ref_{j}\_Count_{w_i})$
    
    计算候选翻译中单词的次数，并选取参考翻译中出现的最大次数进行匹配。


![](https://imgur.com/kHWmcl8.png)

- 组合多元精度得分

  通过统计不同尺度的词组，计算各自的精度，然后将这些精度组合在一起，可以更好地衡量翻译的流畅性与准确性。

  ![](https://imgur.com/Lm9PzuB.png)

#### 3.5.3 短句惩罚

为了避免候选翻译过短导致不合理的高分，BLEU 引入了短句惩罚机制（Brevity Penalty, BP）。其公式为：
- 如果候选翻译长度 `c` 大于参考翻译长度 `r`，则 $BP = 1$。
- 如果 `c` 小于等于 `r`，则 $BP = e^{1 - \frac{r}{c}}$。

#### 3.5.4 最终BLEU得分公式

最终的 BLEU 得分结合了 N-gram 精度和短句惩罚，计算公式为：
$BLEU = BP \times exp\left( \sum_{n=1}^{N} w_n \times log(p_n) \right)$
其中，$p_n$ 表示 n-gram 精度，$w_n$ 是权重。

### 3.6 总结

- 掌握seq2seq模型特点
- 掌握集束搜索方式
- 掌握BLEU评估方法
- 掌握Attention机制
