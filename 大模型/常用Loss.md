#### Point-wise

- **Cross Entropy Loss（交叉熵）**
  为什么要用 log 函数？放大对错误低概率的惩罚

  - **CE Loss**

    $CE = -\sum_{i=1}^{C} y_i \cdot \log\left(p_i\right)​$

    ```python
     # 模型原始输出
    logits = np.array([1.0, 2.0, 3.0]) 
    # softmax
    p = np.exp(logits) / np.sum(np.exp(logits))  
    # ce_loss
    ce = -np.log(p_true)  
    ```

  - **BCE Loss（二分类 / 多标签）**

    正样本时-log(pi)，负样本是-log(1-pi)

    $BCE = - \left[ y \cdot \log(p) + (1 - y) \cdot \log(1 - p) \right]$

    $BCE = -\frac{1}{C} \sum_{i=1}^{C} \left[ y_i \cdot \log(p_i) + (1 - y_i) \cdot \log(1 - p_i) \right]$

    ```python
    def bce_loss(logits, y_true):
        # sigmoid
        p = 1 / (1 + np.exp(-logits))
        # BCE Loss
        res = -np.mean(y_true * np.log(p) + (1 - y_true) * np.log(1 - p))
        return res
    ```

  - **Balanced CE**

    - 背景：即使每个负样本的损失（Loss）都很小，但由于数量巨大，总损失 依然会远大于正样本的总损失 在梯度下降时，模型为了快速降低总损失，会倾向于优先优化负样本，导致模型最终“偷懒”把所有样本都预测为负类。
    - 为什么负样本损失小？因为预测为正样本的概率pi很低，导致log(1-pi)趋于0，负号反转以后梯度很小
    - 方法：通过一个缩放因子a（通常为0.75），正样本为a，负样本为1-a。就是纠偏

- **MSE（均方误差）**
  一个batch_size内，真实值与预测值差的平方的均值

  $\text{MSE} = \frac{1}{N} \sum_{i=1}^N (y_i - \hat{y}_i)^2​$

  ```python
  def mse(logits, y_true):
      # 先算差，再平方
      target = (logits - y_true) * (logits - y_true)
      # 求批量均值
      res = np.mean(target)
      return res
  ```

- **Focal Loss**

  通过对困难/简单样本 进行不同比例的缩放，指数级缩放简单样本的Loss

  - 提出背景：训练数据中部分类别（如罕见意图、小众实体）样本极少，普通交叉熵（CE）会被多数类样本 “淹没”，模型学不会少数类；模型天然倾向于 “挑软柿子捏”—— 优先学好分的易样本（预测概率高），忽略难样本（预测概率低），导致长尾场景效果差。
  - 单样本：$FL(p_i) = -\alpha_i (1 - p_i)^\gamma \log(p_i)$
    批量格式：$FL_{batch} = \frac{1}{N} \sum_{k=1}^N -\alpha_{t_k} (1 - p_{t_k})^\gamma \log(p_{t_k})​$


#### Pair-wise

- **BPR Loss**
  BPR 是 **Pairwise 排序损失**，只关心**相对顺序**，不关心绝对得分。
  训练样本为三元组（user，true_sample，false_sample）

  - 单样本格式：$L_{bpr}(u,i,j) = -\log\left(\sigma\left(\hat{s}_{ui} - \hat{s}_{uj}\right)\right)$

    批量格式：$L_{BPR} = -\frac{1}{|\mathcal{D}|}\sum_{(u,i,j)\in\mathcal{D}} \log\left(\sigma\left(\hat{s}_{ui} - \hat{s}_{uj}\right)\right) + \lambda \cdot \|\Theta\|^2$
    (1)  Si是样本经过 DNN、MF、双塔等模型输出的预测得分，模型学习目标是拉大正负样本的分差。
    (2)  经过 sigmoid 函数，将分差映射到 (0,1)，代表 正样本排在负样本前面的概率。
    (3)  对概率取 负对数：概率越接近 1，Loss 越接近 0；概率越小，Loss 急剧上升。

  - 应用场景：隐式反馈 比如收藏/点击/加购等等行为，不直接打分但是可以通过行为反馈，作为正负样本然后丢给模型取打分。（双塔：正负样本应该分别经过物料塔，user序列经过user塔，然后内积打分。模型的话就是要拉远正样本和负样本的得分来调整embs模型）

- **cosENT Loss**
  通过优化句子对的相对相似性排序，构建更具有判别性的语义空间

  - 训练样本：
    - sentence : sentence 句子对
    - label：1-5的评分，从大到小排序，分数越大越相关
  - 损失函数：$\mathcal{L} = \log\left(1 + \sum_{i,j} \exp\left(\frac{s_i - s_j}{T}\right) \cdot \mathbb{I}(y_i < y_j)\right)$
    排序约束：$\mathbb{I}(y_i < y_j) = \begin{cases} 1, & y_i < y_j \text{（需满足排序约束的样本对）} \\ 0, & y_i \geq y_j \text{（无需约束的样本对）} \end{cases}​$
    当真实标签满足yj > yi 时，要求模型的预测的相似性分数Sj > Si，当预测正确，约束函数 = 0不会产生损失，当si > sj时，约束函数 = 1，会产生损失，并且损失大小与分数插值呈指数级增长
    T为温度系数，实际作为缩放因子，T越大越平滑越鲁棒性，T越小损失越敏感

#### List-wise

- **InfoNCE Loss**

  口语化表述：InfoNCE 损失针对批次中以锚点为中心的正负样本对（单正样本 + 多负样本），通过最大化正样本与锚点的余弦相似度、最小化所有负样本与锚点的余弦相似度之和，以指数缩放 + 对数似然的形式构建损失函数，最终实现正样本与锚点特征拉近、负样本与锚点特征推远的对比学习目标

  - 损失函数：$\mathcal{L}_{\text{InfoNCE}} = -\log \frac{\exp(s(z, z^+)/\tau)}{\exp(s(z, z^+)/\tau) + \sum_{k=1}^{K} \exp(s(z, z_k^-)/\tau)}$
    其中γ为温度系数，等效成缩放因子。s(i,j)为i,j的相似度，通常取余弦相似度
    其中的exp，可以视作一个正负样本的 sampled softmax
  - 训练样本（一个批次）：anchor、正样本、负样本集
  - 具体应用：InfoNCE 正是为解决**样本语义相似度对齐**问题设计的核心损失，其核心目标就是通过优化余弦相似度，让语义相似的样本（正样本对）在特征空间中距离更近（余弦相似度更高），语义不相似的样本（负样本对）距离更远（余弦相似度更低），最终学到具有强判别性的语义特征表示

- **LambdaRank Loss** 