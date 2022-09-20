# Week2
---
## 1. 量化基础
   
   参考paper：[Quantization and Training of Neural Networks for Efficient
Integer-Arithmetic-Only Inference](https://arxiv.org/pdf/1712.05877.pdf)
   
   基础神经网络的参数一般是浮点型float数据 存在大小臃肿 计算较慢等问题 因此提出一种量化方法 把参数映射到整数上存储运算 用可以接受的精度损失换取速度和体积的优化 例如int8量化是指把参数转换成8位int存储
   
   常用公式：
   ![image1](https://img-blog.csdnimg.cn/2021030716171942.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA0MjAyODM=,size_16,color_FFFFFF,t_70)
   
## 2. 对称量化和非对称量化

   参考paper：[Quantizing deep convolutional networks for efficient inference: A whitepaper](https://arxiv.org/abs/1806.08342)
   
   区别在于量化值区间是否关于零点对称 对称量化是非对称量化满足 $Z = 0$ 的特殊情况 对称量化下真实值0对应量化值0
   
   ![image2](https://github.com/BrokenArrow1404/2022-MMLAB-SUMMERCAMP/blob/main/week2/images/image2.png)
   
   左为非对称量化 右为对称量化
   
   非对称量化有两个优点，一是对图像做卷积时有时会需要padding 此时0也是需要参与计算的 设置非0 $Z$ 值就能解决这个问题 二是如果数据极端集中在一小块区域的时候会造成精度丢失（正负不对称） 也需要非对称来解决
   
## 3. PTQ,QAT,校准集

   参考文章：https://zhuanlan.zhihu.com/p/463198140 https://zhuanlan.zhihu.com/p/164901397
   
   PTQ（Post-Training Static Quantization）指先对神经网络进行训练 然后给定一个校准集（calibration） 在校准集上进行推理 统计特征图的分布 手动指定scale的阈值并根据这个阈值计算分布的相似程度（比较常用的算法是KL散度）
   多次调整这个scale值最终得出误差最小的scale
   
   得出scale后可以计算出对应的量化系数 PTQ的一个优势是data free的 对校准集的要求很低
   
   ![image3](https://pytorch.org/assets/images/quantization-practice/ptq-flowchart.svg)
   
   QAT（Quantization-aware training）指在训练的过程中同步更新scale值 并计算当前scale下量化后权重的梯度 并更新量化前的权重 以便减少这种误差 最终这个可学习的scale值就是最优解
   
   这种思路有点类似启发式搜索 由于NCNN使用的是PTQ这里就不深入了解了
   
