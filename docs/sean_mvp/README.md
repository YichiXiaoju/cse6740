# Sean 的 WP3 / WP10 最小版本

**状态：WP3 / WP10 第一版 MVP。代码与候选数据已生成，Sean 已完成 260 条人工初审；N2 部分标签待复核，数据尚不是最终 benchmark。**

基于 Kira PR #1 的 `bd3e1fad03e4931bbb1928ba0de5912ca7361fa5`，不修改她的生成流程。
本地分支以 Kira 分支为基础；若提交 PR，应以 Kira 分支为 base，避免把她的改动重复算作 Sean 的工作。
提交范围仅为 Sean 的 N3、OOD、审查工具及说明，不修改 Kira 原有 N1/N2 生成流程。

## 本次交付

| 内容 | 规模 / 状态 |
|---|---|
| 颜色 N3 候选 | 1,118 条：train 884 / val 98 / test 136；其中 100 条已初审 |
| WP3 审查样本 | 原 N2 100 条 + 新 N3 100 条，已完成初审 |
| 近 OOD 候选 | LAMP / LIGHT_FIXTURE，150 件商品 |
| 远 OOD 候选 | SHOES / SANDAL / BOOT，150 件商品 |
| OOD 配对 | 两档各 pos/N1/N2 150 条、N3 29 条，合计 958 对 |
| OOD 初步抽查 | 每档 N1/N2/N3 各 10 条，共 60 条，已完成初审 |

`n3_candidates.csv`、`ood_pairs_candidates.csv` 中的 label 是**待验证的预期标签**，不是人工确认结果。
近/远 OOD 类别是本次可运行的候选配置，尚未替代组内正式类别决定。若训练范围扩大，必须重跑。

## 已完成的人工初审与当前限制

原始初审判断见 `reviews/ID_review_completed.csv`、`reviews/OOD_review_completed.csv`；初审统计、来源哈希和待复核条目见同目录。空白 `review_sample.csv` 是生成时的原始抽样模板，不代表尚未完成初审。

| 数据 | 类型 | 初审数 | 标为有效负例 | 标为其实匹配 | 无法判断 |
|---|---|---:|---:|---:|---:|
| 家具 | N2 | 100 | 46 | 45 | 9 |
| 家具 | N3 | 100 | 100 | 0 | 0 |
| OOD | N1 | 20 | 15 | 5 | 0 |
| OOD | N2 | 20 | 10 | 9 | 1 |
| OOD | N3 | 20 | 19 | 1 | 0 |

家具 N2 的 45/91=49.5%、OOD N2 的 9/19=47.4% 是排除无法判断后的**初审“其实匹配”标记率**，不是已独立复核的真实假负例率。复看发现 Right/Left Facing、5/6-Drawer 两条初审标签需要复核，原始判断未被擅自覆盖。

`reviews/ID_N3_retained_initial_review.csv` 与 `reviews/OOD_N3_retained_initial_review.csv` 分别保留初审通过的 100 / 19 条 N3。其余 1,018 条 ID N3 和 38 条 OOD N3 仍未初审；不将抽样结果自动推广为全量通过。保留表只有负例，不能单独作为完整二分类评估集。

**N3 标签有效和没有文本捷径是两回事。** 当前目标词仅 red / blue / green / yellow，且复杂商品颜色表达覆盖不足；模型可能利用颜色词和措辞分布预测标签。尚未完成 text-only 对照，因此本版不能宣称无文本偏差。下一版计划优先从真实同类标题提取自然颜色表达，并与正例对齐词汇分布，之后重新抽检；这些改进尚未实现。

对上游家具 N2 的静态诊断还发现：3,978 条中有 1,342 条跨 split 取图、47 条使用原商品同图，69 条的 `(image_path, text)` 与某条正例完全相同却标为负例。不同计数可能重叠。本提交只报告这些发现，不把上游数据修复算作已完成。

## 如何继续审查或复核

