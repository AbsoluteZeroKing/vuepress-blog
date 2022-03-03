---
title: 高性能计算之稳态热传导问题的并行解决方案
date: 2020-11-29 00:27:17.0
updated: 2021-05-24 13:55:42.799
url: https://leesdog.space/archives/paralleliterativemethodsforlinearsystems
summary: 此报告为《高性能计算》这门课的课程报告，这一次的课程最终目标是使用并行的策略利用Jacobi迭代法，Gauss-Seidel 迭代法以及 SOR methods去解决二维稳态热传导问题
categories: 
tags: 
- 高性能计算
- 并发计算
- Fortran
- 雅可比
- Jacobi迭代法
- Gauss-Seidel 法
- SOR法
---

# Parallel Iterative Methods for Linear Systems

[TOC]

## 1. Introduction

​		传热是一门试图预测由于温度差而在材料之间发生的能量传递的科学。热力学将这种能量传递定义为热。传热学不仅试图解释如何传递热能，而且还试图预测在某些特定条件下交换的速率。传热速率是分析的预期目标这一事实指出了传热和热力学之间的差异。热力学涉及平衡系统；它可用于预测将系统从一种平衡状态转换为另一种平衡状态所需的能量。

Heat transfer is the science that seeks to predict the energy transfer that may take place between material bodies as a result of a temperature difference. Thermodynamics teaches that this energy transfer is defined as heat. The science of heat transfer seeks not merely to explain how heat energy may be transferred, but also to predict the rate at which the exchange will take place under certain specified conditions. The fact that a heat-transfer rate is the desired objective of an analysis points out the difference between heat transfer and thermodynamics. Thermodynamics deals with systems in equilibrium; it may be used to predict the amount of energy required to change a system from one equilibrium state to another;

​		传热学中的热传导问题是材料成形模拟中经常需要求解的问题。热传导问题求解得到的是温度场。几乎所有材料成形模拟的过程都与温度场有关。热传导问题的分类有多种。按**是否与时间有关**，可分为**稳态问题**和**瞬态问题**;按**几何模型的维度**，可分为**二维**和**三维**;按**求解方法**，又可分为**Jacobi迭代法**，**Gauss-Seidel 法**等。热传导问题的规模一般较大。当温度网格规模为千万网格时，求解热传导问题需要80~200分钟，如果网格规模达到一亿，则需要5到10个小时。而客户的理想需求则分别为15分钟以内和60分钟以内。在存储需求上，千万网格的数据量大约是10GB，而一亿网格的数据量将超过100GB。对于千万网格的问题规模，网格数据可以存储在大型服务器的内存中，但对于一亿网格的数据量则必须存储在硬盘中。根据以上描述的问题规模，热传导问题的计算瓶颈在于计算周期和数据存储。而并行计算，有望解决计算周期较长的问题。如果能对热传导问题的求解过程进行有效的并行化，则可以显著缩短材料成形模拟的整体周期。所以对热传导问题的并行化求解具有重要意义。

​		The heat conduction problem in heat transfer is a problem that often needs to be solved in material forming simulation. The solution to the heat conduction problem is the temperature field. Almost all material forming simulation processes are related to the temperature field. There are many categories of heat conduction problems. According to whether it is related to time, it can be divided into steady-state problems and transient problems; according to the dimension of the geometric model, it can be divided into two-dimensional and three-dimensional; according to the solution method, it can be divided into Jacobi iterative method, Gauss-Seidel  method, etc. The scale of heat conduction problems is generally larger. When the temperature grid scale is tens of millions of grids, it takes 80 to 200 minutes to solve the heat conduction problem. If the grid scale reaches 100 million, it takes 5 to 10 hours. The ideal needs of customers are within 15 minutes and within 60 minutes. In terms of storage requirements, the data volume of tens of millions of grids is about 10GB, and the data volume of 100 million grids will exceed 100GB. For the problem scale of tens of millions of grids, grid data can be stored in the memory of large servers, but for the amount of data of 100 million grids, it must be stored on the hard disk. According to the scale of the problem described above, the calculation bottleneck of the heat conduction problem lies in the calculation cycle and data storage. Parallel computing is expected to solve the problem of long computing cycle. If the process of solving the heat conduction problem can be effectively parallelized, the overall cycle of material forming simulation can be significantly shortened. Therefore, it is of great significance to solve the heat conduction problem in parallel.

​		此报告为《高性能计算》这门课的课程报告，这一次的课程最终目标是**使用并行的策略利用Jacobi迭代法，Gauss-Seidel 迭代法以及 SOR methods去解决二维稳态热传导问题**。

​		This paper is the course report of the "High Performance Computing". The ultimate goal of this course is to use the parallel strategy to solve the two-dimensional steady-state heat conduction problem by using Jacobi iteration method,Gauss-Seidel  method and SOR methods.

## 2. Problem definition

​		在具有N2个未知数的N×N网格上离散化：假设温度在每个点固定（已知）边界。在内部点，稳态值是（大约）4个相邻值的平均值。你现在需要用雅可比，高斯赛德尔和SOR方法并行地解决上述条件下的稳态热传导问题

​		Discretize on an N × N grid with N2 unknowns:Assume temperature is fixed (and known) at each point on boundary.At interior points, the steady state value is (approximately) the average of the 4 neighboring values.You need to use parallel “Jacobi, Gauss-Seidel and SOR methods” to solve “steady state heat conduct” problem under the above conditions.

## 3. Proposed Methods

###  3.1 Jacobi iteration method

----

#### 3.1.1 Definition

​		Jacobi方法（或Jacobi迭代方法）是一种用于确定对角线优势线性方程组解的算法。 求解每个对角线元素，并插入一个近似值。然后迭代该过程，直到收敛为止。 该算法是矩阵对角化的Jacobi变换方法的简化版本。 该方法以Carl Gustav Jacob Jacobi的名字命名。

​		The Jacobi method (or Jacobi iterative method) is an algorithm for determining the solutions of a diagonally dominant system of linear equations. Each diagonal element is solved for, and an approximate value is plugged in. The process is then iterated until it converges. This algorithm is a stripped-down version of the Jacobi transformation method of matrix diagonalization. The method is named after Carl Gustav Jacob Jacobi.

---

#### 3.1.2 Description

使（1）式展开为n维线性方程（2）

Let
$$
A x=b
$$


be a square system of n linear equations, where:

