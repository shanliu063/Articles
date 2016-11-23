PATH: 基于轨迹记录的身份认证
==========

`机器学习` `HMM`


> **摘要：**文章主要介绍如何将用户的历史运动轨迹作为时间函数在移动设备中进行用户身份的主动认证。将用户的运动视为马尔科夫序列，在模型训练期间，应用位置的边际概率和实时观测信息进行平滑。这种算法MSHMM能够应对测试集中出现的未知数据，并有不错的认证效果。

<i class="icon-pencil"></i> 介绍
-----------

主动认证(AA)是根据用户的生物统计学特征在后台持续进行的身份认证方式。比如设备前置摄像头采集的面部特征，触摸屏幕的手势特点，文字输入模式等都可以作为主动认证方式。当使用手机时，AA通过比较当前用户的使用模式和历史用户模式来决定是否开放手机的使用权限。
基于轨迹记录的身份认证(PATH)也是一种AA，结合其它的AA方法(熔断机制)，PATH通过不断比较用户的历史定位信息来持续更新认证分数。主要工作有：

- *提出解决认证问题的新方法. 考虑了位置数据的稀疏性特点，通过计算置信度给出认证结果*
- *聚类方法. 生成轨迹状态信息，并且能够处理未知数据*
- *对隐马尔可夫模型进行优化，提出了边缘平滑的隐马尔可夫算法--MSHMM*


<i class="icon-pencil"></i> 模型
------------
身份验证系统的认证过程包括这几个步骤：首先，根据某个用户的地理位置数据做聚类，生成包含中心点和半径等信息的簇；然后生成带有时间戳的训练数据序列；最后将最近的n个观察点作为测试数据对用户进行评分认证。

###1. 位置点转换为观察状态
位置服务采集到的用户数据是一个位置点序列，其中每个点p都包含了时间、纬度和经度三个维度。对获取的用户位置数据用DBSCAN进行聚类，得到N个簇。
另外，对每个用户还会指定Tr(位置快速移动变化)和Unk(未知)两种簇，其中不属于任何已知簇的点将被分配给某个未知簇，分配的原则为：

此时，我们构建了2N+2个簇：每个簇及其对应着的未知簇(距离M以内)，以及Tr和Unk；另外，每个观测点可以根据时间添加标记：是否工作日标记为WD或WE，不同时间区间标记为TZ1、TZ2、TZ3。所以对每个观测点都有(2N+2)*2*3种可能；考虑到数据的稀疏性，再加一种Null状态。
另外，许多观测状态可能不会在训练集中出现，但是会发生在测试集中。导致这种状况发生的原因有：

- 定位服务并非时刻可用
- 训练阶段一些已知位置附近的未知($Unk$)状态未出现
- 并非所有时间区间或者日期都会有记录信息

###2. 处理未知数据得到训练模型
为了进行有效的认证就不能简单地将不可预见的观测点概率统一置为0。对训练集的观测值做平滑可以得到所有可能状态的概率分布，Laplace平滑是一种简单有效的方法，但却并未考虑可获取状态的信息。比如C-TZ1-WE在训练集中并未出现，对C-TZ1和C-WE进行独立性假设之后可以根据这两者的概率估计P(C-TZ1-WE)。下一章我们会基于这种假设详细阐述HMM训练模型。


<i class="icon-pencil"></i> 算法
-----------

将位置点转化为观察状态序列之后，有不同的认证方法。主要介绍下面三种算法：

- 简单的时间序列匹配(SM)
- 马尔科夫链(MC)
- 边缘平滑的隐马尔可夫模型(MSHMM)

----------
###1. SM

 **算法**  

    
###2. MC
发生某种状态的概率是由上一时刻的概率和状态转移矩阵决定。假设模型Xn是一条由观测序列构成的长为n的马尔科夫链，它的可能状态集合S有s1,...,sn种状态，其中初始状态为si的概率、状态转移概率和给定上述条件后连续n个观测状态概率分别为：


其中对于未知状态，Laplace平滑会赋给先验概率和状态转移概率一个很小的初始值。

###3. MSHMM
HMM由初始隐状态分布、状态转移概率矩阵和观测概率矩阵决定。在模型学习阶段，使用Baum-Welch算法，给初始隐状态、状态转移矩阵和观测矩阵随机的初值，迭代更新直到参数收敛。

 **算法**  



M-step通过给未出现在观测集合中的点指定非零的发射概率优化了Baum-Welch算法。前文提到，一个观测点由位置、时间区间和是否工作日几个维度，如果采用Laplace平滑假设b只与示性函数相关，给b赋一个很小的初始常值概率；而边缘平滑计算发射概率的时候考虑了位置、时间区间和是否工作日这些信息，并且假设用户所处的位置和时间区间与位置和是否工作日是相互独立的。

<i class="icon-pencil"></i> 结果评估
----------
选取两组数据作为实验对象，并分别计算它们的相似矩阵：位置相似矩阵是通过计算两个用户共同的位置簇占比得到；而观测相似矩阵是在位置相似的基础上考虑了时间和日期的因素，所以比位置相似度值更低。

通过EER(Equal Error Rate) 热度图可以发现观测序列的长度n与聚类簇半径R的关系。直观的理解是，观测数据增多则预测的准确性会增加，EER降低；而簇半径太小或者太大，EER都会增加。根据实验，聚类簇半径R选为20米是一个合理值。

对于一组数据，不同的算法EER结果如图所示。其中，MSHMM在n=16时EER=20.73%，优于其它算法；而对任意n，MSHMM在设置隐状态数目为10时具有最好的评估效果。


**感谢原作者Rama Chellappa的支持。**[查看原文](https://arxiv.org/abs/1610.07935)<br />














