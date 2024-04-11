# 基于大模型的意图识别(修订补充)

在基于大模型的应用开发过程中，意图识别已成为必不可少的一项。我们总结了五种意图识别的方式。



## Prompt模式

Prompt模式是基于大模型进行意图识别的一种基础方式。在这种模式下，系统通过解析用户的输入（即prompt），来理解用户的意图。这种方法的优势在于直接性和简洁性，可以快速响应用户需求。但它也有局限性，尤其是在处理复杂或模糊的用户输入时。

下面是一个基础版本的Prompt案例:

```
# Role:
Dynamic Intent Recognition Analyst with External Search Integration

# Profile:
An expert in natural language processing and intent identification, specialized in integrating external search engines for queries that require additional information beyond the model's current knowledge.

# Skills:
- Natural Language Processing
- Text Classification
- Integration with External Search Engines
- Decision Making under Uncertainty

# Rules:
- Directly classify inputs that require external search engine use as [使用外部搜索] (Use External Search) intent.
- For inputs within the model's knowledge base, identify the most relevant intent.
- Return [默认意图] (Default Intent) for ambiguous intents that don't necessitate external search.

Let's solve this problem slowly and step by step to ensure we have the right answer.

# Workflow:
1. Analyze user input to determine if it falls within the model's knowledge base.
2. If the input clearly requires information from an external search engine, classify it as [使用外部搜索].
3. For inputs within the model's knowledge, identify the most relevant intent.
4. In cases of ambiguity, and where external search is not needed, return [默认意图].
5. Output the identified intent, [使用外部搜索], or [默认意图].

# Tools:
- Text Analysis Techniques
- External Search Engines
- Pattern Recognition Methods

# Example Objective:
For user input like "Find the latest research on Mars exploration", classify as [使用外部搜索] since it requires additional information from an external source.

# Example Output Format:
Text response indicating the identified intent: either a specific intent, [使用外部搜索], or [默认意图].

```



作为demo，上面案例仅支持四种意图。当不属于上面四种时，会返回默认意图。通过上面的Prompt，可以得知这种模式的优点在于简单粗暴，需要支持哪些意图直接把判断依据写上就可以，立竿见影。 但缺点在于:

1. 无法支持判断条件相互干扰的意图
2. 无法支持太多的意图
3. 无法支持提取意图参数，比如判断需要使用[外部搜索]了，但具体搜索啥？ 下游服务只能拿到用户输入的Query，具体干点啥还需要再进行处理。

为了解决这个问题，就引申出下面的第二种方式



## FunctionCall 模型

FunctionCall模型是一种更为高级的意图识别方法。它通过将用户输入映射到特定的函数调用上，从而实现对用户意图的理解。这种方法相比于Prompt模式更为复杂，但能够处理更加复杂的任务和查询，使得意图识别更加精准。



FunctionCall同Prompt相比，可以支持提取更详细和丰富的参数。

如果我们将上面的Prompt转换为FunctionCall则会变为:

```
System Prompt:

# Role:
Dynamic Intent Recognition Analyst with External Search Integration

# Profile:
An expert in natural language processing and intent identification, specialized in integrating external search engines for queries that require additional information beyond the model's current knowledge.

# Skills:
- Natural Language Processing
- Text Classification
- Integration with External Search Engines
- Decision Making under Uncertainty

# Example Objective:
For user input like "Find the latest research on Mars exploration", classify as [使用外部搜索] since it requires additional information from an external source.

# Example Output Format:
Text response indicating the identified intent: either a specific intent, [使用外部搜索], or [默认意图].


Function Call
> 仅对Search Engines进行了改写
{
    "name": "Search Engines",
    "description": "Use a Search Engines for query and answer uers input",
    "parameters": {
        "type": "object",
        "properties": {
            "query": {
                "description": "users input",
                "type": "string"
            }
        },
        "required": [
            "query"
        ]
    }
}
```



FunctionCall 模式可以支持从用户Input中提取更多更丰富的参数，这样就大大简化下游服务的处理逻辑。 理想很美好，但现实却不一定美好。 这种模式的优点显而易见，而缺点却更为重要.

1. 下游服务需要准确回复响应，LLM才能准确判断何时退出。
2. 无法支持判断条件相互干扰的意图, 这种模式本质也是依赖LLM的Prompt能力。
3. 无法支持太多的意图。 FunctionCall里面的内容也会参与到LLM的推理计算，所以定义的Function越多，计算失败的概率就会越大，响应的速度也就越慢。 

现在流行“既要.. 又要..”，如何既要支持更多，又要不丧失准确性呢？那就是FT模式。



## Fine-Tune模式

Fine-Tune模式则是通过对大模型进行微调来适应特定的应用场景。这种模式允许模型更好地理解特定领域的语言和用户意图。通过Fine-Tune，模型可以更加精确地识别和响应用户的需求，特别是在专业或行业特定的环境中。

相对于从零开始构建一个新的模型，Fine-Tune模式是一种成本效益更高的选择。它大幅减少了额外的训练时间和计算资源。再意图识别领域，我们将需要识别的内容做成FAQ类型的语料库投喂给LLM，然后就可以忘记Prompt和FunctionCall，直接像LLM进行提问。



相较于前面两种而言：

#### 优点

