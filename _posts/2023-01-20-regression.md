---
title:  【机器学习】回归：线性回归与逻辑回归
categories:
- MachineLearning
tags:
- ComputerScience 
- MachineLearning 
---
线性回归与逻辑回归教程。


---
# 线性回归（Linear Regression）
回归，指研究一组随机变量 Yn 和另一组 Xn 变量之间关系的统计分析方法。
### 线性形式
假如自变量X与因变量Y的关系为线性关系，或近似为线性关系，可以使用线性回归。
- **一元线性回归**： 形如 $$ Y=\theta_0 + \theta_1 X$$
- **多元线性回归**： 形如 $$ Y=\theta_0 + \theta_1 X_1 + \theta_2 X_2 + ... + \theta_n X_n$$

在X中补充X0=1(全部X0都是1),$$ Y=\theta_0 + \theta_1 X_1 + \theta_2 X_2 + ... + \theta_n X_n=\theta_0 X_0 + \theta_1 X_1 + \theta_2 X_2 + ... + \theta_n X_n$$
若将theta,X,Y看做列向量,显然Y可表示为$$ Y=X^T \theta$$

### 线性回归预测
显然现实中得到的样本点不可能完全是线性的，我们想要的是让样本点尽可能均匀分布在直线两侧。因此我们得到的直线，与真实值必然有误差，记为e.
- 误差：预测值与真实值之差，$$ e = Y_{pred} - Y_{true}$$
- 所有点的误差e可看做列向量，因此线性回归方程带误差项的形式为$$ Y=X^T \theta + e$$
- 我们为了让预测更准确，一个比较直观的想法是让所有点都离直线都尽可能的近，这就是最小二乘法。
- 但统计学能给出更普遍的解释——极大似然估计。

#### 极大似然估计
我们认为：
- 对于所有的样本点，预测模型的准确度应该和选各个不同的点的先后顺序无关的。
- 因此各个样本点的预测的误差应该满足 分布为**高斯分布**的**独立同分布**。

极大似然估计的根本思想：
- 对于选出的样本点，应该是最可能被选出来的。
- 因此选出来的样本点，最能准确的体现整体的特征。
- 即对于所求参数而言，选取对应的样本点的可能性最大。
  
我们选出某个的样本点的概率，$$P(e_i) = \frac{1}{\sqrt{2\pi}\sigma} e^{-\frac{(y_i-\theta ^T x_i)^2} {2 \sigma^2}}$$
选出指定的所有样本点的概率，即为极大似然函数，$$L=P_1 P_2...P_n$$

显然连成的形式是很难计算的。然而我们要求的为L最大时对应的参数值，我们可以取对数简化计算。

- 因为ln(x)函数的是单调递增的,取对数不会影响L的单调性，也就不会影响L的极值点。
- $$ln(L) = \sum_{i=0}^{n}{}ln(\frac{1}{\sqrt{2\pi}\sigma} e^{-\frac{(y_i-\theta ^T x_i)^2} {2 \sigma^2}})$$
- $$ln(L) = nln(\frac{1}{\sqrt{2\pi}\sigma}) -\frac{1}{2\sigma^2}\sum_{i=0}^{n}(y_i-\theta^T x_i)^2$$
  
- 第一项是正值，显然平方项越小L越大。
- 设$$J(\theta)=\frac{1}{2}\sum_{i=0}^{n}(Y_i-\theta^T X_i)^2 = 1/2(X\theta-Y)^T (X\theta-Y)$$
- $$ \frac{\partial J(\theta)}{\partial \theta} = X^T X\theta-X^T Y=0$$
- $$\theta= (X^T X)^{-1}X^T Y$$

```python
theta = np.inv(X.T .dot(Y)).dot(X.T).dot(Y)
```

#### 梯度下降法
虽然由极大似然估计或者最小二乘法已经解决出了参数，但假如逆矩阵不存在，就无法求解。所以我们提出梯度下降法（迭代法）。
- 梯度下降法:求梯度，向梯度相反的方向移动。(效果最好，但最慢)
- $$ \theta_{k+1} =\theta_{k} - \nabla{\theta} \cdot h$$
  - 批量梯度下降：对所有参数求梯度，向梯度相反的方向移动。
    - $$ \nabla{\theta} = \frac{2}{m}X^T(X\theta -y)  $$
    ```python
    for i in range(iter_n)
        gradients = 2/m*X.T.dot(X.dot(theta)-y)
        theta = theta - gradients*h
    ```
  - 随机梯度下降：随机选一个参数求梯度，向梯度相反的方向移动。(效果最差)
  - 小批量梯度下降：对一部分参数求梯度，向梯度相反的方向移动。(效果居中，最常用)

