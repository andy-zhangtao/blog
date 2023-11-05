# Self Attenation 自注意力机制

## 自注意力机制和注意力机制的区别



### 注意力机制

这种机制是受到人类视觉注意力的启发，人类受制于视野生物学特性影响，在视野范围内会挑出重点内容进行关注，而忽略其他非重要信息。

受此启发，在深度学习模型中，注意力机制通常用于处理序列类任务，例如NLP，语音或者图像处理。 注意力机制重点在于聚集数据的重点部分而忽略其他部分。



<img src="https://pic2.zhimg.com/80/v2-e68d0744701e7e019455c85fdd9a597d_1440w.webp" alt="img" style="zoom:50%;" />



它的基本原理是为输入(Input)不同部分分配不同的权重，通过权重显示当前数据对于整个数据的的重要性。

<img src="https://p.ipic.vip/v4a6ly.png" alt="img" style="zoom:67%;" />

即：将Q与K进行点乘运算，$Q \dot K =k1,k2,k3...kn $   得到每个元素与Q的相似度值( $s1,s2,...sn$ ). 做一层SoftMax后得到对应的概率($a1,a2,a3...an$)  。这样经过阶段1，阶段2就可以得到每个词对于Q的重要程度。 根据对概率的计算方式分为三类:

1. 软注意力(Soft Attention)

   模型在训练过程中可以通过反向传播算法调整注意力参数，会在整个输入序列中生成一个权重分布。

2. 硬注意力(Hard Attention)

   随机选择输入的一个子集进行计算，因此无法通过反向传播进行调整。因此会配合例如强化学习等特性的训练方式。

3. 自注意力(Self Attention)

   将输入序列每个元素和其他元素进行比较，计算出整个输入的注意力分布。

在实现层面，注意力机制会认为: 输入Input中每个词对于语义的贡献度是并不一致的，某些词对于理解句子的意义更为关键，因此需要识别出这些关键词汇。

那么如何工作呢？

1. 权重分配

   对于给定的任务，模型会为Input中每个词汇分配一个权重，权重高的词汇则会得到更多的**关注**

2. 上下文向量

   当每个词都有了初始化权重后，模型会使用这些权重生成一个上下文向量。 这个向量是所有词汇权重的加权和，而权重就是每个词经过计算后的权重(并非是初始化权重)。

3. 任务专注

   模型利用上下文向量完成后续特定的任务。

比如说： 我们需要对"The cat sat on the mat。"生成摘要。 如果是非注意力机制的模型，那么每个词的权重都是相同的，'cat'和'sat'是相同的，在这么短的Input中，无法识别出关键词。 而使用了注意力机制后，模型会给'cat'和'mat'更高的权重。最后生成摘要时，模型会特别关注这两个词，因此摘要有可能就是'cat on mat'。



### 自注意力机制



自注意力机制是在注意力机制中，将QKV的数据源都变成了自己。 也就是 $ Q \approx K \approx V $  , 但$ (Q,K,V) \in X $ 。 

<img src="https://p.ipic.vip/vf9z4v.png" alt="img" style="zoom:67%;" />

自注意力机制说的简单一点是，通过X找到X里面的关键点。

在处理方式上，自注意力机制和注意力机制差不多:

<img src="https://p.ipic.vip/lovaa2.jpg" alt="img" style="zoom: 67%;" /> -->   

模型为每个词分配初始化权重(通过Embedding获取)，$QKV$ 分别由$X\dot w_{q},W_{k}, w_{v}$ 做点乘得出。 ($w_{q}, w_{k}, w_{v}$ 由模型训练得出的参数)

<img src="https://p.ipic.vip/5688xm.jpg" alt="img" style="zoom: 67%;" /> <img src="https://p.ipic.vip/o5aj13.jpg" alt="img" style="zoom: 67%;" />

将$q1 \dot k1 = s1$  得出的值 $s1$ 作为 x1 的相关性得分，同理计算$x1,x2,x3..xn$ 的相关性得分。

<img src="https://p.ipic.vip/4f5th9.jpg" alt="img" style="zoom: 67%;" />

将$s1, s2.. sn $进行softmax计算后再点乘v得出z， $softmax(s) \dot v = z$  

最后的z就是每个词在当前Input中的贡献度。 

<img src="https://p.ipic.vip/z2utjw.png" alt="Capture-2023-11-05-195200" style="zoom: 33%;" />

不做注意力，its 的词向量就是单纯的 its，没有任何附加信息。添加了自注意力以后，its会同时表示"law"和"application"两个词汇的信息，但"law"的权重高于"application"。

