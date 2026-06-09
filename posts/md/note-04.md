# 跨域自适应理论学习笔记（四）

## 一、信息瓶颈（Information Bottleneck）：深度网络在"删什么留什么"？

### 1.1 从一个压缩包开始

有一部4K电影（100GB），要发到微信上（限100MB）。必须压缩。
- **无脑压缩**：把画面糊成马赛克，声音砍成单声道。文件小了，但剧情也看不清了。
- **聪明压缩**：保留剧情关键帧、对话清晰度、背景音乐；删掉毛孔细节、远景树叶的抖动、无人注意的阴影。

**信息瓶颈**问的是：在给定压缩率下，**删什么、留什么**，才能保留最多"关于结局的信息"？

### 1.2 互信息（Mutual Information）

#### 熵（Entropy）：不确定性本身

$$H(X) = - \sum_{x}^{}P(x)\log P(x)$$

$X$ 是一个随机变量。在知道它的取值之前，你有多迷茫？
- 公平硬币：$P(\text{正}) = 0.5, P(\text{反}) = 0.5$，$H = \log 2 \approx 0.693$（nats）或 1（bits）
- 太阳从哪边升起：$P(\text{东}) = 1$，$H = 0$

**核心直觉**：熵 = 消除不确定性所需的信息量。确定性事件熵为0，最不确定事件熵最大。

#### 条件熵（Conditional Entropy）

$$H\left( Y|X \right) = \sum_{x}^{}P(x)H\left( Y|X = x \right) = - \sum_{x,y}^{}P(x,y)\log P\left( y|x \right)$$

已经知道 $X$ 的值了，$Y$ 还剩下多少不确定性？
- $Y$ = 是否带伞，$X$ = 是否下雨
- 不知道 $X$ 时，$Y$ 很不确定；知道 $X = \text{下雨}$ 后，$Y$ 几乎确定（带伞）。所以 $H(Y|X)$ 很小。

#### 互信息（Mutual Information）

$$I(X;Y) = H(Y) - H\left( Y|X \right) = H(X) - H\left( X|Y \right)$$

| 符号 | 含义 | 人话 |
|------|------|------|
| $H(Y)$ | $Y$ 的原始不确定性 | 完全不知道 $X$ 时，$Y$ 有多迷 |
| $H(Y|X)$ | 知道 $X$ 后 $Y$ 的不确定性 | 看了 $X$ 之后，$Y$ 还迷不迷 |
| $I(X;Y)$ | 两者之差 | $X$ 帮 $Y$ 消除了多少迷茫 |

等价形式：

$$I(X;Y) = \sum_{x,y}^{}P(x,y)\log\frac{P(x,y)}{P(x)P(y)}$$

如果 $X$ 和 $Y$ 独立，$P(x,y) = P(x)P(y)$，对数项为0，互信息为0。如果它们强相关，联合概率远大于边缘概率乘积，互信息很大。

**关键性质**：
- $I(X;Y) \geq 0$（知道一点总不会更差）
- $I(X;Y) = I(Y;X)$（对称）
- $I(X;Y) \leq \min(H(X), H(Y))$（共同信息不会超过任一方的总信息）

### 1.3 信息瓶颈的数学框架

Tishby等人（1999）提出：好的表示 $T$ 应该是最小充分统计量。

**设定**：
- $X$：输入（原始高维数据，如图片像素）
- $Y$：标签（如"猫"或"狗"）
- $T$：表示（Representation，如神经网络某层的输出）

**目标**：找一个编码器 $P(T|X)$，使得：
1. $T$ 尽可能**压缩** $X$（保留少量信息）
2. $T$ 尽可能**预测** $Y$（保留关于标签的信息）

**数学形式**（Lagrangian）：

$$\min_{P(T|X)}\underbrace{- I(T;Y)}_{\text{预测损失}} + \beta\underbrace{I(T;X)}_{\text{压缩程度}}$$

或等价地写成约束优化：

$$\max_{P(T|X)}I(T;Y)\quad\text{s.t.}\quad I(T;X) \leq R$$

