---
title: 背包问题
published: 2024-11-18
tags: [DP]
category: Knowledge
draft: false
---


# 前言
- 背包问题，一般是对一堆有不同价值和不同体积（重量......）的物品，我们用有限空间的背包去装物品，去求最大价值。
- 背包问题从最简单的 **01背包** 开始，到 **完全背包** **多重背包** 等等
- [洛谷题单](https://www.luogu.com.cn/training/231055)

# 01背包

## 内容
- 一般是有 *n* 个价值为 $v_i$ 重量为 $w_i$ 的物品，且每个物品都只有一个，所以只有取与不取的选择，因此称为 __01背包__
## 做法
- 设数组 $f_{i,j}$ 为遍历到第 *i* 个物品，占用了 *j* 空间时最大价值
- 我们可以知道，对于第 *i* 个物品
   - 如果不取它，那么就有 $f_{i,j}=f_{i-1,j}$ 
   - 如果取它，那么就有 $f_{i,j}=f_{i-1,j-w_i}+v_i$
- 综上我们可以得出递推公式：$f_{i,j}=\max(f_{i-1,j} , f_{i-1,j-w_i}+v_i)$
- 并且我们在递推时要把 *j* 从 $w_i$ 遍历到 *W* (背包空间)
## 代码

<details>
  <summary>01背包</summary>

```
for(int i=1;i<=n;i++){
	for(int j=w[i];j<=W;j++){
		f[i][j]=max(f[i-1][j],f[i-1][j-w[i]]+v[i]);
	}
}
```

</details>

## 优化
- 我们可以发现，每次更新 $f_i$ ，只会用到 $f_{i-1}$ 的元素，因此我们可以使用一种叫做滚动数组的方法来优化空间，做到一维数组就可以用来更新最大价值，既 $f_i=max(f_i,f_{i-w_i}+v_i)$  

>注意我们在更新时要从后往前去更新，因为我们会发现每个 $f_i$ 都是由 $f_{i-w_i}$ 既之前的元素更新来的，如果我们先更新前边的元素，后边的元素更新就会被影响
### 代码

<details>
  <summary>优化</summary>

```cpp
for(int i=1;i<=n;i++){
	for(int j=W;j>=w[i];j--){
		f[j]=max(f[j],f[j-w[i]]+v[i]);
	}
}
```

</details>


# 完全背包

## 内容
- 完全背包就是 *01背包*  的升级版，每个物品从只有一个变成有无限个

## 做法
- 同理 *01背包*  ，但是递推公式变成了 $f_i=\max(f_i,f_{i-cnt*w_i}+cnt*v_i)$ ，*cnt* 为取的物品的个数

## 代码

<details>
  <summary>完全背包</summary>

```cpp
for(int i=1;i<=n;i++){
	for(int j=W;j>=w[i];j--){
		int x=j/w[i];
		for(int k=1;k<=x;k++){
			f[j]=max(f[j],f[j-k*w[i]]+k*v[i]);
		}
	}
}
```

</details>


# 多重背包

## 内容
- 同上，每个物品有有限个

## 做法
- 同理 *01背包*  ，但是递推公式变成了 $f_i=\max(f_i,f_{i-cnt*w_i}+cnt*v_i)$ ，*cnt* 为取的物品的个数

## 代码

<details>
  <summary>多重背包</summary>

```cpp
for(int i=1;i<=n;i++){
	for(int j=W;j>=w[i];j--){
		int x=min(c[i],j/w[i]);
		for(int k=1;k<=x;k++){
			f[j]=max(f[j],f[j-k*w[i]]+k*v[i]);
		}
	}
}
```

</details>


# 混合背包

## 内容
- *01背包*和*完全背包* 和 *多重背包* 的集合体，物品数列可能为**有限个**或者**无限个**

## 做法
- 递推公式同完全背包 $f_i=\max(f_i,f_{i-cnt*w_i}+cnt*v_i)$ ，但是 *cnt* 取决于物品数量

## 代码

<details>
  <summary>混合背包</summary>

```cpp
for(int i=1;i<=n;i++){
	for(int j=W;j>=w[i];j--){
		int x=0;
		if(c[i]==0) x=j/w[i];//如果有无限个
		else x=min(j/w[i],c[i]);
		for(int k=1;k<=x;k++){
			f[j]=max(f[j],f[j-k*w[i]]+k*v[i]);
		}
	}
}
```

</details>


# 分组背包 [洛谷 P1757](https://www.luogu.com.cn/problem/P1757)

## 内容

# 多维费用背包 [洛谷 P1507](https://www.luogu.com.cn/problem/P1507)

# 树形背包 [洛谷 P1997](https://www.luogu.com.cn/problem/P2014)

# 背包优化

## 二进制优化 [洛谷 P1776](https://www.luogu.com.cn/problem/P1776)
	




## 单调队列优化  [洛谷 P1776](https://www.luogu.com.cn/problem/P1776)
