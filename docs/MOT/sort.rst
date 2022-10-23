===========
SORT
===========

.. include:: ../tools.rst


**Abstract**

本文 comtribution:
This paper explores a pragmatic(实用的) approach to multiple object tracking.

检测质量被视为影响跟踪性能的关键因素.
To this end, detection quality is identified as a key factor influencing tracking performance, where changing the detector can improve tracking by up to 18.9%.

仅仅用了 Kalman Filter and Hungarian algorithm 但达到 SOTA 效果


**Introduction**

与许多基于批处理的跟踪方法 [1, 2, 3] 相比，这项工作主要针对在线跟踪，其中仅将 **前一帧** 和 **当前** 帧的检测呈现给跟踪器.


The methods employed by this paper were motivated(受...启发) through observations made on a recently established visual MOT benchmark

- Firstly, there is a resurgence(重新兴起) of mature(成熟) data association techniques including Multiple Hypothesis Tracking (MHT) [7, 3] and Joint Probabilistic Data Association (JPDA) [2] which occupy(占据) many of the top positions of the MOT benchmark. 

- Secondly, the only tracker :blue:`[` that does not use the Aggregate Channel Filter (ACF) [8] detector :blue:`]` is also the top ranked tracker, suggesting that detection quality could be holding back the other trackers. (检测质量影响跟踪的性能)

- Furthermore, the trade-off between accuracy and speed appears quite pronounced, since the speed of most accurate trackers is considered too slow for realtime applications. (大部分精确跟踪器太慢了)

this work explores how simple MOT can be and how well it can perform. (本文的方法简单的一批)


Keeping in line with Occam’s Razor(与奥卡姆剃刀原理一致), appearance features beyond the detection component are ignored in tracking and only the bounding box position and size are used for both motion estimation and data association.
在检测组件之外的外观特征将被忽略, 只有边界框的位置和size将被用来运动估计与数据关联.

Furthermore, issues regarding short-term and long-term occlusion are also ignored, as they occur very rarely and their explicit treatment introduces undesirable complexity into the tracking framework.
关于长短期遮挡的问题也被忽略了, 因为其发生次数少, 并且考虑的话会引入难以想象的复杂度.


We argue that incorporating complexity in the form of object re-identification adds significant overhead into the tracking framework, potentially limiting its use in realtime applications.
目标重识别会增加跟踪框架的开销, 潜在地限制实时应用

This design philosophy is in contrast to many proposed visual trackers that incorporate :blue:`[` a myriad of :blue:`]` :green:`(无数的)`  components to handle various edge cases and detection errors.
这一设计理念与众多之前的 tracker 想抵触, 之前的 tracker 引入无数的组件来处理众多不常出现的情况以及检测出现的异常.

This work instead focuses on efficient and reliable handling of the common frame-to-frame associations.
相反, 该工作主要是进行有效和可靠地处理帧与帧之间的通用 (目标) 匹配. 与其做一些针对检测错误更加鲁棒的工作, 本工作偏向于直接用最先进的检测器.

同时, 本工作直接使用 Kalman 滤波去做运动预测, 用 Hungarian 算法去做数据关联.

This minimalistic formulation of tracking facilitates(促进) both efficiency and reliability for online tracking.
这种简约的跟踪方法提高了在线跟踪的效率和可靠性.


**LITERATURE REVIEW**

Traditionally MOT has been solved using Multiple Hypothesis Tracking (MHT) [7] or the Joint Probabilistic Data Association (JPDA) filters [16, 2], which delay making difficult decisions while there is high uncertainty over the object assignments. 
当目标匹配存在高度不确定性时，这会延迟做出艰难的决定(或者说速度会很慢)。

The combinatorial complexity of these approaches is exponential 
:red:`[` in the number of tracked objects :red:`]`
:blue:`[` making them impractical for realtime applications in highly dynamic environments :blue:`]`. 随着跟踪目标数量上的增长, 这些方法的组合复杂度是指数的, 使得在高动态环境下, 实时跟踪是不现实的.



Many online tracking methods aim to build appearance models of either the individual objects themselves or a global model  through online learning. 
许多跟踪器通过在线学习, 去构建外观模型, 要么构建单个目标个体本身的外观模型, 要么一个构建一个全局外观模型.

In addition to appearance models, motion is often incorporated(被引入) to assist associating detections to tracklets.

本文是受到 Geiger 老哥们的启发, 他们的方法是使用匈牙利算法进行两个阶段的任务, 而我们的工作只有一个阶段, 并且很简单.

- 他们的第一步是将相邻帧的目标进行关联从而形成 tracklets(我翻译成轨迹), 关联过程中的的 affinity matrix (其实就是相似度矩阵) 是通过 geometry 与 appearance 信息联合生成的.
- 然后，再次使用 geometry 和 appearance 线索，将轨迹相互关联以桥接由遮挡引起的破碎轨迹。
- 但是这种两步关联将这种限制为批量计算(我觉的就是并行的意思)。


**METHODOLOGY**