<math xmlns="http://www.w3.org/1998/Math/MathML" display="block">
  <mi>A</mi>
  <mo>=</mo>
  <mrow data-mjx-texclass="INNER">
    <mo data-mjx-texclass="OPEN">[</mo>
    <mtable columnalign="center center center center" columnspacing="1em" rowspacing="4pt">
      <mtr>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>11</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>12</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>1</mn>
              <mi>n</mi>
            </mrow>
          </msub>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>21</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>22</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>2</mn>
              <mi>n</mi>
            </mrow>
          </msub>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mi>n</mi>
              <mn>1</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mi>n</mi>
              <mn>2</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mi>n</mi>
              <mi>n</mi>
            </mrow>
          </msub>
        </mtd>
      </mtr>
    </mtable>
    <mo data-mjx-texclass="CLOSE">]</mo>
  </mrow>
  <mo>,</mo>
  <mi>x</mi>
  <mo>=</mo>
  <mrow data-mjx-texclass="INNER">
    <mo data-mjx-texclass="OPEN">[</mo>
    <mtable columnspacing="1em" rowspacing="4pt">
      <mtr>
        <mtd>
          <msub>
            <mi>x</mi>
            <mrow>
              <mn>1</mn>
            </mrow>
          </msub>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <msub>
            <mi>x</mi>
            <mrow>
              <mn>2</mn>
            </mrow>
          </msub>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <mo>…</mo>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <msub>
            <mi>x</mi>
            <mrow>
              <mi>n</mi>
            </mrow>
          </msub>
        </mtd>
      </mtr>
    </mtable>
    <mo data-mjx-texclass="CLOSE">]</mo>
  </mrow>
  <mo>,</mo>
  <mi>b</mi>
  <mo>=</mo>
  <mrow data-mjx-texclass="INNER">
    <mo data-mjx-texclass="OPEN">[</mo>
    <mtable columnspacing="1em" rowspacing="4pt">
      <mtr>
        <mtd>
          <msub>
            <mi>b</mi>
            <mrow>
              <mn>1</mn>
            </mrow>
          </msub>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <msub>
            <mi>b</mi>
            <mrow>
              <mn>2</mn>
            </mrow>
          </msub>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <mo>…</mo>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <msub>
            <mi>b</mi>
            <mrow>
              <mi>n</mi>
            </mrow>
          </msub>
        </mtd>
      </mtr>
    </mtable>
    <mo data-mjx-texclass="CLOSE">]</mo>
  </mrow>
</math>


将A分解为对角分量D，其余部分为R

Then A can be decomposed into a diagonal component D, and the remainder R:
<math xmlns="http://www.w3.org/1998/Math/MathML" display="block">
  <mi>A</mi>
  <mo>=</mo>
  <mi>D</mi>
  <mo>+</mo>
  <mi>R</mi>
  <mo>,</mo>
  <mi>D</mi>
  <mo>=</mo>
  <mrow data-mjx-texclass="INNER">
    <mo data-mjx-texclass="OPEN">[</mo>
    <mtable columnalign="center center center center" columnspacing="1em" rowspacing="4pt">
      <mtr>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>11</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <mn>0</mn>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <mn>0</mn>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <mn>0</mn>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>22</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <mn>0</mn>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <mn>0</mn>
        </mtd>
        <mtd>
          <mn>0</mn>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mi>n</mi>
              <mi>n</mi>
            </mrow>
          </msub>
        </mtd>
      </mtr>
    </mtable>
    <mo data-mjx-texclass="CLOSE">]</mo>
  </mrow>
  <mo>,</mo>
  <mstyle scriptlevel="0">
    <mspace width="1em"></mspace>
  </mstyle>
  <mi>R</mi>
  <mo>=</mo>
  <mrow data-mjx-texclass="INNER">
    <mo data-mjx-texclass="OPEN">[</mo>
    <mtable columnalign="center center center center" columnspacing="1em" rowspacing="4pt">
      <mtr>
        <mtd>
          <mn>0</mn>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>12</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>1</mn>
              <mi>n</mi>
            </mrow>
          </msub>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>21</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <mn>0</mn>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>2</mn>
              <mi>n</mi>
            </mrow>
          </msub>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mi>n</mi>
              <mn>1</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mi>n</mi>
              <mn>2</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <mn>0</mn>
        </mtd>
      </mtr>
    </mtable>
    <mo data-mjx-texclass="CLOSE">]</mo>
  </mrow>
</math>
然后将上式通过迭代获得x的解

The solution is then obtained iteratively via
<math xmlns="http://www.w3.org/1998/Math/MathML" display="block">
  <msup>
    <mrow>
      <mi mathvariant="bold">x</mi>
    </mrow>
    <mrow>
      <mo stretchy="false">(</mo>
      <mi>k</mi>
      <mo>+</mo>
      <mn>1</mn>
      <mo stretchy="false">)</mo>
    </mrow>
  </msup>
  <mo>=</mo>
  <msup>
    <mi>D</mi>
    <mrow>
      <mo>−</mo>
      <mn>1</mn>
    </mrow>
  </msup>
  <mrow data-mjx-texclass="INNER">
    <mo data-mjx-texclass="OPEN">(</mo>
    <mrow>
      <mi mathvariant="bold">b</mi>
    </mrow>
    <mo>−</mo>
    <mi>R</mi>
    <msup>
      <mrow>
        <mi mathvariant="bold">x</mi>
      </mrow>
      <mrow>
        <mo stretchy="false">(</mo>
        <mi>k</mi>
        <mo stretchy="false">)</mo>
      </mrow>
    </msup>
    <mo data-mjx-texclass="CLOSE">)</mo>
  </mrow>
</math>


其中 $`$x^{k}$`$ 是x的第k个迭代近似解，而 $`$x^{k+1}$`$ 是x的k + 1个迭代近似解，因此, 公式展开后最终变 为(5)
where $`$x^{k}$`$ is the kth approximation or iteration of $`$x$`$ and $`$x^{k+1}$`$ is the next or $`$\mathrm{k}+1$`$ iteration of $`$\mathrm{x}$`$. The element-basedformula is thus:

```math
$$
x_{i}^{(k+1)}=\frac{1}{a_{i i}}\left(b_{i}-\sum_{j \neq i} a_{i j} x_{j}^{(k)}\right), \quad i=1,2, \ldots, n
$$
```

$`$x_{i}^{k+1}$`$ 的计算需要 $`$x^{k}$`$ 中除了自身以外的其他每个元素。 与高斯 - 塞德尔方法不同，我们不能用 $`$x_{i}^{k+1}$`$ 覆盖 $`$x_{i}^{k},$`$ 因为其余的计算将需要该值。 最小存储量是大小为n的两个向量。

The computation of $`$x_{i}^{k+1}$`$ requires each element in $`$x^{k}$`$ except itself. Unlike the Gauss-Seidel method, we can't overwrite $`$x_{i}^{k}$`$ with $`$x_{i}^{k+1},$`$ as that value will be needed by the rest of the computation. The minimum amount of storage is two vectors of size $`$n$`$.

---

#### 3.1.3 Algorithm

输入：解的初始猜值$x^0$，对角矩阵$A$，右端向量$b$，收敛准则

输出：达到收敛时的解

Input: initial guess $`$x^0$`$ to the solution, (diagonal dominant) matrix $`$A$`$, right-hand side vector  $`$b$`$,convergence criterion

Output: solution when convergence is reached 

```pseudocode 
k=0
while convergence not reached do 
	for i:=1 step until n do 
	 μ=0
	 for j :=1 step until n do
	  if j!=i then
	   μ=μ+a_ij*x_j^k
	  end
	 end
	 x_i^(k+1)=(b_i-μ)/a_ii
	end
	k=k+1
end
```

#### 3.1.4 Convergence

标准收敛条件（对于任何迭代方法）是当迭代矩阵的谱半径小于1时：

The standard convergence condition (for any iterative method) is when the spectral radius of the iteration matrix is less than 1:
<math xmlns="http://www.w3.org/1998/Math/MathML" display="block">
  <mi>ρ</mi>
  <mrow data-mjx-texclass="INNER">
    <mo data-mjx-texclass="OPEN">(</mo>
    <msup>
      <mi>D</mi>
      <mrow>
        <mo>−</mo>
        <mn>1</mn>
      </mrow>
    </msup>
    <mi>R</mi>
    <mo data-mjx-texclass="CLOSE">)</mo>
  </mrow>
  <mo>&lt;</mo>
  <mn>1</mn>
