===========
Bytetrack
===========

.. include:: tools.rst




存在即合理

State-of-the-art MOT methods need to deal with true positive / false positive trade-off in detection boxes to eliminate low confidence detection boxes.
Filtering out these objects causes irreversible(不可逆的)) errors for MOT and brings non-negligible missing detection(无法忽略的漏检) and fragmented trajectories(碎片化轨迹).

In this paper, we identify that [the similarity with tracklets] provides a strong cue to distinguish the objects and background in low score detection boxes.
轨迹之间的相似性为 :green:`[` 区分 :red:`[` 低置信度边界框中的物体 与 背景 :red:`]` :green:`]` 提供了有效的帮助.

To the best of our knowledge, 据我所知

As the integrating topic of object detection and association, a desirable solution to MOT is never a detector and the following association; besides, well-designed of their junction area is also important.


Similarity metrics
===================

MOT作为检测和关联的综合课题, MOT的理想解决方案绝不单是 :red:`[` 检测 :red:`]` 与 :red:`[` 关联 :red:`]`, 如何设计二者的结合也很重要.

The innovation(创新) of BYTE lies in the junction area of detection and association, where low score detection boxes are bridges to boost both of them.

notable(值得注意的) improvements are achieved on almost all the metrics including MOTA, IDF1 score and ID switches.

Data association is the core of multi-object tracking, which first computes the similarity between tracklets and detection boxes and leverage different strategies to match them according to the similarity.

SORT [6] combines location and motion cues in a very simple way. It first adopts Kalman filter [29] to predict the location of the tracklets in the new frame and then computes the IoU between the detection boxes and the predicted boxes as the **similarity**.

Some recent methods [59, 71, 89] design networks to learn object motions and achieve more robust results in cases of large camera motion or low frame rate.

- Location and motion similarity are accurate in the short-range matching.
- Appearance similarity are helpful in the long-range matching. 

An object can be re-identified using appearance similarity after being occluded for a long period of time. Appearance similarity can be measured by the cosine similarity of the Re-ID features.

DeepSORT [70] adopts a stand-alone(独立) Re-ID model to extract appearance features from the detection boxes. Recently, joint detection and Re-ID models [33, 39, 47, 69, 84, 85] becomes more and more popular because of their simplicity and efficiency.

Similarity metrics
===================

All these methods focus on how to design better association methods. However, :pink:`[` we argue that :pink:`]` the way detection boxes are utilized :pink:`[` determines the upper bound :pink:`]`  of data association and we focus on how to make full use of detection boxes from high scores to low ones in the matching process.

所有这些方法都集中在如何设计更好的关联方法上。 然而，:pink:`[` 我们认为 :pink:`]` 检测框的使用方式决定了数据关联的 :pink:`[` 上限 :pink:`]`，我们关注的是如何在匹配过程中充分利用从高分到低分的检测框。


- We first associate the high score detection boxes to the tracklets. Some tracklets get unmatched because they do not match to an appropriate high score detection box, which usually happens when occlusion, motion blur or size changing occurs.

- We then associate the low score detection boxes and these unmatched tracklets to recover the objects in low score detection boxes and filter out background, simultaneously(同时).


We find it :red:`important` to use IoU alone as the Similarity#2 in the second association because the low score detection boxes usually contains severe(严重的) occlusion or motion blur and appearance features are not reliable.

Thus, when apply BYTE to other Re-ID based trackers [47, 69, 85], we do not adopt appearance similarity in the second association.


在TransTrack的推理过程中引入了Track Rebirth，以增强对遮挡和短期消失的鲁棒性。 具体来说，如果跟踪框不匹配，它将保持为“非活动”跟踪框，直到K个连续帧保持不匹配为止。无效的跟踪框可以与检测框匹配并重新获得其ID。 类似于《Tracking objects as points》，本文选择K = 32。





















:math:`\mathcal{Hello test}`
:math:`\mathtt{Hello test}`
:math:`\mathsf{Hello test}`
:math:`\mathit{Hello test}`
:math:`\mathnormal{Hello test}`
:math:`\mathrm{Hello test}`
:math:`\mathbf{Hello test}`
:math:`\mathbb{Hello test}`