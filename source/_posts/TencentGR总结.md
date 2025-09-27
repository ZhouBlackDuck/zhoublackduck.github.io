---
title: TencentGR总结
date: 2025-09-15
categories: 比赛
tags:
  - 生成式推荐
  - 推荐算法
---

## 数据格式

数据为用户与物品的交互序列，包含两种格式

1. userprofile：包含用户id，用户特征（均为离散特征）字典，时间戳，其余列为`None`
2. interaction：包含用户id，物品id，物品特征（均为离散特征）字典，交互类型（`0`: 曝光，`1`: 点击），时间戳，其余列为`None`

每个用户的交互序列中，有且仅有一个userprofile，其余为interaction；interaction中，曝光类型数量远大于点击类型；预测数据中存在冷启动数据

另外，部分物品有多模态特征（emb形式），维数分别为32，1024，3584，4096，3584，3584

用户序列数据总量在一百万左右，候选物品在五百万左右

## 数据处理

根据用户序列获取

1. id序列（user_id开头，交互的倒数第二个item_id结束）
2. 正样本序列（下一个序列项是物品，则记录其id作为正样本）
3. 负样本序列（从所有具有物品特征的物品中随机采样，采样不包含序列中的物品）
4. token类型序列（区分userprofile和interaction）
5. next_token类型序列
6. next_action类型序列（下一个token是interaction时的交互类型）
7. 特征序列（每个token的特征）
8. 正样本特征序列
9. 负样本特征序列

所有序列缺失值使用默认值填充，并左填充到统一长度

## Baseline Model

### 组件

1. 所有候选item的emb词表
2. 所有user的emb词表
3. 序列位置编码词表（从-len到len）
4. dropout层
5. 离散特征词表字典（ModuleDict）
6. 多模态特征变换层（ModuleDict）
7. transformer层
8. user特征dnn
9. item特征dnn

### 流程

1. 输入序列特征经过user特征dnn和item特征dnn处理进行融合（使用token类型进行区分）
2. 加入位置编码
3. 对序列掩码输入transformer得到序列表征
4. 同样对正样本序列和负样本序列进行编码
5. 使用next_token类型对损失计算进行掩码，计算bce损失

## 训练

9:1随机分割进行训练

## 改进

## 成绩

- 128/2800+
- 历史最高42

## 可能的改进

