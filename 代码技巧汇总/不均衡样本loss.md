本文是谷歌对CVPR ' 19上发表的一篇文章的综述，文章的标题是Class-Balanced Loss Based on Effective Number of Samples。



**它为最常用的损耗(softmax-cross-entropy、focal loss等)提出了一个针对每个类别的重新加权方案，能够快速提高精度，特别是在处理高度类不平衡的数据时。**



论文的PyTorch实现源码：https://github.com/vandit15/Class-balanced-loss-pytorch



# 样本的有效数量



在处理长尾数据集(其中大部分样本属于很少的类，而许多其他类的样本非常少)的时候，如何对不同类的损失进行加权可能比较棘手。通常，权重设置为类样本的倒数或类样本的平方根的倒数。

![img](https://cy-1256894686.cos.ap-beijing.myqcloud.com/cy/2019-10-01-104407.jpg)



传统的权重调整与这里提出的权重调整





然而，正如上面的图所示，这一过渡是因为随着样本数量的增加，新数据点的带来的好处会减少。新添加的样本极有可能是现有样本的近似副本，特别是在训练神经网络时使用大量数据增强(如重新缩放、随机裁剪、翻转等)的时候，很多都是这样的样本。用有效样本数重新加权可以得到较好的结果。



有效样本数可以想象为n个样本所覆盖的实际体积，其中总体积N由总样本表示。

![img](https://cy-1256894686.cos.ap-beijing.myqcloud.com/cy/2019-10-01-104416.png)

有效样本数量





我们写成：

![img](https://cy-1256894686.cos.ap-beijing.myqcloud.com/cy/2019-10-01-104410.png)

有效样本数量



我们还可以写成下面这样：

![img](https://cy-1256894686.cos.ap-beijing.myqcloud.com/cy/2019-10-01-104409.jpg)

每个样本的贡献





这意味着第j个样本对有效样本数的贡献为βj-1。



上式的另一个含义是，如果β=0，则En=1。同样，当β→1的时候En→n。后者可以很容易地用洛必达法则证明。这意味着当N很大时，有效样本数与样本数N相同。在这种情况下，唯一原型数N很大，每个样本都是唯一的。然而，如果N=1，这意味着所有数据都可以用一个原型表示。



# 类别均衡损失



如果没有额外的信息，我们不能为每个类设置单独的Beta值，因此，使用整个数据的时候，我们将把它设置为一个特定的值(通常设置为0.9、0.99、0.999、0.9999中的一个)。



因此，类别均衡损失可表示为：

![img](https://cy-1256894686.cos.ap-beijing.myqcloud.com/cy/2019-10-01-104415.png)

这里， **L(p,y)** 可以是任意的损失。



# 类别均衡Focal Loss



![img](https://cy-1256894686.cos.ap-beijing.myqcloud.com/cy/2019-10-01-104416.png)



原始版本的focal loss有一个α平衡变量。这里，我们将使用每个类的有效样本数对其重新加权。



类似地，这样一个重新加权的项也可以应用于其他著名的损失(sigmod -cross-entropy, softmax-cross-entropy等)。



# 实现



在开始实现之前，需要注意的一点是，在使用基于sigmoid的损失进行训练时，使用b=-log(C-1)初始化最后一层的偏差，其中C是类的数量，而不是0。这是因为设置b=0会在训练开始时造成巨大的损失，因为每个类的输出概率接近0.5。因此，我们可以假设先验类是1/C，并相应地设置b的值。

### 每个类的权值的计算

![img](https://cy-1256894686.cos.ap-beijing.myqcloud.com/cy/2019-10-01-104408.jpg)

计算归一化的权值



上面的代码行是获取权重并将其标准化的简单实现。

![img](https://cy-1256894686.cos.ap-beijing.myqcloud.com/cy/2019-10-01-104414.jpg)

得到标签的onehot张量



在这里，我们得到权重的独热值，这样它们就可以分别与每个类的损失值相乘。



# 实验



![img](https://cy-1256894686.cos.ap-beijing.myqcloud.com/cy/2019-10-01-104413.png)

类平衡提供了显著的收益，特别是当数据集高度不平衡时(不平衡= 200,100)。



# 结论



利用有效样本数的概念，可以解决数据重叠问题。由于我们没有对数据集本身做任何假设，因此重新加权通常适用于多个数据集和多个损失函数。因此，可以使用更合适的结构来处理类不平衡问题，这一点很重要，因为大多数实际数据集都存在大量的数据不平衡。