</math>


该方法收敛的充分条件是矩阵A为严格对角占优矩阵。 严格对角占优矩阵表示每行对角线项的绝对值大于其他项的绝对值之和

A sufficient (but not necessary) condition for the method to converge is that the matrix A is strictly or irreducibly diagonally dominant. Strict row diagonal dominance means that for each row, the absolute value of the diagonal term is greater than the sum of absolute values of other terms
```math
$$
\left|a_{i i}\right|>\sum_{j \neq i}\left|a_{i j}\right|
$$
```


#### 3.1.5 Implementation

```fortran
do iter=1,maxiter
        uold = u         ! old values
        dumax = 0.d0
        !$omp parallel do private(i) reduction(max : dumax)
        ! private - only be accessed by a single thread.
        do j=1,n
            do i=1,n
                u(i,j) = 0.25d0*(uold(i-1,j) + uold(i+1,j) + &
                                 uold(i,j-1) + uold(i,j+1) + h**2*f(i,j))
                dumax = max(dumax, abs(u(i,j)-uold(i,j)))
            enddo
        enddo

        if (mod(iter,nprint)==0) then
            print 203, iter, dumax
203         format("After ",i8," iterations, dumax = ",d16.6,/)
            endif
        ! check for convergence:
        if (dumax .lt. tol) exit

        enddo
```

---

### 3.2 Gauss-Seidel method

#### 3.2.1 Definition

​		高斯-赛德尔（Gauss-Seidel）方法（也称为利勃曼方法或连续位移方法）是用于求解线性方程组的迭代方法。它以德国数学家卡尔·弗里德里希·高斯（Carl Friedrich Gauss）和菲利普·路德维希·冯·塞德（Philipp Ludwigvon Seidel）的名字命名，类似于Jacobi方法。尽管可以将其应用于对角线上具有非零元素的任何矩阵，但是只有在矩阵严格地对角占优，或对称且为正定的情况下，才能保证收敛。

​		The Gauss–Seidel method, also known as the Liebmann method or the method of successive displacement, is an iterative method used to solve a system of linear equations. It is named after the German mathematicians Carl Friedrich Gauss and Philipp Ludwigvon Seidel, and is similar to the Jacobi method. Though it can be applied to any matrix with non-zero elements on the diagonals, convergence is only guaranteed if the matrix is either strictly diagonally dominant,or symmetric and positive definite.

#### 3.2.2 Description

Gauss-Seidel法是一种用于求解n维线性方程组近似解的方法

The Gauss–Seidel method is an iterative technique for solving a square system of n linear equations with unknown x
```math
$$
Ax=b
$$
```
它由以下迭代式所定义：

It is defined by the iteration
```math
$$
L_{*} \mathbf{x}^{(k+1)}=\mathbf{b}-U \mathbf{x}^{(k)}
$$
```


其中 $`$x^{(k)}$`$ 是方程的第k个近似解或第k次迭代，$`$x^{(k+1)}$`$是 $`$x^{(k)}$`$ 的下一次迭代，A被分解为较简单的两个三角矩阵，其中$`$L_*$`$是下三角矩阵，U是上三角矩阵

where $`$x^{(k)}$`$ is the kth approximation or iteration of $`$x$`$, $`$x^{(k+1)}$`$is the next or k + 1 iteration of $`$x$`$, and the matrix A is decomposed into a lower triangular component $`$L_*$`$ , and a strictly upper triangular component U:$`$A=L_*+U$`$

详情如下：

In more detail, write out A, x and b in their components:
<math xmlns="http://www.w3.org/1998/Math/MathML" display="block">
  <mi>A</mi>
  <mo>=</mo>
  <mrow data-mjx-texclass="INNER">
    <mo data-mjx-texclass="OPEN">[</mo>
    <mtable columnalign="center center center center" columnspacing="1em" rowspacing="4pt">
      <mtr>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>11</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>12</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>1</mn>
              <mi>n</mi>
            </mrow>
          </msub>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>21</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>22</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>2</mn>
              <mi>n</mi>
            </mrow>
          </msub>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mi>n</mi>
              <mn>1</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mi>n</mi>
              <mn>2</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <mo>…</mo>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mi>n</mi>
              <mi>n</mi>
            </mrow>
          </msub>
        </mtd>
      </mtr>
    </mtable>
    <mo data-mjx-texclass="CLOSE">]</mo>
  </mrow>
  <mo>,</mo>
  <mi>x</mi>
  <mo>=</mo>
  <mrow data-mjx-texclass="INNER">
    <mo data-mjx-texclass="OPEN">[</mo>
    <mtable columnspacing="1em" rowspacing="4pt">
      <mtr>
        <mtd>
          <msub>
            <mi>x</mi>
            <mrow>
              <mn>1</mn>
            </mrow>
          </msub>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <msub>
            <mi>x</mi>
            <mrow>
              <mn>2</mn>
            </mrow>
          </msub>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <mo>…</mo>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <msub>
            <mi>x</mi>
            <mrow>
              <mi>n</mi>
            </mrow>
          </msub>
        </mtd>
      </mtr>
    </mtable>
    <mo data-mjx-texclass="CLOSE">]</mo>
  </mrow>
  <mo>,</mo>
  <mi>b</mi>
  <mo>=</mo>
  <mrow data-mjx-texclass="INNER">
    <mo data-mjx-texclass="OPEN">[</mo>
    <mtable columnspacing="1em" rowspacing="4pt">
      <mtr>
        <mtd>
          <msub>
            <mi>b</mi>
            <mrow>
              <mn>1</mn>
            </mrow>
          </msub>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <msub>
            <mi>b</mi>
            <mrow>
              <mn>2</mn>
            </mrow>
          </msub>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <mo>…</mo>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <msub>
            <mi>b</mi>
            <mrow>
              <mi>n</mi>
            </mrow>
          </msub>
        </mtd>
      </mtr>
    </mtable>
    <mo data-mjx-texclass="CLOSE">]</mo>
  </mrow>
</math>


