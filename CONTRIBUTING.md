# 协作约定
- 分支名:`<名字>/<wp编号>-<简述>`,如 `kira/wp1-abo-items`。通过 PR 合并到 `main`,至少一人 review。
- 不提交原始数据、图片、模型权重(见 `.gitignore`)。小的统计表(CSV)可以提交到 `docs/` 或 `scores/`。
- 每个模型只提交一个分数文件到 `scores/`:`scores/<模型名>_<配置>_seed<k>.csv`,两列 `pair_id,score`,分数越高越匹配。
- 阈值只在 val 上选,test 上不调。随机种子和主要超参写进文件名或同目录的说明里。
- 字段名以 `docs/schema.md` 为准,WP0 冻结后不再改;要改先在群里说。
- 先交「小版本」(一个类别、几千对)让别人能跑通,再交全量。
