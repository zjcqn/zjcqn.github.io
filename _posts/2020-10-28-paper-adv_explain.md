---
title: 对抗性样本攻击的成因 EXPLAINING AND HARNESSING ADVERSARIAL EXAMPLES
author: 陈钱牛
date: 2020-10-28 19:44:00 +0800
categories: [Research]
tags: [Paper,Adversarial Attack]
math: true
layout: post
typora-root-url: ..
---

![image-20201029194457302](/assets/img/posts/2020-10-28-Paper_adv_explain and harnessing/image-20201029194457302.png)

# ABSTRACT

- Early attempts at explaining this phenomenon focused on nonlinearity and overfitting.
- We argue instead that the primary cause of neural networks’ vulnerability to adversarial
	perturbation is their linear nature.
- harnessing method：Using this approach to provide examples for adversarial training

# Introduction

- Szegedy et al. (2014b)[^1] find adversarial examples. **This suggests that adversarial examples expose fundamental blind spots in our training algorithms.**speculative explanations have suggested it is due to **extreme nonlinearity** of deep neural networks, perhaps combined with **insufficient model averaging** and **insufficient regularization** of the purely supervised learning problem.
- show evidence to argue the explanation
	- Linear behavior in high-dimensional spaces is sufficient to cause adversarial examples.
- raise a method to provide an additional regularization: **adversarial training**
- 作者认为非线性才是对抗性样本攻击的成因

# related work

- Those most relevant to Szegedy et al. (2014b)[^1]  include:

	> - Box-constrained `L-BFGS` can reliably find adversarial examples.
	> -  On some datasets, such as ImageNet (Deng et al., 2009)[^2], the adversarial examples were so close to the original examples that the differences were indistinguishable to the human eye.
	> - The same adversarial example is often misclassified by a variety of classifiers with different architectures or trained on different subsets of the training data.
	> - Shallow softmax regression models are also vulnerable to adversarial examples.
	> - Training on adversarial examples can regularize the model—however, this was not practicalat the time due to the need for expensive constrained optimization in the inner loop.

- These results suggest that classifiers based on modern machine learning techniques, even those that obtain excellent performance on the test set, are not learning the true **underlying concepts that determine the correct output label**. Instead, these algorithms have built a Potemkin village (骗人的村庄，形式主义) that **works well on naturally occuring data**, but is exposed as a fake when one visits points in space that do not have high probability in the data distribution.

# The Linear Explanation Of Adversarial Examples


$$
\displaystyle{\omega^T\widetilde x=\omega^Tx+\omega^T\eta}
$$










[^1]: Szegedy C, Zaremba W, Sutskever I, et al. Intriguing properties of neural networks[J]. arXiv preprint arXiv:1312.6199, 2013.
[^2]:Deng J, Dong W, Socher R, et al. Imagenet: A large-scale hierarchical image database[C]//2009 IEEE conference on computer vision and pattern recognition. Ieee, 2009: 248-255.