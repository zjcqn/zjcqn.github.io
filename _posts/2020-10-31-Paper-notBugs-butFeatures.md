---
title: 对抗性样本的成因 not Bugs but Features
author: 陈钱牛
date: 2020-10-30 15:33:00 +0800
categories: [Research]
tags: [Paper,Adversarial attack]
math: true
layout: post
typora-root-url: ..
---

Title: Title：Adversarial Examples are not Bugs, they are Features（2019）


![image-20201030233200893](/assets/img/posts/2020-10-31-Paper_not_bugs_but_features/image-20201030233200893.png)

很难解释神经网络是根据什么做预测的，所以更难得知对抗性样本是如何骗过神经网络的。这篇文章将给出了一种对对抗性样本攻击成因的全新的解释。

先前的解释认为对抗性样本攻击是机器模型结构问题，属于训练欠拟合的结果和线性化的结果，只要有更好的训练算法和更大的训练数据集就可以避免这些bug。本文认为所谓“对抗性样本”是源自数据集特征的直接产物，模型只是把数据集的特征给学习了出来。

文章认为机器学习到的图片的特征大于人眼能感知的特征，其中分为**robust**特征和**non-robust**特征。**前者**可以根据人眼能感知的特征来进行分类，能够抵抗微小对抗性扰动；**后者**则是人眼不能感知的特征，它容易受到对抗性攻击而分类错误。