1. **提高特定领域的识别精度**：Fine-Tune模式通过针对特定应用场景调整模型，显著提高了在该领域内的语言理解和用户意图识别的准确性。
2. **增强模型的适应性**：该模式使得大模型能够适应多样化的专业或行业特定环境，增强了模型在不同领域的应用能力。
3. **节省资源与时间**：与从头开始训练模型相比，Fine-Tune节省了大量的计算资源和时间，因为它仅调整已有模型的特定部分。
4. **减少数据需求**：Fine-Tune通常只需要相对较少的特定领域数据即可实现有效的模型调整。

#### 缺点

1. **过度专业化的风险**：过度专注于特定领域可能导致模型在泛化能力上的下降，对于其他类型的数据或场景可能表现不佳。
2. **数据质量依赖性**：Fine-Tune的效果高度依赖于意图识别的质量。如果数据存在偏差或不准确，模型的表现会受到影响。
3. **维护与更新的挑战**：随着领域知识的更新和变化，维护和更新Fine-Tune后的模型可能会变得复杂和耗时。



如果语料库中增加了对参数的描述，那么还可以直接返回参数，效果堪比FunctionCall。



## 基于语义的词语分类模型

在这种模式中，模型通过分析词语之间的语义相似度来识别意图。这种方法不仅关注用户输入的字面意义，而且考虑到词语之间的关联和上下文信息。这使得模型能够更好地理解复杂或含糊的表达，从而提高意图识别的准确性。本质属于“词语分类”。

当用户提供Input后，首先通过Embedding模型生成高纬向量值，然后再根据某种相似度算法在向量库中找到TopK意图。下一步根据某种条件进行抉择: 相似度或者交给LLM。

这种方法的一个关键优势在于能够将单个词语放置在整个句子或对话的上下文中进行分析。这意味着即使用户的表达方式不常规或存在歧义，模型也能通过分析整体上下文来理解其意图。

由于这种模型不依赖于固定的关键词匹配，它能更好地处理新的或未见过的表达方式。这种泛化能力使模型在面对多样化的用户输入时更加鲁棒。



相比前面几种都依赖LLM进行决策，依据向量库几乎没有性能损耗(除了Embedding过程)。 而且每次需要新增意图时，只需要维护向量库就可以了。

#### 优点

1. **增强的语义理解能力**：这种模型通过分析词语间的语义相似度，能够捕捉用户输入的深层含义，而不仅限于字面意思。这种深度理解有助于更准确地识别用户的真实意图。
2. **上下文信息的综合考虑**：模型不只关注单个词语，而是将词语置于更广泛的上下文中考虑，这有助于理解复杂或模糊的表达。
3. **应对模糊表达的能力**：在处理不清晰或多义的输入时，这种方法可以通过分析语义关系来提高识别的准确度。
4. **泛化能力的提升**：相比于仅依赖特定的关键词匹配，这种模型能更好地处理未见过的表达方式，显示出更强的泛化能力。

#### 缺点

1. **对高质量数据的依赖**：为了有效地分析语义相似度，模型需要大量高质量、丰富多样的训练数据，这在某些情况下可能难以获得。
2. **误解读的可能性**：尽管能够处理语义上的细微差别，但仍存在误解读用户意图的风险，特别是在语境不明确或用户使用非标准表达时。
3. **仅能识别意图**：无法提取参数



## 模糊场景下的补充引导模式

在模糊场景下，补充引导模式发挥着重要作用。当用户输入不够清晰或具体时，系统可以提出进一步的问题或提示，以帮助澄清用户的意图。这种互动式的方法不仅提高了意图识别的准确性，也增强了用户体验。

比如用户输入“班车”。 按照上面几种方式，可能会直接推测出所有与班车有关的意图。 这样的意图没有6个，也有3个(班车时间，班车地点，班车是否开通)。 在这种消息量非常短的场景中，就需要引导用户补充更丰富的信息。例如: 

我目前支持: 1. 查询班车时间。 2. 查询班车地点。3. 班车是否开通 。 请问您需要咨询哪一类班车问题?

当用户再进行后续选择的时候，就可以决策出更精确的意图了。

这种方式的实现依赖: Prompt+向量检索。 具体来说，1. 通过向量检索召回TopK潜在意图。 2. 对潜在意图进行分析判断是否可以得到准确意图(通过LLM)。 3. 如果不能获取准确意图，则生成引导问题返回给用户。

通过这三个步骤(在最开始还有一个基于会话历史的问题改写)可以逐步缩短到一个收敛的意图范围。



### 优点

1. **提高准确性**：通过提出补充问题或提示，系统可以更准确地理解用户的意图，特别是在初次输入模糊或不明确时。
2. **增强用户体验**：互动式引导提供了更为个性化和参与感强的用户体验，有助于建立用户的信任和满意度。
3. **降低误解风险**：补充引导可以帮助系统更好地理解用户的需求，从而降低因误解用户意图而产生错误反馈的风险。
4. **适应性强**：这种模式能够根据用户的具体输入动态调整，展现出高度的适应性和灵活性。

### 缺点

1. **可能增加用户负担**：当用户期望快速回答时，过多的提问或提示可能会使用户感到难受。
2. **实现复杂度高**：设计一个有效的补充引导系统需要考虑到多种因素，如提问的时机、内容和方式，这增加了实现的复杂度。
3. **对话流程可能变慢**：借助LLM进行决策导致对话流程变慢。
4. **依赖上下文理解能力**：这种模式的成功在很大程度上依赖于系统理解上下文的能力，如果系统不能准确解读上下文，引导可能会变得无效甚至误导。
5. **不支持参数**：老生常谈的问题，真的不支持参数。
