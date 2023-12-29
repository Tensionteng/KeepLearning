# 引言
## 机器学习分类
### 监督学习
- 回归：平方误差
- 分类/多项分类：交叉熵
- 标记/多标签分类
- 搜索、推荐系统
- 序列学习，如翻译，语音等

### 无监督学习
- 聚类
- 主成分分析
- 因果关系/概率图模型
- 对抗性网络

### 强化学习
- 马尔可夫决策过程（markov decision process）：当环境可被完全观察到时
- 上下文赌博机（contextual bandit problem）：当状态不依赖于之前的操作时
- 多臂赌博机（multi-armed bandit problem）：当没有状态，只有一组最初未知回报的可用动作时

# 卷积神经网络
- 暂退法和计算weight的二范式都是很好的正则化方法
- padding大小为$\frac{kernel\_seize-1}{2}$最佳，可以保持分辨率，即输入输出的张量形状是一样的,torch中，padding是上下左右都补全的，如padding=2，会导致张量上下左右都+2
- `nn.Conv2d(in, out)`中，in是指输入的通道数，out是指输出的通道数，一个卷积层的参数数量为(in * out * kernel_size * kernel_size)

- `(1 * 1)`的卷积核可以改变通道数，降低模型复杂度
- 卷积计算公式: 输出高度 = （输入高度 - Kernel高度 + 2 * padding）/ 步长stride + 1，除法为向下取整
- `BatchNormal(num_feature)`，参数为样本大小，如果是全链接层，那就对应批量大小（因此批量不能为1，否则会减去平均值之后会全部变成0）；如果是卷积层，对应通道数量。批量规范化层一般存在于全输出层或者卷积层之后，激活函数之前，能让中间输出的值更稳定

## google-net
将多个不同维度的卷积核得到的输出堆叠在一起，作为最终的输出。这样的好处有两个，第一，不同的卷积核有不同的感受野，可以学习到不同的特征；第二，大的卷积核能够显著减少参数的数量

## res-net
在深度网络中，很难训练出恒等函数，导致网络退化。res-net的思想在于，将训练恒等函数转为训练残差函数，残差函数的变化率较大，对于深度网络来说，训练起来比较容易。例如，原本训练出的函数为F(x)，残差函数为H(x)=F(x)-x，F(5)=5.1，H(5)=F(5)-5=0.1，5到0.1相差了10倍，深度网络能够比较好地训练出这样的映射。
训练出H(x)后，只需要在最后加上x，即可模拟恒等映射，即H(x)+x= F(x)

在后面的论文中，He提出了改良版的res-net架构，即先BatchNormal再ReLU，再卷积（原本的结构是，先卷积再BatchNormal最后再ReLU）

# 循环神经网络
## 困惑度
困惑度是衡量一个模型好坏的指标。要理解困惑度，要先理解如何计算一个句子的概率。把一个句子分割成n个词元（token），记为x1，x2，x3，...，xn，那么一个句子的概率就是p（x1）p（x2｜x1）p（x3｜x2，x1）...p（xn｜xn-1,xn-2,...,x1）
由于p（xn）都是小数，为了防止溢出，需要将最后的结果乘log（机器学习框架一般是10底或者e底），句子概率最大越好
在训练模型的过程中，需要一个越小越好的损失函数，因此就有了**困惑度**
先看看困惑度的定义：
$$perplexity(W)={p(x_1,x_2,x_3,...,x_n)}^{-\frac{1}{N}}$$
其中，$p(x_1,x_2,x_3,...,x_n)$是句子概率

## GRU
和RNN比，多了两个门，一个是重置门($R_t$)，一个是更新门($Z_t$)
$$R_t=\sigma(X_tW_{xr}+H_{t-1}W_{hr}+b_r)$$
$$Z_t=\sigma(X_tW_{xz}+H_{h-1}W_{hz}+b_z)$$
$$\tilde{H_t}=tanh(X_tW_{xh}+(R_t\odot H_{t-1})W_{hh}+b_h)$$
$$H_t=Z_t\odot H_{t-1}+(1-Z_t)\odot \tilde{H_t}$$