1. 下载并在浏览器打开随交付附上的 `Sean_N2_N3_Review.html`。
2. 对照当前图片和标题，选择「有效负例 / 假负例 / 无法判断」。只有明确矛盾才是不匹配，标题漏说不算错。
3. 审查 N2 100 条、N3 100 条；注明主要原因。缺图或属性无法从图片辨认时选择无法判断。
4. 定期点「备份进度 JSON」，结束时点「导出审查 CSV」。页面的筛选不影响导出：始终导出整批记录。
5. 运行下面的 finalize 命令，只导出**逐条明确审核通过**的 N3；剩余候选仍未审核。
6. 另有 `Sean_OOD_Review.html` 可作候选类别的初步抽查。它不替代 WP3 的 100 + 100 条要求。

页面内嵌图片，无需 API key、服务器或联网。CSV 可继续人工填写；JSON 可在同一批页面中恢复。
浏览器对本地 HTML 的进度保存支持不同，备份 JSON / 导出 CSV 才是可靠的进度交接方式。

## N3 规则

- MVP **仅替换颜色**，不替换材质或形状。材质中可能有 `Stone & Beam` 相关污染，不宜直接替换。
- 使用完整原始 ABO color 字段复核：必须恰有一个 en_US 条目，最多一个标准化值；展示值和标准化值都要在词表中且属于同一色系。
- 标题必须恰有一个已知颜色跨度；使用单词边界与最长短语匹配，`Navy Blue` 一起替换。
- 标题颜色必须与元数据同色系。跳过多颜色跨度、two-tone、multicolor、floral、striped、swatch 等。
- 跳过在输入表中共享次数超过 4 次的图片，称为高频共享图，不假定它们必然都是占位图。
- 颜色词表、别名和排除表在 `src/negatives/build_n3.py`。Ivory/White、Navy/Blue 归入同色系；这些近义关系不互换。
- 新颜色从 red / blue / green / yellow 中确定性抽取，并排除同色和保守定义的邻近色。只改一个匹配跨度，保持图片、item_id、group_id 和 split。
- `n3_audit.csv` 保留原标题、改后标题、原颜色、新颜色和原始属性检查状态；`n3_skipped.csv` 记录逐条跳过原因。
- 即便满足规则，仍可能有多色漏检、颜色词属于型号/品牌、图片与元数据不符等情况，必须人工审查。

## OOD 构造与模型

- 使用 ABO 原始 listings 和图片元数据，保留英文标题及主图路径。
- 全局分组后才抽样：复用 Kira 标题归一化规则，同时合并共享图片、原 Kira group_id。排除与 Kira **全部**已有商品组相连的 OOD 候选。
- 排除高频共享图，每组抽一件，seed=42；每档最多 150 件。近/远档之间也不共享组。
- N1 仅在同一 OOD 档中挑不同组图片；N2 还要求同 product_type，取标题向量余弦相似度最高的不同组图片。
- N2 复用 `sentence-transformers/all-MiniLM-L6-v2` 的模型与规则。此实现用模型作者的全精度 ONNX 导出，CPU 推理、mask mean pooling、L2 归一化、256 token 截断；没有改用 CLIP 或 TF-IDF。
- 固定模型 revision：`1110a243fdf4706b3f48f1d95db1a4f5529b4d41`。后端浮点误差或相似度并列可能使极少数近邻与 SentenceTransformer 实现不同；需要逐条一致时，应统一后端。
- ONNX Runtime 在导入前设置 `ORT_DISABLE_TELEMETRY=1`，并关闭其 telemetry API。
- OOD 的 N3 使用与 ID 一样的颜色规则。N3 只在一部分商品上成立，不能强行凑齐数量。
- OOD 不参与训练或阈值选择，阈值来自 ID val。

## 复现

从仓库根目录执行，Python >= 3.10。首次模型下载约 90 MB，ABO 元数据约 88 MB；不下载整包图片。

