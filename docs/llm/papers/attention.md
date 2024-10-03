---
sidebar_position: 4
title: Attention Is All You Need
---

## Paper Summary

### Intro

之前需要按时序做运算，现在可以用attention，并行度比较高，所以能得到比之前更好的结果。

### Background

如果两个像素隔得远的话，CNN需要很多层卷积一层一层上去才可以把两个像素合并起来，但是attention可以看到所有像素，所以一层就可以把整个序列给你看到。
卷积比较好的是可以做多个输出通道，也想要这个机制，就提出了multi-head attention，可以模拟CNN的多个输出通道。

Transformer是第一个只依赖self-attention的模型，不依赖sequence-aligned RNN or convolution。

### Model Architecture

encoder: maps an input sequence of symbol representations `(x_1, ..., x_n)` to a sequence of continuous representations `z = (z_1, ..., z_n)`

decoder:  Given z, the decoder then generates an output sequence `(y_1, ..., y_m)` of symbols one element at a time.

encoder可以一次看到全部的句子 `(x_1, ..., x_n)` , 但是decoder一次只能看到一个, `auto-regressive`输入又是你的输出，比如 `y_1` 会是 `y_2` 的输入

![img](https://res.cloudinary.com/diuxkoxa8/image/upload/v1716615233/Screen_Shot_2024-05-25_at_1.28.22_PM_yznyz5.png)

## References

[original paper](https://arxiv.org/abs/1706.03762)

[精读](https://www.bilibili.com/video/BV1pu411o7BE)
