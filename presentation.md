---
marp: true
theme: default
paginate: true
backgroundColor: #ffffff
style: |
  section {
    font-family: 'Microsoft YaHei', 'SimHei', sans-serif;
  }
  h1 {
    color: #2563eb;
  }
  h2 {
    color: #1e40af;
  }
  .columns {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
  }
  .highlight {
    background: linear-gradient(90deg, #3b82f6, #8b5cf6);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    font-weight: bold;
  }
  table {
    font-size: 0.8em;
  }
  .small {
    font-size: 0.75em;
  }
---

<!-- _class: lead -->

#  CIFAR-10 图像分类
## 传统机器学习 vs 深度学习方法对比研究

**机器学习期末项目**

###### 江煜 $\space$ 朱洋微 $\space$ 高健峰

---

#  项目概览

## 研究目标
深入探究和对比 **传统机器学习方法(SVM)** 与 **深度学习方法(CNN)** 在图像分类任务上的差异

## 数据集：CIFAR-10
-  **60,000** 张 32×32 彩色图像
-  **10** 个类别：飞机、汽车、鸟、猫、鹿、狗、青蛙、马、船、卡车
-  训练集 50,000 张，测试集 10,000 张
> 我们从[Kaggle](https://www.kaggle.com/competitions/cifar-10/data?select=trainLabels.csv)上获取数据并评价模型

![bg right:40% contain](./resources/CIFAR-10-web.png)

---

#  技术路线

<div class="columns">
<div>

### 传统机器学习方法
1. **HOG 特征提取**
   - 捕捉边缘和纹理信息
2. **PCA 降维**
   - 保留95%方差
   - 减少计算量
3. **SVM 分类器**
   - RBF核函数
   - 网格搜索调参

</div>
<div>


### 深度学习方法
1. **端到端学习**
   - 直接从原始像素学习
   - 自动特征提取
2. **CNN 卷积神经网络**
   - 卷积层提取特征
   - 池化层降维
   - 全连接层分类

</div>
</div>

> 我们使用`git`进行协作，项目代码和材料可在[GitHub](https://github.com/AlanRosston/FinalProject_25A_ML)找到

---

#  Part 1: 传统方法 - HOG特征提取

## HOG (方向梯度直方图) 原理

**核心思想**: 局部目标的表象和形状能够被梯度方向密度分布描述



![bg right:50% contain](./resources/hog_tempalte.png)

---

#  PCA 降维分析

## 方差解释比例

| 主成分数量 | 累积方差 |
|:---:|:---:|
| 4 | >20% |
| 50 | $\approx$ 88% |
| **155** | **>95%** |

###  关键发现
> 保留95%信息需要**155维**，说明图像特征的**复杂性**和**冗余性**

手工特征选择的**困难性**由此可见

<style>
.top-right-image {
  position: absolute;
  top: 200px;
  right: 50px;
  width: 800px;
}
</style>

<img src="./resources/PCA_Variation.png" class="top-right-image">



---

#  t-SNE 可视化分析

## HOG特征的可分离性

<div class="columns">
<div>

### 观察结果
-  无法找到完全分离的簇
-  cat 与  deer 分布相近
-  dog 与  ship 距离较远
- 相似类别**严重重叠**，但有些类别之间的差距足够大

</div>
<div>

### 问题揭示
传统特征难以区分：
- 🐱 猫 vs 🐕 狗
- 🐦 鸟 vs 🦌 鹿
- ✈️ 飞机 vs 🚢 船

**类内差异大，类间差异小**

</div>
</div>

---

![bg 60%](./resources/t-sne-picture.png)

---



#  Part 2: SVM 模型训练

### 核函数对比实验

对比不同核函数（linear、rbf、poly、sigmoid）在 SVM 模型中的表现，通过训练时间和测试集准确率两个指标，选出最优的核函数。
![](resources/svm_FourKernel.png)


---
#  超参数优化

## 粗粒度：<br>网格搜索 + 三折交叉验证

![bg right contain](resources/svm_heatmap.png)

### 参数影响分析
- **gamma** 影响显著。显然在gamma=0.002取最优，可进一步精确探索
- **C** 影响相对较小。在最优gamma区间内变化不大

---

## 细粒度：细化gamma+五折交叉验证
gamma: 0.001到0.003之间随机取30个值(主要集中在0.002附近)
![bg left:50% contain](resources/svm_gamma.png)

### 分析
1. 最优gamma值：0.002020（对应准确率：0.5974）
2. 0.002附近gamma值的平均准确率：0.5964
3. gamma偏离0.002越远，准确率变化趋势：下降

**最优参数**：C=5, gamma=0.002020

---

#  SVM 分类结果分析

![bg left:50% contain](resources/svm_report.png)

## 测试集准确率: **59.55%**

<div class="columns">
<div>

###  表现好的类别
| 类别 | F1-Score |
|:---:|:---:|
| automobile | 0.75 |
| truck | 0.72 |
| frog | 0.67 |

</div>
<div>

###  表现差的类别
| 类别 | F1-Score |
|:---:|:---:|
| cat | 0.31 |
| bird | 0.49 |
| dog | 0.52 |

</div>
</div>

---

#  混淆矩阵分析：

![bg left:45% contain](resources/svm_ConfusionMatrix.png)

## 严重的类别混淆现象

###  Cat 的误判分析
- 200个样本中仅 **60个** 正确分类 (30%)
- **58个** 被误判为  dog

###  根本原因
- 32×32 低分辨率丢失关键细节
- 猫/狗的毛发、轮廓特征相似
- **SVM + HOG 无法捕捉细粒度语义差异**

---

# ROC曲线分析

![bg right:45% contain](resources/svm_ROC.png)

**类别间差异**

表现优异的类别：automobile（AUC=0.944）、truck（AUC=0.946）、frog（AUC=0.932）等，ROC 曲线靠近左上角，AUC 接近 0.95，说明这些类别被模型准确区分的概率很高；

表现较差的类别：cat（AUC=0.777），ROC 曲线更靠近随机虚线，AUC 明显低于其他类别，是模型的薄弱项。

---

# Cat类别调优

先单独尝试了类别加权重新训练模型，效果甚微，分类报告无变化<br>

## 定制化特征工程
挖掘 cat 类独有的视觉特征（形状、纹理、边缘等），弥补原有通用特征对 cat 类区分度不足的缺陷。

---
![bg left:60% contain](resources/svm_report2.png)

## 分析
- 猫类别表现有所提升（10%）
- 其他类别未受影响 
- 整体准确率有微小提升（0.1%）

---
# kaggle提交结果

![bg right:50% contain](resources/SVM-Kaggle2.png)

**分数**：
1. 0.59690（gamma=0.002020）
2. 0.59540 (gamma=0.002)

---

#  Part 3: 深度学习方法

## CNN 卷积神经网络架构

```
输入 (3×32×32) 
    → Conv2d(3→6, 5×5) → ReLU → MaxPool(2×2)
    → Conv2d(6→16, 5×5) → ReLU → MaxPool(2×2)
    → Flatten(16×5×5=400)
    → Linear(400→120) → ReLU
    → Linear(120→84) → ReLU
    → Linear(84→10)
输出 (10类概率)
```

**特点**: 端到端学习，自动提取特征

---

#  深度学习 vs 传统机器学习

## 核心差异对比

<div class="columns">
<div>

### 传统机器学习 (SVM)
-  依赖**手工特征设计**
-  HOG仅捕捉边缘/纹理
-  难以学习语义信息
-  特征与分类器**分离**

</div>
<div>

### 深度学习 (CNN)
-  **自动学习**层级特征
-  从边缘→纹理→语义
-  端到端优化
-  特征提取与分类**联合优化**

</div>
</div>

---

#  性能对比：关键结果

## Kaggle 测试集得分

<div style="text-align: center; font-size: 1.5em; margin: 2em 0;">

| 方法 | 准确率 | 提升 |
|:---:|:---:|:---:|
| HOG + SVM | **59.54%** | 基准 |
| CNN | **62.05%** | +4.2% |

</div>

###  CNN 胜出！
即使是**简单的2层卷积网络**也超越了精心优化的传统方法

---

# Kaggle提交得分

$\space$
$\space$
$\space$
$\space$
$\space$
$\space$
$\space$
$\space$
$\space$
$\space$
$\space$

![bg 60%](./resources/CIFAR-10_Result_DeepLearning.png)

---

#  深度学习的优势总结

## 为什么 CNN 更强？

### 1️ 自动特征学习
- 无需手工设计 HOG、SIFT 等特征
- 从数据中自动学习最优表示

### 2️ 层级特征抽象
- 浅层学习边缘、颜色
- 深层学习纹理、形状、语义

### 3️ 端到端优化
- 特征提取与分类器**联合训练**
- 全局最优而非局部最优

---

#  为什么 HOG+SVM 落后？

## 传统方法的局限性

###  特征表达能力不足
- HOG 只能捕捉**局部梯度统计**
- 丢失颜色、纹理细节
- 无法理解**语义信息**

###  类别混淆严重
- 🐱猫/🐕狗 难以区分（大小相仿）
- ✈️飞机/🚢船 容易混淆（相似轮廓）

###  特征与分类器分离
- PCA可能丢弃**判别性特征**
- SVM无法反馈优化特征提取

---

#  深度学习如何解决这些问题？

## CNN 的解决方案

<div class="columns">
<div>

###  丰富的特征学习
- 卷积核自动学习**多种模式**
- 保留颜色、纹理、形状信息
- 学习**高层语义特征**

</div>
<div>

###  反向传播优化
- 分类误差反馈到特征提取
- **端到端联合优化**
- 特征更具判别性

</div>
</div>

###  提升空间巨大
使用 ResNet、VGG 等深层网络可达 **90%+** 准确率！

---

#  未来展望

## 深度学习的发展方向

###  更强的网络架构
- ResNet、EfficientNet、Vision Transformer
- 更深的网络，更强的表达能力

###  先进训练技术
- 数据增强 (Data Augmentation)
- 迁移学习 (Transfer Learning)
- 自监督学习 (Self-supervised Learning)

---

![bg 80%](./resources/leader-board.png)

---

#  总结与结论

## 核心发现

<div style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white; padding: 1.5em; border-radius: 10px; margin: 1em 0;">

###  深度学习展现显著优越性

- 即使是**简单CNN**也超越了精心优化的传统方法
- CNN自动学习**层级化特征**，避免繁琐的手工设计
- 端到端训练实现**全局最优**

</div>

###  这也正是计算机视觉领域向深度学习**全面转移**的根本原因

---

<!-- _class: lead -->

# 感谢聆听！

## Q & A


### 小组分工

**江煜**(组长)：项目第二部分(HOG特征提取与可视化)和报告汇总; PPT制作与汇总
**朱洋微**：项目第三部分(SVM分类器训练); 参与PPT制作
**高健峰**：项目第四部分(CNN训练); 参与PPT制作

