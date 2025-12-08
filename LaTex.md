## 基础语法

### 1. 公式环境

- **行内公式**: 使用 `$公式$` 或 `\(公式\)`
- **独立公式**: 使用 `$$公式$$` 或 `\[公式\]`
- **编号公式**: 使用 `equation` 环境

### 2. 常用数学符号

#### 上下标

latex

```
x^2, y_1, x^{2y}, z_{i,j}
```



#### 分数

latex

```
\frac{a}{b}, \dfrac{a}{b} (显示更大)
```



#### 根号

latex

```
\sqrt{x}, \sqrt[n]{x}
```



#### 希腊字母

latex

```
\alpha, \beta, \gamma, \Gamma, \Delta
```



#### 运算符

latex

```
\sum, \prod, \int, \lim, \sin, \cos, \log
```



## 完整示例

latex

```latex
\documentclass{article}
\usepackage{amsmath} % 数学包

\begin{document}

行内公式示例: $E = mc^2$ 是质能方程。

独立公式:
\[
f(x) = \frac{1}{\sqrt{2\pi\sigma^2}} e^{-\frac{(x-\mu)^2}{2\sigma^2}}
\]

编号公式:
\begin{equation}
\int_{-\infty}^{\infty} e^{-x^2} dx = \sqrt{\pi}
\end{equation}

矩阵:
\[
\mathbf{A} = \begin{pmatrix}
a_{11} & a_{12} \\
a_{21} & a_{22}
\end{pmatrix}
\]

多行公式:
\begin{align}
(a+b)^2 &= a^2 + 2ab + b^2 \\
(a-b)^2 &= a^2 - 2ab + b^2
\end{align}

\end{document}
```