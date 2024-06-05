
# CNN

[tut](https://www.bilibili.com/video/BV1zF411V7xu)

## CNN vs 传统神经网络

| **Feature**                        | **Convolutional Neural Network (CNN)**                       | **Traditional Neural Network (MLP)**                         |
| ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Input Data Format**              | 3D data (Height, Width, Depth), e.g., image data<br />**例子**：一个 32x32 的 RGB 图像的输入形状为 (32, 32, 3)。 | 1D vector, all input features flattened into a single vector<br />**例子**：一个 32x32 的 RGB 图像会被展开成长度为3072 的一维向量。 |
| **Structure**                      | Includes convolutional layers, pooling layers, and fully connected layers | Primarily composed of fully connected layers (input layer, hidden layers, output layer) |
| **Parameter Sharing**              | Convolutional kernels share parameters across the input data, reducing the number of parameters | Each neuron has independent weights, no parameter sharing    |
| **Connection Pattern**             | Sparse connections, convolution operations connect only to local regions | Fully connected, each neuron in one layer connects to all neurons in the previous layer |
| **Feature Extraction**             | Automatically extracts local features, excels at capturing spatial information | Requires manual feature design, struggles with spatial information |
| **Spatial Invariance**             | Good at handling translation, scaling, and rotation invariance | Sensitive to spatial transformations in the input data       |
| **Computational Efficiency**       | Parameter sharing and sparse connections enhance efficiency, suitable for large-scale data | Fully connected layers have high computational cost and many parameters |
| **Application Domains**            | Image classification, object detection, image segmentation, video processing, natural language processing, etc. | Data classification, regression analysis, and other structured data tasks |
| **Handling High-Dimensional Data** | Superior in handling high-dimensional data (e.g., images)    | Less efficient with high-dimensional data, requires flattening into 1D vector |
| **Hierarchical Feature Learning**  | Convolutional layers extract features hierarchically from low-level to high-level | Lacks natural hierarchical feature learning, requires manual design of multiple hidden layers |
| **Training Time**                  | Generally shorter, especially for high-dimensional data      | Longer, particularly when dealing with large-scale and high-dimensional data |

<u>视频笔记: CNN的框架</u>

每次卷积结束后需要加上一个非线性变换（和bp神经网络一致），执行几次卷积得到一个比较大的特征图就进行一次池化，最后的结果要转化为分类的概率值（全连接层）。

由于全连接层参数不能为3维，需要把处理过后的特征图拉成一个特征向量，将该向量转化为5分类的概率值。

带参数计算才能称作层（卷积层和全连接层）。



## 卷积层 (Convolutional Layer)

### 卷积层主要特点

1. **卷积操作**：
   - 卷积层通过卷积核（convolutional kernels/filters）在输入数据上进行滑动窗口操作。每个卷积核是一个小的权重矩阵，通常比输入数据的尺寸小。
   - 卷积核在输入数据上滑动，通过点积计算得到特征图（Feature Map），这个过程类似于图像处理中的滤波操作(filtering in image processing)。

2. **参数共享**：
   - 卷积核在整个输入数据上共享相同的参数，这大大减少了模型的参数数量，提高了计算效率，并降低了过拟合的风险。

3. **局部连接**：
   - 卷积层中的每个神经元只与上一层的局部区域相连接，而不是像全连接层那样与所有神经元相连接。这种局部连接使得卷积层能够捕捉局部特征。

4. **非线性激活**：
   - 卷积操作的输出通常会经过非线性激活函数（如ReLU）处理，以引入非线性能力，使得网络可以学习更复杂的特征。

### 工作原理

1. **卷积核初始化**：
   - 卷积核是一个小矩阵，初始化时包含随机的权重值。卷积核的尺寸通常是固定的，如3x3、5x5等。

2. **滑动窗口操作**：
   - 卷积核在输入数据上按一定的步长（Stride）滑动。每次滑动时，卷积核与对应的输入子区域进行元素级的点积运算，并将结果相加，生成一个单一的输出值。

3. **生成特征图**：
   - 滑动窗口操作完成后，所有的输出值组成特征图，特征图反映了输入数据的局部特征。

4. **添加偏置和激活**：
   - 卷积操作的结果可以加上一个偏置值（Bias），然后通过激活函数（如ReLU）进行非线性变换，得到最终的输出。

### 数学表示

假设输入数据为 $I$，卷积核为 $K$，则卷积操作可以表示为：

$$
(I * K)(i, j) = \sum_{m}\sum_{n} I(i + m, j + n) \cdot K(m, n)
$$

其中，$(I * K)(i, j)$ 表示卷积操作在位置 $(i, j)$ 处的输出值，$I(i + m, j + n)$ 是输入数据在 $(i + m, j + n)$ 处的值，$K(m, n)$ 是卷积核在 $(m, n)$ 处的权重。

[视频解释](https://www.bilibili.com/video/BV1zF411V7xu/?p=3)

### 卷积层的作用

1. **特征提取**：卷积层能够有效地提取输入数据的局部特征，如边缘、纹理等。
2. **减少参数**：通过参数共享和局部连接，卷积层显著减少了模型的参数数量，使得模型更高效。
3. **增强鲁棒性**：卷积层对输入数据的平移、缩放和旋转具有一定的鲁棒性，这对于图像处理任务尤为重要。

卷积层是卷积神经网络中至关重要的一部分，通过层层卷积，可以逐步提取输入数据的高层次特征，使得模型能够胜任复杂的任务，如图像分类、目标检测等。

### 特征图

可以是多个，用多个convolutional kernel/filters就可以。

To determine the size of the feature map (output dimensions) produced by a convolutional layer given an input, you can use the following formula:

$ \text{Output Size} = \frac{(\text{Input Size} - \text{Kernel Size} + 2 \times \text{Padding})}{\text{Stride}} + 1 $

**Example: 32x32 Image with 3x3 Kernel, No Padding, and Stride of 1**

1. **Input Size $H_{in}, W_{in}$**: The height and width of the input image. In this case, both are 32.

2. **Kernel Size $K$**: The size of the convolutional kernel. Here, the kernel is 3x3, so $ K = 3 $.

3. **Padding $P$**: The number of pixels added to the border of the input. If no padding is used, $ P = 0 $.

4. **Stride $S$**: The number of pixels by which the kernel moves over the input image. In this case, $ S = 1 $.

Plugging these values into the formula:

$$ \text{Output Height} = \frac{(32 - 3 + 2 \times 0)}{1} + 1 = \frac{29}{1} + 1 = 30 $$

$ \text{Output Width} = \frac{(32 - 3 + 2 \times 0)}{1} + 1 = \frac{29}{1} + 1 = 30 $

Therefore, the output feature map size would be 30x30.



### 卷积层涉及参数

- 滑动窗口步长(Stride)
- 卷积核尺寸
- 边缘填充(padding): 使边界点被更公平地对待
- 卷积核个数



## 池化层(Pooling Layer)

池化层（Pooling Layer）是卷积神经网络（CNN）中的一个关键组件，主要**用于减小特征图的尺寸（Downsampling）**，从而减少参数的数量和计算量，并帮助防止过拟合。池化层通过对输入特征图应用某种池化操作来实现这一点。常见的池化操作包括最大池化（Max Pooling）和平均池化（Average Pooling）。

### 最大池化（Max Pooling）
最大池化在池化窗口内选择最大的值作为输出。例如，如果使用大小为2x2的池化窗口，则在2x2的区域内选择最大值并将其作为该区域的输出。这样做的目的是**保留最显著的特征**，并对小的平移和形变具有一定的鲁棒性。

### 平均池化（Average Pooling）
平均池化在池化窗口内计算平均值作为输出。同样地，如果使用大小为2x2的池化窗口，则在2x2的区域内计算平均值并将其作为该区域的输出。平均池化更倾向于**保留整体特征，而不是最显著的特征**。

### 池化层的参数
池化层主要有以下几个参数：
1. **池化窗口的大小（Kernel Size）**：决定了每次池化操作的区域大小。
2. **步幅（Stride）**：决定了池化窗口移动的步长，影响输出特征图的大小。
3. **填充（Padding）**：决定是否在输入的边缘添加填充值，以控制输出特征图的尺寸。

### 作用
1. **降维和减少计算量**：通过池化操作减小特征图的尺寸，减少参数数量和计算量。
2. **防止过拟合**：减少模型参数数量，有助于防止过拟合。
3. **提高平移不变性**：对输入图像的小的平移和形变具有一定的鲁棒性，提高模型的泛化能力。

池化层在卷积神经网络中通常与卷积层交替使用，以逐步减少特征图的尺寸并提取高层次特征。



## VGG

<u>视频笔记：VGG网络</u>

卷积大小都是3 x 3，经过池化层后会损失一部分信息，在下一次卷积过程中使得特征值翻倍（用特征图的个数来弥补长宽的损失）。


### 感受野（Receptive Field）

在卷积神经网络（Convolutional Neural Networks, CNNs）中，感受野（Receptive Field）指的是网络中某一层的一个神经元在输入图像上对应的区域大小。更具体地说，一个卷积神经元的感受野是指它能够“看到”的输入数据的范围。

以下是一些关键点：

1. **局部连接**：在CNN中，卷积层的每个神经元只与前一层的一个小局部区域连接，而不是像全连接层那样与所有神经元连接。这个小局部区域就是当前层神经元的感受野。

2. **堆叠层**：随着网络层数的增加，感受野会逐层增加。例如，第一个卷积层的感受野可能是3x3，但经过多个卷积层和池化层的处理后，深层的感受野会变得更大。

3. **感受野的计算**：感受野大小取决于卷积核的大小、池化层的大小以及步幅（stride）。通过逐层累积计算，可以确定深层神经元对应的输入图像的区域。

4. **重要性**：较大的感受野使得神经元能够捕捉更广泛的图像信息，适用于捕捉全局特征；而较小的感受野则适用于捕捉局部特征。选择适当的感受野大小对于网络的性能和特定任务的效果至关重要。

<u>Q: 感受野是当前层受最初input的影响范围吗</u>

是的，在卷积神经网络（CNN）中，感受野是指当前层的神经元受到最初输入（即原始图像）的影响范围。具体来说，某一层的一个神经元的感受野描述了在输入图像上它所“看到”或受到影响的区域大小。

以下是详细解释：

1. **局部连接和卷积核**：在卷积层中，每个神经元只与前一层的一个局部区域连接。例如，假设卷积核（滤波器）的大小是3x3，那么每个神经元在当前层会接收到前一层3x3区域的输入。

2. **感受野的递归计算**：随着网络层数的增加，每一层的感受野会逐渐增大。例如，如果第一层的卷积核是3x3，那么第一层的每个神经元的感受野就是3x3。如果第二层的卷积核也是3x3，那么第二层的每个神经元的感受野是前一层的3x3区域中的每一个点各自的3x3区域的组合，即5x5。这个过程会递归进行，从而逐渐扩大感受野。

3. **步幅和池化层的影响**：步幅（stride）和池化（pooling）操作也会影响感受野的大小。步幅大于1会跳过一些输入位置，导致感受野增长得更快。而池化层（如最大池化或平均池化）会将输入的感受野进一步扩大。

4. **感受野的总大小**：通过计算每一层的感受野，可以确定网络中任意层的神经元在输入图像上所对应的区域范围。这个范围即为该层神经元的感受野。

<u>Q: 如果堆叠3个3 x 3的卷积层，并且保持stride=1，其感受野就是 7 x 7 的了，这跟使用7 x 7的卷积核结果一样，那为什么非要堆叠3个小卷积呢?</u>

假设输入大小都是hwc，并且都使用c个卷积核(得到c个特征图)，可以来计算一下其各自所需参数数:

- 一个7 x 7卷积核所需参数数 $ = C \times (7 \times 7 \times C) = 49 C^2$

- 3个3 x 3卷积核所需参数数 $ = 3 \times C \times (3 \times 3 \times C) = 27 C^2$

可以看出:

1. 堆叠小的卷积核所需的参数数更少一些。

2. 用堆叠小的卷积核卷积过程更多，特征提取也会更细致，加入的非线性变换(ReLu)也随著增多，还不会增加权重参数个数。

**这就是VGG网络的基本出发点，用小的卷积核来完成特征提取操作。**

<u>Q: VGG中卷积核都是 3 x 3 的吗</u>

在VGG网络中，绝大部分的卷积核确实都是 3 x 3的。这种设计有几个关键的优势：

1. **减少参数数量**：多个 3 x 3 的卷积层相比于一个大卷积核（如 7 x 7）来说，参数数量更少，从而减少了模型的复杂性。
2. **增加非线性**：每个 3 x 3 卷积层后面通常会接一个非线性激活函数，这样可以增加模型的非线性能力，从而能够学习到更复杂的特征。
3. **提取细节特征**：小的卷积核能够更细致地提取特征，捕捉到更小的细节信息。

然而，虽然大部分的卷积核是 3 x 3，在网络的最开始和结束部分，VGG也会使用一些不同大小的卷积核，比如 1 x1 和 2 x 2 的卷积核来调整通道数量和进行降维。



## Resnet

并不是层数越多越好，在堆叠层数时可能有不好的进行影响，需要设计一个方案进行剔除。

可以理解为经过卷积层后学的不好的覆盖了，重在重新构建堆叠原始学好的那部分，即使之前不好的部分有影响，但只要学习使得该部分权值为0即可（好的我用，不好的我权重为0，相当于白搞，也就是对我有用的我加上，没用的我也加上但是不影响，最后效果至少一定不会比原来更差）。



## Data Augmentation

用来增加train的数据量，比如翻转图片、旋转图片、放大/缩小图片，[实现的时候](https://www.bilibili.com/video/BV1zF411V7xu?p=18)可以用`pytorch`的`transforms`模块
