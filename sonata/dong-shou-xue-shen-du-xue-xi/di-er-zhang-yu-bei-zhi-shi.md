# 第二章 预备知识



### 预备知识

#### 写在前面

对于这本《动手学深度学习》，此前已经有了大概的了解，是新手入门深度学习的一本好书。本次的笔记将着重标出MXNet与Pytorch的操作上的一些差异，并拎出仍然模糊的概念，进一步加深印象。对比较简单的部分就不再赘述了。

#### 基本操作

**导入环境包**

在MXNet中，以NDArray为基本单位，进行数据间的频繁操作，即在程序的开头导入nd，通过`nd.arrange()、nd.zero()`等方式调用函数。

如果用户使用Pytorch，基本单位将变成tensor，当然也需要在开头导入环境包torch，然后直接将nd改成torch即可完成大部分操作，且具有相同的执行效果。

```text
from mxnet import nd
import torch
```

**运算**

根据MXNet中的例子，我们也创建一个X和Y的Tensor来运算。

```text
import torch
X = torch.arange(12)
X = X.reshape((3, 4))
​
# 要注意两者在使用特定list时，制定创建torch时的差别。
Y = torch.tensor([[2, 1, 4, 3], [1, 2, 3, 4], [4, 3, 2, 1]])
print(X, '\n', Y)
```

输出结果如下所示：

```text
tensor([[ 0,  1,  2,  3],
        [ 4,  5,  6,  7],
        [ 8,  9, 10, 11]]) 
 tensor([[2, 1, 4, 3],
        [1, 2, 3, 4],
        [4, 3, 2, 1]])
```

在基本运算里面，比较常用的除了加减以外，还有两个矩阵的乘法（nd.dot / torch.dot）、矩阵间的连接（nd.concat / torch.concat）、求和（nd.sum / tensor.sum）等等。另外，条件判别式（X == Y）可判断两个矩阵在相应位置上的值是否相等，并输出一个形状相同的新矩阵，如果相等则置1，不等置0。也就是说`X[0][0] == Y[0][0]`，如果相等，新矩阵的第一行第一列的值则为1。

⚠️ 矩阵间的连接很好用，例如在搭建神经网络的过程中的多分类任务时，可以通过矩阵拼接后的归一化处理完成答案的预测。但是我们需要注意的是，两个拼接的矩阵在维度上是有特定需求的。

**广播机制**

两个形状不同的矩阵进行元素运算，将触发广播机制，将维度变得尽可能地大，同时多出的部分是复制的已有的部分。然后我刚刚突发奇想，好奇能不能对维度分别为`[4， 2]`进行广播，发现报错了哈。说明广播机制还是有严格的维度匹配要求的～

如果相加的两个数组的shape不同, 就会触发广播机制,

1. 程序会自动执行操作使得A.shape==B.shape；
2. 对应位置进行相加；

运算结果的shape是：A.shape和B.shape对应位置的最大值。

比如:A.shape=\(1,9,4\)，B.shape=\(15,1,4\)，那么A+B的shape是\(15,9,4\)。

⚠️注意不要混淆ndim和shape这两个基本概念

有两种情况能够进行广播：

1. A.ndim &gt; B.ndim, 并且A.shape最后几个元素包含B.shape, 比如下面三种情况。

```text
A.shape=(2,3,4,5), B.shape=(3,4,5)
A.shape=(2,3,4,5), B.shape=(4,5)
A.shape=(2,3,4,5), B.shape=(5)
```

1. A.ndim == B.ndim, 并且A.shape和B.shape对应位置的元素要么相同要么其中一个是1。

```text
A.shape=(1,9,4), B.shape=(15,1,4)
A.shape=(1,9,4), B.shape=(15,1,1)
```

#### 自动求梯度

```text
out.backward()
print(x.grad)
```

反向计算得到的梯度时，需要注意这里的out为标量,所以直接调用backward\(\)函数即可.

但是当out为数组时,用先定义一样大小的Tensor例如`grad_output`执行`.backgrad(grad_output)`语句，或者对out各位置求和后得到标量再求导。

```text
# 求和
x = torch.ones(2,requires_grad=True)
z = x + 2
z.sum().backward() #    求和
print(x.grad)
​
>>> tensor([1., 1.])
```

我们再仔细想想，对z求和不就是等价于z**点乘**一个一样维度的全为1的矩阵吗？\(点乘只是相对于一维向量而言的，对于矩阵或更高为的张量，可以看做是对每一个维度做点乘\)

```text
# 点乘
x = torch.ones(2,requires_grad=True)
z = x + 2
z.backward(torch.ones_like(z)) # grad_tensors需要与输入tensor大小一致
print(x.grad)
​
>>> tensor([1., 1.])
```

#### 参考文献

1. [https://blog.csdn.net/littlehaes/article/details/103807303](https://blog.csdn.net/littlehaes/article/details/103807303)
2. [https://www.jianshu.com/p/d475db3506f5](https://www.jianshu.com/p/d475db3506f5)
3. [https://www.cnblogs.com/marsggbo/p/11549631.html](https://www.cnblogs.com/marsggbo/p/11549631.html)

