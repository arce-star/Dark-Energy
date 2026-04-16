# Dark-Energy

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue)](https://www.python.org/)

一个用于分析 Tonry 2003 超新星数据的 Python 项目，通过线性回归、最大似然估计（MLE）和贝叶斯 MCMC 方法精确测量哈勃常数 $H_0$。

## 项目概述

本项目利用 Ia 型超新星的观测数据来验证哈勃定律，并估算哈勃常数 $H_0$：

- **数据集**: 使用 Tonry 2003 公布的红移-距离模数数据集。
- **物理模型**: 哈勃定律 $v = H_0 \times d$。
- **核心目标**: 获得 $H_0$ 的最优估计及其不确定性。

## 项目结构

```bash
Dark_Energy/
├── data/                    # 原始天文数据
│   └── Tonry_2003.vot       # Tonry 2003 观测表格
├── src/                     # 源代码
│   ├── data.py         # 数据加载及预处理
│   ├── visualization.py     # 数据可视化
│   ├── linear.py            # 线性拟合
│   ├── MLE.py               # 最大似然估计
│   └── MCMC.py            # MCMC采样
├── notebooks/               # Jupyter 笔记本
│   └── Hubble_Constant_MCMC.ipynb
├── results/                 # 分析结果
│   ├── plots/               # 数据预处理图
│   ├── linear/              # 线性拟合图
│   ├── MLE/                 # 最大似然估计图
│   └── MCMC/                # MCMC采样图
├── requirements.txt
└── README.md
```

## 快速开始

### 安装步骤

1. **克隆仓库**

   ```bash
   git clone https://github.com/arce-star/Dark-Energy/hubble-constant-analysis.git
   cd hubble-constant-analysis
   ```

2. **安装依赖**

   ```bash
   pip install -r requirements.txt
   ```

### 基本使用

运行主分析笔记本：

```python
# 打开Jupyter笔记本
jupyter notebook notebooks/darkenergy.ipynb
```

## 分析流程

### 1. 数据预处理 (Data Preprocessing)

- **格式解析**: 使用 `astropy.io.ascii` 读取 VOTable 数据。
- **量纲转换**: 
  - 将距离模数（Distance Modulus）转换为线性距离（Mpc）。
  - 处理速度单位（km/s）。
- **误差传播**: 考虑到距离模数是对数尺度，将其误差转换为线性空间下的非对称误差。

### 2. 可视化分析 (Visualization)

- **对数坐标分布**: 解决近处数据密集、远处数据稀疏的拟合权重倾斜问题。
- **误差条图**: 在线性坐标下展示速度与距离的关系及其观测不确定性。
- **数据筛选**: 选取距离在 0-700 Mpc 范围内的良好样本进行拟合。

### 3. 线性拟合 (Linear Fitting)

- **坐标变换**: 考虑到距离测量误差远大于速度测量误差，将距离设为因变量 $d = m \cdot v + b$。
- **加权最小二乘 (WLS)**:
  - 构建协方差矩阵 $C$。
  - 通过矩阵运算 $\mathbf{w} = (A^T C^{-1} A)^{-1} A^T C^{-1} \mathbf{y}$ 获取解析解。
  - 估算初步的哈勃常数 $H_0 = 1/m$。

### 4. 最大似然估计 (MLE)

- **似然函数定义**: 引入固有弥散因子 $f$，构建高斯似然函数。
- **异方差建模**: 总方差 $\sigma^2 = \sigma_{\text{err}}^2 + \text{model}^2 \cdot \exp(2\ln f)$。
- **数值优化**: 使用 `scipy.optimize.minimize` 寻找参数空间的最优解，作为 MCMC 的初始位置。

### 5. MCMC 采样 (MCMC Sampling)

- **贝叶斯框架**: 
  - **先验 (Prior)**: 设定参数的物理合理范围（无信息先验）。
  - **后验 (Posterior)**: $\text{log-post} = \text{log-prior} + \text{log-likelihood}$。
- **采样策略**: 
  - 使用 `emcee` 的仿射不变集成采样器。
  - 32 个行走者进行 5000 次迭代。
- **结果导出**:
  - 绘制参数演化迹图，检查自相关时间确保收敛。
  - 绘制 **Corner Plot** 展示 $H_0$ 与其它参数的关联。

## 技术细节

### 核心概率模型

对于观测数据 $y$，我们建立如下似然模型：

$$ \ln \mathcal{L} = -\frac{1}{2} \sum \left[ \frac{(y - \text{model})^2}{\sigma^2} + \ln \sigma^2 \right] $$

其中总方差 $\sigma^2 = \sigma_{\text{obs}}^2 + \text{model}^2 \cdot \exp(2\ln f)$。

### 主要依赖库

- **Astropy**: 天文坐标与单位计算。
- **Emcee**: 仿射不变 MCMC 采样器。
- **Corner**: 后验分布可视化。
- **Scipy**: 数值优化与矩阵运算。

## 结果示例

项目生成的分析结果包括：

- **哈勃图**: 包含 WLS、MLE 与 MCMC 后验线条的对比。
- **参数轨迹图**: 用于诊断 MCMC 链的收敛状态。
- **Corner Plot**: 展示 $H_0$、截距与弥散因子的多维概率分布。

## 物理背景

哈勃常数 $H_0$ 是宇宙学中最重要的参数之一：

- 它决定了宇宙的膨胀速率和年龄。
- Ia 型超新星作为“标准烛光”，是测量宇宙距离梯度的关键工具。

## 贡献

欢迎提交 Issue 和 Pull Request 来改进拟合方法或增加新的数据集。

## 致谢

- 数据来源于 Tonry et al. (2003) 的Ia超新星项目。
- 感谢 `emcee` 团队提供的卓越采样工具。
- 感谢 github.com/wj198414/ASTRON1221 项目