$R_t$注重于，在更新$H_t$的时候，看多少$X_t$的信息(尽量关注$X_t$，要不要遗忘$H_{t-1}$)

$Z_t$注重于，在更新$H_t$的时候，看多少$H_{t-1}$的信息(尽量不关注$X_t$,要不要用$H_{t-1}$更新$H_{t}$)

## LSTM
lstm和gru相比就多了一个C，可以看作是第二个隐状态，C没有经过tanh激活函数，所以范围较大，能够保存比H更多的状态

## 双向RNN
训练的时候直接把batch反过来即可做到“从未来获取过去”，最后两层隐藏层`concat`，得到最终的output

但是，双向RNN**只能用作特征提取或者填空**，不能用作预测，因为无法预知未来

# Attention

## attention的具体计算过程

1. 根据`Query`和`Key`计算两者的相似性或相关性
   - `Query`表示输入，`Key`表示输出，如将Tom and Jerry翻译成汤姆和杰瑞的时候，Tom是Query，汤姆是Key，这里汤姆也是Value
   - 计算方法（不同的计算方法对应不同的attention网络）
     - 向量点积
     - 余弦相似度
     - 引入额外的神经网络计算
2. 对步骤1中的原始分值进行归一化处理，如`softmax`
3. 将步骤2中得到的权重系数和Value进行加权求和
    - key和value可以相同也可以不同，key是用于计算相似度的，value才是真正在网络中进行计算的值

## attention的seq2seq

普通seq2seq只会把encoder最后一层RNN的hidden_state和decoder的embedding concat起来，作为输入，虽然最后一层RNN的hidden_state包含了前面序列的所有信息，但是这种高度集中的信息，难免还是有些失真，对于后面的词来说多了很多噪音。

而使用attention之后，可以根据输入，获取相应位置的hidden_state，concat起来。例如我要翻译hello， world，普通seq2seq只能concat hello+world的hidden_state，使用attention之后，我知道“你好”这个单词和hello关系更密切，就可以concat hello的hidden_state，这样预测更准确。


## Self Attention

- query，key和value都是同一个东西
- 相比RNN，更容易捕获句子中**长距离**的相互依赖特征
  - 因为RNN在计算长距离关系的时候，需要经过若干时间步的累积，在这过程中，有效信息很有可能被遗忘
  - 在attention中，句子中任意两个单词（query和key）可以直接通过一个函数计算关联度

与CNN、RNN不同，self-attention不包含位置信息，可以通过增加一个位置信息矩阵P（P的维度和输入X一致），将X+P作为自编码输入
$$P \epsilon \R^{n \times d}: P_{i,2j}=sin(\frac{i}{10000^{2j/d}}),P_{i,2j+1}=cos(\frac{i}{10000^{2j/d}})$$

位于$i+\delta$处的位置编码可以通过$i$处的位置编码线性变换得到，记录的是**特征的相对位置**

## Transformer

- Transformer是一个纯使用注意力机制的编码器，解码器架构
- 编码器和解码器都有n个Transformer块
- 每个块使用多头（自）注意力，基于位置的前馈网络，层归一化（不是批量归一化）

encoder：mul-head -> res -> mlp -> res

encoder和decoder之间传递信息：把encoder的输出作为decoder第一个attention块的key和query，encoder本身的输入作为query。attention本身提取出来的特征信息较少，因此往往需要较大的数据量。

# Bert

动机：预训练的模型已经抽取了足够多的特征，新的任务只需要加一个简单的输出层即可复用

结构：只有编码器的Transformer

# GNN
最简单的GNN层只有三个mlp，分别对应节点，边，全局。这么做没有利用到图本身的结构，例如一个节点，经过一个GNN层的时候，只有他自己的向量得到了计算，是孤立的。

解决方法：`pooling`，即将节点本身和其邻居节点的向量相加，再进入mlp