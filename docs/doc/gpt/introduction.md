# GPT 模型学习的一些心得体会 - 01

> GPT 模型特点以及网络结构

> **请注意下面所有的图片均使用 DallE 生成，并非完全正确，仅用于辅助理解！**

GPT 模型非常适合处理和生成自然语言文本，并能够捕捉文本中的复杂模式和关系。其可以实现这种功能依赖于它的神经网络，这个网络是一个 Transformer 的 decoder，它的输入是一个文本序列，输出是一个文本序列，这个网络的结构特点如下：

1. **基于 Transformer 架构**：GPT 完全采用了 Transformer 架构，特别是其解码器部分，以处理序列数据。
2. **自注意力机制**：Transformer 中的核心是自注意力机制，它允许模型在不同的位置之间建立权重，从而捕捉长距离的依赖关系。
3. **多头注意力**：GPT 使用多头注意力来并行捕捉输入数据的不同方面的信息。
4. **堆叠的层**：GPT 模型由多个这样的 Transformer 层堆叠而成，使其能够学习更复杂的模式。
5. **位置编码**：由于 Transformer 本身不考虑序列的顺序，GPT 在输入中加入位置编码以捕捉序列中的位置信息。
6. **大规模参数**：GPT 模型通常有数十亿到数千亿的参数，这使其能够存储大量的知识和信息。
7. **单向自注意力**：与标准的 Transformer 不同，GPT 使用单向（或称为"掩码"）自注意力，确保在生成文本时只考虑之前的词，而不是之后的词。