Then the decomposition of A into its lower triangular component and its strictly upper triangular component is given by:
<math xmlns="http://www.w3.org/1998/Math/MathML" display="block">
  <mi>A</mi>
  <mo>=</mo>
  <msub>
    <mi>L</mi>
    <mrow>
      <mo>∗</mo>
    </mrow>
  </msub>
  <mo>+</mo>
  <mi>U</mi>
  <mstyle scriptlevel="0">
    <mspace width="1em"></mspace>
  </mstyle>
  <mtext> where </mtext>
  <mstyle scriptlevel="0">
    <mspace width="1em"></mspace>
  </mstyle>
  <msub>
    <mi>L</mi>
    <mrow>
      <mo>∗</mo>
    </mrow>
  </msub>
  <mo>=</mo>
  <mrow data-mjx-texclass="INNER">
    <mo data-mjx-texclass="OPEN">[</mo>
    <mtable columnalign="center center center center" columnspacing="1em" rowspacing="4pt">
      <mtr>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>11</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <mn>0</mn>
        </mtd>
        <mtd>
          <mo>⋯</mo>
        </mtd>
        <mtd>
          <mn>0</mn>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>21</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>22</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <mo>⋯</mo>
        </mtd>
        <mtd>
          <mn>0</mn>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <mrow>
            <mo>⋮</mo>
          </mrow>
        </mtd>
        <mtd>
          <mrow>
            <mo>⋮</mo>
          </mrow>
        </mtd>
        <mtd>
          <mo>⋱</mo>
        </mtd>
        <mtd>
          <mrow>
            <mo>⋮</mo>
          </mrow>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mi>n</mi>
              <mn>1</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mi>n</mi>
              <mn>2</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <mo>⋯</mo>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mi>n</mi>
              <mi>n</mi>
            </mrow>
          </msub>
        </mtd>
      </mtr>
    </mtable>
    <mo data-mjx-texclass="CLOSE">]</mo>
  </mrow>
  <mo>,</mo>
  <mstyle scriptlevel="0">
    <mspace width="1em"></mspace>
  </mstyle>
  <mi>U</mi>
  <mo>=</mo>
  <mrow data-mjx-texclass="INNER">
    <mo data-mjx-texclass="OPEN">[</mo>
    <mtable columnalign="center center center center" columnspacing="1em" rowspacing="4pt">
      <mtr>
        <mtd>
          <mn>0</mn>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>12</mn>
            </mrow>
          </msub>
        </mtd>
        <mtd>
          <mo>⋯</mo>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>1</mn>
              <mi>n</mi>
            </mrow>
          </msub>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <mn>0</mn>
        </mtd>
        <mtd>
          <mn>0</mn>
        </mtd>
        <mtd>
          <mo>⋯</mo>
        </mtd>
        <mtd>
          <msub>
            <mi>a</mi>
            <mrow>
              <mn>2</mn>
              <mi>n</mi>
            </mrow>
          </msub>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <mrow>
            <mo>⋮</mo>
          </mrow>
        </mtd>
        <mtd>
          <mrow>
            <mo>⋮</mo>
          </mrow>
        </mtd>
        <mtd>
          <mo>⋱</mo>
        </mtd>
        <mtd>
          <mrow>
            <mo>⋮</mo>
          </mrow>
        </mtd>
      </mtr>
      <mtr>
        <mtd>
          <mn>0</mn>
        </mtd>
        <mtd>
          <mn>0</mn>
        </mtd>
        <mtd>
          <mo>⋯</mo>
        </mtd>
        <mtd>
          <mn>0</mn>
        </mtd>
      </mtr>
    </mtable>
    <mo data-mjx-texclass="CLOSE">]</mo>
  </mrow>
</math>
由上可得线性方程组可以重写为：

The system of linear equations may be rewritten as:
$$
L_*x=b-Ux
$$

现在，Gauss-Seidel法使用右侧的x的先前值来求解该表达式的左侧x。从分析上讲，这可以写为：

The Gauss–Seidel method now solves the left hand side of this expression for x, using previous value for x on the right hand side. Analytically, this may be written as:
```math
$$
\mathbf{x}^{(k+1)}=L_{*}^{-1}\left(\mathbf{b}-U \mathbf{x}^{(k)}\right)
$$
```


通过利用化简后的三角矩阵形式，可以使用正向替换顺序计算$x^{(k+1)}$的元素：

However, by taking advantage of the triangular form of , the elements of x(k+1) can be computed sequentially using forward substitution:
```math
$$
x_{i}^{(k+1)}=\frac{1}{a_{i i}}\left(b_{i}-\sum_{j=1}^{i-1} a_{i j} x_{j}^{(k+1)}-\sum_{j=i+1}^{n} a_{i j} x_{j}^{(k)}\right), \quad i=1,2, \ldots, n
$$
```
通常继续执行该过程，直到通过迭代进行的值的变化低于某个误差阈值为止。

The  procedure is  generally continued  until  the  changes made  by  an  iteration are  below some tolerance, such as a sufficiently small residual.

#### 3.2.3 Algorithm

由于在此算法中计算元素时可以覆盖它们，因此仅需要一个存储向量，并且省略了向量索引。 该算法如下：

Since elements can be overwritten as they are computed in this algorithm, only one storage vector is needed, and vector indexing is omitted. The algorithm goes as follows:

```
Inputs: A, b

Output: ∅

Choose an initial guess ∅ to the solution

repeat until convergence 

​	for i from 1 until n do 

​		θ<-0

​		for j from 1 until n do

​			if j ≠ i then 

​				θ<-θ+$a_{ij}∅_j$

​			endif

​		end (j-loop)

​		$\phi_{i} \leftarrow \frac{1}{a_{i i}}\left(b_{i}-\sigma\right)$

​	end (i-loop)    

​	check if convergence is reached

end (repeat)
```



#### 3.2.4 Convergence

Gauss-Seidel方法的收敛性取决于矩阵A。即，已知在以下情况下该过程收敛：

- A是对称的正定

- A在对角线中严格或不可避免地占主导地位。

即使不满足这些条件，高斯-塞德尔方法有时也会收敛。

The convergence properties of the Gauss–Seidel method are dependent on the matrix A. Namely, the procedure is known to converge if either:

- A is symmetric positive-definite

- A is strictly or irreducibly diagonally dominant.

The Gauss–Seidel method sometimes converges even if these conditions are not satisfied.

#### 3.2.5 Implementation

```fortran
!$omp parallel do private(i) reduction(max : dumax)
    ! private - only be accessed by a single thread.
    ! 红黑树第一部分
    do j=1,n
        do i=1,n
            if(mod(i+j,2)==0) then
                u(i,j) = 0.25d0*(uold(i-1,j) + uold(i+1,j) + &
                             uold(i,j-1) + uold(i,j+1) + h**2*f(i,j))
            dumax = max(dumax, abs(u(i,j)-uold(i,j)))
            endif
        enddo
    enddo  
    !$omp parallel do private(i) reduction(max : dumax)
    ! 红黑树第二部分
    do j=1,n
        do i=1,n
            if(mod(i+j,2)==1) then
                u(i,j) = 0.25d0*(u(i-1,j) + u(i+1,j) + &
                             u(i,j-1) + u(i,j+1) + h**2*f(i,j))
            dumax = max(dumax, abs(u(i,j)-uold(i,j)))
            endif
        enddo
    enddo
```



---

### 3.3 SOR method

#### 3.3.1 Definition

SOR算法是解线性方程组的迭代加速方法 ,通过选择恰当的松弛因子ω ,它能使收敛速度较慢的迭代法变的收敛快 ,使发散的迭代法可能变成收敛

The SOR algorithm is an iterative acceleration method for solving linear equations. By selecting the appropriate relaxation factor ω, it can make the iterative method with slower convergence speed to converge faster, so that the divergent iterative method may become convergent.

#### 3.3.2 Description

SOR法前面的工作大致和Gauss-Seidel method一致，易得 ：

