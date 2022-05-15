---
title: Paper｜FoolImages：DNN视觉和人眼视觉的差异
author: 陈钱牛
date: 2020-10-28 19:33:00 +0800
categories: [Research]
tags: [Paper,Adversarial attack]
math: true
layout: post
typora-root-url: ..
banner:
  video:
  loop: true
  volume: 0.8
  start_at: 8.5
  image: https://bit.ly/3xTmdUP
  opacity: 0.618
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  subheading_style: "color: gold"
---


# Deep Neural Networks are Easily Fooled: High Confidence Predictions for Unrecognizable Images (CVPR 2015)

## Abstract

- It is easy to produce images that are completely unrecognizable to humans, but that DNNs believe to be recognizable objects with **99.99%** confidence

## Introduction 

- we use **evolutionary algorithms** or **gradient ascent** to generate images that are given high prediction scores by CNNs.
- We also find that, for MNIST DNNs, it is not easy to prevent the DNNs from being fooled by retraining them with fooling images labeled as such.

![image-20201029143930269](/assets/img/posts/2020-10-28-Paper_fool_images/image-20201029143930269.png)

## Method 

- DNN models
	- DNN net: AlexNet, 
	- datasets: ImageNet, MNIST
- Generating images with evolution
	- a new evolutionary algorithm called **the multi-dimensional archive of phenotypic elites MAP-Elites**, which performs well on  many classes.
	- two encodings: direct, and indirect (CPPN)
	- CPPN networks start with no hidden nodes, and nodes are added over time, encouraging evolution to first search for simple, regular images before adding complexity





