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
