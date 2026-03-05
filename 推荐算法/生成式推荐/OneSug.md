## OneSug

![img](https://pic3.zhimg.com/v2-05b60bb763739227fadd4cd93be717f2_r.jpg)

#### 面试总结

首先通过bge全量微调，得到更加适配当前语义场景的，pre与query共同对齐的aligned bge。然后利用大量的query对rqvae模型进行预训练，训练码本与encoder层参数。然后再将词典中的query全部进行量化，并根据query id-sid的映射，根据sid建立倒排索引，能拿到sid下的query集合。接下来是推理阶段，通过pre，通过表层含义比如bge+cos相似度和后验点击querys，聚合他们的信息，然后拿到映射的sid，根据倒排索引取出语义相似的querys，然后根据这些querys输出llm，进行语义上的聚合，生成用户最想要的query



#### 具体步骤

#### （1）训练阶段

- **Aligned BGE**
  通过 数据清洗 或 借助大模型 构造 **prefix2query & query2query (pairwise)** 高质量数据集，对BGE-base进行全量微调，让 prefix 和 query 映射到同一embedding空间上。

- **RQ-VAE 量化**
  通过大量的query embedding，训练出codebook+encoder，使得rq-vae模型具备将embedding映射到码本空间，将query emb转成离散的 **sid + token embedding**

- **离散构建语义检索库**

  将现有词典query的全部量化，并根据 **SID : Query** 的映射，简历每个 SID 的 倒排索引

#### （2）推理阶段

- **向量增强**：
  用户输入搜索词prefix，通过 文本相关度 / 前缀匹配 / 后验 召回query，然后经过BGE得到query embedding进行max pooling，在对prefix embedding进行加权聚合进行增强
- **量化索引**
  将 prefix 增强后的 embedding，经过rq-vae量化得到sid，取得sid下语义相似的querys
- **LLM Query改写**
  将 prefix , querys 通过送入 LLM，通过编写Prompt提示词，让LLM生成最有可能的Query。这里可以通过高质量数据集，对大模型进行Lora微调 或 全量微调 进行效果优化