The previous work of the SOR method is roughly the same as the Gauss-Seidel method, easy to get the equation below：
<math xmlns="http://www.w3.org/1998/Math/MathML" display="block">
  <mtable displaystyle="true" columnalign="right left right left right left right left right left right left" columnspacing="0em 2em 0em 2em 0em 2em 0em 2em 0em 2em 0em" rowspacing="3pt">
    <mtr>
      <mtd>
        <msubsup>
          <mi mathvariant="bold-italic">x</mi>
          <mrow>
            <mi>i</mi>
          </mrow>
          <mrow>
            <mo stretchy="false">(</mo>
            <mi>k</mi>
            <mo>+</mo>
            <mn>1</mn>
            <mo stretchy="false">)</mo>
          </mrow>
        </msubsup>
      </mtd>
      <mtd>
        <mi></mi>
        <mo>=</mo>
        <mrow data-mjx-texclass="INNER">
          <mo data-mjx-texclass="OPEN">(</mo>
          <msub>
            <mi mathvariant="bold-italic">b</mi>
            <mrow>
              <mi>i</mi>
            </mrow>
          </msub>
          <mo>−</mo>
          <msub>
            <mi mathvariant="bold-italic">a</mi>
            <mrow>
              <mi>i</mi>
              <mn>1</mn>
            </mrow>
          </msub>
          <msubsup>
            <mi mathvariant="bold-italic">x</mi>
            <mrow>
              <mn>1</mn>
            </mrow>
            <mrow>
              <mo stretchy="false">(</mo>
              <mi>k</mi>
              <mo>+</mo>
              <mn>1</mn>
              <mo stretchy="false">)</mo>
            </mrow>
          </msubsup>
          <mo>−</mo>
          <mo>⋯</mo>
          <mo>−</mo>
          <msub>
            <mi mathvariant="bold-italic">a</mi>
            <mrow>
              <mi>i</mi>
              <mo>,</mo>
              <mi>i</mi>
              <mo>−</mo>
              <mn>1</mn>
            </mrow>
          </msub>
          <msubsup>
            <mi mathvariant="bold-italic">x</mi>
            <mrow>
              <mi>i</mi>
              <mo>−</mo>
              <mn>1</mn>
            </mrow>
            <mrow>
              <mo stretchy="false">(</mo>
              <mi>k</mi>
              <mo>+</mo>
              <mn>1</mn>
              <mo stretchy="false">)</mo>
            </mrow>
          </msubsup>
          <mo>−</mo>
          <msub>
            <mi mathvariant="bold-italic">a</mi>
            <mrow>
              <mi>i</mi>
              <mo>,</mo>
              <mi>i</mi>
              <mo>+</mo>
              <mn>1</mn>
            </mrow>
          </msub>
          <msubsup>
            <mi mathvariant="bold-italic">x</mi>
            <mrow>
              <mi>i</mi>
              <mo>+</mo>
              <mn>1</mn>
            </mrow>
            <mrow>
              <mo stretchy="false">(</mo>
              <mi>k</mi>
              <mo stretchy="false">)</mo>
            </mrow>
          </msubsup>
          <mo>−</mo>
          <mo>⋯</mo>
          <mo>−</mo>
          <msub>
            <mi mathvariant="bold-italic">a</mi>
            <mrow>
              <mi>i</mi>
              <mo>,</mo>
              <mi>n</mi>
            </mrow>
          </msub>
          <msubsup>
            <mi mathvariant="bold-italic">x</mi>
            <mrow>
              <mi>n</mi>
            </mrow>
            <mrow>
              <mo stretchy="false">(</mo>
              <mi>k</mi>
              <mo stretchy="false">)</mo>
            </mrow>
          </msubsup>
          <mo data-mjx-texclass="CLOSE">)</mo>
        </mrow>
        <mrow>
          <mo>/</mo>
        </mrow>
        <msub>
          <mi mathvariant="bold-italic">a</mi>
          <mrow>
            <mi>i</mi>
            <mi>i</mi>
          </mrow>
        </msub>
      </mtd>
    </mtr>
    <mtr>
      <mtd></mtd>
      <mtd>
        <mi></mi>
        <mo>=</mo>
        <msubsup>
          <mi mathvariant="bold-italic">x</mi>
          <mrow>
            <mi>i</mi>
          </mrow>
          <mrow>
            <mo stretchy="false">(</mo>
            <mi>k</mi>
            <mo stretchy="false">)</mo>
          </mrow>
        </msubsup>
        <mo>+</mo>
        <mrow data-mjx-texclass="INNER">
          <mo data-mjx-texclass="OPEN">(</mo>
          <msub>
            <mi mathvariant="bold-italic">b</mi>
            <mrow>
              <mi>i</mi>
            </mrow>
          </msub>
          <mo>−</mo>
          <munderover>
            <mo data-mjx-texclass="OP">∑</mo>
            <mrow>
              <mi>j</mi>
              <mo>=</mo>
              <mn>1</mn>
            </mrow>
            <mrow>
              <mi>i</mi>
              <mo>−</mo>
              <mn>1</mn>
            </mrow>
          </munderover>
          <msub>
            <mi mathvariant="bold-italic">a</mi>
            <mrow>
              <mi>i</mi>
              <mi>j</mi>
            </mrow>
          </msub>
          <msubsup>
            <mi mathvariant="bold-italic">x</mi>
            <mrow>
              <mi>j</mi>
            </mrow>
            <mrow>
              <mo stretchy="false">(</mo>
              <mi>k</mi>
              <mo>+</mo>
              <mn>1</mn>
              <mo stretchy="false">)</mo>
            </mrow>
          </msubsup>
          <mo>−</mo>
          <munderover>
            <mo data-mjx-texclass="OP">∑</mo>
            <mrow>
              <mi>j</mi>
              <mo>=</mo>
              <mi>i</mi>
            </mrow>
            <mrow>
              <mi>n</mi>
            </mrow>
          </munderover>
          <msub>
            <mi mathvariant="bold-italic">a</mi>
            <mrow>
              <mi>i</mi>
              <mi>j</mi>
            </mrow>
          </msub>
          <msubsup>
            <mi mathvariant="bold-italic">x</mi>
            <mrow>
              <mi>j</mi>
            </mrow>
            <mrow>
              <mo stretchy="false">(</mo>
              <mi>k</mi>
              <mo stretchy="false">)</mo>
            </mrow>
          </msubsup>
          <mo data-mjx-texclass="CLOSE">)</mo>
        </mrow>
        <mrow>
          <mo>/</mo>
        </mrow>
        <msub>
          <mi mathvariant="bold-italic">a</mi>
          <mrow>
            <mi>i</mi>
            <mi>i</mi>
          </mrow>
        </msub>
      </mtd>
    </mtr>
  </mtable>
</math>


为了得到更好的收敛效果，可在修正项前乘以一个松弛因子ω，于是可得迭代格式：