| 项 | 含义 | 人话 |
|----|------|------|
| $I(T;Y)$ | 表示与标签的互信息 | 表示里有多少"关于答案"的信息 |
| $I(T;X)$ | 表示与输入的互信息 | 表示里有多少"关于原始数据"的信息 |
| $\beta$ | Lagrange乘子 | 压缩力度：$\beta$ 越大，越狠地压缩 |

**核心直觉**：
- $I(T;X)$ 很大：表示 $T$ 记住了输入的很多细节（过拟合风险）
- $I(T;X)$ 很小：表示 $T$ 高度压缩，但可能把判别信息也压掉了
- $I(T;Y)$ 很大：表示 $T$ 保留了预测标签的关键线索
- **最优解**：在压缩曲线上找到 $I(T;Y)$ 最大的点

### 1.4 信息平面（Information Plane）

把神经网络的每一层画在二维平面上：
- 横轴：$I(T;X)$（该层保留了多少输入信息）
- 纵轴：$I(T;Y)$（该层保留了多少标签信息）

**Tishby的观察**（对简单网络）：
1. **拟合期**：网络快速增加 $I(T;Y)$，同时 $I(T;X)$ 也在增加（学习有用特征）
2. **压缩期**：$I(T;X)$ 开始下降（遗忘输入细节），但 $I(T;Y)$ 保持或缓慢上升（泛化期）

**人话**：神经网络先"学会"，再"忘掉没用的"。

### 1.5 与跨域自适应的联系

**核心命题**：跨域自适应中，好的表示 $T$ 应该满足：
- $I(T;Y)$ **跨域大**（保留与任务相关的信息）
- $I(T;\text{Domain})$ **跨域小**（不保留与域身份相关的信息）

其中 $\text{Domain}$ 是指示变量（源=0，目标=1）。

**数学目标**：

$$\max_{T}I(T;Y) - \beta I(T;X) - \lambda I\left( T;\text{Domain} \right)$$

不仅要压缩输入，还要**专门压缩"域身份信息"**。

**与IRM的对比**：
- IRM：通过约束"同一个分类器在所有域上最优"来迫使 $T$ 丢弃域相关信息
- 信息瓶颈：直接量化和最小化 $I(T;\text{Domain})$，更显式

**实现难点**：$I(T;\text{Domain})$ 需要从样本估计。如果域分类器能完美区分源域和目标域样本，说明 $I(T;\text{Domain})$ 很大，表示 $T$ 泄露了域身份。

### 1.6 变分信息瓶颈（VIB）

互信息涉及未知分布 $P(X)$ 和 $P(Y|X)$，实际中无法直接计算。

**VIB的近似**：

$$I(T;X) = \mathbb{E}_{P(x)}\left\lbrack D_{KL}\left( P(T|X = x) \parallel P(T) \right) \right\rbrack$$

其中 $P(T)$ 是边缘分布。VIB用**变分近似** $Q(T)$ 代替 $P(T)$，用**重参数化技巧**（Reparameterization Trick）采样 $T$。

**实际损失函数**：

$$\mathcal{L}_{VIB} = \underbrace{\mathbb{E}_{x,y}\left\lbrack - \log Q\left( Y|T \right) \right\rbrack}_{\text{交叉熵（预测损失）}} + \beta\underbrace{\mathbb{E}_{x}\left\lbrack D_{KL}\left( P(T|X = x) \parallel Q(T) \right) \right\rbrack}_{\text{KL散度（压缩惩罚）}}$$

- 第一项：预测要准（最大化 $I(T;Y)$ 的下界）
- 第二项：表示分布要接近某个简单先验 $Q(T)$（通常是标准高斯），防止记住太多输入细节

---

## 二、流形上的最优传输：当数据不在"平地"而在"山谷"

### 2.1 为什么需要流形？

在广州开车。导航说"从天河到番禺直线距离20公里"。但实际上要走华南快速干线，绕过山、跨过江，实际路程40公里。

**原因**：城市不是一张平坦的纸，道路网络是一个**流形（Manifold）**——局部看起来像平面，但整体有弯曲、有约束。