```bash
python -m pip install -r requirements-sean.txt

python src/data/download_abo_metadata.py --out-dir data/raw

python src/negatives/build_n3.py \
  --items-csv docs/mini_version/items.csv \
  --listings-dir data/raw/listings/metadata \
  --out-dir data/processed/sean

python src/negatives/build_review.py \
  --items-csv docs/mini_version/items.csv \
  --pairs-csv docs/mini_version/pairs.csv data/processed/sean/n3_candidates.csv \
  --n3-audit data/processed/sean/n3_audit.csv \
  --out-dir data/processed/sean/review

python src/negatives/build_ood.py \
  --existing-items-csv docs/mini_version/items.csv \
  --listings-dir data/raw/listings/metadata \
  --images-csv data/raw/images/metadata/images.csv.gz \
  --model-revision 1110a243fdf4706b3f48f1d95db1a4f5529b4d41 \
  --out-dir data/processed/sean
```

生成的主审查页是 `data/processed/sean/review/review.html`。
换 OOD 类别可使用 `--near-types` / `--far-types`，与 Kira 输入表 product_type 重叠时脚本报错。
数据和图片的本地根目录仍遵守 Kira schema，image_path 相对 `data/images/`。

模型同学需要所有 OOD 图片时，执行：

```bash
python src/data/download_pair_images.py --csv docs/sean_mvp/ood_items.csv
```

此脚本按表中路径下载并校验图片；失败会输出 manifest 并返回非零退出码。含图审查页只下载抽中的样本。

### 导出审查结果后

假设浏览器导出的文件保存为 `data/processed/sean/sean_review_completed.csv`：

```bash
python src/negatives/finalize_review.py \
  --review-csv data/processed/sean/sean_review_completed.csv \
  --sample-csv docs/sean_mvp/review_sample.csv \
  --n3-candidates docs/sean_mvp/n3_candidates.csv \
  --out-dir data/processed/sean/reviewed
```

输出 `n3_approved.csv` 与 `review_summary.json`。假负例率 = `match / (match + mismatch)`；pending 与 uncertain 不计入这个分母，但单独报告数量和无法判断率。
工具保留的 `false_negative_rate` 字段是按传入判断计算的比率；对于本次尚未完成二次复核的 N2，报告时必须称为初审标记率。`n3_approved.csv` 也只表示按当前传入判断保留，并不代表独立复核或消除文本捷径。
**不能因为 100 条抽检通过，就把 1,118 条全部写成“人工审核通过”。**
若需审核更多候选，增大 `build_review.py --n-per-type`，并在 finalize 时提供对应那批 sample CSV。

### OOD 小批抽查页面

`build_review.py` 同样支持 `--items-csv .../ood_items.csv --pairs-csv .../ood_pairs_candidates.csv --n3-audit .../ood_n3_audit.csv`；
可用 `--types N1 N2 N3 --n-per-type 20` 抽 60 条。交付附带的 OOD 页面按 near/far 各抽 10 条每种类型，更便于检查两档。

## 验证和剩余事项

- `python -m unittest discover -s tests -v`：10 个测试，覆盖颜色过滤、保留 split、OOD 跨档采样、已有组隔离、待审数据不能冒充已审核、导出数据防串批。
- 实际数据检查和图片验证见 `validation.json`。
- 已完成 200 条 ID 与 60 条 OOD 初审。N2 争议标签待复核，原始人工记录保持不变。
- 尚未跑模型基线或 text-only 捷径检查，这是 Jack / Aaron / Jane 的后续对接工作。
- Kira 原 N1/N2 构造存在跨 split 候选问题，本次不改其已有配对；主审查仍抽原始 N2，避免改变审查对象。正式实验前需组内修复并重新对齐样本。
- 当前 pos/N1/N2/N3 数量不平衡，不能直接把多数类准确率当作模型提升。最终比例、类别范围和分负例类型评估口径仍需组内确认。
- 原始 metadata、图片、模型缓存以及含图片的 HTML 均不提交 git；仓库只保留代码、规则、派生候选表与小统计文件。

来源：ABO 官方公开数据（https://amazon-berkeley-objects.s3.amazonaws.com/）；遵循仓库约定，仅用于非商业课程项目。