In order to get a better convergence effect, a relaxation factor ω can be multiplied before the correction term, so the iterative format can be obtained
<math xmlns="http://www.w3.org/1998/Math/MathML" display="block">
  <msubsup>
    <mi mathvariant="bold-italic">x</mi>
    <mrow>
      <mi>i</mi>
    </mrow>
    <mrow>
      <mo stretchy="false">(</mo>
      <mi>k</mi>
      <mo>+</mo>
      <mn>1</mn>
      <mo stretchy="false">)</mo>
    </mrow>
  </msubsup>
  <mo>=</mo>
  <msubsup>
    <mi mathvariant="bold-italic">x</mi>
    <mrow>
      <mi>i</mi>
    </mrow>
    <mrow>
      <mo stretchy="false">(</mo>
      <mi>k</mi>
      <mo stretchy="false">)</mo>
    </mrow>
  </msubsup>
  <mo>+</mo>
  <mi>ω</mi>
  <mrow data-mjx-texclass="INNER">
    <mo data-mjx-texclass="OPEN">(</mo>
    <msub>
      <mi mathvariant="bold-italic">b</mi>
      <mrow>
        <mi>i</mi>
      </mrow>
    </msub>
    <mo>−</mo>
    <munderover>
      <mo data-mjx-texclass="OP">∑</mo>
      <mrow>
        <mi>j</mi>
        <mo>=</mo>
        <mn>1</mn>
      </mrow>
      <mrow>
        <mi>i</mi>
        <mo>−</mo>
        <mn>1</mn>
      </mrow>
    </munderover>
    <msub>
      <mi mathvariant="bold-italic">a</mi>
      <mrow>
        <mi>i</mi>
        <mi>j</mi>
      </mrow>
    </msub>
    <msubsup>
      <mi mathvariant="bold-italic">x</mi>
      <mrow>
        <mi>j</mi>
      </mrow>
      <mrow>
        <mo stretchy="false">(</mo>
        <mi>k</mi>
        <mo>+</mo>
        <mn>1</mn>
        <mo stretchy="false">)</mo>
      </mrow>
    </msubsup>
    <mo>−</mo>
    <munderover>
      <mo data-mjx-texclass="OP">∑</mo>
      <mrow>
        <mi>j</mi>
        <mo>=</mo>
        <mi>i</mi>
      </mrow>
      <mrow>
        <mi>n</mi>
      </mrow>
    </munderover>
    <msub>
      <mi mathvariant="bold-italic">a</mi>
      <mrow>
        <mi>i</mi>
        <mi>j</mi>
      </mrow>
    </msub>
    <msubsup>
      <mi mathvariant="bold-italic">x</mi>
      <mrow>
        <mi>j</mi>
      </mrow>
      <mrow>
        <mo stretchy="false">(</mo>
        <mi>k</mi>
        <mo stretchy="false">)</mo>
      </mrow>
    </msubsup>
    <mo data-mjx-texclass="CLOSE">)</mo>
  </mrow>
  <mrow>
    <mo>/</mo>
  </mrow>
  <msub>
    <mi mathvariant="bold-italic">a</mi>
    <mrow>
      <mi>i</mi>
      <mi>i</mi>
    </mrow>
  </msub>
</math>
写成矩阵形式即为：

Written in matrix form is:
<math xmlns="http://www.w3.org/1998/Math/MathML" display="block">
  <mtable displaystyle="true" columnalign="right left right left right left right left right left right left" columnspacing="0em 2em 0em 2em 0em 2em 0em 2em 0em 2em 0em" rowspacing="3pt">
    <mtr>
      <mtd>
        <msup>
          <mi mathvariant="bold-italic">x</mi>
          <mrow>
            <mo stretchy="false">(</mo>
            <mi>k</mi>
            <mo>+</mo>
            <mn>1</mn>
            <mo stretchy="false">)</mo>
          </mrow>
        </msup>
      </mtd>
      <mtd>
        <mi></mi>
        <mo>=</mo>
        <msup>
          <mi mathvariant="bold-italic">x</mi>
          <mrow>
            <mo stretchy="false">(</mo>
            <mi>k</mi>
            <mo stretchy="false">)</mo>
          </mrow>
        </msup>
        <mo>+</mo>
        <mi>ω</mi>
        <msup>
          <mi mathvariant="bold-italic">D</mi>
          <mrow>
            <mo>−</mo>
            <mn>1</mn>
          </mrow>
        </msup>
        <mrow data-mjx-texclass="INNER">
          <mo data-mjx-texclass="OPEN">(</mo>
          <mi mathvariant="bold-italic">b</mi>
          <mo>+</mo>
          <mi mathvariant="bold-italic">L</mi>
          <msup>
            <mi mathvariant="bold-italic">x</mi>
            <mrow>
              <mo stretchy="false">(</mo>
              <mi>k</mi>
              <mo>+</mo>
              <mn>1</mn>
              <mo stretchy="false">)</mo>
            </mrow>
          </msup>
          <mo>+</mo>
          <mi mathvariant="bold-italic">U</mi>
          <msup>
            <mi mathvariant="bold-italic">x</mi>
            <mrow>
              <mo stretchy="false">(</mo>
              <mi>k</mi>
              <mo stretchy="false">)</mo>
            </mrow>
          </msup>
          <mo>−</mo>
          <mi mathvariant="bold-italic">D</mi>
          <msup>
            <mi mathvariant="bold-italic">x</mi>
            <mrow>
              <mo stretchy="false">(</mo>
              <mi>k</mi>
              <mo stretchy="false">)</mo>
            </mrow>
          </msup>
          <mo data-mjx-texclass="CLOSE">)</mo>
        </mrow>
      </mtd>
    </mtr>
    <mtr>
      <mtd>
        <msup>
          <mi mathvariant="bold-italic">x</mi>
          <mrow>
            <mo stretchy="false">(</mo>
            <mi>k</mi>
            <mo>+</mo>
            <mn>1</mn>
            <mo stretchy="false">)</mo>
          </mrow>
        </msup>
      </mtd>
      <mtd>
        <mi></mi>
        <mo>=</mo>
        <mo stretchy="false">(</mo>
        <mi mathvariant="bold-italic">D</mi>
        <mo>−</mo>
        <mi>ω</mi>
        <mi mathvariant="bold-italic">L</mi>
        <msup>
          <mo stretchy="false">)</mo>
          <mrow>
            <mo>−</mo>
            <mn>1</mn>
          </mrow>
        </msup>
        <mo stretchy="false">[</mo>
        <mo stretchy="false">(</mo>
        <mn>1</mn>
        <mo>−</mo>
        <mi>ω</mi>
        <mo stretchy="false">)</mo>
        <mi mathvariant="bold-italic">D</mi>
        <mo>+</mo>
        <mi>ω</mi>
        <mi mathvariant="bold-italic">U</mi>
        <mo stretchy="false">]</mo>
        <msup>
          <mi mathvariant="bold-italic">x</mi>
          <mrow>
            <mo stretchy="false">(</mo>
            <mi>k</mi>
            <mo stretchy="false">)</mo>
          </mrow>
        </msup>
        <mo>+</mo>
        <mi>ω</mi>
        <mo stretchy="false">(</mo>
        <mi mathvariant="bold-italic">D</mi>
        <mo>−</mo>
        <mi>ω</mi>
        <mi mathvariant="bold-italic">L</mi>
        <msup>
          <mo stretchy="false">)</mo>
          <mrow>
            <mo>−</mo>
            <mn>1</mn>
          </mrow>
        </msup>
        <mi mathvariant="bold-italic">b</mi>
      </mtd>
    </mtr>
  </mtable>
</math>

#### 3.3.3 Algorithm

```
Inputs: A, Y，w

Output: x

do
	for i= 0 to n-1
	  xold[i]=x[i];
	  for j=0 to n-1
	  	if i!=j 
	  	sum+=a[i][j]*x[j]
	  x[i]=(b[i]-sum)/a[i][i],其中a[i][i]不等于0；
	for i=0 to n-1
	  x[i]=w*x[i]+(1-w)*xold[i]
	for i=0 to n-1
	while flag==0
	判断误差error是否大于给定误差eps，若大于继续迭代，否则flag=1
	end while
	step3：输出所求的解向量和迭代次数
	end

```



#### 3.3.4 Convergence

SOR收敛的必要条件是0 < ω< 2。

SOR收敛的充分条件是A对称正定，且0 < ω< 2或者若A严格对角占优或不可约弱对角占优，且0 < ω≤1。

The necessary condition for SOR convergence is 0 <ω <2.

The sufficient condition for SOR convergence is that A is symmetric and positive definite, and 0 <ω <2 or if A is strictly diagonally dominant or irreducible weakly diagonally dominant, and 0 <ω≤1.

