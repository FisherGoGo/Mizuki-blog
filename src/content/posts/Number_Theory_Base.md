---
title: 数论基础
published: 2025-01-07
tags: [MATH]
category: Knowledge
draft: false
---

# 前言

- 这篇只算是自己的一个笔记，从此[视频](https://www.bilibili.com/video/BV1Zf4y1r7qE/?spm_id_from=333.999.top_right_bar_window_history.content.click&vd_source=f0c9904d89ec29a147dec0f6756bca3f)学习
- 本篇会从整除，取余等概念开始，涉及**GCD，LCM，裴蜀定理**等内容

# 整除&取余

- 若 $a\bmod b=0$ ，则称$b$整除$a$，写作 $b\mid a$  
- 若 $a\bmod b\ne 0$ ，则称$b$不整除$a$，写作 $b\nmid a$  
- 若 $a = bq + r$ ，则  $a \bmod b = r$
- 取余运算有以下性质：
	- $(a+b)\bmod m = ((a\bmod m)+(b\bmod m))\bmod m$
	- $(a\times b)\bmod m = ((a\bmod m)\times (b\bmod m))\bmod m$
	- $(a - b)\bmod m = ((a\bmod m)-(b\bmod m)+m)\bmod m$
	- 关于相除的模运算会在后边逆元处讲到

# 快速幂

- 一般计算幂运算，如$a^n$，时间复杂度是$O(n)$ ，效率太低了
- 因此我们有两种方法
- 第一种是将计算$a^n$转化为计算$a^{\tfrac{n}{2}} \times a^{\tfrac{n}{2}}$，用递归实现
	- 代码：
	 ```cpp
	 long long qpow(long long a,int n){
		 if(n==1) return a;
		 if(n==0) return 1;
		 long long temp=qpow(a,n/2);
		 if(n%2==0) return temp*temp;
		 else return temp*temp*a;
	 }
	  ```
- 另外一种不用递归的方法是我们可以把例如$a^{11}$转化为$a^8\times a^2\times a^1$
- 我们观察$11$的二进制为$1011$，那么就$a^{8*1}\times a^{4*0}\times a^{2*1}\times a^{1*1}$
	- 代码：
	```cpp
	long long qpow(long long a,int n){
		long long ans=1;
		while(n){
			if(n&1) ans*=a;
			a*=a;
			n>>=1;
		}
		return ans;
	}
    ```

# 最大公约数 GCD & 最大公倍数 LCM

- 我们将$a,b$的最大公约数记为 $\gcd(a,b)$ ，以下将简记为 $(a,b)$
- $\gcd(a,0)=a$，$\gcd(a,b)=\gcd(a,-b)$
- $\gcd(a,b,c,d)=\gcd(\gcd(a,b),\gcd(c,d))$
- 关于如何快速计算最大公约数，我们有两种解法
## 辗转相除法

- 第一种是辗转相除法 $\gcd(a,b) = \gcd(b,a\bmod b)$ 
- 证明：我们设$a=bq+c$ ，设$d$ ，$d\mid a$，$d\mid b$ ，所以就有 $d\mid (a-qb)$，既$d\mid c$，得证
- 代码：
```cpp
int gcd(int a,int b){
	if(b==0) return a;
	return gcd(b,a%b);
}
```
## 更相减损术

- 第二种为$\gcd(a.b) = \gcd(b,a-b)$ 
- 证明我们设 $d$ ，$d\mid a$ ，$d\mid b$ ，那么$d\mid (a-b)$ ，得证

## LCM

- $lcm(a,b)= \frac {a\times b} {\gcd(a,b)}$

# 裴蜀定理

- 对于任意正整数$a,b$，存在整数$x,y$，使$ax+by=\gcd(a,b)$成立，既方程$ax+by=(a,b)$有整数解
## 证明

- 若 $a=0$ ，$\gcd(a,b)=b$ ，那么 $0x+by=b$有无数组解
- 若都不为$0$
- 让我们证集合 $A=\{xa+yb;x,y\in \mathbb{Z} \}$ 的最小元素为$\gcd(a,b)$
- 我们设 $d_0 =x_0a + y_0b \in A$ 为最小元素
- 再设 $p=x_1a+y_1b \in A$
- 则$p$在模$d_0$意义的代数余子式为 $p=qd_0+r$ ，其中$0\le r <d_0$
- $r=p-qd_0=x_1a+y_1b-q(x_0a+y_0b)=(x_1-qx_0)a+(y_1-qy_0)b\in A$ 
- 又因为 $0\le r <d_0$，所以$r=0$
- 所以 $d_0 \mid p$
- 所以$A$中任意正元素都为$d_0$的倍数，那么 $d_0 \mid a$，$d_0\mid b$
- 所以 $d_0=\gcd(a,b)$，得证
## 定理一

- 若二元方程 $ax+by=c$有整数解 $x=x_0 \; y=y_0$ ，那么一切解可写为
$$\begin{cases} x=x_0-b_1t\\y=y_0+a_1t\end{cases}$$
- 其中$a_1=\frac{a}{\gcd(a,b)} \; b_1=\frac{b}{\gcd(a,b)}$
- 证明：
	- 充分性：将$a_1$与$b_1$代入原方程得 $ax_0+by_0+t(ba_1-b_1a)=c$ ，化简后得原式
	- 必要性：设 $a'x+b'y=c$ ，那么 $a(x'-x_0)+b(y'-y_0)=0$ ，将其中一项移到右边并同除以$\gcd(a,c)$ 得 $a_1(x'-x_0)=-b_1(y'-y_0)$，因为 $\gcd(a_1,b_1)=1$，所以 $(y'-y_0)\mid a_1 \; (x'-x_0)\mid -b_1$ ，转化后可得上式

## 定理二

- 二元一次方程 $ax+by=c$ 有解的充要条件为 $\gcd(a,b)\mid c$
- 证明：
	- 必要性：若有整数解，$(a,b)$ 可整除左右式，既 $(a,b)\mid c$
	- 充分性：若 $(a,b)\mid c$，根据裴蜀定理可知 $ax+by=(a,b)$ 有解，我们再根据定理一，可知原方程也有解

# 扩展欧几里得 EXGCD

- 用递归的方法来求解二元一次方程 $ax_1+by_1=(a,b)$
- 要求解原方程，可以先求解 $bx_2+(a\bmod b)y_2=(b,a\bmod b)$ 的解
- 其中 $x_1=y_2 \; y_1=x_2-\left\lfloor\dfrac{a}{b}\right\rfloor y_2$
## 证明

- $$ax_1+by_1=(a,b)=(b,a\bmod b)$$
- $$bx_2+(a\bmod b)y_2 = (b,a\bmod b)=bx_2+(a-\left\lfloor\dfrac{a}{b}\right\rfloor b)y_2$$
- $$ay_2+b(x_2-\left\lfloor\dfrac{a}{b}\right\rfloor y_2)=(a,b)$$
## 代码
```cpp
int exgcd(int a,int b,int &x,int &y){
	if(b==0){
		x=1;
		y=0;
		return a;
	}
	int d=exgcd(b,a%b,x,y);
	int temp=x;
	x=y;
	y=temp-a/b*y;
	return d;
}
```

# 同余与逆元

## 同余

- 若 $a\bmod m = b\bmod m$，则记作 $a\equiv b\pmod{m}$
- 若 $m\mid (a-b)$，则 $a\equiv b\pmod{m}$ ，将其代数余子式写出后可简单看出
### 性质
- 若 $a\equiv b\pmod{m}$，则 $(a,m)=(b,m)$
- 证明
 $$\begin{cases}(a,m)=(m,a\bmod m)=(m,c)\\(b,m)=(m,b\,\bmod m)=(m,c)\end{cases}$$
- 若 $a\equiv b\pmod{m}$，则$ak\equiv bk\pmod{m}$
- 证明
 $$\begin{cases}ak=kp_1m+kq\\bk=kp_2m+kq\end{cases} $$
 $$\begin{cases}ak\bmod m=kq\bmod m\\bk\bmod m=kq\bmod m\end{cases}$$
- 若 $a\equiv b\pmod{m}$，设 $d$，$d\mid a \ d\mid b$，$(d,m)=1$，则有$\frac{a}{d}\equiv \frac{b}{d}\pmod{m}$
## 同余式
- $f(x)$ 的系数为整数的多项式，$m$ 为正整数，将 $f(x) \equiv b\pmod m$ 为模 $m$ 同余式
- 我们基本上讨论一次同余式
 $$\begin{cases} ax\equiv b\pmod m\\ ax\not\equiv0\pmod{m}\end{cases}$$
- 使同余式成立的一个 $x$ 叫做同余式的一个解
- 一次同余式（线性同余方程）有解的充要条件是 $(a,m)\mid b$
- 证明： 
 $$ax=my+b \Rightarrow ax-my=b \Rightarrow ax+my=b$$
- 然后通过定理二可证

## 逆元

- 线性同余方程 $ax\equiv 1\pmod m$ $(a,m)=1$ 的一个解 $x$ 为 $a$ 模 $m$ 的逆元
- 应用：求除法的模，$Exgcd$，求方程 $ax+my=1$ 的解等
- 若我们要求 $\dfrac{a}{b} \bmod p$ ，我们设 $k$ 为 $b$ 的逆元，那么原式可变为 
 $$\left( \left(\dfrac{a}{b}\bmod m\right)\times \left(bk\bmod m\right)\right)\bmod m \Rightarrow ak\bmod m$$
- 其中 $bk\bmod m=1$ 

# 费马小定理

- 若 $p$ 是素数，那么 $a^p\equiv a\pmod p\Rightarrow a^{p-1}\equiv 1\pmod p$ 
- 应用：$a$ 模 $p$ 的一个逆元为 $a^{p-2}\bmod p$ ，可以用快速幂来算
## 证明

- 设集合 $\left\{1,2,3......p-1\right\}$ 模 $p$ 的值各不相同
- 再设 $a$ 不是 $p$ 的倍数，那么 $\left\{a,2a,3a......a(p-1)\right\}$ 模 $p$ 也互不相同
> 若 $x\not\equiv y\pmod p$ ，则 $ax\not\equiv ay\pmod p$ 
- 将其模 $p$ 后为 $(p-1)$ 的排列
- 所以 $(p-1)!\equiv (a\times 2\times a\times 3......\times (p-1)a)\pmod p$，将其左右都除以 $(p-1) !$ 可得上式

# 唯一分解定理

- 任意大于 $1$ 的整数可以被表示为质数的乘积既 
$$a=p_1\times p_2\times p_3......\times p_n$$
- 或者
$$a=p_1^{a_1}\times p_1^{a_2}\times p_3^{a_3}......\times p_n^{a_n}$$
- 那么
$$\gcd(a,b)=p_1^{\min(a_1,b_1)}\times p_2^{\min(a_2,b_2)}......\times p_n^{\min(a_n,b_n)}$$
$$lcm(a,b)=p_1^{\max(a_1,b_1)}\times p_2^{\max(a_2,b_2)}......\times p_n^{\max(a_n,b_n)}$$ 
## 代码

```cpp
int num[1005],cnt[1005];
void UD(int a){
	int Cnt=0;
	for(int i=2;i*i<=n;i++){
		if(n%i==0) Cnt++,num[Cnt]=i;
		while(n%i==0){
			n/=i;
			cnt[Cnt]++;
		}
	}
	if(n>1) num[++Cnt]=n;
	return ;
}
```
# 素数筛

- 线性筛（欧拉筛）是一种保证所有数都只被筛去一次的算法，所以时间复杂度因此降低为 $O(n)$ 
```cpp
int getprime(int n){
	for(int i=2;i<=n;i++){
		if(!notprime[i]) pri[++cnt]=i;
		for(int j=1;j<=cnt;j++){
			if(i*pri[j]>n) break;
			notprime[i*pri[j]]=1;
			if(i%pri[j]==0) break;
		}
	}
}
```
- 循环每个数，都跟比它小的质数做乘积并标记，直到这个质数能整除这个数
- 因为如果 $i\bmod pri_j =0$ ，说明 $i$ 是 $pri_j$ 的倍数，那么 $i$ 可以写成 $k\times pri_j$ ，那么 $i\times pri_{j+1},i\times pri_{j+2}.......i\times pri_{cnt}$ 都是 $pri_j$ 的倍数，在后边遍历时会被 $pre_j,k\times pri_{j+1}......k\times pri_{cnt}$ 给筛掉，所以 $break$ 掉，那么每个数只会被其最小质因数筛去
# 欧拉函数

- 定义 $\varphi (n)$ 表示的是小于等于 $n$ 且与 $n$ 互质的数的个数，其中 $\varphi(1)=1$ ，$\varphi(p)=p-1$ ，$p$ 为质数
- 若 $(a,b)=1$ ，那么 $\varphi(a\times b)=\varphi(a)\times \varphi(b)$   
- 公式：
$$\varphi(n)=n\times \prod_{i=1}^n \dfrac{p_i-1}{p_i}$$
其中根据唯一分解定理
$$n=\prod_{i=1}^s p_i^{k_i}$$
- 证明：根据积性可知
$$\begin{aligned}\varphi(n)&=\prod_{i=1}^s \varphi(p_i^{k_i}) \\ &=\prod_{i=1}^s (p_i-1)\times p_i^{k_i-1}\\ &=\prod_{i=1}^s p_i^{k_i}\times(1-\frac{1}{p_i}) \\&=n\prod_{i=1}^s (1-\frac{1}{p_i})\end{aligned}$$

## 代码

- 求单个欧拉函数的值：按照公式来
```cpp
int euler_phi(int n){
	int ans=n;
	int mx=int(sqrt(n+0.5));
	for(int i=2;i<=mx;i++){
		if(n%i==0){
			ans=ans/i*(i-1);
			while(n%i==0) n/=i;
		}
	}
	if(n>1) ans=ans/n*(n-1);
	return ans;
}
```
- 批量处理欧拉函数：欧拉筛法
```cpp
int phi[maxn],vis[maxn],cnt=0;
int euler_phi(int n){
	phi[1]=1;
	for(int i=2;i<=m;i++){
		if(!vis[i]){
			phi[i]=i-1;
			pri[++cnt]=i;
		}
		for(int j=1;j<=cnt;j++){
			if(i*pri[j]>n) break;
			vis[i*pri[j]]=1;
			if(i%pri[j]!=0) phi[i*pri[j]]=phi[i]*(pri[j]-1);
			else{
				phi[i*pri[j]]=phi[i]*pri[j];
				break;
			}
		}
	}
}
```