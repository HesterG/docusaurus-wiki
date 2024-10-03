---
sidebar_position: 3
title: Deep Residual Learning for Image Recognition
---

## Paper Summary

### Abstract

The abstract introduces the residual learning framework to ease the training of deep neural networks, which reformulates layers as learning residual functions instead of directly fitting desired mappings. This approach allows for the creation of deeper networks, significantly improving performance on benchmarks like ImageNet and COCO.

### 1. Introduction

**Background**

- **Deep Convolutional Neural Networks (CNNs)**: The section starts by highlighting the significant breakthroughs achieved by deep convolutional neural networks (CNNs) in image classification tasks. These networks integrate low, mid, and high-level features through multiple layers, creating robust representations for image recognition.

**Importance of Depth**

- **Network Depth**: Emphasizes that network depth is crucial for achieving high accuracy in various visual recognition tasks. Deeper networks tend to perform better as they can learn more complex features.
- **Leading Models**: References successful deep models like VGG nets and GoogleNet, which have depths ranging from 16 to 30 layers, demonstrating the effectiveness of increased depth in neural networks.

**Challenges**

- **Vanishing/Exploding Gradients**: Discusses the well-known problem of vanishing and exploding gradients, which can make training deep networks difficult. These issues can hamper the convergence of networks during training.

  当网络很深的时候会出现梯度爆炸或者消失，解决他的一个方法是weights在初始化的时候做得好一点，还有做<u>batch normalization</u> (通过对每个小批量的数据进行标准化，减小均值，除以标准差，再进行缩放和平移来加速训练。它允许更高的学习率，提高训练稳定性，并有正则化效果。)

- **Degradation Problem**: Introduces the degradation problem, where increasing network depth beyond a certain point leads to higher training errors. This is not due to overfitting but because deeper networks become harder to optimize.

**Addressing the Challenges**

- **Residual Learning Framework**: Proposes a solution to the degradation problem by introducing a residual learning framework. Instead of directly fitting a desired mapping, the network learns **residuals**. The core idea is to reformulate the layers to learn residual functions with reference to the layer inputs, making it easier to optimize deeper networks.

  核心就是residual connections

- **Shortcut Connections**: Explains the use of shortcut connections that perform identity mapping, which allows the gradient to flow more easily through the network. These shortcuts do not add extra parameters or computational complexity, making them efficient.

**Contributions**

- **Empirical Evidence**: The introduction concludes by stating that comprehensive empirical evidence shows that residual networks (ResNets) are easier to optimize and can gain accuracy from increased depth. The authors evaluate these networks on large-scale datasets like ImageNet and CIFAR-10, showing significant improvements in performance.

### 2. Related Work

- **Residual Representations**: Reviews existing methods like VLAD and Fisher Vector that use residuals for encoding in image recognition tasks.
- **Shortcut Connections**: Discusses the history and use of shortcut connections in neural networks, including highway networks that use gating functions.

### 3. Deep Residual Learning

#### 3.1. Residual Learning

- **Concept**: Introduces the concept of residual learning where the network learns residuals $ F(x) = H(x) - x $ instead of directly learning the mapping $ H(x) $.
- **Motivation**: The hypothesis is that it is easier to optimize the residual mapping than the original mapping.

#### 3.2. Identity Mapping by Shortcuts

- **Formulation**: Describes the mathematical formulation of residual learning using identity mappings through shortcut connections.
- **Advantages**: Explains that identity shortcuts introduce no additional parameters or computational complexity, making them efficient for practical use.

#### 3.3. Network Architectures

- **Plain Networks**: Describes the baseline architecture inspired by VGG nets, with layers mostly consisting of 3×3 filters.
- **Residual Networks**: Converts plain networks into residual networks by adding shortcut connections, leading to different variants like ResNet-34, ResNet-50, ResNet-101, and ResNet-152.

#### 3.4. Implementation

- **Training**: Details the training procedures, including data augmentation, initialization, and optimization techniques used for training the networks on ImageNet.

### 4. Experiments

#### 4.1. ImageNet Classification

**Experiment Setup**

