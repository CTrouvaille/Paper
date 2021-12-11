# Paper
# TRUSTED MULTI-VIEW CLASSIFICATION

**多视图分类(MVC)通常侧重于通过使用来自不同视图的信息来提高分类精度,不同视图的信息来提高分类精度，通常将它们集成到下游任务的统一综合表示中。**

**然而，动态评估不同样本的视图质量也是至关重要的，以便提供可靠的不确定性估计，这表明预测是否可以可信。**

**SO 通过在证据层次上动态整合不同的视图，为多视图学习提供了一个新的范式。**

**论文的算法通过整合多个视图和多个视图的证据来提高分类可靠性和鲁棒性**。为了实现这一点，使用狄利克雷分布模拟概率分布，用来自不同观点的证据参数化，并与邓普斯特谢弗理论（证据理论）相结合。统一的学习框架引入了准确的不确定性，从而赋予了模型对分布外样本的可靠性和鲁棒性。



动机：

分类效果很好，但是当没有良好的视图时（比如传感器异常）将会产生不可靠的预测。所以在安全相关关键应用上受到限制（例如自动驾驶）。

于是，我们做一个多视图分类的范式，来产生可信任的决策。

### 现有方法

通过算法为每一个视图假设一个相等的值；或者为每一个视图分配/学习一个固定的权重。

原因：假设这些视图的质量和重要性在所有样本上都是稳定的。

缺点：在现实中，视图的质量往往因不同的样本而不同，模型设计应该适应这个变化。

分类效果很好，但是当没有良好的视图时（比如传感器异常）将会产生不可靠的预测。所以在安全相关关键应用上受到限制（例如自动驾驶）。



所以**需要为每个样本的预测提供预测的准确的不确定性，甚至每个样本的每一个视图。**



### 不确定性算法

* 贝叶斯方法。

传统的贝叶斯方法通过推断参数的后验分布来估计不确定性。

eg：拉普拉斯近似、马尔科夫链蒙特卡罗、变分技术

但是与普通的神经网络相比，参数加倍以及收敛困难，计算成本太大。如今有算法通过在测试阶段引入dropout来估计不确定性，减少了计算成本。

* 非贝叶斯方法

deep ensemble、evidential deep learning、deterministic uncertainty estimate

但是这些方法都聚焦于在单视图数据上估计不确定性。

然而融合多视图不确定性能提高性能和可靠性。



### SO

提出一个多视图分类模型来结合不同视图信息来做出可信任的决策。

在证据层上结合不同视图    VS   在特征/输出层结合

从而产生稳定合理的不确定性估计，提高分类的可靠性和鲁棒性。

狄利克雷分布用于模拟类概率的分布，用来自不同观点的证据进行参数化，并与邓普斯特-谢弗理论（证据理论）相结合。



### 模型结构

![](https://raw.githubusercontent.com/CTrouvaille/FigureBed/main/img/20211211143337.png)

每一个视图都会得到一系列$\mathbf{e}^{v}=\left[e_{1}^{v}, \cdots, e_{K}^{v}\right]$，称为证据。然后狄利克雷分布由证据得到${\alpha}^{v}=\left[\alpha_{1}^{v}, \cdots, \alpha_{K}^{v}\right]$,其中$\alpha_{k}^{v}=e_{k}^{v}+1$。然后置信质量$b$和不确定性$u$由下列公式得到：
$$
b_{k}^{v}=\frac{e_{k}^{v}}{S^{v}}=\frac{\alpha_{k}^{v}-1}{S^{v}} \quad \text { and } \quad u^{v}=\frac{K}{S^{v}}
$$
其中，$S^{v}=\sum_{i=1}^{K}\left(e_{i}^{v}+1\right)=\sum_{i=1}^{K} \alpha_{i}^{v}$，叫做狄利克雷强度。

（其实也就是所有的α相加和。但是因为$\alpha_{k}^{v}=e_{k}^{v}+1$，所以对$α$求和得到$S$后，相当于$e$求和之后再加上$K$，而$M$相当于计算每一个$e$占$S$的比重，所以多出的$u$就等于多加上的$k$除以$S$)

证据足够强时（e），不确定性u可以忽略（趋于0）；证据非常弱时，不确定性变为极大（趋于1）

每一个视图都会得到一组$M$，然后将多个视图的$M$拼接在一起。不采用特征级融合、决策层融合、数据级融合拼接。融合方式如下：

