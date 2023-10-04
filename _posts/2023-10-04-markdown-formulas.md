---
title:  【Markdown】Markdown 公式快速入门
categories:
- Markdown
tags:
- Markdown 
---

Markdown 用了一段时间了，发现Markdown写公式非常方便，也比较常用。

因此想写一个Markdown 公式的快速入门教程，也作为一个自用的备忘录。


---
# Markdown 公式快速入门

### 公式格式

#### 行内公式
markdown 用 `$ $` 来表示公式，如 `$y=x$` 会显示成 $y=x$ .

#### 行间公式
markdown 用 `$$ $$` 来表示公式，如 `$$y=x$$` 会显示成 $$y=x$$ 行间公式会居中放大显示。

#### 分割与组合
- 空格 ` ` 会分割不同元素， 不会渲染
  - 如 `$\sin x$` 会显示成 $\sin x$, 而`$\sinx$`非法;
  - 如 `$\sin x        y$` 会显示成 $\sin x        y$, 空格不显示;
- 大括号 `{}` 的内容将被视作一个整体， 不会渲染
  - 如 `$e^{x+y}$` 会显示成 $e^{x+y}$;
  - 相应的 `$e^x+y$` 会显示成 $e^x+y$;

#### 转义字符
- 转义字符通过在特殊字符前面加上反斜杠`\`来实现。
- 渲染符号：
  - 常用函数，如`$\sin x$` 显示为 $\sin x$
  - 常用符号，如`$x \leq y$` 显示为 $x \leq y$
  - ......
- 直接显示特殊字符：
  - 空格作为公式解析的分割符，渲染时默认忽略空格
    - 如 `$a b$` 显示为 $a b$， `$a\ b$` 显示为 $a\ b$
  - 大括号作为公式解析的组合符，渲染时默认忽略大括号
    - 如 `$\{ x \}$` 显示为 $\{ x \}$， `${ x }$` 显示为 ${ x }$
  - ......

#### 上下标
- 上标：`^` 
  - 如 `$e^{x}$` 会显示成 $e^{x}$
- 下标：`_` 
  - 如 `$x_{n^2}$` 会显示成 $x_{n^2}$

#### 矢量
- 单字母矢量：`\vec`
  - 如 `$\vec a$` 显示为 $\vec a$

- 多字母矢量：`\overrightarrow`
  - 如 `$\overrightarrow{xy}$` 显示为 $\overrightarrow{xy}$
​
#### 括号
- 不同于大括号 `{}` , `()` , `[]` 和 `|` 表示符号本身。
- 用 `\left` 和 `\right` 显示自适应大小的的括号或分隔符。 对所有括号，括号大小和邻近的公式适应。
  - 如 `(\frac{x}{y})` 显示为 $(\frac{x}{y})$
  - 相应的 `\left(\frac{x}{y}\right)` 显示为 $\left(\frac{x}{y}\right)$
- 更多括号见后表。

#### 常用符号
常用初等的数学符号：
- 分式: `\frac`
  - 如`$\frac{a}{b}$` 显示为 $\frac{a}{b}$
- 根式：`\sqrt`
  - 如`$\sqrt[x]{y}$` 显示为 $\sqrt[x]{y}$
- 累加：`\sum`
  - 如`$\sum_{i=0}^{n}$` 显示为 $\sum_{i=0}^{n}$
- 累乘：`\prod`
  - 如`$\prod_{i=0}^{n}$` 显示为 $\prod_{i=0}^{n}$

常用微积分的数学符号：
- 极限：`\lim`
  - 如`$\lim_{x\to 0}$` 显示为 $\lim_{x\to 0}$
- 导数: `\prime` 显示为 $\prime$
- 微分算子: `\mathrm{d}` 显示为 $\mathrm{d}$
- 偏微分算子：`\partial` 显示为 $\partial$
- 积分：`\int`
  - 如`$\int_{a}^{b}$` 显示为 $\int_{a}^{b}$
  - 二重积分 `\iint`  $\iint$
  - 三重积分 `\iiint`  $\iiint$
  - 曲线积分 `\oint`  $\oint$

更多符号见后表。

#### 多行公式
markdown 的多行公式和 latex 语法基本一致。
- 起始标记 `\begin{}`
- 结束标记 `\end{}`
- 行结束标记 `\\`
- 指定对齐点 `&`

#### 方程组
方程组是一种多行公式。
- 方程组：在起始 `\begin{}` ，结束 `end{}` 标记的`{}`中填入`{cases}`
  - 例：
  ```
  $$
  \begin{cases}
  x+y=a \\
  x-y=b \\
  \end{cases}
  $$
  ```
  $$
  \begin{cases}
  x+y=a \\
  x-y=b \\
  \end{cases}
  $$



#### 矩阵
矩阵是一种多行公式。
- 矩阵：在起始 `\begin{}` ，结束 `end{}` 标记的`{}`中填入
  - `matrix` ：无边框
  - `pmatrix` ：小括号边框
  - `bmatrix` ：中括号边框
  - `Bmatrix` ：大括号边框
  - `vmatrix` ：单竖线边框
  - `Vmatrix` ：双竖线边框
  - 例：(使用`matrix`无边框)
  ```
  $$\begin{matrix}
  1&0&0\\
  0&1&0\\
  0&0&1\\
  \end{matrix}$$
  ```
  $$\begin{matrix}
  1&0&0\\
  0&1&0\\
  0&0&1\\
  \end{matrix}$$


- 省略元素:
  - 横省略号：`\cdots`
  - 竖省略号：`\vdots`
  - 斜省略号：`\ddots`
  - 例：
  ```
  $$\begin{vmatrix}
  {a_{11}}&{a_{12}}&{\cdots}&{a_{1n}}\\
  {a_{21}}&{a_{22}}&{\cdots}&{a_{2n}}\\
  {\vdots}&{\vdots}&{\ddots}&{\vdots}\\
  {a_{m1}}&{a_{m2}}&{\cdots}&{a_{mn}}\\
  \end{vmatrix}$$
  ```
  $$\begin{vmatrix}
  {a_{11}}&{a_{12}}&{\cdots}&{a_{1n}}\\
  {a_{21}}&{a_{22}}&{\cdots}&{a_{2n}}\\
  {\vdots}&{\vdots}&{\ddots}&{\vdots}\\
  {a_{m1}}&{a_{m2}}&{\cdots}&{a_{mn}}\\
  \end{vmatrix}$$


### 希腊字母表
谈到数学公式，少不了用到希腊字母。

以下是希腊字母表：

 | 小写输入 | 小写显示 | 大写输入 | 大写显示 | 
 | :----: | :----: | :----: | :----: |  
 | `\alpha` | $\alpha$ | `\Alpha` | $\Alpha$ | 
 | `\beta` | $\beta$ | `\Beta` | $\Beta$ | 
 | `\gamma` | $\gamma$ | `\Gamma` | $\Gamma$ | 
 | `\delta` | $\delta$ | `\Delta` | $\Delta$ | 
 | `\epsilon` | $\epsilon$ | `\Epsilon` | $\Epsilon$ | 
 | `\zeta` | $\zeta$ | `\Zeta` | $\Zeta$ | 
 | `\eta` | $\eta$ | `\Eta` | $\Eta$ | 
 | `\theta` | $\theta$ | `\Theta` | $\Theta$ | 
 | `\iota` | $\iota$ | `\Iota` | $\Iota$ | 
 | `\kappa` | $\kappa$ | `\Kappa` |$\Kappa$ | 
 | `\lambda` | $\lambda$ | `\Lambda` | $\Lambda$ | 
 | `\nu` | $\nu$ | `\Nu` | $\Nu$ | 
 | `\mu` | $\mu$ | `\Mu` | $\Mu$ | 
 | `\xi` | $\xi$ | `\Xi` | $\Xi$ | 
 | `\pi` | $\pi$ | `\Pi` | $\Pi$ | 
 | `\rho` | $\rho$ | `\Rho` | $\Rho$ | 
 | `\sigma` | $\sigma$ | `\Sigma` | $\Sigma$ | 
 | `\tau` | $\tau$ | `\Tau` | $\Tau$ | 
 | `\upsilon` | $\upsilon$ | `\Upsilon` | $\Upsilon$ | 
 | `\phi` | $\phi$ | `\Phi` | $\Phi$ | 
 | `\chi` | $\chi$ | `\Chi` | $\Chi$ | 
 | `\psi` | $\psi$ | `\Psi`| $\Psi$ | 
 | `\omega` | $\omega$ | `\Omega` | $\Omega$ | 

实际上希腊字母变换有如下规则：
- 大写希腊字母，是 小写希腊字母 首字母大写
  -   `\alpha` | $\alpha$ 
  -   `\Alpha` | $\Alpha$ 
- 斜体希腊字母，是 希腊字母 前加var前缀
  -   `\Gamma` | $\Gamma$ 
  -   `\varGamma` | $\varGamma$ 



### 常见运算符号表

#### 基础/代数运算

| 符号输入 | 符号显示 | 
| :----: | :----: | 
| `\pm`  | $\pm$   | 
| `\times`  | $\times$  |  
| `\div` | $\div$   | 
| `\mid` | $\mid$ | 
| `\cdot` | $\cdot$ |
| `\circ` | $\circ$ |
| `\ast` | $\ast$ |
| `\bigodot` | $\bigodot$ |
| `\bigotimes` | $\bigotimes$ |
| `\bigoplus` | $\bigoplus$ |
| `\leq` | $\leq$ |
| `\geq` | $\geq$ |
| `\neq` | $\neq$ |
| `\approx` | $\approx$ |
| `\equiv` | $\equiv$ |
| `\sum` | $\sum$ |
| `\prod` | $\prod$ |
| `\coprod` | $\coprod$ |
| `\log` | $\log$ |
| `\lg` | $\lg$ |
| `\ln` | $\ln$ |


#### 比较运算
| 符号输入 | 符号显示 |
| ---- | ---- |
| `\leq` |  $x \leq y$ |
| `\geq` |  $x \geq y$ |
| `\nleq` |  $x \nleq y$ |
| `\not \leq` | $x \not \leq y$ |
| `\ngeq` | $x \ngeq y$ |
| `\not \geq` | $x \not \geq y$ |
| `\neq` | $x \neq y$ |
| `\approx` | $x \approx y$ |
| `\equiv` | $x \equiv y$ |


#### 集合运算

| 符号输入 | 符号显示 |
| :---: | :---: |
| `\emptyset` | $\emptyset$ |
| `\in` | $\in$ |
| `\notin` | $\notin$ |
| `\subset` | $\subset$ |
| `\supset` | $\supset$ |
| `\subseteq` | $\subseteq$ |
| `\supseteq` | $\supseteq$ |
| `\bigcap` | $\bigcap$ |
| `\bigcup` | $\bigcup$ |
| `\bigvee` | $\bigvee$ |
| `\bigwedge` | $\bigwedge$ |
| `\biguplus` | $\biguplus$ |
| `\bigsqcup` | $\bigsqcup$ |

#### 三角运算

| 符号输入 | 符号显示 |
| :---: | :---: |
| `\bot` | $\bot$ |
| `\angle` | $\angle$ |
| `90^\circ` | $90^\circ$ |
| `\sin` | $\sin$ |
| `\cos` | $\cos$ |
| `\tan` | $\tan$ |
| `\cot` | $\cot$ |
| `\sec` | $\sec$ |
| `\csc` | $\csc$ |

#### 微积分运算

| 符号输入 | 符号显示 |
| :---: | :---: |
| `\infty` | $\infty$ |
| `\lim` | $\lim$ |
| `\mathrm{d}` | $\mathrm{d}$ |
| `\partial` | $\partial$ |
| `\prime` | $\prime$ |
| `\nabla` | $\nabla$ |
| `\int` | $\int$ |
| `\iint` | $\iint$ |
| `\iiint` | $\iiint$ |
| `\oint` | $\oint$ |

#### 逻辑运算

| 符号输入 | 符号显示 |
| :---: | :---: |
| `\because` | $\because$ |
| `\therefore` | $\therefore$ |
| `\forall` | $\forall$ |
| `\exists` | $\exists$ |
| `\not=` | $\not=$ |
| `\not>` | $\not>$ |
| `\not\subset` | $\not\subset$ |

#### 箭头符号

| 符号输入 | 符号显示 |
| :---: | :---: |
| `\uparrow` | $\uparrow$ |
| `\downarrow` | $\downarrow$ |
| `\Uparrow` | $\Uparrow$ |
| `\Downarrow` | $\Downarrow$ |
| `\rightarrow` | $\rightarrow$ |
| `\leftarrow` | $\leftarrow$ |
| `\Rightarrow` | $\Rightarrow$ |
| `\Leftarrow` | $\Leftarrow$ |
| `\longrightarrow` | $\longrightarrow$ |
| `\longleftarrow` | $\longleftarrow$ |
| `\Longrightarrow` | $\Longrightarrow$ |
| `\Longleftarrow` | $\Longleftarrow$ |

#### 特殊括号
| 符号输入 | 符号显示 |
| ---- | ---- |
| `\langle \rangle` | $\langle a+b \rangle$ |
| `\lceil \rceil` | $\lceil a+b \rceil$ |
| `\lfloor \rfloor` | $\lfloor a+b \rfloor$ |
| `\lbrace \rbrace` | $\lbrace a+b \rbrace$ |
| `\overline` | $\overline{a+b+c+d}$ |
| `\underline` | $\underline{a+b+c+d}$ |
| `\overbrace` | $\overbrace{a+\underbrace{b+c}_{1.0}+d}^{2.0}$ |
| `\underbrace` | $\underbrace{a+d}_3$ |

