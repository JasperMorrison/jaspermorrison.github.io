---
layout: post
title: （知识蒸馏）Knowledge Distillation
categories: "AI"
tags: AI 无监督学习 知识蒸馏 KD Knowledge-Distillation
author: Jasper
---

* content
{:toc}

第一次认识知识蒸馏是从模型压缩的角度，使用泛化能力更强的大模型（如Resnet-50），蒸馏出一个小模型（Resnet-18），提升小模型的泛化能力。它的思辨和原理是什么，论文《Distilling the Knowledge in a Neural Network》，给出了答案。为了表达出该技术的思辨过程，尽量保留原文意思。



# Abstract

如何提升性能，简单点，使用各种trick，各式网络结构，不同的数据分布，训练很多模型，比较哪个好就用哪个。然而，在部署阶段，超大网络的代价和计算量是无法接受的。已经被证实，大/多模型的knowledge可以被压缩到single小模型中，我们使用了多种压缩手段完成这种压缩工作。在MNIST数据集，我们获得了不错的结果。我们还介绍了一种新型的组合类型，由 full models 和 specialist models 组成，其中，specialist models 可以明确区分fine-grained classes，而full models会混淆它们。这些 specialist models可以经过独立并行训练来获得。

本文中，generalize，一般化，泛化都是混用的，提升到思辨的高度，它们都是一个意思。

# Introduction

在大规模机器学习中（large-scale machine learning），我们通常在训练阶段和部署阶段使用相似的模型，即使它们的要求并不相同。训练阶段，使用大量的，甚至是冗余的数据，并消耗大量的算力，来提取特征。在部署阶段，却要求低时延和低算力。我们期望训练一个复杂的、表现力强的大模型，并用上各种花哨的手段，如dropout，来提升模型性能。一旦获得大模型，我们可以将其knowledge蒸馏到小模型上，方便部署。而且，这样的蒸馏技术已经证明可行。

在一个给定训练好的，经过学习的，带有固定参数的大模型，一切都固定了，很难直接观察到，如何在修改模型后，仍能保留其knowledge。对knowledge更抽象的认识，它应当是从具体的个体中独立出来的一种表示，是一种从input到output的映射关系。通常来说，大模型训练的目标是，最大化average log probability，也就是概率最大化，以此来区分样本的所属类别。但更深的理解应该考虑到负样本，训练好的模型给所有的样本都赋予了概率值，只是某些比另一些的概率更大而已。这种统一的思考方式，告诉我们，大模型是如何进行一般化的（一般化，说明它的表现力很广，很强）。比如说，一辆宝马被识别为一辆垃圾车的几率很小，但是这种小的几率，也比它被识别为胡萝卜的几率要大（这个小几率是由于模型的一般化能力所给予的，模型有给予正样本高概率的能力，也有给予负样本低概率的能力，generalize well）。

通常被接受的说法是，目标函数被训练为，将正样本的计算结果尽可能的靠近正样本。除此以外，模型被优化为尽可能地可以泛化到新的样本上。这就表面，最好是训练一个泛化能力强的模型（generalize well），然而，训练泛化能力强的模型的方法所需要的信息不是那么容易获得。当我们将大模型蒸馏到小模型上时，小模型能够以大模型同样的方式进行泛化（generalize），它很可能比基于标注样本进行直接训练的泛化能力更强。

从大模型蒸馏出小模型，一种可行方法是使用 soft class probabilities（这个概率信息来自大模型），称为“soft targets”。可以使用相同的数据集，也可以使用子集来进行蒸馏。当soft target拥有高的信息熵时，能比hard target提供更多的信息，在梯度中拥有更低的方差，因此，小模型可以使用更少的数据和更大的学习率。（这里的soft target和hard target，区别在于在计算softmax时，有没有除T，除T后曲线更平滑，方差更小，能保留更多信息）

就像MNIST，模型的结果往往给正确的结果以很高的confidence，但仍然有很多信息隐藏在非常小概率值的信息中，这体现在softtarget函数。比如数字2，会给予非常低的概率值使得它属于数字3和数字7。这个低概率信息是非常有用的，比如，至少，我们知道哪个2更像3，哪个2更像7。但是这个小概率值对交叉熵损失的贡献极小，因为它们基本很接近0。即使它们很有用，但太小终究是个问题，因为我们获取和利用它们。Caruana and his collaborators 另辟蹊径，他们直接使用logits (the inputs to the final softmax)，而不是softmax函数产生的概率值来进行蒸馏，目标函数是最小化大模型与小模型的logits平方距离。而，我们则使用soft targets。

迁移数据集（蒸馏数据集），可以是unlabeled data，也可以是训练阶段使用的train data。我们发现，使用train data效果更好，尤其是我们在目标函数中添加一个small term。（此small term即能使得小模型更积极地预测正例，同时能很好地匹配大模型的soft targets）

# Distillation

![](/images/AI/distillation_probability.png)

Zi：输出层的logits
Pi：logits经过softmax后的概率值
T：a temperature，起到平滑softmax的作用

训练和蒸馏阶段，T被设置得很高，一旦完成蒸馏，小模型的T被设置为1.

带T的softmax称为soft targets，T==1表示hard targets。hard targets 与 soft targats 正好相差1/T^2倍。

![](/images/AI/distillation_loss.png)

直接将蒸馏的损失（梯度）定义为两个概率值的差值，并当T很大时，损失正好等于logits差值/(NT^2)。



（进行中。。。）