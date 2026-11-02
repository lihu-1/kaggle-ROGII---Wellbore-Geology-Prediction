# ROGII Wellbore Geology Prediction 推理方案

> **当前状态：**本文介绍目前的最终推理方案；其余内容仍在制作与复盘中，后续将逐步补充。

## 任务与入口

对测试数据生成 `tvt` 预测。入口为 [inference.ipynb](inference.ipynb)，包含基线运行和后续候选方案两个有序单元。

## 方法

1. 自动定位包含测试数据和提交模板的比赛目录，调用 E70a2 reduced-Ridge 推理包，生成基线 `submission.csv`。
2. 对基线检查列、ID 顺序、重复值及预测是否为有限数，并记录文件 SHA-256。
3. 保存安全 Ridge8 基线，定位已有 physics 输出；将 physics runtime 和相应模型组织到工作目录，调用外部 `prefix_gold_candidate.py`，传入基线、physics 输出和 physics runner 生成候选结果。

第二阶段的具体特征、组合与修正规则由挂载的外部脚本和模型包实现，不能仅凭此 notebook 完整复现其内部算法。注释掉的单元是备选代码，不是默认执行流程。

## 运行与依赖

Kaggle 中需挂载比赛数据、`lihu6666/ridge` 的 E70a2 reduced-Ridge 包和 `lihu6666/input-qianzhui` 代码数据集，确认 notebook 引用的脚本及模型存在后按顺序运行。外部资源未包含在仓库中。最终提交前应再次核对输出和模板；本次未执行推理或验证分数。

## 实现细节

### 1. 自动发现数据与基线包

第一单元在 `/kaggle/input` 下寻找同时包含 `sample_submission.csv` 和 `test` 目录的比赛数据，并寻找唯一的 `run_kaggle_e70a2.py`。如果未发现已解压启动器，则寻找唯一 E70a2 bundle ZIP，在解压前检查绝对路径与 `..` 路径。多个候选或缺失时直接报错，避免静默使用错误文件。

### 2. E70a2 reduced-Ridge 推理

启动器接收数据目录、输出路径和工作目录；参数指定 `cuda:0`、4 个 jobs、每批 4 口井。执行完成后对输出的列、ID 顺序、重复 ID 和 TVT 有限性进行校验，并打印文件哈希。具体特征构造、Ridge 训练及内部融合逻辑由外部 bundle 定义，当前 notebook 没有展开这些实现。

### 3. Prefix Gold 候选处理

第二单元复制 `prefix_gold_candidate.py` 与 `run_kaggle_prefix_gold.py` 到可写目录，把旧的 `horizontal.row_idx` 访问修补为 `row_ids(horizontal)`，并通过断言检查修补结果。随后将已有提交文件备份为 `submission_safe_ridge8.csv`（若备份已存在则沿用），在工作目录查找唯一的非 prefix_gold physics 提交文件。

运行前检查 TCN、Ridge、GRU 和两个 point-rate 模型文件；将 physics runtime 复制到工作目录，并以符号链接连接原模型目录。候选运行器接收测试数据、安全基线、physics 提交和 physics runner，设置 `max-alpha=0.30`，将结果写回 `submission.csv`，并指定审计文件目录。注释称这一步在测试井 TVT_input 内做伪切分后的候选处理；具体权重选择和公式在外部脚本中，不能仅根据参数推定。

### 4. 输出检查

第一单元只验证基线；第二单元覆盖最终输出后，建议再次检查列顺序、ID、有限值和审计 JSON。旧备份会被复用，重跑前应确认它对应本次基线。第三单元全部注释，属于未执行的 affine5 备选流程。
