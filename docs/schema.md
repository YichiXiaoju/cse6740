# 统一数据格式(草稿,WP0 冻结前以 Kira 的最终版为准)

## pairs 表字段
| 字段 | 含义 |
|---|---|
| pair_id | 每个图文对的唯一 id |
| item_id | 图所属商品的 id |
| image_path | 主图路径 |
| text | 英文标题(N3 为改写后的标题) |
| label | 1 = 匹配,0 = 不匹配 |
| neg_type | pos / N1 / N2 / N3 |
| category | 类别(product_type 或一级类目) |
| year | 年份(跨时间测试用) |
| split | train / val / test,以及 OOD、跨时间测试集的名字(待 Kira 定) |

## items 表(WP1 产出)
item_id, title, image_path, product_type, node, color, material, group_id, split

## 分数文件
`pair_id,score`,分数越高越匹配。

## 「匹配」的定义(建议,待组内确认)
标题里说的每一句图里都不矛盾即算匹配;标题漏说一部分(如黑背白垫的椅子只写「黑色椅子」)不算错。Luna 的 prompt 也按此写。
