# 小版本(家具,WP0–WP2)

给 Jack / Aaron / Jane 先跑通自己脚本用的小版本数据。类别选的是**家具**(product_type 白名单见
`src/data/build_items.py` 里的 `FURNITURE_TYPES`,排除了 `HOME_FURNITURE_AND_DECOR`、
`HOME_BED_AND_BATH`、`FURNITURE_COVER` 这几个混了非家具商品的大类,也过滤掉了布料色卡
"xxx Swatch" 这种误标成家具的条目)。

## 数据规模
- 3978 件商品(items.csv),3173 个近重复分组(group_id)
- train/val/test = 3189 / 393 / 396(按 group_id 划分,同组不跨 split)
- pairs.csv:3978 pos + 3978 N1(随机负例) + 3978 N2(同类难负例),共 11934 行

## 怎么复现
```
python src/data/count_categories.py --listings-dir data/raw/listings/metadata \
    --out-csv docs/category_counts.csv --out-md docs/category_counts.md

python src/data/build_items.py --listings-dir data/raw/listings/metadata \
    --images-csv data/raw/images/metadata/images.csv.gz \
    --images-out data/images/small \
    --out-csv data/processed/items.csv --out-counts-csv data/processed/items_counts.csv

python src/negatives/build_pairs.py --items-csv data/processed/items.csv \
    --out-csv data/processed/pairs.csv --spotcheck-csv docs/n1_n2_spotcheck_sample.csv
```
ABO listings/images 元数据从 `https://amazon-berkeley-objects.s3.amazonaws.com/` 按需下载(不是整个
tar,按分片/按需单张图拉取,几分钟内能跑完)。图片本身不进 git,`docs/mini_version/items.csv` 和
`pairs.csv` 是给大家对齐字段用的,要跑模型还是要自己跑一遍上面的脚本把图下下来。

## 做了哪些决定
1. **类别 = 家具**,对应 open-decisions.md 选项 B(只用家具)。还没做 OOD(家居装饰/服饰鞋靴),
   那是 Sean 的 WP10。
2. **N2 相似度**用 `sentence-transformers/all-MiniLM-L6-v2`(非 CLIP),同 product_type 内、排除同
   group_id,取 cosine 相似度最高的一个。
3. **N1/N2 比例**按 pos:N1:N2 = 1:1:1 处理(每条标题各出一次 pos、一次 N1、一次 N2),对应
   open-decisions.md 选项 3a 的思路,但那条是 N1:N2:N3 的比例,我这边先按这个套用,N3 出来后
   这个假设需要 Sean/组里确认一下。
4. **近重复分组**:共享 main_image_id 或者去掉自己的 color/material 词 + 常见尺寸词后标题相同,
   就归一组。发现 3 张 ABO 的占位图(没有真实图,各型号共用同一张"暂无图片"图)被几十个不相关
   商品共用,已经把"图片共享"这条规则限制在最多共享 4 次以内,超过的当占位图处理,不用来合并分组。
5. **抽检**:`docs/n1_n2_spotcheck_sample.csv` 放了 N1/N2 各 20 条(共 40,比文档建议的 50 对少一点,
   先凑够能跑通;真要定假负例比例还是需要大家手动过一遍填 `actually_a_match` 列)。我自己随机看了
   几张图,N1/N2 的负例构造逻辑是对的,但没有替代组内的正式人工审查。

## 已知局限 / 没做的事
- 只处理了 en_US 标题;同一商品的其它语言版本没有用上。
- WP1 文档要求的"颜色/材质标准化值出现在标题里"计数表做了,但只在全量 `docs/category_counts.csv`
  里(不分品类),家具专属的这张还没单独拆出来,需要的话我可以补。
- N2 候选目前允许跨「大类里不同小类」(比如 CHAIR 里躺椅和办公椅会互相当负例),没有按三级 node
  路径再细分,如果组里觉得太粗可以再收紧。