- 每次移动的距离为步长h（学习率）或eta。
  - 步长偏小：收敛速度慢，迭代次数多；
  - 步长偏大：不收敛，学习效果差；
  - 因此通常尽量选择小步长，除非慢到受不了！ 
  
- 小批量梯度下每次选择的变量个数为batch,根据硬件和要求合理选择。
  - batch越大速度越慢，精度相对高
  - batch越小速度越快，精度相对低 
  - 因此通常尽量选择大batch，除非慢到受不了！ 

- epoch:每遍历一轮样本称为一个epoch.

# 多项式回归
- 多项式回归本质也是线性回归，实质是将X中各项组合后产生高阶项(例如2阶X1和X2，产生X1^2, X1X2, X2^2),对所有生成的项做多元线性回归。
- 核心算法与线性回归相同。
- 多项式复杂度degree：并非越高越好，过高容易过拟合。

# 正则化（防止过拟合）
为了减轻过拟合，可以在loss（损失函数）添加正则化项（惩罚项）。
本质是将无偏估计变为有偏估计，虽然有了bias,但是降低了方差。
- 例如 $$ \frac{\lambda}{n}\sum_{i=0}^{n}|\theta_i| $$ 
$$ \frac{\lambda}{2n}\sum_{i=0}^{n}\theta_i^2 $$
- lambda为调节参数（tuning parameter），为0时就是无惩罚项的最小二乘法。
- 显然某些系数越大，产生的惩罚项也越大，导致损失函数增大；
- 为减小损失函数，回归将使各个系数的差减小，因此可以压缩系数的范围。
### lasso回归 L1
- 正则化项 $$ \frac{\lambda}{n}\sum_{i=0}^{n}|\theta_i| $$
- 调节参数够大时，某些系数会被压至0。
- 因此，lasso具有变量选择的效果，比岭回归模型更具有解释性。lasso是个稀疏模型（sparse model）。
### 岭回归（Ridge回归）L2
- 正则化项 $$ \frac{\lambda}{2n}\sum_{i=0}^{n} \theta_i^2 $$
- 岭回归本质是将最小二乘的无偏估计变为有偏估计，虽然有了bias,但是降低了方差。


---
# 逻辑回归（Logistic Regression）
逻辑回归虽然叫回归，但实际是一个分类算法(主要是二分类)！
- 决策边界：既可以线性，也可以非线性。

### Sigmoid 函数
- $$ g(z) = \frac{1}{1+e^{-z}} $$
- 显然定义域z覆盖全体实数R，值域[0,1].
- 相当于把实数数轴压缩到[0,1]范围（可看做概率）。
- 将线性回归的预测值映射到Sigmoid函数，相当于将预测值映射为概率。

### 预测函数
- $$ Y=\theta_0 + \theta_1 X_1 + \theta_2 X_2 + ... + \theta_n X_n =\theta^T X $$
- $$ h_\theta (x) = g(Y)=g(\theta^T X) $$
- $$ h_\theta (x) = \frac{1}{1+e^{-\theta^T X}} $$
- 对于二分类，y=0或1，$$p(y=1)=h_\theta (x),p(y=0)=1-h_\theta (x)$$  可整合为 $$ p(y)=(h_\theta (x))^y(1-h_\theta (x))^{1-y} $$
- 似然函数$$L=P_1 P_2 ...P_n$$
- 由于对数函数和sigmoid函数都单调，可以取对数似然：
$$l(\theta) = ln(L)=\sum_{i=0}^{n}(y_i ln(h_\theta(x_i))+(1-y_i) ln(1-h_\theta(x_i)))  $$ 
- 此时梯度为正，可变换使其成为梯度下降：$$  J(\theta)=-l(\theta)/n  $$
- $$ \frac{\partial J(\theta)}{\partial \theta_i} = \frac{1}{n} \sum_{j=0}^{n} (h_\theta(x_j)-y_j)x_i$$

### 多分类
- 为了扩大样本间的差异，可以用exp做变换：
- softmax 概率 $$p_k=\frac{e^{s_k(x)}}{\sum_{j=1}^{K}  e^{s_j(x)}}  $$
- 损失函数（交叉熵）$$J(\Theta)=- \frac{1}{m}\sum_{i=1}^{m} \sum_{k=1}^{K}y_{k,i} ln(p_{k,i})  $$


---