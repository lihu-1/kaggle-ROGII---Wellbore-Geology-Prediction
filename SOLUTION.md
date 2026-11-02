# ROGII Wellbore Geology Prediction 推理方案

## 任务与入口

对测试数据生成 `tvt` 预测。入口为 [inference.ipynb](inference.ipynb)，包含基线运行和后续候选方案两个有序单元。

## 方法

1. 自动定位包含测试数据和提交模板的比赛目录，调用 E70a2 reduced-Ridge 推理包，生成基线 `submission.csv`。
2. 对基线检查列、ID 顺序、重复值及预测是否为有限数，并记录文件 SHA-256。
3. 保存安全 Ridge8 基线，定位已有 physics 输出；将 physics runtime 和相应模型组织到工作目录，调用外部 `prefix_gold_candidate.py`，传入基线、physics 输出和 physics runner 生成候选结果。

第二阶段的具体特征、组合与修正规则由挂载的外部脚本和模型包实现，不能仅凭此 notebook 完整复现其内部算法。注释掉的单元是备选代码，不是默认执行流程。

## 运行与依赖

Kaggle 中需挂载比赛数据、`lihu6666/ridge` 的 E70a2 reduced-Ridge 包和 `lihu6666/input-qianzhui` 代码数据集，确认 notebook 引用的脚本及模型存在后按顺序运行。外部资源未包含在仓库中。最终提交前应再次核对输出和模板；本次未执行推理或验证分数。
