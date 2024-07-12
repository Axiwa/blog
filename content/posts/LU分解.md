---
title: LU分解、Chelosky分解、QR分解
categories: Math
date: 2022-05-30
katex: true
markup: pandoc
---
花式解决$Ax = b$
<!--more-->

[LU Decomposition for Solving Linear Equations](https://courses.engr.illinois.edu/cs357/sp2020)

[线性代数笔记10——矩阵的LU分解](https://www.cnblogs.com/bigmonkey/p/9555710.html)

[线性方程组求解与LU分解](https://www.math.pku.edu.cn/teachers/lidf/docs/statcomp/html/_statcompbook/matrix-solve.html)

## Gauss Elimination
Suppose we have a set of linear equations:

$$
\begin{bmatrix}
a_{11} & a_{12} & .. & a_{1n} \\
a_{21} & a_{22} & .. & .. \\
.. &  &  &  \\
 &  &  & a_{nn} \\
\end{bmatrix}\begin{bmatrix}
 x_1\\
 x_2\\
 ..\\
x_n
\end{bmatrix}
= \begin{bmatrix}
 b_1\\
 b_2\\
 ..\\
b_n
\end{bmatrix}
$$

The most brute force way is to eliminate one variable each row from top to bottom, and then plug in the sovled variable to previous ones from bottom to top. It is something like this:

Step 1: Let each row except 1st do not contain $x_1$. Then eliminate $x_2$ in each row except 1st and 2nd

$$
\begin{bmatrix}
a_{11} & a_{12} & .. & a_{1n} \\
0 & a_{22}-a_{12}\frac{a_{21}}{a_{11}} & .. & .. \\
.. &  &  &  \\
0 &  &  & a_{nn}-a_{1n}\frac{a_{n1}}{a_{11}} \\
\end{bmatrix}\begin{bmatrix}
 x_1\\
 x_2\\
 ..\\
x_n
\end{bmatrix}
= \begin{bmatrix}
 b_1\\
 b_2-b_1\frac{a_{21}}{a_{11}}\\
 ..\\
b_n-b_1\frac{a_{n1}}{a_{11}}
\end{bmatrix}
$$

$$
\begin{bmatrix}
a_{11} & a_{12} & .. & a_{1n} \\
0 & a_{22}-a_{12}\frac{a_{21}}{a_{11}} & .. & .. \\
0 & 0 & a_{33}-a_{13}\frac{a_{31}}{a_{11}}-(a_{23}-a_{13}\frac{a_{21}}{a_{11}})\frac{a_{32}-a_{12}\frac{a_{31}}{a_{11}}}{a_{22}-a_{12}\frac{a_{21}}{a_{11}}} &  \\
.. & .. &  &  \\
0 & 0 &  & a_{nn}-a_{1n}\frac{a_{n1}}{a_{11}}-(a_{2n}-a_{1n}\frac{a_{21}}{a_{11}})\frac{a_{n2}-a_{12}\frac{a_{n1}}{a_{11}}}{a_{22}-a_{12}\frac{a_{21}}{a_{11}}} \\
\end{bmatrix}\begin{bmatrix}
 x_1\\
 x_2\\
 ..\\
x_n
\end{bmatrix}
$$

$$
= \begin{bmatrix}
 b_1\\
 b_2-b_1\frac{a_{21}}{a_{11}}\\
 ..\\
b_n-b_1\frac{a_{n1}}{a_{11}}-(b_2-b_1\frac{a_{21}}{a_{11}})\frac{a_{n2}-a_{12}\frac{a_{n1}}{a_{11}}}{a_{22}-a_{12}\frac{a_{21}}{a_{11}}}
\end{bmatrix}
$$

If every time the denominator is not zero, finally we will get an upper matrix
$$
\begin{bmatrix}
u_{11} & u_{12} & .. & u_{1n} \\
0 & u_{22} & .. & .. \\
.. &  &  &  \\
0 &  &  & u_{nn} \\
\end{bmatrix}\begin{bmatrix}
 x_1\\
 x_2\\
 ..\\
x_n
\end{bmatrix}
= \begin{bmatrix}
 \widetilde{b_1}\\
 \widetilde{b_2}\\
 ..\\
\widetilde{b_n}
\end{bmatrix}
$$

This process can be seen as

$$
\begin{aligned}
E_1Ax = E_1b \\
E_2E_1Ax = E_2E_1b\\
\cdots \\
E_n..E_1Ax = E_n...E_1b = Ux\\
(E_n...E_1)^{-1}Ux = b \\
LUx = b
\end{aligned}
$$

where $E_i$

$$
E_1 = \begin{bmatrix}
1 & 0 & ...&0\\
-\frac{a_{21}}{a_{11}} & 1 & ...&0 \\
-\frac{a_{31}}{a_{11}}&0&1...&0\\
...\\
-\frac{a_{n1}}{a_{11}}&..&&1
\end{bmatrix}
$$

$$
E_2 = \begin{bmatrix}
1 & 0 & ...&0\\
0 & 1 & ...&0 \\
0& -\frac{a_{32}-a_{12} \frac{a_{31}}{a_{11}}}{a_{22}-a_{12} \frac{a_{21}}{a_{11}}}  &1...&0\\
...\\
0&-\frac{a_{n 2}-a_{12} \frac{a_{n 1}}{a_{11}}}{a_{22}-a_{12} \frac{a_{21}}{a_{11}}}&&1
\end{bmatrix}
$$

... 

are lower triangular matrix that eliminates the element in one column. The inverse of a lower triangular matrix is still lower triangular matrix, so A can be decomposed as $A = LU$ where $L$ is a lower triangular matrix and $U$ is an upper triangular matrix. However, what if in this process an element on the diagonal (for example at row i) is zero? We are sure that there would be a row (row > i) that has non-zero at col i (otherwise A is not invertable). We then exchange this two rows using a perturbation matrix $P_i$ that reorders the rows of A. 

$$
...E_iP(i, s_i)...A = ...E_iP(i, s_i)...b
$$


For every row, we can exchange two rows using $P_i$ such that the current row i has the largest absolute value of i th column element. This is called "Pivoting". It can be proved([https://link.zhihu.com/?target=https%3A//www.amazon.com/Numerical-Linear-Algebra-Lloyd-Trefethen/dp/0898713617](https://link.zhihu.com/?target=https%3A//www.amazon.com/Numerical-Linear-Algebra-Lloyd-Trefethen/dp/0898713617)) that

$$
PA = LU
$$

where 

$$
P=P\left(n-1, s_{n-1}\right) \ldots P\left(2, s_{2}\right) P\left(1, s_{1}\right)
$$

The first step uses $2n(n-1)$ operations (multiplication + addition = 2 floating-point operations), the second uses $2(n-1)(n-2)$... so Step 1 uses ~ $O(2n^3/3)$

Step 2: 

Now we can solve $x_n$ using $u_{nn}x_n = \widetilde{b_n}$. Then we plug $x_n$ to the second last row we can solve $x_{n-1}$, and so on... by back substitution we can solve $Ub = \widetilde{b}$ in $O(n^2)$

## LU decomposition

We have seen that an invertable matrix A can be decomposed to $(P^{T})LU$. How can we get $P, L, U$?

### By Gauss elimination

From Gauss elimination we know 
$$
L = E^{-1}_1E^{-1}_2...E^{-1}_n
$$

And we can get $L$ by multiplication and inverse of these lower triangular matrices. Note that for lower triangular matrices $E_i$ with diagnal element 1, the inverse and multiplication is quite simple, and we can build $L$ with $E_i$([https://zhuanlan.zhihu.com/p/84210687](https://zhuanlan.zhihu.com/p/84210687)). Thus the complexity of LU decomposition and solve this linear equation is

1) LU decomposition: $2n^2/3$

2) Solve $Ly = b$: $n^2$

3) Solve $Ux = y$: $n^2$

We use LU decomposition but not Gauss elimination because we don't want to bother to modify $b$ every time, especially when we have many different $b$


### By recursion
???

## LUP decomposition
Every time we left-multiply a perturbation matrix $P$ and get 

$$
E_{n-1} P_{n-1} \ldots E_{2} P_{2} E_{1} P_{1} A=U
$$
=>
$$
PA = LU,\text { where } P=\prod_{i=1}^{n-1} P_{i}
$$

Though swapping two rows each time, the complexity is still $O(2n^3/3)$ (我觉得好像是这样...最多多出来$O(n^2)$次操作...)

### By recursion