![](https://raw.githubusercontent.com/CTrouvaille/FigureBed/main/img/20211211150213.png)
$$
b_{k}=\frac{1}{1-C}\left(b_{k}^{1} b_{k}^{2}+b_{k}^{1} u^{2}+b_{k}^{2} u^{1}\right), u=\frac{1}{1-C} u^{1} u^{2}
$$
其中，$C=\sum_{i \neq j} b_{i}^{1} b_{j}^{2}$，图中空白部分的集合。空白部分表示各模态做出分类判别结果出现冲突的情况，把它舍弃掉。$\frac{1}{1-C}$用于归一化。

新的$b_{k}$的计算其实就是计算该类别的分数占所有预测情况分数的比例。之所以加上$b_{k}*u$，是因为当一个模态打出不确定的预测时，就以另一个模态的分类预测为准，即也是打出属于该分类的分数。

证据理论融合合理性及优势：

* 当两种模态不确定性都较高（置信度低）时，最终分类一定是低置信度；

* 当两种模态不确定性都较低（置信度高）时，最终分类一般是高置信度；（可能存在决策冲突的例外情况）

* 当仅有一种模态不确定性较低时，最终分类是由该模态决定；

* 当两种模态决策冲突时，最终分类可信度降低。

**使用主观逻辑相比softmax的优势**：与softmax输出相比，使用主观不确定性更适合多决策的融合。主观逻辑提供了一个**额外的mass函数(u)**，允许模型区分缺乏证据。在我们的模型中，主观逻辑提供了每个视图的总体不确定程度，这在某种程度上对于可信分类和可解释性很重要。

**如何获取证据$e$？**

将softmax层换成激活函数（RELU）来确保输出为非负值，此向量就当作证据$e$.

### 损失函数

传统的分类使用交叉熵函数：
$$
\mathcal{L}_{c e}=-\sum_{j=1}^{K} y_{i j} \log \left(p_{i j}\right)
$$
$p_{i j}$为第i个样本预测为j类的概率。

需要对其进行改良，首先对其在狄利克雷分布上积分，得到下列损失函数：
$$
\mathcal{L}_{a c e}\left(\boldsymbol{\alpha}_{i}\right)=\int\left[\sum_{j=1}^{K}-y_{i j} \log \left(p_{i j}\right)\right] \frac{1}{B\left(\boldsymbol{\alpha}_{i}\right)} \prod_{j=1}^{K} p_{i j}^{\alpha_{i j}-1} d \mathbf{p}_{i}=\sum_{j=1}^{K} y_{i j}\left(\psi\left(S_{i}\right)-\psi\left(\alpha_{i j}\right)\right)
$$
上述损失函数确保了每个样本的正确标签比其他类别产生更多的证据，但它不能保证对不正确的标签产生更少的证据。也就是说，在模型中，预计不正确标签的证据会缩小到0。

所以对其加入正则约束，引入狄利克雷类分布先验信息融入，抑制不合理分布：
$$
\begin{array}{l}
K L\left[D\left(\mathbf{p}_{i} \mid \tilde{\boldsymbol{\alpha}}_{i}\right) \| D\left(\mathbf{p}_{i} \mid \mathbf{1}\right)\right] \\
\quad=\log \left(\frac{\Gamma\left(\sum_{k=1}^{K} \tilde{\alpha}_{i k}\right)}{\Gamma(K) \prod_{k=1}^{K} \Gamma\left(\tilde{\alpha}_{i k}\right)}\right)+\sum_{k=1}^{K}\left(\tilde{\alpha}_{i k}-1\right)\left[\psi\left(\tilde{\alpha}_{i k}\right)-\psi\left(\sum_{j=1}^{K} \tilde{\alpha}_{i j}\right)\right]
\end{array}
$$
于是，对于单模态的损失函数为：
$$
\mathcal{L}\left(\boldsymbol{\alpha}_{i}\right)=\mathcal{L}_{a c e}\left(\boldsymbol{\alpha}_{i}\right)+\lambda_{t} K L\left[D\left(\mathbf{p}_{i} \mid \tilde{\boldsymbol{\alpha}}_{i}\right) \| D\left(\mathbf{p}_{i} \mid \mathbf{1}\right)\right]
$$
$\lambda_{t}$为平衡因子，大于0。在实践中，我们可以逐渐增加$\lambda_{t}$的值，以防止网络过于关注训练的KL散度的初始阶段，否则可能会导致缺乏良好的探索参数空间，导致网络输出一个均匀的分布。

对于多个模态，采用多任务策略损失函数：
$$
\mathcal{L}_{\text {overall }}=\sum_{i=1}^{N}\left[\mathcal{L}\left(\boldsymbol{\alpha}_{i}\right)+\sum_{v=1}^{V} \mathcal{L}\left(\boldsymbol{\alpha}_{i}^{v}\right)\right]
$$



