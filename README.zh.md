# FBCNet 中文导读

> 本仓库是 [ravikiran-mane/FBCNet](https://github.com/ravikiran-mane/FBCNet) 的 fork，用于学习脑机接口（BCI）中的 EEG 分类方法。原始 README 见 [README.md](README.md)。

论文：Mane et al., *FBCNet: A Multi-view Convolutional Neural Network for Brain-Computer Interface*, arXiv:2104.01233 (2021)

## 这个模型解决什么问题

用头皮脑电（EEG）做运动想象（Motor Imagery）分类时，有两个核心困难：

1. **信噪比低**——单次试验的 EEG 信号淹没在噪声里
2. **样本少**——每个被试通常只有几百次试验，深层网络极易过拟合

FBCNet 的思路是先做**多视图频谱分解**，把原始信号拆成多个窄带，再在每带上分别提取空间判别特征，最后用**方差层**压缩时间维度。这样既保留了频谱-空间的可判别信息，又大幅降低了参数量。

## 四阶段结构

| 阶段 | 操作 | 作用 |
| --- | --- | --- |
| 1. 多视图表示 | 用多个窄带滤波器对原始 EEG 做频谱滤波 | 得到多个频带的并行视图 |
| 2. 空间变换学习 | Depthwise Convolution | 在每个视图上学习空间判别模式 |
| 3. 时间特征提取 | Variance layer（方差层，非池化） | 压缩时间维度，保留能量信息 |
| 4. 分类 | 全连接层 | 输出类别 |

关键设计是第 3 步用方差代替常规池化——EEG 的判别信息主要体现在各频带的能量差异上，方差层正好对应这个物理含义。

## 代码结构

```
codes/
  classify/
    cv.py                   入口：被试内 10 折交叉验证
    ho.py                   入口：跨被试（hold-out）评估
  centralRepo/              核心模块
    networks.py             网络定义（FBCNet / DeepConvNet / EEGNet）
    baseModel.py            训练循环、早停、评估指标
    eegDataset.py           数据集加载
    transforms.py           数据增强与频带滤波
    saveData.py             数据预处理与缓存
    stopCriteria.py         早停判据
    samplers.py             采样策略（类别不平衡处理）
  cvFiles/                  预生成的交叉验证划分索引（JSON）
  netInitModels/            各网络的初始权重（.pth）
Instructions                完整环境搭建与运行步骤（英文）
req.txt                     Python 依赖
FBCNet-V2.png               网络结构图
results.png                 对比实验结果
```

## 支持的数据集

- **BCI Competition IV-2a**（4 类运动想象，9 被试）
- **Korea University EEG dataset**

`codes/cvFiles/` 里已有这两个数据集的交叉验证划分索引，保证结果可复现。

## 运行

环境要求见 `req.txt`（作者在 Python 3.7 + Ubuntu 上测试过），完整步骤见 `Instructions`。

```bash
# 被试内交叉验证
python codes/classify/cv.py

# 跨被试评估
python codes/classify/ho.py
```

数据路径需要在脚本里按实际情况修改。

## 学习路径建议

如果是第一次接触这个仓库，建议按以下顺序阅读：

1. `networks.py` 里的 `FBCNet` 类——只有几十行，看得懂结构就理解了全文
2. `transforms.py` 的频带滤波部分——理解多视图怎么来的
3. `baseModel.py` 的训练循环
4. `cv.py` / `ho.py`——了解评估协议

## 注意

- 本 fork 与上游当前状态**完全一致**（ahead=0, behind=0），没有本地改动
- 初始权重 `.pth` 已在 `codes/netInitModels/` 中，无需另行下载
- `deepConvNet_*.pth` 各约 1.1MB，是三个对比基线中体积最大的
