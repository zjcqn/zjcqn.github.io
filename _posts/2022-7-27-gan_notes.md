---
layout: post
title: GANs in Action：阅读笔记
author: 陈钱牛
categories: [Research]
tags: [GAN]
math: true
typora-root-url: ..
---

> 内容来自  ***GANs in Action* (《GAN实战》)** 

## DCGAN 深度卷积生成对抗网络
简单介绍，最为最基础的版本演示GAN训练过程。

## MM-GAN/NS-GAN/WGAN
GAN结构的三个核心版本：
- 最大-最小GAN (MM-GAN)
- 非饱和GAN (NS-GAN)
- Wasserstein GAN (WGAN)


## PGGAN 渐进式增长生成对抗网络 （Progressive Growing of GAN）
在顶级机器学习会议ICLR2018上提出时引起了轰动，以至于谷歌立即将其整合为TensorFlowHub中的几个模型之一。这项技术被深度学习的鼻祖之一Yoshua Bengio称赞为“好得令人难以置信”，在其发布后，立即成为学术报告和实验项目的最爱。

NVIDIA研究人员花了两个月才跑出来，说明GAN的设计不是轻松的活啊。  

## CGAN 条件生成对抗网络
通过给定标签控制生成器生成对应的内容，比如把马生成斑马。

## CycleGAN 条件生成对抗网络
输入图片转换图片中的特征（域），例如将苹果变成橙子。

## 对抗样本
- 为什么对抗样本需要限制不可感知，如果取消这一限制并加以利用，比如制作成汽车上的涂鸦等，是否能够造成更大的威胁？