**数据同理**：
- 人脸图片：理论上在 $\mathbb{R}^{1024 \times 1024}$ 空间里，但实际所有人脸都蜷缩在一个低维流形上（眼睛、鼻子、嘴的相对位置有限）
- MNIST手写数字：784维（28×28），但实际有效维度可能只有10-20维

**问题**：如果在全空间 $\mathbb{R}^{d}$ 上算Wasserstein距离，可能会穿过"不可能的图片"区域（比如眼睛长在鼻子下面），距离失真。

### 2.2 流形的数学定义（直观版）

一个 $k$ 维流形 $\mathcal{M}$ 嵌入在 $\mathbb{R}^{d}$ 中（$k \ll d$）：
- **局部**：每一点 $p \in \mathcal{M}$ 附近，$\mathcal{M}$ 看起来就像 $\mathbb{R}^{k}$（可以铺一张 $k$ 维坐标纸）
- **全局**：$\mathcal{M}$ 可能有弯曲、扭转、洞，但局部总是"平的"

**例子**：
- 地球表面：2维流形嵌入在3维空间里
- 所有"3"的手写图片：可能是一个10维流形嵌入在784维像素空间里

### 2.3 Grassmann流形：子空间的"空间"

#### 从PCA说起

主成分分析（PCA）把数据投影到最重要的 $k$ 个方向上。这 $k$ 个方向构成一个**子空间**。

**问题**：如果有两个域的数据，各自做PCA得到两个 $k$ 维子空间，怎么比较这两个子空间？

**不能直接用欧氏距离**：子空间是一组基向量，基的选取不唯一（可以旋转）。需要比较的是**子空间本身**，而不是某一套特定的基。

#### Stiefel流形与Grassmann流形

**Stiefel流形** $V_{k}(\mathbb{R}^{d})$：$\mathbb{R}^{d}$ 中所有**标准正交** $k$-标架的集合。

**人话**：所有可能的"$k$ 个互相垂直的单位向量组"的集合。每个点是一个 $d \times k$ 的矩阵 $U$，满足 $U^{T}U = I_{k}$（列正交）。

**Grassmann流形** $G_{k}(\mathbb{R}^{d})$：$\mathbb{R}^{d}$ 中所有**$k$ 维线性子空间**的集合。

**人话**：Stiefel流形上，不同的基可以描述同一个子空间（比如旋转一下基，子空间不变）。Grassmann流形把这些"描述同一个子空间的基"**捏成同一个点**。

**关系**：

$$G_{k}\left( \mathbb{R}^{d} \right) = V_{k}\left( \mathbb{R}^{d} \right)/O(k)$$

Grassmann = Stiefel 模掉正交群 $O(k)$（所有 $k$ 维旋转的集合）。

**维度**：

$$\dim\left( G_{k}\left( \mathbb{R}^{d} \right) \right) = k(d - k)$$

描述一个 $k$ 维子空间在 $d$ 维空间中的"姿态"，需要 $k(d - k)$ 个自由度。

**例子**：
- $d = 3, k = 1$（3维空间中的直线）：$\dim = 1 \times 2 = 2$。确实，3D空间中过原点的直线由球面上的一个点表示（2维）。
- $d = 3, k = 2$（3维空间中的平面）：$\dim = 2 \times 1 = 2$。过原点的平面也是2维（法向量方向）。

### 2.4 主角度（Principal Angles）：两个子空间有多"对齐"？

设两个子空间 $\mathcal{S}_{1}, \mathcal{S}_{2} \in G_{k}(\mathbb{R}^{d})$，各自由正交基 $U_{1}, U_{2}$ 表示。

**主角度** $\theta_{1} \leq \theta_{2} \leq ... \leq \theta_{k}$ 递归定义：

$$\cos\theta_{i} = \max_{u \in \mathcal{S}_{1}, v \in \mathcal{S}_{2}}u^{T}v$$

约束：$u, v$ 单位长度，且与之前所有对正交。

**人话**：找两个子空间中"最像"的方向，夹角为 $\theta_{1}$；然后在剩下的正交方向中找"第二像"的，夹角为 $\theta_{2}$；以此类推。

**计算**：

$$\cos\theta_{i} = \sigma_{i}\left( U_{1}^{T}U_{2} \right)$$

