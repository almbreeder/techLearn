### 《Joint type inference on entities and relations via graph convolutional networks》(ACL 2019)

利用GCN进行关系提取，与前面看到的几篇论文不同，在关系提取阶段**将模型识别的实体作为关系提取部分的输入**，而不是将实体作为已知条件，利用联合参数模型进行关系分类。模型的结构和[SpERT模型](./week1)有些相似，利用了entity-type、entity-entity(relation)之间的信息来训练整个模型。
图卷积网络的输入并没有建立在依存树的基础上，作者不仅将实体看作一个node，而且将relation也作为图中的node，通过relation node与相关实体来构建图。文中提出了三种形式的图**static graph**、**hard dynamic graph**和 **soft dynamic graph**。
- **static graph**：默认所有的实体是有关系的，构成一个全连接的图
- **hard dynamic graph**：不是所有的实体间都有关系的，作者在建立图之前通过**binary relation classification**的一个二分器来预测实体间是否有关系，有关系则赋值为1，反之为0（**相当于剪枝策略**）。
- **soft dynamic graph**：就是将预测的概率作为邻接矩阵的权重，**实验结果与static graph相似，没什么提升，反而hard效果较好**

模型结构:
![](./images/8_26(1).PNG)
模型可以分为两部分，第一部分是**利用sequence-based model提取实体向量和关系向量**，第二部分将实体向量和关系向量构建成图作为GCN的输入，进而提取实体特征和关系特征，实现实体类别的识别和实体关系的识别。

**loss function:**

采用的是BILOU标记策略，计算实体序列预测的损失函数
$$
\mathcal{L}_{\mathrm{span}}=-\frac{1}{|s|} \sum_{i=1}^{|s|} \log P\left(\hat{t}_{i}=t_{i} \mid s\right)
$$
计算实体间是否存在关系的损失函数
$$
\mathcal{L}_{\mathrm{bin}}=-\sum_{r_{i j}} \frac{\log P\left(\hat{b}=b \mid r_{i j}, s\right)}{\# \text { candidate relations } r_{i j}}
$$
计算实体类型识别的损失函数
$$
\mathcal{L}_{\mathrm{ent}}=-\frac{1}{|\hat{\mathcal{E}}|} \sum_{e_{i} \in \hat{\mathcal{E}}} \log P\left(\hat{y}=y \mid e_{i}, s\right)
$$
计算关系分类的损失函数
$$
\mathcal{L}_{\mathrm{rel}}=-\sum_{r_{i j}} \frac{\log P\left(\hat{l}=l \mid r_{i j}, s\right)}{\# \text { candidate relations } r_{i j}}
$$
总的损失函数为：
$$
\mathcal{L} = \mathcal{L}_{\mathrm{span}} + \mathcal{L}_{\mathrm{bin}} + \mathcal{L}_{\mathrm{ent}} + \mathcal{L}_{\mathrm{rel}}
$$

缺点：
1. 由于使用的是BILOU标记策略，无法解决实体重叠的问题
2. 无法解决识别关系重叠的问题