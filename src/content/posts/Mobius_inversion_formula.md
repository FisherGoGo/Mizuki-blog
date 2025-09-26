---
title: 莫比乌斯反演
published: 2025-07-18
tags: [MATH]
category: Knowledge
draft: false
---

# 莫比乌斯函数

- 对于常数函数 $\mathbf{1}$ 的迪利克雷逆元，我们将其称为莫比乌斯函数 $\mu$ 
- 我们不难得出 $\mu(p)=-1$ 和 $\mu(p^n)=0(n\ge 2)$ ，其中 $p$ 为质数，且 $\mu$ 为积性函数
- 那么就有以下形式
$$\mu(n)=\begin{cases}1,\quad while \;n=1\\(-1)^m,\quad while \; n=p_1p_2...p_m\\0,\quad otherwise\end{cases}$$
- 莫比乌斯函数被用于以下莫比乌斯反演公式：
$$g(n)=\sum_{d\mid n}f(d) \xLeftrightarrow{} f(n)=\sum_{d\mid n}\mu(n)g\left(\frac{n}{d}\right)$$
- 或者写成[狄利克雷卷积](https://fishergogo.github.io/2025/04/07/Dirichlet_product/)的形式，此时这个 $f$ 也被称为 $g$ 的莫比乌斯变换：
$$f*\mathbf{1}=g \xLeftrightarrow{} f=g*\mu$$

## 计算
- 对于莫比乌斯函数的计算我们可以在 $O(n)$ 内用欧拉筛算出
```cpp
int mu[N];
bool isnp[N];
vector <int> primes;
void init(int n){
	mu[1]=1;
	for(int i=2;i<=n;i++){
		if(!isnp[i]) primes.push_back(i),mu[i]=-1;
		for(int p:primes){
			if(p*i>n) break;
			isnp[p*i]=1;
			if(i%p==0){
				mu[p*i]=0;
				break;
			}
			else mu[p*i]=mu[p]*mu[i];
		}
	}
}
```

## 倍数形式

- 上边的式子被叫做莫比乌斯反演的因数形式，还有一个倍数形式：
$$g(n)=\sum_{n\mid N}f(N) \xLeftrightarrow{} f(n)=\sum_{n\mid N}g(N)\mu \left(\frac{N}{n}\right)$$
- 让我们通过容斥原理来理解：
- 设 $N=kn$ ，那我们就可以把 $\sum_{n\mid N}g(N)\mu \left(\frac{N}{n}\right)$ 写成 $\sum_{k=1}^{\infty}g(kn)\mu(k)$ 
- 相当于我们要计算 $f(n)$ 的值，就先计算 $g(n)=f(n)+f(2n)+f(3n)+...$ 的值，然后再减去所有 $2n$ 的倍数， $3n$ 的倍数， $5n$ 的倍数......的函数值，有些值会被重复减去，那么我们再加回来，在这个容斥过程中容斥系数是莫比乌斯函数值
- 又或者我们考虑逆推这个式子：
$$\begin{aligned}\sum_{n\mid N}g(N)\mu \left(\frac{N}{n}\right)&=\sum_{k=1}^{\infty}\mu(k)g(kn)\\&=\sum_{k=1}^{\infty}\mu(k)\sum_{kn\mid N}f(N)\\&=\sum_{n\mid N}f(N)\sum_{k\mid \frac{N}{n}}\mu(k)\\&=\sum_{n\mid N}f(N)\varepsilon\left(\frac{N}{n}\right)\\&=f(n)\end{aligned}$$
# 整除分块

- 其实这是一个常用的算法，在这里为下边做准备先了解一下
- 假如我们要求 $\sum_{d=1}^n \left\lfloor\frac{n}{d}\right\rfloor$  ，朴素求法是 $O(n)$ 的，但是我们可以发现有许多的连续项的值相同
- 我们设每一段的左端点为 $l$ ，那么右端点就是 $r=\left\lfloor\frac{n}{\left\lfloor\frac{n}{l}\right\rfloor}\right\rfloor$ 
- 我们设 $\left\lfloor\frac{n}{l}\right\rfloor=\left\lfloor\frac{n}{r}\right\rfloor$ ，则 $\left\lfloor\frac{n}{l}\right\rfloor \le \frac{n}{r}$ ，所以 $\frac{1}{\left\lfloor\frac{n}{l}\right\rfloor}\ge \frac{r}{n}$ ，又得 $r\le \frac{n}{\left\lfloor\frac{n}{l}\right\rfloor}$ ，又因为 $r$ 为整数，所以其最大值为 $\left\lfloor\frac{n}{\left\lfloor\frac{n}{l}\right\rfloor}\right\rfloor$ 

```cpp
for(int l=1,r;l<=n;l=r+1){
	r=n/(n/l);
	ans+=(r-l+1)*(n/l);
}
```

- 可以证明总段数不超过 $2\sqrt{n}$ ，所以复杂度为 $O(\sqrt{n})$ 

# 提取公因数

- 设有 $\sum_{i=1}^n\sum_{j=1}^m\sum_{d\mid gcd(i,j)}f(d,i,j)$ 
- 我们发现我们可以枚举 $d$ ，其中 $d\le \min(n,m)$ ，并且对于每个 $d$ 都有 $d\mid i\;, d\mid j$ ，我们令 $x=\frac{i}{d},\; y=\frac{j}{d}$ ，则 $\frac{1}{d}\le x\le \frac{n}{d},\frac{1}{d}\le y\le \frac{m}{d}$ ，也相当于 $1\le x\le \left\lfloor\frac{n}{d}\right\rfloor,\; 1\le\ y\le \left\lfloor\frac{m}{d}\right\rfloor$ 
- 所以原式化作 
$$\sum_{d=1}^{\min(n,m)}\sum_{x=1}^{\left\lfloor\frac{n}{d}\right\rfloor} \sum_{y=1}^{\left\lfloor\frac{m}{d}\right\rfloor}f(d,xd,yd)$$
- 而且这个表达式可以扩展
$$\sum_{d=1}^{\min(n,m)}\sum_{x=1}^{\left\lfloor\frac{n}{d}\right\rfloor} \sum_{y=1}^{\left\lfloor\frac{m}{d}\right\rfloor}\sum_{z=1}^{\left\lfloor\frac{l}{d}\right\rfloor}f(d,xd,yd,zd)$$

# 例题

## 求 $1\le x\le n,1\le y\le m$ ，且 $\gcd(x,y)=1$ 的二元组数

- 我们定义 $\left[f(x)=0\right]=\begin{cases}1,\; while\; f(x)=0\\0,\; otherwise\end{cases}$ 
- 那就是求 $\sum_{i=1}^n\sum_{j=1}^m\left[\gcd(i,j)=1\right]=\sum_{i=1}^n\sum_{j=1}^m\varepsilon\left(\gcd(i,j)\right)$  
- 根据 $\mu*\mathbf{1}=\varepsilon$ ，得到 $\sum_{i=1}^n\sum_{j=1}^m\sum_{d\mid \gcd(i,j)}\mu(d)$ ，我们直接对这个式子提取公因式
$$\begin{aligned}\sum_{i=1}^n\sum_{j=1}^m\sum_{d\mid \gcd(i,j)}\mu(d)&=\sum_{d=1}^{\min(n,m)}\sum_{i=1}^{\left\lfloor\frac{n}{d}\right\rfloor}\sum_{j=1}^{\left\lfloor\frac{m}{d}\right\rfloor}\mu(d)\\&=\sum_{d=1}^{\min(n,m)}\mu(d)\sum_{i=1}^{\left\lfloor\frac{n}{d}\right\rfloor}\sum_{j=1}^{\left\lfloor\frac{m}{d}\right\rfloor}\mathbf{1}\\&=\sum_{d=1}^{\min(n,m)}\mu(d)\left\lfloor\frac{n}{d}\right\rfloor\left\lfloor\frac{m}{d}\right\rfloor\end{aligned}$$

- 接下来直接用整除分块就可以了

```cpp
for(int l=1,r;l<=min(n,m);l=r+1){
	r=min(n/(n/l),m/(m/l));
	ans+=(sum[r]-sum[l-1])*(n/l)*(m/l);
}
```
- 证明 $\mu*\mathbf{1}=\varepsilon$ ：
- 设 $n=\prod_{i=1}^{k} p_{i}^{c_{i}},n'=\prod_{i=1}^{k}p_{i}$ ，因为 $\mu(p^{n})=0\;(n\ge 2)$ ，所以有 
$$\sum\limits_{d|n}\mu(d)=\sum\limits_{d|n'}\mu(d)=\sum\limits_{i=0}^{k} \binom{k}{i}\cdot(-1)^{i}=(1+(-1))^{k}$$
- 可知当且仅当 $k=0,n=1$ 时值为 $1$ 否则为 $0$ 

## 求 $1\le x\le n,1\le y\le m$ ，且 $\gcd(x,y)=k$ 的二元组数

- 我们求$\sum_{i=1}^n\sum_{j=1}^m\left[\gcd(i,j)=k\right]$ ，那么我们令 $i\rightarrow ki,\; j\rightarrow kj$ ，就可以得到
$$\sum_{i=1}^{\left\lfloor\frac{n}{k}\right\rfloor}\sum_{j=1}^{\left\lfloor\frac{m}{k}\right\rfloor}\varepsilon(\gcd(i,j))$$
- 同上处理即可

## 求 $\sum_{i=1}^n\sum_{j=1}^n\gcd(i,j)$ 

$$\begin{aligned}\sum_{i=1}^n\sum_{j=1}^n\gcd(i,j)&=\sum_{i=1}^n\sum_{j=1}^n \mathbf{Id}(\gcd(i,j))\\ &=\sum_{i=1}^n\sum_{j=1}^n(\mathbf{1}*\varphi)(\gcd(i,j))\\&=\sum_{i=1}^n\sum_{j=1}^n\sum_{d\mid gcd(i,j)}\varphi(d)\\&=\sum_{d=1}^n\sum_{i=1}^{\left\lfloor\frac{n}{d}\right\rfloor}\sum_{j=1}^{\left\lfloor\frac{n}{d}\right\rfloor}\varphi(d)\\&=\sum_{d=1}^n\left\lfloor\frac{n}{d}\right\rfloor^2\varphi(d)\end{aligned}$$

- 其中 $\mathbf{Id}=\mathbf{1}*\varphi$ ，既欧拉恒等式的证明
$$\begin{aligned}(\mathbf{1}*\varphi)(n)&=\sum_{d\mid n}\varphi(d)=n\end{aligned}$$
- 我们考虑集合 $\{1,2,...,n\}$ ，对于 $n$ 的每个正因数 $d$ ，定义集合 $S_d=\{k\in\{1,2,...,n\}:\gcd(k,n)=d\}$ ，显然 $\bigcup_{d\mid n}S_d=\{1,2,...,n\}$ ，且当 $d_1\ne d_2$ 时， $S_{d_1}\cap S_{d_2}=\varnothing$ ，所以 $n=\sum_{d\mid n}|S_d|$ 
- 若 $\gcd(k,n)=d$ ，我们让 $k=d\cdot m,\;n=d\cdot s$ ，那么 $\gcd(m,s)=1$ ，且 $1\le m\le s$ 
- 所以满足 $gcd(m,\frac{n}{d})=1$ 且 $1\le m\le \frac{n}{d}$ 的个数等于 $\gcd(k,n)=d$ 的个数，既 $|S_d|=\varphi(\frac{n}{d})$ 
- 我们带入 $n=\sum_{d\mid n}|S_d|$ ，得 $n=\sum_{d\mid n}\varphi(\frac{n}{d})$ ，我们再令 $\frac{n}{d}=t$ ，在 $d$ 遍历所有 $n$ 的正因数时 $t$ 也在遍历所有 $n$ 的正因数，所以得证

## $\sum_{i=1}^n \mathrm{lcm}(i,n)$ 

$$\begin{aligned}\sum_{i=1}^n \mathrm{lcm}(i,n)&=\sum_{i=1}^n\dfrac{i\cdot n}{\gcd(i,n)}\end{aligned}$$
- 我们复制一份可得：
$$\dfrac{1}{2}\left(\sum_{i=1}^{n-1}\dfrac{i\cdot n}{\gcd(i,n)}+\sum_{i=n-1}^1\dfrac{i\cdot n}{\gcd(i,n)}\right)+n=\dfrac{1}{2}\cdot \sum_{i=1}^n\dfrac{n^2}{\gcd(i,n)}+\dfrac{n}{2}$$
- 这样子我们就可以把相同的 $\gcd(i,n)$ 合并在一起算，同上一题，式子化为
$$\dfrac{1}{2}\cdot\sum_{d\mid n}\dfrac{n^2\cdot \varphi(\frac{n}{d})}{d}+\dfrac{n}{2}$$
- 我们再让 $d'=\dfrac{n}{d}$ ，作这样我们就可以提取公因式
$$\dfrac{1}{2}n\cdot\left(\sum_{d'\mid n}d'\cdot\varphi(d')+1\right)$$
- 我们设 $g(x)$ ，已知其为积性函数，所以可以 $O(n)$ 筛出
$$g(x)=\sum_{d\mid n}d\cdot\varphi(d)$$
- 让我们来推导一下：
- 先考虑 $g(p_j^k)$ 的值，它的约数只有 $p_j^0,p_j^1,...,p_j^k$ ，所以
$$g(p_j^k)=\sum_{w=0}^kp_j^w\cdot \varphi(p_j^w)=\sum_{w=0}^kp_j^{2w-1}\cdot (p_j-1)\quad \left(\varphi(p_j^w)=p_j^{w-1}\cdot (p_j-1)\right)$$
- 所以有
$$g(p_j^{k+1})=g(p_j^k)+p_j^{2k+1}\cdot (p_j-1)$$
- 那么对于线性筛中的 $g(i\cdot p_j)\quad(p_j\mid i)$ ，令 $(i=a\cdot p_j^w)\quad gcd(a,p_j)=1$ ，得
$$g(i\cdot p_j)=g(a)\cdot g(p_j^{w+1})$$
$$g(i)=g(a)\cdot g(p_j^w)$$
$$g(i\cdot p_j)-g(i)=g(a)\cdot p_j^{2w+1}\cdot (p_j-1)$$
$$g(i)-g\left(\dfrac{i}{p_j}\right)=g(a)\cdot p_j^{2w-1}\cdot (p_j-1)$$
$$g(i\cdot p)=g(i)+\left(g(i)-g\left(\dfrac{i}{p_j}\right)\right)\cdot p_j^2$$

## 求树上多少条简单路径，其点集满足 $\mathbf{lcm(V)}$ 刚好为 $x$

- 2025.7.18 航电第一场 1007
- 我们定义以下函数：
$$f(x):简单路径\;\mathbf{lcm}\;为\;x\;的约数的个数$$
$$g(x):简单路径\;\mathbf{lcm}\;为\;x\;的个数$$
- 我们可以写出一下式子：
$$f(x)=\sum\limits_{d|x} g(d)$$
- 那么就是最基础的莫比乌斯反演，得到
$$g(x)=\sum\limits_{d|x} \mu\left( \frac{x}{d} \right)f(d)$$
- 统计 $f(x)$ 是简单的，并查集然后统计连通块大小即可
- 式子反演即可




