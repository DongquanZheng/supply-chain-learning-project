# 现有算法家族与候选强底座

状态：文献候选清单，不代表算法已经选定。服务器 Frozen V1、TabZilla 和 TabArena 不受影响。

## 1. 首选强底座：Relational Deep Learning / HGT

### 已有机制

Relational Deep Learning（ICML 2024）把多张关系表转换为带时间的异构图；RelBench（NeurIPS 2024）提供时间切分、异构图构造和关系预测基线。Heterogeneous Graph Transformer（WWW 2020）进一步让节点类型、边类型和相对时间直接决定消息变换。

这与供应链数据的自然结构一致：

- 国家、港口、新闻事件和周可以作为不同实体类型；
- WITS 贸易关系是有类型、有方向的边；
- GDELT 事件与国家、时间存在明确配对；
- 风险预测只允许读取预测时点之前的关系记录。

### 为什么优先

它直接使用真实关系，而不是把所有来源压成单一向量。关系语义被破坏时，可以保持图规模、参数量和训练预算不变，因此适合当前 capacity gate 与 semantic gate。

### 必须先验证

先运行原始 RDL/HGT，在相同开发切分上比较 TabM。只有原始关系模型稳定胜过 TabM，才把它认定为供应链强底座并进行公式修改。

## 2. 最合适的单公式改动

HGT 的核心是关系特定的消息矩阵。候选改动不是增加 gate 或 adapter，而是直接重参数化消息映射：

原关系消息：

`m_(u->v) = W_(tau(u), r, tau(v)) h_u`

候选关系条件低秩消息：

`m_(u->v) = U_(tau(v)) Diag(c_(r, delta_t)) V_(tau(u))^T h_u`

其中：

- `tau(u), tau(v)` 是发送端和接收端实体类型；
- `r` 是真实关系类型，如本地事件、贸易传播或运营观测；
- `delta_t` 是合法时间差；
- `U,V` 是跨关系共享的任务方向；
- `c_(r,delta_t)` 在低维坐标中改变真实关系的作用。

这条公式迁移了 V1 中“学习共享方向”和“乘法调制坐标”的有效部分，同时把调制变量从不明确的 sample prompt 改成可验证的真实关系。它仍然只是待检验候选。

匹配 null 使用完全相同的 `U,V,c` 参数量，只把 `r,delta_t` 换成预先固定的置换关系标识。这样性能差异不能由额外容量解释。

## 3. 必要对照：LMF

Low-rank Multimodal Fusion（ACL 2018）用模态特定因子分解高阶融合张量，在低成本下保留乘法跨模态交互。它与 V1 的低秩乘法直觉最接近，适合作为“把五个来源当五个模态”的融合基线。

局限：LMF 知道输入来自不同模态，但不天然知道国家、贸易边和时间对应是否正确。因此它不能单独解决语义可识别问题。

用途：作为低秩乘法融合强对照，不与 HGT 堆叠。

## 4. 冲突与可靠度对照：TMC

Trusted Multi-View Classification（ICLR 2021）为每个视图输出 Dirichlet evidence，并用 Dempster–Shafer 规则融合，显式建模视图不确定性和冲突。

局限：它主要回答“某个来源是否可信”，不表示贸易关系或文本—运营对应关系。动态证据融合还可能引入额外容量。

用途：作为 reliability asymmetry 和 source conflict 条件下的强对照，不作为默认核心结构。

## 5. 时间错位对照：MulT

Multimodal Transformer（ACL 2019）用方向性跨模态注意力处理未对齐序列和长距离跨模态依赖。

局限：当前33维开发面板已经按周聚合；已有实验也没有支持连续时间漂移解释。直接使用 MulT 可能增加大量不必要自由度。

用途：只在保留原始事件时间戳、运营记录时间戳以后，作为 temporal misalignment 对照。

## 6. 推荐顺序

1. 用新评测集构造原生的时间异构关系图。
2. 跑未修改的 RDL/HGT 与 TabM；这一步决定强底座是否成立。
3. 同时跑 LMF、TMC 作为融合与可靠度对照，但不混合结构。
4. 若 RDL/HGT 胜过 TabM，只替换关系消息矩阵为上面的低秩关系条件公式。
5. 固定比较原 HGT、修改版、参数匹配 null 和语义破坏版。
6. 若原 HGT 不能胜过 TabM，停止修改 HGT；转而检查 LMF 是否能成为强底座，而不是继续给 HGT 加模块。

## 7. 当前判断

- 现有研究已经分别处理真实关系、低秩乘法、来源可靠度和时间错位。
- 没有一个现成算法同时证明“收益来自真实供应链语义而不是容量”。
- 最有希望的主线是 RDL/HGT，因为它与国家—新闻—贸易网络—时间的关系结构最一致。
- V1 最适合迁移的是受限乘法坐标变换，不是 prompt、时间流或多轴结构本身。

## 8. 原始论文与实现

- Relational Deep Learning, ICML 2024: https://proceedings.mlr.press/v235/fey24a.html
- RelBench, NeurIPS 2024: https://proceedings.neurips.cc/paper_files/paper/2024/hash/25cd345233c65fac1fec0ce61d0f7836-Abstract-Datasets_and_Benchmarks_Track.html
- RelBench code: https://github.com/stanford-star/relbench
- Heterogeneous Graph Transformer: https://arxiv.org/abs/2003.01332
- HGT code: https://github.com/acbull/pyHGT
- Low-rank Multimodal Fusion, ACL 2018: https://aclanthology.org/P18-1209/
- Multimodal Transformer, ACL 2019: https://aclanthology.org/P19-1656/
- Trusted Multi-View Classification, ICLR 2021: https://openreview.net/pdf?id=OOsR8BzCnl5
- TMC code: https://github.com/Han-Zongbo/TMC