其中 $\sigma_{i}(\cdot)$ 是第 $i$ 个奇异值。

**例子**：
- 两个子空间完全重合：所有 $\theta_{i} = 0$
- 两个子空间完全正交：所有 $\theta_{i} = \pi/2$
- 一般情形：$k$ 个角度在 $0$ 到 $\pi/2$ 之间

### 2.5 Grassmann上的测地线：子空间之间的"最短路径"

在欧氏空间里，两点之间最短路径是直线。在流形上，最短路径是**测地线（Geodesic）**。

**Grassmann测地线**：

设 $U_{1}, U_{2}$ 是两个子空间的正交基表示。先做SVD：

$$U_{1}^{T}U_{2} = A\cos\Theta B^{T}$$

其中 $\Theta = \text{diag}(\theta_{1},...,\theta_{k})$ 是主角度。

从 $U_{1}$ 到 $U_{2}$ 的测地线（参数 $t \in [0,1]$）：

$$U(t) = U_{1}A\cos(t\Theta)A^{T} + \left( I - U_{1}U_{1}^{T} \right)U_{2}B\left( \sin(t\Theta)/\sin\Theta \right)A^{T}$$

**直觉**：
- 在已经对齐的方向上（$\theta_{i}$ 小），直接按比例插值
- 在未对齐的方向上（$\theta_{i}$ 大），沿着"球面大圆"的路径旋转过去

**Grassmann距离**（测地线长度）：

$$d_{G}\left( \mathcal{S}_{1},\mathcal{S}_{2} \right) = \sqrt{\sum_{i = 1}^{k}\theta_{i}^{2}}$$

把 $k$ 个主角度的平方和开根号。完全重合=0，完全正交=$\sqrt{k(\pi/2)^{2}}$。

### 2.6 与跨域自适应的联系：子空间对齐

**核心思想**：如果源域和目标域的数据各自蜷缩在各自的子空间上，跨域自适应就是**把源域子空间旋转/变形到目标域子空间**。

**具体方法**：
1. **子空间插值**：在Grassmann测地线上找一个中间子空间，作为域无关的公共表示
2. **主角度对齐**：最小化主角度和，即让两个子空间尽可能对齐
3. **指数映射**：在Grassmann流形的切空间上做线性操作，再通过指数映射回到流形（类似于"在地球表面做导航，先在平面地图上规划，再投影回球面"）

**与Bures-Wasserstein的区别**：

| 工具 | 描述对象 | 几何结构 | 对齐目标 |
|------|----------|----------|----------|
| Bures-Wasserstein | 高斯分布（均值+协方差） | 概率分布空间 | 均值平移 + 协方差变形 |
| Grassmann | $k$ 维子空间 | 子空间集合 | 旋转基向量使子空间重合 |

**联系**：PCA子空间由协方差矩阵的前 $k$ 个特征向量张成。如果我们用高斯近似分布，Bures-Wasserstein对齐协方差，Grassmann对齐协方差的特征子空间——两者是**互补**的：一个对齐"形状大小"，一个对齐"方向"。

### 2.7 流形上的最优传输

如果两个域的数据分布 $P_{S}, P_{T}$ 都支持在某个流形 $\mathcal{M}$ 上，最优传输应该在**流形内部**进行，不能"穿过"流形外的虚空。

**数学形式**：

$$W_{\mathcal{M}}\left( P_{S},P_{T} \right) = \inf_{\gamma \in \Pi(P_{S},P_{T})}\int_{\mathcal{M} \times \mathcal{M}}d_{\mathcal{M}}(x,y)\, d\gamma(x,y)$$

其中 $d_{\mathcal{M}}(x,y)$ 是流形上的**测地距离**（沿流形表面走的最短路径），而非欧氏距离 $\parallel x - y \parallel$。

**计算困难**：
- 流形未知（需要从数据估计）
- 测地距离需要解微分方程
- Sinkhorn算法需要知道流形的体积测度

