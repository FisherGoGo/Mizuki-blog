---
title: 狄利克雷卷积
published: 2025-04-07
tags: [MATH]
category: Knowledge
draft: false
---

- 狄利克雷卷积是定义在**数论函数**间的一种二元运算，可以这样定义 
$$(f* g)(n)=\sum_{xy=n}f(x)\cdot g(y)$$
- 或者等价地写作
$$(f*g)(n)=\sum_{d\mid n}f(d)\cdot g(\frac{n}{d})$$
- 有一些常用的数论函数：
	- 欧拉函数 $\varphi(n)$ 
	- 单位函数 $\varepsilon (n) = \begin{cases}1,while \; n=1 \\ 0,otherwise\end{cases}$ 
	- 幂函数 $\mathbf{Id}_k(n)=n^k$ ，当 $k=1$ 时为恒等函数 $\mathbf{Id}(n)$ ，当 $k=0$ 时为常数函数 $\mathbf{1} (n)$  
	- 除数函数 $\sigma_k (n)=\sum_{d\mid n} d^k$ ，当 $k=1$ 时为因数和函数 $\sigma(n)$ ，当 $k=0$ 时为因数个数函数 $\sigma_0(n)$ 
- 这些函数在 $\gcd(a,b)=1$ 时为积性函数，既有 $f(a)f(b)=f(ab)$ ，其中单位函数和幂函数为完全积性函数
- 若 $f$ 和 $g$ 为积性函数，那么 $f*g$ 也为积性函数，证明如下
$$(f*g)(1)=f(1)g(1)=1$$

