###《Graph Convolution over Pruned Dependency Trees Improves Relation Extraction》（ACL2018）
### Introduction
作者利用图卷积网络来提取实体间的关系，将依存树转换成对应的图作为图卷积网络的输入，本质上该方法是基于依存树提取特征。相比于Tree-BiLSTM，图卷积的可并行性大大提高了运行效率。论文中的另一个创新点是提出了一种新的剪枝方法$path-centric\space pruning$，作者发现 shortest dependency path 方法虽然效率很高，但是会遗漏掉一些关键信息（eg. not/no），从而导致理解的意思完全不同。作者在最短依赖路径的基础上进行拓展获取更多可能的信息，例如将距离每个节点的为1的邻居节点添加到最短依赖路径上形成一条性的路径，这就是文中所采用的剪枝策略。

### Model
图卷积的关系提取可以分为三个步骤：预处理、特征提取和关系分类
**预处理**：
根据剪枝的规则将句子转换成对应的图，输入至图卷积网络中进行特征提取
**特征提取**：
利用GCN来提取依赖树中的关系特征，GCN由$L$个子层构成，这是因为每一个卷积层仅处理一阶邻域信息，通过叠加若干卷积层可以实现多阶邻域的信息传递。
$l$层中每个节点的输出满足公式
$$
h_{i}^{(l)}=\sigma\left(\sum_{j=1}^{n} \tilde{A}_{i j} W^{(l)} h_{j}^{(l-1)} / d_{i}+b^{(l)}\right)
$$
$\tilde{A}= A +I$,$A$是图的邻接矩阵，加上$I$是一个单位向量是为了将自己的特征信息保留，$\tilde{A}_{i j}$表示$node_{j}$和$node_{j}$之间的连接关系（作者发现有向图和无向图对结果的影响不大，故采用了无向图，所以邻接矩阵是一个对称矩阵）。$d_{i}=\sum_{j=1}^{n} \tilde{A}_{i j}$表示$token_{i}$的度，除以每个node的度进行归一化处理，防止由于节点度的分布不同产生较大差异。
**关系分类**:
将提取后的hidden representation中的subject、object和sentence进行maxpolling的操作后进行concatenate，输入一个全连接层中进行关系的分类。

### Experiment
- 对于剪枝策略的实验：
发现当K=1时效果最好，适当添加off-path信息效果更好。实验表明使用完整的依存树性能比LCA子树效果更差，表明了过多的信息也会损害模型的提取的效果，给网络增加分类的负担。
- 序列模型和图卷积的互补
序列模型擅长处理长度较短的句子，图卷积则擅长处理长度较长的句子。本文的采用了一个简单的结合方法
$$P(r \mid x)=\alpha \cdot P_{G}(r \mid x)+(1-\alpha) \cdot P_{S}(r \mid x)$$
分别给图卷积$P_{G}$和序列模型$P_{S}$一个预测的权重（文中$$\alpha = 0.6$），发现模型的F1score提升了2.0+。
而采用Bi-LSTM对句子进行编码的C-GCN，剪枝对于C-GCN没有太大的影响，作者认为经过Bi-LSTM编码的序列中提取到了上下文的信息，序列中已经存储了off-path的特征。


## 《Graph Neural Networks with Generated Parameters for Relation Extraction》
这篇文章是关于关系推理的，传统的关系推理只能建立再预定义好的图上。本文利用参数生成的方法从无结构的信息（文本）推理出实体间的关系。

模型结构分为三部分：
**Encoding Module**：
将句子编码并生成转换矩阵$A_{ij}^{(n)}$
$$
\mathcal{A}_{i, j}^{(n)}=[MLP_{n}(f\left(E\left(x_{0}^{i, j}\right), E\left(x_{1}^{i, j}\right), \cdots, E\left(x_{l-1}^{i, j}\right) ; \theta_{e}^{n}\right))]
$$
其中$x_{t}^{i,j} = [x_{t},pos_{t}^{i,j}]$，$i,j \in {1,2,3}$，$pos_{t}^{i,j}$表示的是$x_{t}$距离实体1和实体2的长度向量，所以一共有三种转换矩阵$\mathcal{A}_{1, 2}^{(n)},\mathcal{A}_{2, 3}^{(n)},\mathcal{A}_{3, 1}^{(n)}$，其中3表示的是与非实体的距离的向量。$E(*)$表示的是向量化，$f(*)$表示的是BiLSTM编码，MLP为多层感知器(激活函数为tanh)，$[*]$调整矩阵维度。
**Propagation Module**：
利用GNN网络对模型进行特征向量的提取
$$
\mathbf{h}_{i}^{(n+1)}=\sum_{v_{j} \in \mathcal{N}\left(v_{i}\right)} \sigma\left(\mathcal{A}_{i, j}^{(n)} \mathbf{h}_{j}^{(n)}\right)
$$
$\mathcal{N}\left(v_{i}\right)$表示为实体$v_{i}$的所有邻居实体

**Classification Module**
将关系的表示向量输入，得到每个类别的概率。
$$
\boldsymbol{r}_{v_{i}, v_{j}}=\left[\left[\boldsymbol{h}_{v_{i}}^{(1)} \odot \boldsymbol{h}_{v_{j}}^{(1)}\right]^{\top} ;\left[\boldsymbol{h}_{v_{i}}^{(2)} \odot \boldsymbol{h}_{v_{j}}^{(2)}\right]^{\top} ; \ldots ;\left[\boldsymbol{h}_{v_{i}}^{(K)} \odot \boldsymbol{h}_{v_{j}}^{(K)}\right]^{\top}\right]
$$
两个节点之间的关系推理使用了这两个节点在每一层GNN的输出信息构建了特征向量，使用多层感知机+softmax输出分类概率
$$
\mathbb{P}\left(r_{v_{i}, v_{j}} \mid h, t, s\right)=\operatorname{softmax}\left(\operatorname{MLP}\left(\boldsymbol{r}_{v_{i}, v_{j}}\right)\right)
$$

[参考文章](https://blog.csdn.net/TgqDT3gGaMdkHasLZv/article/details/103839705)