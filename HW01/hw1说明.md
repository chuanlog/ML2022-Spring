# 作业 1（HW01）说明：COVID-19 确诊病例预测（回归 Regression）

> 本说明基于同目录下的 `HW01.ipynb`（已翻译为中文版）整理，帮助你快速理解作业目标与完成步骤。
> 官方完整要求（评分细则、数据字段表、提交格式等）请以 `HW01.pdf` 为准。

---

## 一、作业是什么

- **任务类型**：回归（Regression）
- **目标**：用**深度神经网络（DNN）**预测 COVID-19 的阳性率（目标列 `tested_positive`）。
- **学习目标**：
  1. 用 DNN 解决一个回归问题；
  2. 理解 DNN 的基础训练技巧（特征选择、模型结构、优化器、正则化、调参）；
  3. 熟悉 **PyTorch** 的基本使用。
- **最终产出**：一个 `pred.csv` 预测文件，上传到对应的 Kaggle 比赛页面进行评分。

---

## 二、数据说明

| 文件 | 规模 | 说明 |
| --- | --- | --- |
| `covid.train.csv` | 2699 × 118 | 训练数据，含标签 |
| `covid.test.csv` | 1078 × 117 | 测试数据，**不含最后一天的阳性率（需要你预测）** |

- 每行代表一个州在连续 5 天的数据：`id + 37 个州标识 + 16 种特征 × 5 天 + 目标值(tested_positive)`。
- 最后一列 `tested_positive` 是回归目标；测试集没有这一列。
- 数据下载：notebook 顶部提供 Google Drive 链接；若失效，可从 [Kaggle](https://www.kaggle.com/t/a3ebd5b5542f0f55e828d4f00de8e59a) 下载后手动上传到工作区。

---

## 三、目录文件

- `HW01.pdf`：官方作业说明文档。
- `HW01.ipynb`：**已翻译为中文版**的 Notebook，包含完整的代码框架与中文注释，按单元格顺序运行即可。
- `hw1说明.md`：本说明文档。
- （运行后会产生）`models/model.ckpt`：训练保存的最优模型。
- （运行后会产生）`pred.csv`：提交用的预测结果。

> 注：仓库里还保留了 `HW01.ipynb.bak`（翻译前原始英文备份）和 `translate_ipynb.py`（翻译脚本），如不需要可自行删除。

---

## 四、你需要完成的工作

代码框架已经搭好，你主要需要**改进以下四处（notebook 中均标有 `TODO` 或明确提示）**：

### 1. 模型结构 —— `My_Model` 类
- 默认是一个简单的 3 层全连接网络：`Linear(input_dim,16) → ReLU → Linear(16,8) → ReLU → Linear(8,1)`。
- **你需要**：尝试更深 / 更宽的网络、不同的激活函数、加 Dropout / BatchNorm 等，注意各层维度要匹配。
- 位置：notebook 中「神经网络模型」代码单元格。

### 2. 特征选择 —— `select_feat` 函数
- 当 `config['select_all'] = True` 时使用全部 117 个特征；设为 `False` 时，函数里 `feat_idx = [0,1,2,3,4]` 只选了前 5 列（这是**示例占位**，需要你改进）。
- **你需要**：分析并挑选对预测有用的特征列，提升模型表现。

### 3. 优化器与正则化 —— `trainer` 函数
- 损失函数固定为 **MSELoss（不可修改）**。
- 默认优化器是 `SGD(lr=1e-5, momentum=0.9)`。
- **你需要**：
  - 尝试 `Adam` / `AdamW` 等不同优化器（参考 [PyTorch optim](https://pytorch.org/docs/stable/optim.html)）；
  - 加入 **L2 正则化**（`weight_decay` 参数或自行实现），缓解过拟合。

### 4. 超参数调优 —— `config` 字典
可调整项包括：
- `seed`：随机种子（保证可复现）；
- `select_all`：是否使用全部特征；
- `valid_ratio`：验证集占比（默认 0.2）；
- `n_epochs`：训练轮数；
- `batch_size`：批次大小；
- `learning_rate`：学习率；
- `early_stop`：连续多少轮验证集无提升就提前停止；
- `save_path`：模型保存路径。

---

## 五、运行与完成步骤

按 notebook 单元格顺序执行即可：

1. **下载数据**：运行「下载数据」单元格（或手动上传 `covid.train.csv` / `covid.test.csv`）。
2. **导入套件**：NumPy、Pandas、PyTorch、tqdm、TensorBoard 等。
3. **工具函数**：`same_seed`（固定随机种子）、`train_valid_split`（划分训练/验证集）、`predict`（对测试集预测）。这部分无需修改。
4. **数据集**：`COVID19Dataset` 已将 numpy 数组封装为 PyTorch `Dataset`。无需修改。
5. **神经网络模型 / 特征选择 / 训练循环 / 配置参数**：在这里完成上面「四」中的改进。
6. **数据加载器**：读取数据、划分、选特征、构建 `DataLoader`。无需修改（除非你改了特征逻辑）。
7. **开始训练！**：运行训练单元格，模型会在验证集 loss 最优时自动保存到 `./models/model.ckpt`，并用 early stopping 控制训练长度。
8. **生成提交文件**：加载最优模型，对测试集预测，调用 `save_pred` 生成 `pred.csv`（格式：`id,tested_positive`）。

> 提示：训练默认跑在 GPU（`device = 'cuda' if ... else 'cpu'`），无 GPU 时会自动用 CPU，只是更慢。

---

## 六、提交方式

1. 运行完 notebook 后，确认生成了 `pred.csv`。
2. 将其上传到作业对应的 **Kaggle 比赛**页面（notebook 顶部有 Kaggle 链接），即可获得公开/私下的分数。

---

## 七、提升分数的建议（供参考）

- **特征工程**：COVID 数据里并非所有 16×5 特征都相关，尝试相关性分析、去除冗余特征。
- **模型容量**：适当加深/加宽网络，但注意过拟合（用验证集监控）。
- **正则化**：Dropout、BatchNorm、L2（`weight_decay`）。
- **学习率策略**：可用 `torch.optim.lr_scheduler` 做学习率衰减。
- **训练稳定性**：固定 `seed`、合理设置 `batch_size` 与 `learning_rate`，关注 Train/Valid loss 曲线（notebook 已接 TensorBoard）。

---

## 八、注意事项

- 损失函数（MSELoss）**不要修改**，这是评分基准。
- 提交文件格式必须是 `id,tested_positive` 两列，与 `save_pred` 输出一致。
- 具体评分占比、排名规则、迟交政策等以 `HW01.pdf` 官方文档为准。
