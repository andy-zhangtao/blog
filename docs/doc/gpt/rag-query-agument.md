# RAG系列 - 查询增强



查询增强是在RAG流程的召回阶段。具体来说属于召回前需要做的事情。

<img src="/doc/gpt/assets/image-20231230145812876.png" alt="image-20231230145812876" style="zoom:50%;" />

查询增强的作用在于结合当前query的上下文推导出最合适的query，可以用下面的公式来表达：
$$
f(x) = transform(query + context) 
$$
这么做的目的在于直接通过query进行召回不是最优的，query直接召回的弊端如下:

1. 只可以做相似度检索，无法做相似性检索(相似度值得是数学维度上的相似度算法，相似性值得是语义上的相似性)。例如:

   当前query = “北京如何?” 通过此句做召回，会大量召回与北京有关的内容，因为缺乏具体的上下文信息(context)会导致召回内容无法用于LLM进行推理。

2. 当前query对于recall来说未必是最优的。 自然语言天然存在“没有最佳范式”这样的特性，同一个问题可以有不同的问法。而对于召回而言，“残缺”的问法可能会导致召回内容不全。 所有有必要在召回之前进行“问法补全”。 例如: 当前query = “这本书多钱？”。在此阶段会增强为: “这本书的价格是多少? ”， “这本书当前售价为多少?” 等等，相对而言对recall更为友好的query。



当前进行query enhance的方式大致如下三种：

1. Condense_Question模型

   > 通过结合上下文，将当前query改写为包含context的query。

   1. 根据上下文和最新的query生成一条独立于context的孤立query
   2. 将最新的query进行召回

   优点: 

   ​	每次都保存最新的context语义

   缺点：

   ​	每次都进行recall，因此无法回答例如“我问过什么？”之类的闲聊问题

   <img src="/docs/doc/gpt/assets/image-20231230153743327.png" alt="image-20231230153743327" style="zoom:50%;" />

2. Context 模型

   > 仅对query进行有限制的增强(去除错别字, 词汇纠正)

   1. 使用有限增强的query进行召回
   2. 将召回内容插入到context之中，并进行推理

   优点:

   ​	有完整的上下文对话历史，LLM可以回顾过往记录

   缺点:

   ​	无法解决recall数据不完整的问题

<img src="/docs/doc/gpt/assets/image-20231230154115688.png" alt="image-20231230154115688" style="zoom:50%;" />

1. Condense_Plus_Context 模型

   > 前面两种模式的综合模型

   1. 将对话和最新的用户消息压缩为一个独立的问题
   2. 使用最新的query进行recall，并将context+recall内容插入到context中进行推理

   优点:

   ​	可以解决recall数据不完整和LLM无法得知过往历史的问题

   缺点:

   ​	每次要求LLM进行推理的数据量偏大，context无法容纳很多数据

<img src="/docs/doc/gpt/assets/image-20231230154342856.png" alt="image-20231230154342856" style="zoom:50%;" />