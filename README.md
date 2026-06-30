# GFP 候选序列推理流程说明

本仓库用于运行 GFP / sfGFP 候选序列的计算筛选与推理流程。主要功能包括：读取输入序列、计算或加载 ESM2 / SaProt 表征、构建结构与理化特征、调用亮度预测模型进行打分，并结合 ThermoMPNN / FoldX 结果辅助筛选最终候选序列。

## 1. 环境配置要求

推荐在 Linux 服务器上运行，若需要重新计算 ESM2 / SaProt embedding，建议使用 GPU。

```text
Python: 3.11
Conda 环境名: GFP
推荐 GPU: NVIDIA RTX 3090 Ti 或其他 CUDA GPU
```

创建主环境示例：

```bash
conda create -n GFP python=3.11 -y
conda activate GFP
```

安装 Jupyter kernel：

```bash
pip install jupyter ipykernel
python -m ipykernel install --user --name GFP --display-name "GFP"
```

## 2. Python 依赖库

主流程依赖以下 Python 包：

```text
numpy
pandas
scipy
scikit-learn
xgboost
torch
transformers
biopython
tqdm
joblib
matplotlib
seaborn
openpyxl
jupyter
ipykernel
```

可使用以下命令安装：

```bash
pip install numpy pandas scipy scikit-learn xgboost torch transformers biopython tqdm joblib matplotlib seaborn openpyxl jupyter ipykernel
```

如果需要运行 ThermoMPNN，建议单独准备环境，例如：

```bash
conda activate deepstab_env
```

ThermoMPNN 的具体依赖请按照其原项目说明安装。

## 3. 外部模型与工具依赖

本流程需要以下本地模型或外部工具。换服务器运行时，需要在 notebook 的配置单元格中修改对应路径。

```text
ESM2 model:       /home/liuchang/esm2_t33_650M
SaProt model:     /home/liuchang/saprot_650M
SaProt code:      /home/liuchang/saprot/SaProt
Foldseek binary:  /home/liuchang/anaconda3/envs/GFP/bin/foldseek
ThermoMPNN code:  /data/liuchang/ThermoMPNN-main
FoldX binary:     /data/liuchang/tools/foldx5.1/foldx
```

注意：

```text
1. ESM2 / SaProt 模型权重不包含在本仓库中，需要自行下载或配置。
2. FoldX 需要单独申请和安装，不能直接随仓库上传。
3. ThermoMPNN 和 FoldX 结果仅作为稳定性 proxy，用于辅助筛选候选序列。
```

## 4. 输入文件

运行主 notebook 前，需准备以下文件：

```text
GFP_data.xlsx
AAseqs of 5 GFP proteins.txt
Exclusion_List.csv
fold_avgfp_model_0.cif
fold_amacgfp_model_0.cif
fold_cgregfp_model_0.cif
fold_pplugfp_model_0.cif
fold_sfgfp_model_0.cif
sfGFP.pdb
```

这些文件默认放在项目根目录下。如路径不同，需要在 notebook 配置区修改。

## 5. 推理代码运行方式

启动 Jupyter：

```bash
conda activate GFP
jupyter lab
```

打开主 notebook：

```text
hechengbio_server_full_recompute_embeddings(1).ipynb
```

根据需要选择运行模式。

快速调试模式：

```python
FAST_DEV_MODE = True
FULL_RUN = False
```

正式推理 / 完整搜索模式：

```python
FAST_DEV_MODE = False
FULL_RUN = True
```

运行顺序：

```text
1. 运行配置与 helper function 单元格
2. 读取输入数据和参考序列
3. 加载或计算 ESM2 / SaProt embedding
4. 构建 pocket features 和 structure-role features
5. 加载或训练亮度预测模型
6. 对候选序列进行打分
7. 可选运行 ThermoMPNN 稳定性初筛
8. 可选运行 FoldX 多突变稳定性复核
9. 导出最终候选序列和 submission.csv
```

## 6. 主要输出文件

默认输出目录：

```text
server_full_recompute_outputs/
```

常见输出文件：

```text
recomputed_candidate_scores.csv
recomputed_top6_detailed_report.csv
recomputed_submission_top6.csv
thermompnn_single_mutation_scan.csv
foldx_review_summary_deduped.csv
foldx_all_ran_candidates_detailed_summary.csv
candidate_filter_funnel.csv
lofo_submodel_metrics.csv
crossfamily_model_weights.csv
```

最终提交文件：

```text
submission.csv
```

提交文件格式：

```text
Team_Name,Seq_ID,Sequence
```

## 7. 注意事项

```text
1. 重新计算 embedding 会耗时较长，建议优先复用缓存。
2. 修改输入序列、模型路径或特征版本后，应重新计算对应 embedding。
3. FoldX 的 mutant file 文件名应为 individual_list.txt，或以 mutant_file 开头。
4. FoldX 的 Average_*.fxout 中，SD 不是 ddG；应使用 total energy 作为 FoldX ddG proxy。
5. 模型预测、ThermoMPNN 和 FoldX 均为计算筛选依据，不能替代真实实验验证。
```
