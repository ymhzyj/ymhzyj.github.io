---
layout: post
title: Soft teacher
description: "论文阅读笔记"
last-updated:   05-11-2022 11:29:45 -0400
tag: soft teacher
type: card-img-top
categories: latin text
image: http://placehold.it/750X300?text=Header+Image
type: card-img-top
categories: post ipsum
author: Rui Zhang
card: card-1
---


[End-to-End Semi-Supervised Object Detection with Soft Teacher](https://openaccess.thecvf.com/content/ICCV2021/html/Xu_End-to-End_Semi-Supervised_Object_Detection_With_Soft_Teacher_ICCV_2021_paper.html)的相关内容，文章发表在ICCV2021，下面是本人对于文章的总结：
# #文章摘要
Soft-Teacher提出了一种端到端的半监督目标检测算法，能够逐步提升伪标签的质量，
从而也让整个训练过程受益。算法在目标检测和实例分割任务上都取得了最高的效果，真的
完全监督任务也有较好的精度提升效果。代码在[github](https://github.com/microsoft/SoftTeacher)，提供了基于 mmdetection 实现的目标检测代码。
# 引言
获取有效的标注是深度学习技术发展的瓶颈，因此才需要半监督和自监督技术来利用无
标注数据。针对目标检测任务，基于伪标签的半监督目标检测算法是一类主流的半监督学习
算法。过去基于伪标签的算法通常需要多个阶段的训练过程，第一阶段利用小规模有标签数
据初始化检测器为无标签数据标注伪标签，第二阶段利用伪标签重新训练模型。这类方法的
性能受到初始化过程检测模型性能的显著影响，伪标签质量不佳的情况下，检测性能也会降
低。在一部分工作是希望能够尽量降低错误伪标签的影响，增强对于噪声标签的鲁棒性。
这篇文章为了解决伪标注和两阶段训练过程繁琐的问题，提出了端到端的基于伪标签的
半监督目标检测框架。一个训练的批里包含指定比例的有标签和无标签的数据，同时使用师
生模型进行训练。学生模型负责训练检测任务，教师模型负责更新伪标签，其参数采用学生
模型的指数滑动平均，两个模型相互促进，逐步提升检测性能。并且教师模型不是直接提供
伪标签，而是对由学生网络提供的伪标注进行进一步评估，通过阈值判断候选框是前景/背景，
提高伪标注的可信性，在本文中被称为 Soft teacher。此外针对伪标签还使用了边界框抖动的
方法，首先将学生网络提供的边界框进行随机便宜，然后将偏移的边界框根据教师网络提供
的定位回归，通过测量回归的方差判断当前候选框的可靠性。

# 方法
![方法](assets/img/posts/softteacher.png#pic_center)

算法端到端训练整体流程如图 1 所示，分别包括教师模型和学生模型。学生模型由有标
注数据和伪标注数据监督，使用目标检测的损失函数。伪标注包含两组，分别用于训练分类
和回归任务。教师模型是学生模型的指数滑动平均。
整体的损失函数包括有监督和无监督损失：
$$L=L_s+\alpha L_\mu$$