**近似方法**：
1. **局部线性嵌入（LLE）**：在数据点周围用切平面近似流形，局部用欧氏距离
2. **图距离**：把数据点连成k近邻图，最短路径作为测地距离近似
3. **黎曼Sinkhorn**：在已知流形结构时，把熵正则化最优传输推广到黎曼流形上

---

## 三、元学习与域自适应：MAML的数学

### 3.1 从"刷题"到"学会考试"

传统机器学习：给你10000道北京奶茶店的题，你狂刷，形成一套解题套路。

元学习（Meta-Learning）：给你北京、上海、成都、西安各1000道题。你不是死记每座城市的答案，而是学一种"快速适应新城市口味"的**学习方法**。

**测试时**：给你广州10道题，你只需要做几道就能摸清广州规律。

### 3.2 MAML的设定：Task的分布

MAML（Model-Agnostic Meta-Learning, Finn et al., 2017）假设：
- 有一个**Task的分布** $p(\mathcal{T})$
- 每个Task $\mathcal{T}_{i}$ 有自己的训练集 $\mathcal{D}_{i}^{\text{train}}$ 和测试集 $\mathcal{D}_{i}^{\text{test}}$
- 在域自适应语境下：每个"Task"就是一个**域**（北京域、上海域...）

**目标**：学一个**初始化参数** $\theta$，使得对于从 $p(\mathcal{T})$ 采样的新Task，只需**一步或几步梯度更新**，就能在该Task上表现好。

### 3.3 Bi-Level优化：外层与内层

#### 内层（Task-level Adaptation）

对于每个Task $\mathcal{T}_{i}$，从初始化 $\theta$ 出发，走 $K$ 步梯度下降：

$$\theta_{i}' = \theta - \alpha\nabla_{\theta}\mathcal{L}_{\mathcal{T}_{i}}(\theta)$$

**一步版本（K=1）**：

$$\theta_{i}' = \theta - \alpha\nabla_{\theta}\mathcal{L}_{\mathcal{T}_{i}^{\text{train}}}(\theta)$$

#### 外层（Meta-level Optimization）