![](https://p.ipic.vip/lx5440.png)

上图体现了 GPT 模型的网络基本结构，

1. **SELF-ATTENTION**: 自注意力机制是 Transformer 模型的核心，允许模型为输入序列中的每个词分配权重。
2. **POSITION ATTENTION**: 指位置编码或位置嵌入，它是 Transformer 模型的一个重要组成部分，用于传达序列中单词的位置信息。
3. **LAYERS**: 指的是神经网络中的层。
4. **MULTI-HEAD ATTENTION**: Transformer 模型的核心组成部分，它允许模型在多个“头”或并行的自注意力机制中同时处理信息。

GPT 模型的神经网络是基于 Transformer 的解码器，它的输入是一个文本序列，输出是一个文本序列。而 Transformer 的解码器是由多个相同的层堆叠而成，每个层都包含自注意力和前馈神经网络。这些层的输出会被传递给下一层，最后一层的输出会被传递给一个线性层，用于生成下一个词的概率分布。

自注意力机制，它允许模型为输入序列中的每个词分配权重。这些权重可以用于计算输入序列中每个词的加权平均值，从而捕捉序列中的关键信息。自注意力机制的核心是计算输入序列中每个词与其他所有词之间的相似度，然后将这些相似度转换为权重。这些权重可以用于计算输入序列中每个词的加权平均值，从而捕捉序列中的关键信息。

## 什么是 Transformer 架构?

Transformer 架构的关键特点和组件：

1. **自注意力机制**：这是 Transformer 的核心组件，允许模型在不同位置的输入序列之间计算权重，从而捕捉长距离的依赖关系。
2. **多头注意力**：通过多个自注意力层并行处理，模型可以同时关注输入数据的不同方面和特征。
3. **位置编码**：因为 Transformer 本身不具有序列的顺序感知能力，所以需要添加位置编码来为模型提供序列中的位置信息。
4. **前馈神经网络**：在每个 Transformer 层中，自注意力的输出会被传递给一个前馈神经网络。
5. **残差连接**：Transformer 中的每个子层（如自注意力和前馈神经网络）都有一个残差连接，帮助防止梯度消失问题。
6. **规范化层**：每个子层的输出都会通过一个规范化层，确保模型的稳定性。
7. **堆叠层**：Transformer 模型由多个相同的层堆叠而成，每层都包含自注意力和前馈神经网络。
8. **编码器-解码器结构**：原始的 Transformer 模型包括编码器和解码器部分。编码器处理输入数据，解码器生成输出。但在某些模型如 BERT（只使用编码器）和 GPT（只使用解码器）中，这种结构被简化。

`自注意力机制`，`多头注意力`和`位置编码`内容比较多，留待后续再分析。

我们从`前馈神经网络`开始学习。

### 前馈神经网络

前馈神经网络是 Transformer 模型中的一个重要组件，它由两个线性层和一个激活函数组成。前馈神经网络的输入是自注意力的输出，输出是一个向量，它会被传递给下一个 Transformer 层。

前馈神经网络示意图:

1. **结构**：前馈神经网络在结构上是相对简单的。它由两层全连接层组成，中间有一个激活函数（如 ReLU）。
2. **功能**：前馈神经网络在 Transformer 层中的作用是对自注意力的输出进行进一步的非线性变换。这增加了模型的表达能力，使其能够学习更复杂的函数。
3. **流程**：在 Transformer 的每一层中，输入首先通过自注意力机制进行处理，得到一个输出。这个输出然后被传递给前馈神经网络进行进一步的处理。前馈神经网络的输出再与原始输入相加（残差连接），然后通过规范化层，得到这一层的最终输出。

![](https://p.ipic.vip/tecyvh.png)

在前馈神经网络中，全连接层的作用是将输入向量转换为一个新的向量，这个向量的维度通常比输入向量的维度大得多。这个新的向量会被传递给激活函数，激活函数的作用是对向量中的每个元素进行非线性变换。这个变换后的向量会被传递给下一个全连接层，这个过程会重复多次，直到得到最终的输出向量。

- 为什么需要前馈神经网络？

  1. **非线性变换**：前馈神经网络的作用是对自注意力的输出进行进一步的非线性变换。这增加了模型的表达能力，使其能够学习更复杂的函数。

- 为什么需要全连接层？

  1. **维度变换**：全连接层的作用是将输入向量转换为一个新的向量，这个向量的维度通常比输入向量的维度大得多。

- 为什么需要激活函数？

  1. **非线性变换**：激活函数的作用是对向量中的每个元素进行非线性变换。

### 残差链接

在神经网络中，残差连接是一种直接将某一层的输入传递到其后几层的技巧。具体来说，如果有一个层的输入为**`x`**，输出为**`F(x)`**，那么残差连接将这两者相加，得到**`x + F(x)`**作为该层的最终输出。

![](https://p.ipic.vip/3g0s9q.png)

1. **输入层**：标记为**`x`**的矩形框。
2. **子层处理**：旁边有一个标记为**`F(x)`**的矩形框，代表子层（如自注意力或前馈神经网络）的处理。
3. **直接连接**：从**`x`**直接引出的箭头，绕过**`F(x)`**，直接连接到输出层。
4. **子层输出连接**：从**`F(x)`**引出的箭头，连接到输出层。
5. **输出层**：标记为**`x + F(x)`**的矩形框，代表残差连接的结果。
6. **箭头**：使用蓝色箭头表示数据流动的方向。
7. **标签**：确保每个部分都有清晰的标签并注明其功能，如"输入"、"子层处理"、"残差连接"等。

**为什么需要残差连接**：在深度神经网络中，随着层数的增加，梯度（用于更新网络权重的值）可能会变得非常小，这被称为"梯度消失"问题。当梯度接近于 0 时，网络权重的更新会变得非常缓慢，导致训练过程几乎停滞。残差连接可以帮助梯度更直接地流过多个层，从而缓解梯度消失问题。

**在 Transformer 中的应用**：Transformer 架构中的每个子层（例如自注意力子层和前馈神经网络子层）都使用了残差连接。这意味着每个子层的输出都会与其输入相加，然后再传递给下一个子层或层的下一部分。

### 规范化层

规范化层是 Transformer 模型中的一个重要组件，它的作用是确保模型的稳定性。规范化层通常会在每个子层的输出上进行操作，其输出会被传递给下一个子层或层的下一部分。

在 Transformer 架构中，每个子层（例如自注意力子层和前馈神经网络子层）的输出在传递到下一个层或用于其他计算之前，都会先经过一个规范化层。

规范化层的主要目的是调整和标准化子层的输出，使其具有稳定的均值和方差。这样做可以帮助模型更快地收敛，并防止训练过程中出现的梯度爆炸或梯度消失问题。

具体来说，规范化层通常使用的是层归一化（Layer Normalization）技术。在层归一化中，对于每个输入样本，它会独立地标准化每一层的输出，使其具有零均值和单位方差。

![](https://p.ipic.vip/u462kw.png)

- 规范化层的作用？

1. **稳定性**：规范化层的作用是确保模型的稳定性。规范化层通常会在每个子层的输出上进行操作，其输出会被传递给下一个子层或层的下一部分。

2. **梯度消失**：规范化层的主要目的是调整和标准化子层的输出，使其具有稳定的均值和方差。这样做可以帮助模型更快地收敛，并防止训练过程中出现的梯度爆炸或梯度消失问题。

### 堆叠层

1. **堆叠层**：在神经网络中，"堆叠"通常意味着将多个相同类型的层叠加在一起。在 Transformer 模型中，这意味着有多个 Transformer 层，每个层都是相同的结构。
2. **每层都包含自注意力和前馈神经网络**：这描述了每个 Transformer 层的内部结构。每个层都有两个主要的子层：
   - **自注意力**：这是 Transformer 的核心，它允许模型为输入序列中的每个元素分配不同的注意力权重，这样模型就可以根据上下文关注到不同的信息。
   - **前馈神经网络**：这是一个普通的神经网络，它独立地处理自注意力的输出结果，为每个位置提供额外的计算能力。

![](https://p.ipic.vip/61d3so.png)

> 每个层由自注意力子层（蓝色）和前馈神经网络子层（绿色）组成

- 堆叠层的作用？

1. **学习复杂模式**：堆叠层的作用是使模型能够学习更复杂的模式。每个层都包含自注意力和前馈神经网络，这些层的输出会被传递给下一层，最后一层的输出会被传递给一个线性层，用于生成下一个词的概率分布。

### 编码器-解码器结构

Transformer 架构最初是为了处理序列到序列（seq2seq）任务而设计的，如机器翻译。在这种情境下，Transformer 包含两个主要部分：编码器和解码器。

1. **编码器**:
   - **目的**：编码器的任务是接受输入序列（例如，一个英文句子）并将其转化为一个连续的表示，通常被称为“上下文”或“编码”。
   - **结构**：编码器由多个相同的 Transformer 层堆叠而成。每个层都包含自注意力机制和前馈神经网络。
   - **工作原理**：输入序列首先通过自注意力机制，这允许模型为输入序列中的每个元素分配不同的注意力权重。然后，自注意力的输出被传递给前馈神经网络。经过所有编码器层后，我们得到一个编码的上下文表示。
2. **解码器**:
   - **目的**：解码器的任务是接受编码器的输出并生成目标序列（例如，一个法文句子）。
   - **结构**：解码器也由多个相同的 Transformer 层堆叠而成。但与编码器不同，解码器的每个层都包含两个自注意力机制：一个是“掩码自注意力”（处理目标序列），另一个是“编码器-解码器注意力”（将编码器的输出与解码器的当前状态结合起来）。
   - **工作原理**：目标序列首先通过掩码自注意力，然后与编码器的输出结合，最后通过前馈神经网络。这一过程在每个解码器层中都会重复。

![](https://p.ipic.vip/e0e1n8.png)

具体到实例当中: 在 Transformer 架构中，特别是在序列到序列的任务（如机器翻译）中，编码器-解码器结构是常见的。但对于像 GPT 这样的语言模型，它只使用了解码器部分。

例如预测’今天天气是’这句话时。

首先，将输入序列“今天天气是”编码为一个固定大小的上下文向量。然后，这个上下文向量被传递给解码器，解码器一次生成一个输出符号，直到生成一个结束符号或达到某个长度限制。在每次生成一个符号后，该符号被添加到输入序列中，并再次传递给解码器，直到整个序列生成完成。

![](https://p.ipic.vip/dhppiz.png)

- 编码器-解码器结构的作用？

1. **序列到序列任务**：编码器-解码器结构最初是为了处理序列到序列（seq2seq）任务而设计的，如机器翻译。在这种情境下，Transformer 包含两个主要部分：编码器和解码器。

2. **编码器**：编码器的任务是接受输入序列（例如，一个英文句子）并将其转化为一个连续的表示，通常被称为“上下文”或“编码”。编码器由多个相同的 Transformer 层堆叠而成。每个层都包含自注意力机制和前馈神经网络。输入序列首先通过自注意力机制，这允许模型为输入序列中的每个元素分配不同的注意力权重。然后，自注意力的输出被传递给前馈神经网络。经过所有编码器层后，我们得到一个编码的上下文表示。

GPT 中训练的是单向语言模型，其实就是直接应用 Transformer Decoder；
Bert 中训练的是双向语言模型，应用了 Transformer Encoder 部分，不过在 Encoder 基础上还做了 Masked 操作