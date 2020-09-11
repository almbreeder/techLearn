### 《Inter-sentence Relation Extraction with Document-level Graph Convolutional Neural Network》（ACL 2019)
#### 简介
这是一篇inter-sentence level的关系提取，inter-sentence level的关系提取除了要学习到intra-sentence level的local关系，还需要non-local信息来联系。句子的语法~~语义（没有LSTM之类的提取语义关系~~)关系和序列信息（relative position）为local information。Adjacent sentence 和 **coreference** 信息为non-local信息。该模型利用GCN提取local和non-local信息，借助多示例学习提取实体关系。

####模型结构：
![](./images/8_31(1).png)

**Input Layer:**
将文本进行向量化，同时加入距离目标实体的相对位置信息$x_{i} = {w_{i};d_{i}^{1};d_{i}^{2}}$

**GCN:**
1）图的构建
可以看到GCNN模型涉及了许多不同维度的信息，为每种信息都创建了对应的邻接矩阵，为了后续实验的方便（文中说是避免over-parameterisation）,将不需要的信息统归为rare类的一个邻接矩阵。
2）GCN layer
$$
\mathbf{x}_{i}^{k+1}=f\left(\sum_{u \in \nu(i)}\left(\mathbf{W}_{l(i, u)}^{k} \mathbf{x}_{u}^{k}+\mathbf{b}_{l(i, u)}^{k}\right)\right)
$$
其中$\mathbf{W}_{l(i, u)}^{k}$表示的是第$k$层信息种类为$l$的参数，$\mathbf{x}_{i}^{k+1}$表示第$k+1$层第$i$个token的隐表达，$\nu(i)$是为token $i$ 的邻居集合。

**Bi-affline Layer:**
将上一层的输出分别作为两个多层感知器的输入，得到head和tail的表达式（并不是entities的表达，而是将整个句子做了一个处理）。遍历所有的knowledge id相同的实体，通过bi-affine layer计算head entity和 tail entity的confidence scores,
$$
\operatorname{scores}\left(e^{\text {head}}, e^{\text {tail}}\right)= 
\log \sum_{i \in E^{\text {head}}, j \in E^{\text {tail}}} \exp \left(\left(\mathbf{x}_{i}^{\text {head}} \mathbf{R}\right) \mathbf{x}_{j}^{\text {tail}}\right)
$$
其中$R \in \mathbb{R}^{d \times r \times d}$，$LogSumExp$函数可以类似于Max函数的功能。
MIL的作用是可以对远程监督提供的样本进行一个降噪的功能，减少负样本的影响。


###《Simultaneously Self-Attending to All Mentions for Full-Abstract Biological Relation Extraction》（NAACL 2018）
上一篇文章采用的就是该模型的思路，将提取关系信息的部分的Transformer换成了GCN，并添加了coreference作为全局变量构建图网络。作者主要解决的是document-level生物医疗文本的关系提取任务。模型的**Transformer部分由多头注意力和卷积层堆叠而成**，并利用MIL学习解决关系分类问题。

模型结构：
<div align = center>
    <img src = "./images/9_4(1).png">
</div>

### 《Downstream Model Design of Pre-trained Language Model for Relation Extraction Task》（arXiv 2020）
这篇论文在SemEval 2010 Task 8, NYT, WebNLG这三个数据集上都达到了SOTA。这篇论文给每个关系对计算所有可能出现的关系概率，与HBT模型的思路类似。

[github链接](https://github.com/slczgwh/REDN)

模型结构：
![](./images/9_6(1).png)