$$\min_{\theta}\sum_{\mathcal{T}_{i} \sim p(\mathcal{T})}^{}\mathcal{L}_{\mathcal{T}_{i}^{\text{test}}}\left( \theta_{i}' \right) = \sum_{\mathcal{T}_{i}}^{}\mathcal{L}_{\mathcal{T}_{i}^{\text{test}}}\left( \theta - \alpha\nabla_{\theta}\mathcal{L}_{\mathcal{T}_{i}^{\text{train}}}(\theta) \right)$$

| 符号 | 含义 | 人话 |
|------|------|------|
| $\theta$ | 元参数（初始化） | 所有Task共享的起点 |
| $\alpha$ | 内层学习率 | Task内部微调的步伐 |
| $\theta_{i}'$ | Task $i$ 的适配参数 | 从起点出发，走了几步后的位置 |
| $\mathcal{L}_{\mathcal{T}_{i}^{\text{test}}}(\theta_{i}')$ | Task $i$ 测试集上的损失 | 用这个Task专属参数，在没见过的数据上考几分 |
| $\sum_{\mathcal{T}_{i}}$ | 对所有采样Task求和 | 平均来看，这个起点好不好 |

**核心直觉**：外层优化不是在找"某个Task上最好的参数"，而是在找"最容易微调到任意Task上最好参数的参数"。

### 3.4 二阶导数的"噩梦"与FOMAML

外层损失对 $\theta$ 求导：

$$\nabla_{\theta}\mathcal{L}_{\mathcal{T}_{i}^{\text{test}}}\left( \theta_{i}' \right) = \nabla_{\theta_{i}'}\mathcal{L}_{\mathcal{T}_{i}^{\text{test}}}\left( \theta_{i}' \right) \cdot \frac{d\theta_{i}'}{d\theta}$$

而：

$$\frac{d\theta_{i}'}{d\theta} = I - \alpha\nabla_{\theta}^{2}\mathcal{L}_{\mathcal{T}_{i}^{\text{train}}}(\theta)$$

**人话**：外层梯度不仅涉及**一阶导数**（梯度），还涉及**二阶导数**（Hessian矩阵）！

Hessian矩阵：损失函数对每个参数对的二阶偏导，大小 $d \times d$。对于神经网络，$d$ 可能是百万级，Hessian计算和存储都是灾难。

#### FOMAML（First-Order MAML）的近似

忽略二阶项，直接令 $\frac{d\theta_{i}'}{d\theta} \approx I$：

$$\nabla_{\theta}^{\text{FOMAML}} \approx \nabla_{\theta_{i}'}\mathcal{L}_{\mathcal{T}_{i}^{\text{test}}}\left( \theta_{i}' \right)$$

**人话**：假装内层梯度更新没有改变参数的"依赖结构"，直接把Task测试集上的梯度当作对元参数 $\theta$ 的梯度。

**效果**：近似效果往往不错，计算量从 $O(d^{2})$ 降到 $O(d)$。

### 3.5 MAML与跨域自适应的血缘关系

**直接联系**：
- 把每个源域看作一个Task
- MAML学到的初始化 $\theta$ 是一个"域无关"的起点
- 遇到目标域（新Task），只需几步微调就能适应

**与PAC-Bayes的联系**：
- MAML的初始化 $\theta$ 就是PAC-Bayes中的**先验中心**
- MAML的微调过程 $\theta \rightarrow \theta'$ 对应PAC-Bayes中的**后验更新**
- MAML效果好 = 后验离先验近 = PAC-Bayes界紧

**与信息瓶颈的联系**：
- MAML要求初始化能压缩"跨域共性"，同时保留"快速适应的个性"
- 这相当于在信息平面上找到一个"靠近原点但纵坐标高"的点

### 3.6 Meta-Regularization：防止元过拟合

MAML的风险：如果Task分布 $p(\mathcal{T})$ 的多样性不够，学到的初始化可能只记住见过的Task类型，对新Task无效。

**数学表现**：外层训练损失很低，但新Task上微调后测试损失很高。

**解决方案**：
1. **任务增强**：对已有Task做数据增强（加噪声、旋转、遮挡），扩充Task分布
2. **元正则化**：在外层目标中加正则项，如 $\parallel \theta \parallel^{2}$ 或 $D_{KL}(Q \parallel P)$
3. **贝叶斯MAML（BMAML）**：把 $\theta$ 视为随机变量，学一个分布而非点估计，用集成方法减少方差

---

## 四、度量学习：从"近朱者赤"到分布距离

### 4.1 从一个社交圈子的比喻开始

想象一个大学社团：
- **正样本**：同社团的成员，应该坐在一起
- **负样本**：其他社团的人，应该坐得远
- **度量学习**：学一个"座位安排规则"，让同社团的人距离近，不同社团的人距离远

在跨域自适应中：
- 源域和目标域的**同类样本**应该拉近（正样本对）
- **不同类样本**应该推远（负样本对）
- 即使来自不同域，同类样本也要近

### 4.2 对比损失（Contrastive Loss）

**设定**：
- 锚点（Anchor）$x_{a}$：当前样本
- 正样本（Positive）$x_{p}$：与 $x_{a}$ 同类的样本（可能来自不同域）
- 负样本（Negative）$x_{n}$：与 $x_{a}$ 不同类的样本

**损失函数**：

$$\mathcal{L}_{\text{contrastive}} = \mathbb{E}\left\lbrack \parallel f(x_{a}) - f(x_{p}) \parallel^{2} - \parallel f(x_{a}) - f(x_{n}) \parallel^{2} + \text{margin} \right\rbrack_{+}$$

其中 $[z]_{+} = \max(0,z)$ 是hinge函数，$f$ 是特征提取器。

| 项 | 含义 | 人话 |
|----|------|------|
| $\parallel f(x_{a}) - f(x_{p}) \parallel^{2}$ | 同类样本在特征空间的距离 | 同社团的人坐多近 |
| $\parallel f(x_{a}) - f(x_{n}) \parallel^{2}$ | 异类样本在特征空间的距离 | 不同社团的人坐多远 |
| margin | 安全边界 | 至少要保持的"社交距离" |
| $[\cdot]_{+}$ | 只惩罚违反规则的情况 | 如果已经坐得对，不罚；如果坐错了，罚差多少 |

**工作原理**：
- 如果 $\parallel f(x_{a}) - f(x_{p}) \parallel^{2} + \text{margin} < \parallel f(x_{a}) - f(x_{n}) \parallel^{2}$：正样本比负样本近得多，损失为0（满意）
- 如果 $\parallel f(x_{a}) - f(x_{p}) \parallel^{2} + \text{margin} > \parallel f(x_{a}) - f(x_{n}) \parallel^{2}$：正样本离得太远或负样本离得太近，损失为正（惩罚）

### 4.3 InfoNCE：对比学习的概率版本

InfoNCE（Noise Contrastive Estimation）：

$$\mathcal{L}_{\text{InfoNCE}} = - \mathbb{E}\left\lbrack \log\frac{\exp\left( f(x_{a})^{T}f(x_{p})/\tau \right)}{\sum_{i}^{}\exp\left( f(x_{a})^{T}f(x_{i})/\tau \right)} \right\rbrack$$

其中 $\tau$ 是温度参数，分母遍历一个batch中所有样本（包括多个负样本）。

| 符号 | 含义 | 人话 |
|------|------|------|
| $f(x_{a})^{T}f(x_{p})$ | 锚点与正样本的特征内积 | 相似度（越大越像） |
| $\tau$ | 温度 | 控制分布的"尖锐程度"：$\tau$ 小，区别放大；$\tau$ 大，区别模糊 |
| 分子 | 正样本的未归一化概率 | 同社团的"亲密度" |
| 分母 | 所有样本的未归一化概率和 | 与所有人的"亲密度"总和 |
| 整体 | 负对数似然 | 让正样本的相对概率最大化 |

**与softmax的关系**：InfoNCE本质上是一个**以锚点为query、正样本为正确key、其他样本为错误key的softmax交叉熵**。

### 4.4 三元组损失（Triplet Loss）

$$\mathcal{L}_{\text{triplet}} = \sum_{(x_{a},x_{p},x_{n})}^{}\left\lbrack \parallel f(x_{a}) - f(x_{p}) \parallel^{2} - \parallel f(x_{a}) - f(x_{n}) \parallel^{2} + \text{margin} \right\rbrack_{+}$$

**与对比损失的区别**：
- 对比损失通常一次只对比一个正样本和一个负样本（或对称版本）
- 三元组损失显式构造三元组 $(a,p,n)$，更精细

**困难：采样策略**
- **Easy Triplets**：已经满足margin，损失为0。对训练无贡献。
- **Hard Triplets**：负样本比正样本还近（违反规则最严重），损失最大，信息量最大。
- **Semi-hard Triplets**：负样本比正样本远，但没有满足margin（差一点点），损失为正但不大。

实际中通常用Semi-hard采样（在线挖掘batch中的semi-hard三元组），因为Hard triplets可能导致训练不稳定（梯度爆炸）。

### 4.5 度量学习与分布距离的深层联系

**核心发现**：对比损失和InfoNCE实际上在**估计互信息的下界**。

**InfoMax原则**：

$$I(X;T) \geq \mathbb{E}_{p(x,t)}\left\lbrack \log Q(T|X) \right\rbrack - \mathbb{E}_{p(t)}\left\lbrack \log Q(T) \right\rbrack$$

InfoNCE损失是这个下界的一个**具体实现**。

**与MMD的联系**：
- MMD：显式对齐两个分布的均值嵌入
- 度量学习：通过样本对距离，隐式塑造特征空间的局部几何
- **联系**：如果正样本对包含所有源域-目标域的同类配对，度量学习实际上在**局部对齐条件分布** $P(X|Y = y)$

**与Wasserstein的联系**：
- Wasserstein：全局最优传输计划
- 度量学习：局部配对约束
- **联系**：度量学习可以看作Wasserstein的**贪婪近似**——每次只考虑一个锚点的最近邻传输，而非全局优化

### 4.6 在跨域自适应中的应用：DANN + 度量学习

**DANN**（Domain-Adversarial Neural Network）+ **度量损失**的组合：

特征提取器 $f$ 的输出进入三个分支：
1. 标签分类器（预测 $Y$）
2. 域判别器（对抗训练，让域不可分）
3. 度量学习头（拉近跨域同类样本，推远跨域异类样本）

总损失：

$$\mathcal{L} = \mathcal{L}_{\text{cls}} + \lambda\mathcal{L}_{\text{adv}} + \mu\mathcal{L}_{\text{metric}}$$

不仅要"骗过域判别器"（让域不可分），还要"主动组织特征空间"（让同类聚类）。两者互补：对抗损失处理全局分布，度量损失处理局部结构。

---

## 五、核心困惑

### 困惑 1：信息瓶颈的 $\beta$ 到底怎么选？

$$\mathcal{L} = - I(T;Y) + \beta I(T;X)$$

$\beta$ 控制压缩力度。在跨域自适应中，还需要一个 $\lambda I(T;\text{Domain})$ 项。

**问题**：三个超参数（$\beta, \lambda$，还有网络深度/宽度）如何协调？有没有**无监督**的方式确定"保留多少、丢弃多少"？

**思考**：这可能与**率失真理论（Rate-Distortion Theory）**有关——给定信道容量（网络瓶颈大小），最优编码自动决定压缩率。变分自编码器（VAE）中的 $\beta$-VAE 通过遍历 $\beta$ 发现"解耦"（Disentanglement）的相变点。跨域自适应中，可能存在类似的相变：当 $\lambda$ 超过某个阈值，域身份信息被突然压缩掉。

### 困惑 2：Grassmann流形上的优化怎么算？

神经网络参数优化通常用SGD，参数在欧氏空间 $\mathbb{R}^{d}$ 里自由移动。但如果某些参数必须生活在Grassmann流形上（比如子空间基），简单的梯度下降会破坏约束（正交性）。

**问题**：Grassmann上的**投影梯度下降**（Projected Gradient Descent）或**指数映射**（Exponential Map）怎么实现？计算代价是否比欧氏空间大很多？

**思考**：实际中，人们通常用**QR分解**或**SVD**在每一步后重新正交化基向量，而不是严格地在流形上做黎曼优化。这相当于"先随便走，再拉回流形"。这种近似是否收敛？收敛速度如何？对于大规模网络（如Transformer），Grassmann约束的层是否可行？

### 困惑 3：MAML的收敛性黑洞

MAML的bi-level优化中，内层优化通常只做 $K = 1$ 或 $K = 5$ 步梯度下降。对于深度网络，这几步远不收敛到Task最优解。

**问题**：如果内层没有收敛，外层梯度是否还有意义？MAML的"学习初始化"是否真的在优化"几步后到达的局部极小值"，还是在优化某种"轨迹中点"？

**思考**：有理论工作证明，在凸情形下MAML收敛。但神经网络非凸，内层轨迹复杂。FOMAML忽略二阶项，实际上是在优化"一步梯度后的线性近似"。这解释了为什么MAML对网络结构敏感（如需要careful的batch normalization初始化）。

### 困惑 4：度量学习的采样偏差与域偏移

度量学习依赖"正样本对"和"负样本对"的构造。在跨域场景中：
- 正样本对：源域的"猫"和目标域的"猫"
- 负样本对：源域的"猫"和目标域的"狗"

**问题**：如果目标域样本很少（比如每个类只有1个），负样本对几乎全是"源域-源域"或"目标域-目标域"，缺乏"跨域-跨类"的负样本。度量学习是否会退化成一个"单域聚类算法"，失去跨域对齐能力？

**思考**：这被称为**跨域采样不平衡**。缓解方法包括：记忆库（Memory Bank）存储历史特征、动量编码器（Momentum Encoder）增加负样本多样性、以及**混合样本**（Mixup）构造虚拟跨域负样本。但这些方法的数学保证都很弱。

---

## 六、一句话总结

跨域自适应的终极形态，是**在信息论的指导下**（该删什么），**在流形几何的约束下**（该沿什么路径搬运），**在元学习的框架中**（该如何快速适应），**在度量学习的局部结构里**（该如何组织特征空间），同时回答四个维度的问题——而这四个维度的交集，可能正是"智能"本身的一个切面。
