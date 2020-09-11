### 《Span-based Joint Entity and Relation Extraction with Transformer Pre-training》（ECAI 2020）
SpERT是一个使用**片段分类方法**作为解码器的**联合参数模型**，片段分类方法就是对n个tokens的segment进行遍历，寻找到所有可能的实体。例如，北京大学可以分解为e(北)，e(北京)，e(北京大)，e(北京大学)。相比与序列标注法这种方法可以很好的识别重叠实体，但片段的文本过长时会产生大量的负样本。

SpERT的模型结构可以分为三层，分别为span classification、span filtering和relation classification。

**Span Classification**：这一层负责识别所有的实体，sentence经过预训练模型BERT进行编码，得到结果$segment=(e_{i}, e_{i+1}, ..., e_{i+k})$，将$f_{maxpooling}(segment)+embedding_{width}+cls$作为span filtering的输入，$cls$是输入句子的特征表示，$embedding_{width}$表示向量的跨度特征
**Span Filtering**：对实体进行筛选，大于10tokens的实体直接排除
**Relation Classification**：识别实体之间的关系。在将（subject,object）输入linear层之前，加入上下文信息（suject,contextual embedding,object）,**contextual embedding**是subject和object之间的信息。
SpERT提取实体的关系的方式是**ETC(extract-then-classify)**，这种方法的缺陷是，需要对N个实体进行$N*N*relation_nums$次的概率计算分类，来判断实体之间的关系。作者在Span Classification和relation classification阶段采取了负采样来减少计算代价。

### 《Attention guided graph convolutional networks for relation extraction》(ACL 2019)
AGGCN利用GCN从dependency tree中提取特征，得到实体间的关系。**这篇论文在GCN的基础上加上了Attention Guided Layer(AG)，利用multi-head attention训练多个邻接矩阵使模型可以关注到不同的子空间的信息**。邻接矩阵$A_{ij}$表示$node_{i}$和$node_{j}$之间的关系强度，通过这种“soft pruning”的方法可以更全面地提取句子中的特征信息。Attention机制也可以为后续的Desenly Connected Layer过滤掉噪音以及不重要的信息，提高模型的提取能力。 

>**The key idea** behind the attention guided layer is to use attention for inducing relations between nodes, especially for those connected by indirect, multi-hop paths. These soft relations can be captured by differentiable functions in the model.

模型结构分为三部分，分别是预处理、图卷积部分以及关系提取部分
**预处理**：将sentence转换成关系树，以邻接矩阵的形式存储
**图卷积部分**：这部分是主要负责提取关系特征，由M个相同的Block组成，每个Block有Attention Guided Layer、$N$个Densely Connected Layer和Linear Combination Layer
- Attention Guided Layer：
这是本文最关键的部分，将预处理的邻接矩阵由多头注意力机制分为N个不同的邻接矩阵，从而从多个角度提取不同关系的特征。相比于基于规则的硬剪枝方法，将不符规则的部分权重直接设为零，这种soft pruning的方式可以更好地提取句子中潜在的信息。同时可以过滤后续Densely Connected Layer中无用的信息和噪音。
$$
\tilde{\mathbf{A}}^{(\mathbf{t})}=\operatorname{softmax}\left(\frac{Q \mathbf{W}_{i}^{Q} \times\left(K \mathbf{W}_{i}^{K}\right)^{T}}{\sqrt{d}}\right) V
$$
- Densely Connected Layer：
为了可以训练更深的GCN，采用了（密集连接）dense connections的方法。密集连接就是对前每一层都加一个单独的shortcut，使得任意两层网络都可以直接“沟通”。Densely Connected Layer由多个子层，子层$l$的计算输出为
$$
\mathbf{h}_{t_{i}}^{(l)}=\rho\left(\sum_{j=1}^{n} \tilde{\mathbf{A}}_{i j}^{(t)} \mathbf{W}_{t}^{(l)} \mathbf{g}_{j}^{(l)}+\mathbf{b}_{t}^{(l)}\right)
$$
其中
$$\mathbf{g}_{j}^{(l)}=[x_{j};h_{j}^{1};...;h_{j}^{l-1}]$$
第$l$层的输入是前$l-1$层的一个组合输入。这样浅层和深层的特征可以随意组合，会使模型的结果更加robust。从 wide-network 角度来看， DenseNet 可以被看作一个真正的宽网络，在训练时会有比 ResNet 更稳定的梯度，收敛速度自然更好。
- Linear Combination Layer：
$$
\mathbf{h}_{c o m b}=\mathbf{W}_{c o m b} \mathbf{h}_{o u t}+\mathbf{b}_{c o m b}
$$
其中$h_{out}=[h^{1};...;h^{N}]$，将N个Densely Connected Layer的最终输出通过一层线性层整合在一起。
**关系提取部分**：利用一个全连接网络进行关系的预测分类
$$
h_{\text {final}}=\operatorname{FFNN}\left(\left[h_{\text {sent}} ; h_{e_{1}} ; \ldots h_{e_{i}}\right]\right)
$$
$h_{sent}$为句子中非实体部分的编码表示，$h_{e_{i}}$表示实体i的隐表示（每个token都经过了max pooling操作）。