- **Dataset**: The experiments were conducted on the ImageNet 2012 classification dataset, which comprises 1.28 million training images and 50,000 validation images across 1,000 classes.
- **Evaluation Metrics**: The models were evaluated using top-1 and top-5 error rates on the validation set. Final results were also obtained on the test set.

**Plain Networks**

- **Baseline Architectures**: The authors first evaluated plain (non-residual) networks with 18 and 34 layers, which served as baselines for comparison.
- **Design**: These plain networks followed a similar architectural philosophy to VGG nets, using mostly 3x3 convolutional layers.

**Results for Plain Networks**

- **Validation Error**: The 34-layer plain network exhibited higher validation error compared to the 18-layer plain network, demonstrating the degradation problem, where increasing depth leads to higher error rates.
- **Training Error**: The training errors of the 34-layer plain network were also consistently higher than those of the 18-layer network throughout the training process.

**Residual Networks (ResNets)**

- **Introduction of Shortcuts**: The authors introduced shortcut connections to the plain networks to create residual networks (ResNets) with 18 and 34 layers.

  Resnet的核心就是这些shortcut connections

- **Identity Mapping**: These shortcuts performed identity mapping, which added no extra parameters or computational complexity.

**Results for Residual Networks**

- **Performance Improvement**: The 34-layer ResNet significantly outperformed both the 18-layer ResNet and the 34-layer plain network, showing lower training and validation errors.
- **Error Rate Reduction**: The 34-layer ResNet reduced the top-1 error rate by 3.5% compared to its plain counterpart.

**Deeper Architectures**

- **Extended Depth**: The authors extended the ResNet architecture to deeper networks with 50, 101, and 152 layers.

- **Bottleneck Design**: For deeper networks, they used a bottleneck design, where each residual block consists of three layers (1x1, 3x3, 1x1 convolutions) to <u>reduce parameters and computational cost</u>.

  34到50主要是因为加了这个bottleneck的设计

**Implementation Details**

- **Data Augmentation**: The training images underwent standard data augmentation techniques, including random cropping and horizontal flipping.
- **Normalization and Initialization**: Batch normalization was applied after each convolutional layer and before activation. The weights were initialized following previous methods.
- **Optimization**: The networks were trained using stochastic gradient descent (SGD) with a mini-batch size of 256. The learning rate started at 0.1 and was divided by 10 when the validation error plateaued. The models were trained for up to 60 epochs.

**Final Results**

- **152-Layer ResNet**: The 152-layer ResNet achieved a top-5 error rate of 3.57% on the ImageNet test set, setting a new state-of-the-art performance.
- **Ensemble Models**: An ensemble of these residual networks further improved performance, showcasing the robustness and generalization capabilities of ResNets.

**Conclusion**

- The experiments demonstrated that the residual learning framework significantly improves the training of very deep networks, allowing them to achieve superior performance on large-scale image classification tasks like ImageNet.
- Residual networks effectively address the degradation problem, showing their scalability to extremely deep architectures and setting new performance benchmarks.

#### 4.2. CIFAR-10 and Analysis

- **Setup**: Explains the setup for experiments on the CIFAR-10 dataset.
- **Results**: Presents the performance of residual networks on CIFAR-10, demonstrating their ability to be trained to very deep levels (up to 1202 layers) without degradation.
- **Analysis**: Analyzes layer responses to show that residual networks generally have smaller responses, supporting the hypothesis that residual functions are easier to optimize.

#### 4.3. Object Detection on PASCAL and MS COCO

- **Setup**: Describes the setup for object detection tasks using Faster R-CNN and compares the performance of ResNet-101 with VGG-16.
- **Results**: Shows significant improvements in object detection accuracy, particularly on the COCO dataset, attributed to the deep residual networks.

### 5. Conclusion

- **Summary**: Recaps the main findings, emphasizing the success of residual learning in training deep networks.
- **Impact**: Highlights the impact of residual networks in advancing the state of the art in image recognition and object detection tasks.

ResNet通过引入残差块和快捷连接，有效保持了梯度传递，解决了梯度消失的问题。这使得随机梯度下降法（SGD）能够更有效地更新参数，加快训练速度，同时提高模型性能，因为梯度足够大，参数更新更迅速，网络收敛更快。

## References

[original paper](https://arxiv.org/pdf/1512.03385)
