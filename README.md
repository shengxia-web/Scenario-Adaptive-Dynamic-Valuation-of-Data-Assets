# Scenario-Adaptive-Dynamic-Valuation-of-Data-Assets
A Scenario-Adaptive Graph-Based Model for Dynamic Valuation of Data Assets
# DataAssetGraphData (DLG-DR-23) 预处理代码
本仓库提供论文 *"A Scenario-Adaptive Graph-Based Model for Dynamic Valuation of Data Assets"* 
统一实体与关系模式（Schema Mapping）
缺失值过滤（缺失率 > 50% 的节点删除）
长尾特征对数变换（`log1p`）
z-score 标准化
边权重归一化（基于调用频率）
时间序列构建与按时间顺序切分（80% 训练 / 20% 测试）
生成 PyTorch Geometric (`Data`) 对象，按时间步组织
## 目录结构
├── preprocess_dlg.py # 主预处理脚本
├── requirements.txt # Python 依赖
├── data/
│ └── DLG-DR-23/ # 原始数据目录（用户自行放置）
│ ├── DLG_1/
│ │ ├── nodes.csv
│ │ ├── edges.csv
│ │ └── time_series.csv
│ ├── DLG_2/
│ │ └── ...
│ └── ...
└── processed/ # 预处理输出目录（自动创建）
├── DLG_1/
│ ├── nodes_processed.csv
│ ├── edges_processed.csv
│ ├── time_series_processed.csv
│ ├── train_times.json
│ └── test_times.json
└── ...
## 数据格式说明
原始数据应为每个 DLG 一个文件夹，包含以下 CSV 文件：
### nodes.csv
每行描述一个数据资产节点，必须包含以下字段（字段名可能不同，脚本会自动映射）：
node_id：节点唯一标识符（字符串或整数）
quality：数据质量综合评分
access_freq：访问频率
business_contrib：业务贡献度
risk：风险等级
cost：存储与计算成本
其他列将被保留但不在特征中使用
### edges.csv
每行描述一条有向边，字段：
src：源节点 ID
dst：目标节点 ID
call_freq：调用频率，用于计算边权重；若缺失则默认为 1.0
其他列将被忽略
## 依赖安装
Python 3.8+ 

bash
pip install -r requirements.txt

numpy>=1.21.0
pandas>=1.3.0
scipy>=1.7.0
torch>=1.10.0
torch-geometric>=2.0.0
networkx>=2.6
scikit-learn>=1.0.0

使用方法
1. 准备原始数据
将 DLG-DR-23 数据集解压到 data/DLG-DR-23/ 目录下，确保每个 DLG 子文件夹中存在 nodes.csv、edges.csv 和可选的 time_series.csv。

2. 执行预处理
直接运行脚本（处理全部 18 个 DLG）：

bash
python preprocess_dlg.py
脚本会：

遍历 data/DLG-DR-23/ 下的所有子文件夹（默认识别为 DLG_*）

对每个 DLG 执行完整预处理

将结果保存到 processed/ 对应子文件夹中

3. 自定义配置
可在脚本开头的参数区调整：

python
DATA_ROOT = "./data/DLG-DR-23"          # 原始数据根目录
OUTPUT_ROOT = "./processed/DLG-DR-23"   # 输出根目录
MISSING_THRESHOLD = 0.5                 # 缺失率阈值
TIME_SPLIT_RATIO = 0.8                  # 训练集比例
RANDOM_SEED = 42
4. 在模型中使用预处理结果
预处理生成每个 DLG 的训练/测试时间步列表（JSON 格式），以及处理后的节点/边/时间序列表格。若需要使用 PyTorch Geometric 的 Data 对象，可参考脚本中的 build_pyg_data() 函数，它返回按时间步分割的 Data 列表。

预处理步骤详解
统一 Schema
将原始字段名映射到论文统一特征：quality, access_freq, business_contrib, risk, cost。

缺失节点过滤
对于每个节点，计算上述 5 个特征的缺失比例，若超过 MISSING_THRESHOLD（默认 50%）则删除该节点及其关联边。

特征变换

对 access_freq 和 cost 进行 log1p 变换（因通常呈长尾分布）。

用中位数填充剩余缺失值。

对所有统一特征进行 Z-score 标准化（减去均值，除以标准差）。

边权重计算
依据论文公式 (1)，若缺失则默认权重为 1.0。

时间序列整合
若提供 time_series.csv，则将其与静态节点特征合并；否则生成单一时间步（t=0）。最终形成包含 node_id, timestamp 和所有特征的宽表。

时间切分
将时间步排序后，前 80% 作为训练集，后 20% 作为测试集（保持时间顺序，避免未来信息泄露）。

输出保存

处理后的节点表、边表、时间序列表（CSV）

训练/测试时间戳列表（JSON）

输出文件说明
nodes_processed.csv	过滤并标准化后的节点特征（静态）
edges_processed.csv	包含归一化权重 weight 的边表
time_series_processed.csv	完整时间序列特征（每个节点每个时间步一行）
train_times.json	训练时间步列表
test_times.json	测试时间步列表
