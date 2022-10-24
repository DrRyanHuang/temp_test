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

Detection
**********

The proposed method is described by 

- the key components of detection, 
- propagating object states into future frames, (将当前目标状态传播到下一帧, 其实就是使用卡尔曼滤波预测下一帧中目标位置)
- associating current detections with existing objects, and 
- managing the lifespan of tracked objects. (管理被跟踪对象的生命周期)

本文为了利用快速发展的基于 CNN 的检测器, 于是使用了 Faster RCNN.

    The first stage extracts features and proposes regions for the second stage which then classifies the object in the proposed region.

Faster RCNN 的一大优势是两阶段参数共享


As we are only interested in pedestrians we ignore all other classes and only pass person detection results with output probabilities greater than 50% to the tracking framework.
(忽略其他类, 只把概率大于 50% 的结果送给跟踪器框架)

原文用 ``FrRCNN(ZF)`` 、 ``FrRCNN(VGG16)`` 和 ``ACF(Aggregate Channel Filter)`` 做了比较:
    we found that the detection quality has a significant impact on tracking performance when comparing the FrRCNN detections to ACF detections.

FrRCNN(VGG16) 的表现使得 ``MDP`` 和 ``SORT`` 的效果都很好.


Estimation Model(估计模型下一帧的位置)
****************************************

We approximate the inter-frame displacements(帧间位移) of each object with a linear constant velocity model which is independent of other objects and camera motion.

每一个目标的状态建模为:

.. math::
    \bf{x} = [u, v, s, r, \dot{u}, \dot{v}, \dot{s}]^T

:math:`u` 和 :math:`v` 是目标中心水平和竖直方向的像素位置, 而 :math:`s` 和 :math:`r` 分别代码边界框的面积与纵横比.

Note that the aspect ratio is considered to be constant !!!

- When a detection is associated to a target, the detected bounding box is used to update the target state where the velocity components are solved optimally via a Kalman filter framework.
- 当新检测结果与目标相关联时，检测到的边界框用于更新目标状态，其中速度分量通过卡尔曼滤波器框架进行求解.

- If no detection is associated to the target, its state is simply predicted without correction using the linear velocity model.
- 如果没有与目标相关联的新检测结果, 则只需使用线速度模型对其状态进行预测, 而无需进行校正.


Data Association
******************

当前的所有目标都 通过上一帧的位置 来估计 当前帧的位置, 之后与当前帧的检测结果进行匹配.
是的, 就是用之前提到的匈牙利算法进行匹配, cost矩阵是已有目标与预测出来的当前位置的IoU距离.

Additionally, a minimum IOU is imposed to reject assignments where the detection to target overlap is less than :math:`IoU_{min}`.
当检测结果与已有目标的IoU小于 :math:`IoU_{min}` 时, 则无需进行匹配.

文章发现边界框的 IoU 距离隐含地处理了由通过的其他目标引起的短期遮挡。

后边这都是啥????? 啥意思这是:

- Specifically, when a target is covered by an occluding object, only the occluder is detected, since the IOU distance appropriately favours detections with similar scale. 
- This allows both the occluder target to be corrected with the detection while the covered target is unaffected as no assignmentm is made.


Creation and Deletion of Track Identities
******************************************************

When objects enter and leave the image, unique identities need to be created or destroyed accordingly.

**Create 轨迹**

本文认为, 任何 重叠率低于 :math:`IoU_{min}` 的检测结果都是一个未跟踪的目标.

The tracker is initialised using the geometry of the bounding box with the velocity set to zero.
轨迹初始化, 用第一帧的边界框位置初始化, 速度设置为0.

Since the velocity is unobserved at this point(此时) the covariance(协方差) of the velocity component is initialised with large values, reflecting this uncertainty. 
(由于此时未观察到速度，因此将速度分量的协方差初始化为较大的值，反映了这种不确定性)


Additionally, the new tracker then undergoes a probationary period where the target needs to be associated with detections to accumulate enough evidence in order to prevent tracking of false positives.
(此外, 新的跟踪器随后会经历一段试用期, 在此期间, 目标需要与检测相关联, 以积累足够的证据, 以防止跟踪误报。) :blue:`[不知道代码中有没有体现]`

**Delete 轨迹**

Tracks are terminated if they are not detected for :math:`T_{Lost}` frames.

This prevents an unbounded growth in the number of trackers and localisation errors caused by predictions over long durations without corrections from the detector.
(避免轨迹的数量无限制增长 同时 避免由于长时间线性预测而没有检测器矫正而引起的错误)

文章中 :math:`T_{Lost}=1` 有两个原因:

- 本文使用的是匀速模型(the constant velocity model), 无法很好的预测出真实状态
- 其次, 本文工作主要是 frame-to-frame tracking, object re-identification 不是本文的工作范围(beyond the scope of this work).
- Additionally, early deletion of lost targets aids efficiency. (早期删除丢失目标有助于提升效率??? 你要不都删了吧, 这效率多高[狗头])


Should an object reappear, tracking will implicitly resume under a new identity. :red:`[倒装句强调]`
如果一个之前小时的目标重新出现, 跟踪将隐式地在一个新Id下出现.

