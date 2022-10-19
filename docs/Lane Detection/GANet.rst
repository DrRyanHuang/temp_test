===========
GANet
===========

.. include:: ../tools.rst



Lane detection requires predicting complex topology(拓扑) shapes of lane lines and distinguishing different types of lanes simultaneously.

Due to the slender shapes of lane lines and the need for instance-level discrimination, it is crucial to formulate lane detection task appropriately.
由于车道线很长的形状以及需要区分每个车道线实例, 所以需要选择合适的方案.

.. image:: ../images/1.lane_detec_anch_vs_kpts.png


Anchor-based
------------

In Figure 1a, Similar to object detection, a group of straight lines with various orientations are defined
as anchors. Points on anchors are regressed to lane lines by predicting the offsets between anchor points and lane
points.
(将线直接定义为 Anchors 我感觉这个很不错)

Afterward, Non-Maximum Suppression (NMS) is applied to select lane lines with the highest confidence.
(有些好奇, 这个NMS怎么实现呢?)

优缺点：车道线实例区分容易, 但是由于预定义 Anchor 问题, 缺少灵活性 
Although this kind of method is efficient in lane discrimination, it is inflexible because of the predefined anchor shapes.
:red:`[` 真的吗, 我不信, 这多灵活啊 :red:`]`

The strong shape prior limits the ability of describing various lane shapes, resulting in sub-optimal performances of
these methods(导致性能不佳).


keypoint estimation and association
------------------------------------

To describe complex shapes of lane lines flexibly, Qu et al. [21] propose to formulate lane detection as a keypoint estimation and association problem, which takes a bottom-up design as illustrated in Figure 1b.

Concretely(具体来说), lanes are represented with a group of ordered key points evenly sampled in a sparse manner.
车道线用一组 :blue:`[` 以稀疏方式均匀采样的有序关键点 :blue:`]` 表示.

Each key point is associated with its neighbours by estimating the spatial offsets between them.
:blue:`(step by step 的形式进行预测, 最终整合成一条线)`

优缺点：灵活但低效和耗时 | 同时错误容易累加
Though keypointbased methods are flexible on the shape of lane lines, 

- it is inefficient and time-consuming to associate only one keypoint to its belonged lane line at each step.
- Besides, the point-by-point extension of keypoints is easy to cause error accumulation due to the lack of global view.


GANet
-----

we formulate the lane detection problem from a new keypoint-based perspective(视角) where each keypoint is directly regressed to its belonged lane. (每个关键点直接回归所属车道)

.. image:: ../images/1.GANet_LFA.png

As illustrated in Figure 1c, each lane line is represented uniquely with its starting point, which is easy to determine without ambiguity.
如图 1c 所示，每条车道线都以其起点唯一地表示，这很容易确定，没有歧义。

:blue:`(那我接下来只需要预测每个KPT到起点的偏移即可, 下一段用了 associate 这个词)`

To associate a keypoint properly, we estimate the offset from the keypoint to its corresponding starting point.

:blue:`(那我接下来只需要预测每条线的起点了对不? )`

:red:`(但原文不是这样做的, 他们的方法暂时无需预测起点)` :
Keypoints :blue:`[` whose approximated starting points fall into the same neighborhood area :blue:`]` will be assigned to the same lane line instance, thus separating keypoints into different groups.
全图每个车道线关键点都能通过其预测的偏移量反推其起点, 起点距离比较近的将被划分为同一条线.

这里总结一下他们的优点：

- Our assignment of keypoints to their belonged lanes is independent of each other and makes the parallel implementation feasible, which greatly improves the efficiency of postprocessing. 
- (每一个KPT分配给他们所属的车道线这个过程是互相独立的, 故可并行实现)

- Besides, the keypoint association is more robust to the accumulated single-point errors since each keypoint owns a global view.
- (此外, 由于每个KPT都拥有全局视图, 因此关键点关联对累积的单点误差更加鲁棒)
- :blue:`[` 低情商: 每个都预测一下起点 :blue:`]`
- :blue:`[` 高情商: 《global view》 :blue:`]`

:red:`step by step 预测 => 找一个通用的基点预测 (缓解单点误差)`


这里再说一下他们遇到的问题和解决方式:

Although keypoints belonging to the same lane line are integrated during post-processing, it is important to ensure the correlations between adjacent points in order to obtain a continuous curve.
虽然属于同一车道线的关键点在后处理过程中被整合，但重要的是要确保相邻点之间的相关性以获得连续曲线。

:green:`可能单独使用以上策略, 获得的曲线不连续, 于是说明了接下来引出来的方法(不知道消融实验有没有)`

To this end, we develop a local information aggregation module named Lane-aware Feature
Aggregator (LFA) to enhance the correlations between adjacent keypoints.
开发局部信息聚合模块(LFA)来加强相邻关键点之间的相关性.

To adapt to the slender and long shapes of lanes, we :red:`[` modify the sampling positions of the standard 2D deformable convolution :blue:`(` by predicting offsets to adjacent points :blue:`)` :red:`]` **to sample within a local area** on the lane each time.

为了适应车道的细长形状，我们通过预测到相邻点的偏移量来修改标准二维可变形卷积[3]的采样位置，以便每次在车道上的局部区域内进行采样。
:red:`(此处需要看代码)`

We further add an auxiliary loss to facilitate estimating the offset predicted on each key point.
我们进一步添加了一个辅助损失，以方便估计在每个关键点上预测的偏移量。

Our LFA module complements(补充) the global association process to enable both local and global views(我们的 LFA 模块补充了全局关联过程，以启用局部和全局视图), which is essential for dense labeling tasks like lane detection.


Contributions
---------------

- GANet
- LFA module (enhance correlations among adjacent keypoints to supplement(补充) local information)
- benchmarks 上获得高效结果


相关工作不想看了
------------------

- Segmentation-based methods
- Detection-based methods
- Keypoint-based methods