- 然后我们设 $a$ 和 $b$ 互质，有
$$(f*g)(a)=\sum_{d\mid a}f(d)g(\frac{a}{d}) \qquad (f*g)(b)=\sum_{d\mid b}f(d)(g(\frac{b}{d})$$
$$(f*g)(ab)=\sum_{d\mid ab}f(d)g(\frac{ab}{d})$$
$$\begin{aligned}(f*g)(a)\cdot (f*g)(b)&=\sum_{d_1\mid a}f(d_1)g(\frac{a}{d_1})\cdot \sum_{d_2\mid b}f(d_2)g(\frac{b}{d_2})\\&=\sum_{d_1\mid a,d_2\mid b}f(d_1)g(\frac{a}{d_1})\cdot f(d_2)g(\frac{b}{d_2})\\&=\sum_{d_1\mid a,d_2\mid b}f(d_1d_2)g(\frac{ab}{d_1d_2})\end{aligned}$$

- 因为 $\gcd(a,b)=1$ ，所以他们两个的因数可以表示成 $a$ 的因数和 $b$ 的因数的乘积

# 数论函数之间的关系

- 根据狄利克雷卷积的定义，我们可以得到一些数论函数之间的关系

## 除数函数与幂函数
$$(f*\mathbf{1})(n)=\sum_{d\mid n}f(d)\mathbf{1}(\frac{n}{d})=\sum_{d\mid n}f(d)$$
$$(\mathbf{Id}_k*1)(n)=\sum_{d\mid n}\mathbf{Id}_k(d)=\sum_{d\mid n}d^k=\sigma_k$$
## 欧拉函数与恒等函数

- 因为 $(\varphi* \mathbf{1})(n)=\sum_{d\mid n}\varphi(\frac{n}{d})$ 
- 当 $d=p^m$ （ $p$ 为质数）时，我们有：
$$\begin{aligned} \sum_{d\mid n}\varphi(n)&=\varphi(1)+\sum_{i=1}^m \varphi(p^i) \\ &=1+\sum_{i=1}^m(p^i-p^{i-1}) \\ &=p^m=d\end{aligned}$$
- 所以有 $(\varphi*\mathbf{1})(p^m)=p^m$ 
- 现在我们让 $n$ 为任意整数，根据唯一分解定律他可以被分解为 $\prod p^m$ ，由于 $\varphi*\mathbf{1}$ 为积性函数，所以有 $(\varphi * \mathbf{1})(\prod p^m)=\prod(\varphi * \mathbf{1})(p^m)$ ，所以有 $(\varphi * \mathbf{1})(n)=n$ ，既 $\varphi * \mathbf{1}=\mathbf{Id}$  

# 性质

- 接下来我们来证明一些性质

## 交换律 

$$(f*g)(n)=\sum_{xy=n}f(x)g(y) \xLeftrightarrow{x\leftrightarrow y}\sum_{yx=n}f(y)g(x)=(g*f)(n)$$

## 结合律

$$\begin{aligned}((f*g)*h)(n)&=\sum_{xy=n}(f*g)(x)\cdot h(y) \\&=\sum_{xy=n} \left(\sum_{uv=x}f(u)g(v)\right)h(y) \\&=\sum_{xy=n}\sum_{uv=x}f(u)g(v)h(y) \\&=\sum_{uvy=n}f(u)g(v)h(y) \\& \xLeftrightarrow{u\rightarrow x,v\rightarrow u,y\rightarrow v} \sum_{xuv=n}f(x)g(u)h(v) \\&=\sum_{xy=n} \left(f(x)\cdot \sum_{uv=y}g(u)h(v)\right)\\&=\sum_{xy=n}f(x)\cdot (g*h)(y)\\&=\left(f*(g*h)\right)(n)\end{aligned}$$
## 加法分配律

$$\begin{aligned}(f*(g+h))(n)&=\sum_{xy=n}f(x)(g+h)(y) \\&=\sum_{xy=n}f(x)g(y)+\sum_{xy=n}f(x)h(y)\\&=(f*g)(n)+(f*h)(n)\end{aligned}$$
## 单位元
$$(\varepsilon *f)(n)=\sum_{d\mid n}\varepsilon(d)f(\frac{n}{d})=f(n)$$
- 所以 $\varepsilon$ 为狄利克雷卷积的单位元

## 逆元

- 如果有 $f*g=\varepsilon$  ，那么我们称 $g$ 为 $f$ 的狄利克雷逆元，记作 $f^{-1}$ 
- 让我们来一点一点推
$$(f*f^{-1})(1)=\sum_{d\mid n}f(d)f^{-1}(\frac{1}{d})=f(1)f^{-1}(1)=\varepsilon(1)=1$$
- 由此可得 $f^{-1}=\frac{1}{f(1)}$ ，这说明 $f$ 存在逆元的必要条件是 $f(1)\ne 0$ 
$$(f*f^{-1})(2)=\sum_{d\mid 2}f(d)f^{-1}(\frac{2}{d})=f(1)f^{-1}(2)+f(2)f^{-1}(1)=\varepsilon(2)=0$$
- 所以 $f^{-1}(2)=-\dfrac{f(2)f^{-1}(1)}{f(1)}$ ，我们也可以同上得出 $f^{-1}(3)=-\dfrac{f(3)f^{-1}(1)}{f(1)}$ ， $f^{-1}(4)=-\dfrac{f(4)f^{-1}(1)}{f(1)}-\dfrac{f(2)f^{-1}(1)}{f(1)}$ 
- 我们合理猜测下边这个函数即为 $f^{-1}$ 
$$g(n)=\begin{cases}\dfrac{1}{f(n)},while\;\;n=1\\ -\dfrac{1}{f(1)}\sum_{d\mid n,d>1}f(d)g(\frac{n}{d}),otherwise\end{cases}$$
- 我们来推导验证一下：
- $n=1$ 时， $(f*g)(n)=\sum_{d\mid 1}f(d)g(\frac{1}{d})=f(1)g(1)=1$
- $n>1$ 时
$$\begin{aligned}(f*g)(n)&=\sum_{d\mid n}f(d)d(\frac{n}{d})\\&=f(1)g(n)+\sum_{d\mid n,d>1}f(d)g(\frac{n}{d})\\&=-\dfrac{f(1)}{f(1)}\sum_{d\mid n,d>1}f(d)g(\frac{n}{d})+\sum_{d\mid n,d>1}f(d)g(\frac{n}{d})\\&=0\end{aligned}$$