#### 3.3.5 Implementation

```fortran
!$omp parallel do private(i) reduction(max : dumax)
	do j=1,n
        do i=1,n
            if(mod(i+j,2) == 0) then
                u(i,j) =(1-w)*uold(i,j) + 0.25d0*w*(uold(i-1,j) + uold(i+1,j) + &
                             uold(i,j-1) + uold(i,j+1) + h**2*f(i,j))
            endif
            dumax = max(dumax, abs(u(i,j)-uold(i,j)))
        enddo
    enddo  
    !$omp parallel do private(i) reduction(max : dumax)
    do j=1,n
        do i=1,n
            if(mod(i+j,2) == 1) then
                u(i,j) = (1-w)*uold(i,j) + 0.25d0*w*(u(i-1,j) + u(i+1,j) + &
                             u(i,j-1) + u(i,j+1) + h**2*f(i,j))
            dumax = max(dumax, abs(u(i,j)-uold(i,j)))
            endif
        enddo
    enddo
```



## 4. Experiments

**实验环境**： XUbuntu虚拟机，分配了2个处理器，每个处理器2个内核，内存4G

**Experimental environment**: XUbuntu virtual machine, allocated 2 processors, 2 cores per processor, 4G memory

![](http://image.leesdog.space/HPSC2.jpg)

### 4.1 Jacobi iteration method

#### 4.1.1 Experiment process

首先对整个温度矩阵进行初始化，默认情况是将整个矩阵的第一列以及第一行全部设置为0，然后将最后一列和最后一行中间温度设置为1，最后将其他地方温度设置为0

First initialize the entire temperature matrix. The default is to set the first column and the first row of the entire matrix to 0, then set the middle temperature of the last column and the last row to 1, and finally set the temperature of other places to 0

```fortran
 u(i,j) = 0.25d0*(uold(i-1,j) + uold(i+1,j) + &
                                 uold(i,j-1) + uold(i,j+1) + h**2*f(i,j))
```

u代表当前迭代的矩阵值，uold代表上一次迭代矩阵结果，h**2*f(i,j)为误差值，本次实验环境为稳态，所以误差设置基本为0

$`$u$`$ represents the matrix value of the current iteration, $`$uold$`$ represents the matrix result of the last iteration, $`$h**2*f(i,j) $`$is the error value, this experiment environment is steady state, so the error setting is basically 0.



循环整个矩阵

Loop the entire matrix

```fortran
 do j=1,n
        do i=1,n
            u(i,j) = 0.25d0*(uold(i-1,j) + uold(i+1,j) + &
                             uold(i,j-1) + uold(i,j+1) + h**2*f(i,j))
            dumax = max(dumax, abs(u(i,j)-uold(i,j)))
        enddo
 enddo
```

当前一次迭代和后一次迭代之间无任何差异之后，退出迭代，完成循环，输出迭代次数

After there is no difference between the current iteration and the next iteration, exit the iteration, complete the loop, and output the number of iterations

```
if (mod(iter,nprint)==0) then
        print 203, iter, dumax
 203         format("After ",i8," iterations, dumax = ",d16.6,/)
            endif
        ! check for convergence:
        if (dumax .lt. tol) exit
```

整个并行雅可比算法如下，雅可比算法的特点是每次运行时需要将i设置为线程私有，保证每个线程之间i是互不干扰的，j不断向后逐行循环，每一次循环都使用的是前一次迭代的数据，并没有使用当前迭代的数据：

The entire parallel Jacobi algorithm is as follows. The characteristic of the Jacobi algorithm is that i needs to be set as thread private each time it is run to ensure that i does not interfere with each other between each thread. Only j continuously loops backwards line by line, each cycle both use the data of the previous iteration, rather than the current iteration data:

```fortran
do iter=1,maxiter
        uold = u         ! old values
        dumax = 0.d0
        !$omp parallel do private(i) reduction(max : dumax)
        ! private - only be accessed by a single thread.
        ! #pragma omp parallel for private(i) reduction(max : dumax)
        do j=1,n
            do i=1,n
                u(i,j) = 0.25d0*(uold(i-1,j) + uold(i+1,j) + &
                                 uold(i,j-1) + uold(i,j+1) + h**2*f(i,j))
                dumax = max(dumax, abs(u(i,j)-uold(i,j)))
            enddo
        enddo

        if (mod(iter,nprint)==0) then
            print 203, iter, dumax
203         format("After ",i8," iterations, dumax = ",d16.6,/)
            endif
        ! check for convergence:
        if (dumax .lt. tol) exit

        enddo
```

#### 4.1.2 Result

![](http://image.leesdog.space/HPSC3.jpg)





### 4.2 Gauss-Seidel method

#### 4.2.1 Experiment process

代码实现的前面部分与雅可比算法一致，主要的区别在于后面，对于高斯-赛德尔法，每次迭代中如果运算数值更新变为新数据，需要立即将下一步计算数据变为新值：

The front part of the code is consistent with the Jacobi algorithm. The main difference lies in the latter. For the Gauss-Seidel method, if the calculated value is updated to new data in each iteration, the next calculation data needs to be immediately changed to the new value:

```fortran
    do j=1,n
        do i=1,n
                u(i,j) = 0.25d0*(u(i-1,j) + uold(i+1,j) + &
                             u(i,j-1) + uold(i,j+1) + h**2*f(i,j))
            dumax = max(dumax, abs(u(i,j)-uold(i,j)))
            endif
        enddo
    enddo  
```

$`$u（i-1,j）$`$和$`$u（i,j-1）$`$都是已经更新的数据

$`$u(i-1,j)$`$ and $`$u(i,j-1)$`$ are both updated data

多线程并行算法中，由于每个线程之间速度并不一定相同，可能会出现一个线程运行速度较快，但他需要其他还未计算出来达到数据，进而导致其使用了之前的数据而使得结果错误，为了解决该问题，我们选择使用红黑排序的思想，将整个代码分为两个部分，第一部分是计算红黑排序中红色部分或黑色部分，另一部分是根据上一步得到的结果计算红黑排序中的另一部分：

In a multi-threaded parallel algorithm, since the speed of each thread is not necessarily the same, there may be a thread running faster, but it needs other data that has not been calculated, which will cause it to use the previous data and make the result Error, in order to solve this problem, we chose to use the idea of red-black sorting. The whole code is divided into two parts. The first part is to calculate the red part or the black part in the red-black sorting. The other part is to calculate the red part based on the result obtained in the previous step. Another part of the black sort:

```fortran
    !$omp parallel do private(i) reduction(max : dumax)
    ! private - only be accessed by a single thread.
    ! 红黑树第一部分
    do j=1,n
        do i=1,n
            if(mod(i+j,2)==0) then
                u(i,j) = 0.25d0*(uold(i-1,j) + uold(i+1,j) + &
                             uold(i,j-1) + uold(i,j+1) + h**2*f(i,j))
            dumax = max(dumax, abs(u(i,j)-uold(i,j)))
            endif
        enddo
    enddo  
    !$omp parallel do private(i) reduction(max : dumax)
    ! 红黑树第二部分
    do j=1,n
        do i=1,n
            if(mod(i+j,2)==1) then
                u(i,j) = 0.25d0*(u(i-1,j) + u(i+1,j) + &
                             u(i,j-1) + u(i,j+1) + h**2*f(i,j))
            dumax = max(dumax, abs(u(i,j)-uold(i,j)))
            endif
        enddo
    enddo
```



#### 4.2.2 Result

![](http://image.leesdog.space/HPSC4.jpg)



### 4.3 SOR method

#### 4.3.1 Experiment process

该方法实现过程中，和高斯-赛德尔法的区别在于加入了松弛因子，通过松弛因子w来将计算根据权值分为两部分，一部分是将新结果得到的值乘上w，另一部分原值乘上（1-w），定义矩阵为u，之前的矩阵为uold，松弛因子为w，为了计算当前的u(i,j)，需要用到上一次迭代过程中该数值的旧值，所以整个代码最终结果为：

In the implementation process of this method, the difference from the Gauss-Seidel method is that the relaxation factor is added. The relaxation factor w is used to divide the calculation into two parts according to the weight. One part is to multiply the value obtained by the new result by w, and the other part is the original Multiply the value by (1-w), the matrix is defined as u, the previous matrix is uold, and the relaxation factor is w. In order to calculate the current u(i,j), the old value of the value in the previous iteration needs to be used, So the final result of the entire code is:

```fortran
	   do j=1,n
            do i=1,n
                if(mod(i+j,2) == 0) then
                    u(i,j) =(1-w)*uold(i,j) + 0.25d0*w*(u(i-1,j) + uold(i+1,j) + &
                                 u(i,j-1) + uold(i,j+1) + h**2*f(i,j))
                endif
                dumax = max(dumax, abs(u(i,j)-uold(i,j)))
            enddo
        enddo  
```

多线程时，仍然是通过红黑排序进行计算的，和高斯-赛德尔法类似，分为两部分，将整个代码分为两个部分，第一部分是计算红黑排序中红色部分或黑色部分，另一部分是根据上一步得到的结果计算红黑排序中的另一部分，唯一的区别是在计算上我们加入了松弛因子w

When multi-threaded, it is still calculated by red-black sorting, which is similar to the Gauss-Seidel method. It is divided into two parts. The entire code is divided into two parts. The first part is to calculate the red or black part in the red-black sorting. The other part is to calculate the other part of the red and black sorting based on the results obtained in the previous step. The only difference is that we have added a relaxation factor w in the calculation.

```fortran
	!$omp parallel do private(i) reduction(max : dumax)
	do j=1,n
        do i=1,n
            if(mod(i+j,2) == 0) then
                u(i,j) =(1-w)*uold(i,j) + 0.25d0*w*(uold(i-1,j) + uold(i+1,j) + &
                             uold(i,j-1) + uold(i,j+1) + h**2*f(i,j))
            endif
            dumax = max(dumax, abs(u(i,j)-uold(i,j)))
        enddo
    enddo  
    !$omp parallel do private(i) reduction(max : dumax)
    do j=1,n
        do i=1,n
            if(mod(i+j,2) == 1) then
                u(i,j) = (1-w)*uold(i,j) + 0.25d0*w*(u(i-1,j) + u(i+1,j) + &
                             u(i,j-1) + u(i,j+1) + h**2*f(i,j))
            dumax = max(dumax, abs(u(i,j)-uold(i,j)))
            endif
        enddo
    enddo
```

#### 4.3.2 Result

SOR由于对w的取值不同结果也会有变化，根据其收敛准则定义，w取值应在0-2之间，通过不断取值我得到了关于w取不同值以及不同线程数所花费时间的表格：

SOR will change due to different values of w. According to the definition of its convergence criterion, the value of w should be between 0-2. Through continuous value selection, I get information about the time it takes to take different values of w and the number of threads.

|    W     | 1 thread  | 2 thread  |
| :------: | :-------: | :-------: |
|   0.1    |  44.195   |  30.319   |
|   0.2    |  26.811   |  18.309   |
|   0.3    |  18.957   |  14.361   |
|   0.4    |  14.731   |   9.714   |
|   0.5    |  11.172   |   7.848   |
|   0.6    |   9.888   |   6.414   |
|   0.7    |   7.765   |   5.526   |
|   0.8    |   6.868   |   4.587   |
|   0.9    |   5.515   |   3.876   |
|   1.0    |   4.756   |   3.232   |
|   1.1    |   4.019   |   2.768   |
|   1.2    |   3.522   |   2.326   |
|   1.3    |   2.759   |   1.939   |
|   1.4    |   2.405   |   1.599   |
|   1.5    |   1.945   |   1.385   |
|   1.6    |   1.456   |   1.049   |
|   1.7    |   1.057   |   0.780   |
|   1.8    |   0.782   |   0.548   |
|   1.9    |   0.372   |   0.264   |
|   1.91   |   0.308   |   0.229   |
|   1.92   |   0.271   |   0.203   |
|   1.93   |   0.275   |   0.177   |
|   1.94   |   0.206   |   0.153   |
| **1.95** | **0.138** | **0.099** |
|   1.96   |   0.187   |   0.143   |
|   1.97   |   0.227   |   0.154   |
|   1.98   |   0.353   |   0.237   |
|   1.99   |   0.699   |   0.509   |

![](http://image.leesdog.space/HPSCpic11.png)

![](http://image.leesdog.space/HPSC5.jpg)



## 5. Conclusions

​		三种算法经验证得到的热成像图与T-contour一致（见下图），就结果上来讲，三种算法中效果较好的是经过试验权值w后的超松弛算法，效果最差的是雅可比算法，在超松弛算法中总体趋势随着w在0-2之间逐步增加，速度也逐渐变快，但当w达到1.95左右时再增加则反而会导致速度变慢

<center>    <img   src="http://image.leesdog.space/pcolor.png">  
    <h6>
    图1 温度图
    </h6></center>


<center>    <img   src="http://image.leesdog.space/contour.png">    <br>    <div style=""><h6>
    图2 contour图
    </h6></div> </center>

​		整个课程设计过程中，通过查看老师所给的Reference以及Wikipedia上面的定义，我对Jacobi，Gauss-Serdel以及SOR算法都有了比较深刻的认识理解，同时也掌握了一定Fortran语言的语法，能够编写使用makefile文件，能够使用python库进行简单画图，同时掌握了三种算法的并行实现思想，了解学习了红黑排序思想

​		就并行计算上而言，Jacobi算法的思想在于并行时不立刻更新值，而是等待一次循环结束后进行更新；Gauss-Serdel算法的思想在于实时计算实时更新；而SOR算法又是在Gauss-Serdel算法的基础上添加了松弛因子w用于加速新得到值的计算

​		在整个学习过程中我感触比较深的是红黑排序的思想，通过这种思想能较为简单的在Fortran里实现并行计算



​	Throughout the course design process, by looking at the Reference given by the teacher and the definition on Wikipedia, I have a deep understanding of Jacobi, Gauss-Serdel and SOR algorithms, and at the same time I have mastered a certain Fortran language grammar, and can write Using makefile files, you can use the python library for simple drawing, and at the same time master the parallel implementation ideas of the three algorithms, understand and learn the red-black sorting ideas

 	In terms of parallel computing, the idea of the Jacobi algorithm is not to update the value immediately during parallelization, but to wait for the update after the end of a loop; the idea of the Gauss-Serdel algorithm is to calculate real-time updates in real time; and the SOR algorithm is in Gauss- On the basis of the Serdel algorithm, a relaxation factor w is added to accelerate the calculation of the newly obtained value

 	During the whole learning process, I felt deeply the idea of red and black sorting. Through this idea, parallel computing can be realized in Fortran more easily.
