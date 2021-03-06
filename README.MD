
### [[云上贵州2017]智慧交通预测挑战赛](https://tianchi.aliyun.com/competition/entrance/231598/introduction)
**线上第四名**<br/>
<br/><br/>
这场比赛采用天池特色AB规则，其中A榜大约评测25天，B榜大约评测3天，每天评测1次，最终成绩取A榜和B榜的平均值。<br/><br/>
向官方建议仅取B榜成绩，被无视。<br/><br/>
最终得了线上第四名，因为没有奖金而没有参加答辩（不然大概会降到第五名）。<br/><br/>
这场比赛开始的时候要求大家用数加平台，但资源始终紧缺，于是中途修改规则，允许大家线下生成结果。这场比赛之后我就再也没有见过数加。<br/><br/>
我设计了一个略为复杂的规则，之前也分享过几次，从来没有人耐心地看完过。不过这里还是稍微讲一下吧。<br/><br/>
简单来说，预测一个时间片的平均通行时间，可以用来参考的信息当然包括历史每一天同一时段的平均通行时间，和当天早些时候的平均通行时间，对这两部分数据做常数回归就好。于是我的方案要分成两部分，历史预测和近左预测。<br/><br/>
#### 常数回归<br/>
给定一个序列，和一个损失函数，我们可以找到一个常数使得这个常数在这个序列上的损失最小，这个可以称为「常数回归」。例如，MAE的常数回归值是中值，MSE的常数回归值是均值，MQE的常数回归值非常复杂，一般可以用均值、最小值和最大值的平均值来估算。如果这个序列中不同元素的重要性不同，我们也可以给每个元素加一个权重。<br/><br/>
有时这个损失函数形式较为复杂，或者加了一些权重，我们便不能简单计算出这个权重的值。解决这个问题的办法有很多，但是我只会最简单最粗暴的办法，那便是遍历。就是从序列的最小值到最大值，以一个步长遍历，选择损失最小的值。<br/><br/>
时间序列问题常可以使用常数回归。这是因为时间序列问题受各种因素干扰，趋势一般不可预测。鬼才知道明天会发生什么（冷笑话：知道明天会发生什么的都是鬼才）。那么我们便只能假定趋势大概是平稳的，明天的情况大概和今天差不多。<br/><br/>
#### 历史预测<br/>
经过对数据的观察和统计，对赛题的理解和分析，（这里如何凑成排比句一向是难题，大概要接「对人生的思考和感悟」？）我们显然可以发现一个路段在相近的时间的平均通行时间相差较小。例如，一个路段在某一天的8:13和8:15的平均通行时间可能比较接近。因而对于某一路段某一日某一待测时间片，我们用来预测的样本包括历史各日的同一时间片和其附近的（前后三十个）时间片。例如，待测时间片为8:31，则样本为历史各日的8:01、8:03、...、9:01等时间片的通行时间。<br/><br/>
样本的权重，显然应该和它与待测时间片的距离有关，距离越大的，权重应该越小，反之亦然。那么我们可以用距离的倒数作为权重（倒数的平方、四次方也可以试试）；此外，平均通行时间受曜日的影响，因而权重还应该和曜日有关。与待测日同曜日的日子的权重最大，同为工作日或周末的日子的权重要小一些，其馀日子的权重最小，我们将这三种情况的权重分别设为1、0.5和0。将上述两种权重相乘，就得到了最终的权重。<br/><br/>
用上述所选样本和所计算出的权重，做常数回归，就是历史预测。<br/><br/>
#### 近左预测<br/>
对于某一路段某一日某一待测时间片，取当日该路段的稍早的数据作为样本，取它们与待测时间片的距离的倒数作为权重，做常数回归，即为近左预测。（「近左这个怪异的名字有些历史原因」）<br/><br/>
#### 融合<br/>
计算出这两种预测之后，就要将它们融合，这里就简单采用加权融合就好。<br/>
但是这里有个问题，每一个路段的权重不一样，有的路段近左预测的权重大一些，有的小一些。我也没找到什么规律，我也不知道这个跟道路的长度和宽度是不是有关系。<br/><br/>
于是我采用了比较粗暴的办法。我把测试集向前移了30天，然后对于每个路段，以0.05为步长，从0遍历到1，看看哪个权重在这30天表现最好，就对这个路段使用这个权重。<br/><br/>
要是嫌我的方法麻烦的话，直接取某些中间结果作为特征训练机器学习模型也可以。我只是懒得用机器学习。<br/>
