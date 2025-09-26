---
title: 最近公共祖先
published: 2025-01-10
tags: [GRAPH]
category: Knowledge
draft: false
---


# 定义

- 最近公共祖先称为**LCA**，就是两个节点的公共祖先中，离根节点最远的那个
- 我们记点集 $S=\left\{v_1,v_2,v_3...v_n\right\}$ 的最近公共祖先为 $LCA\left(v_1,v_2,v_3...v_n\right)$ 或者 $LCA(S)$ 

# 性质

- $LCA(u)=u$ 
- 如果 $u$ 是 $v$ 的祖先，那么 $LCA(u,v)=u$ 
- 如果 $u$ 不是 $v$ 的祖先，且 $v$ 也不是 $u$ 的祖先，那么 $v$ 和 $u$ 位于 $LCA(u,v)$ 的两棵不同子树上
- 前序遍历中 $LCA(S)$ 出现在所有 $S$ 元素之前，后序遍历则在之后
- $LCA(A\cup B)=LCA(LCA(A),LCA(B))$ 
- 两点的 $LCA$ 必定在两点最短路径上
- $d(u,v)=h(u)+h(v)-2h(LCA(u,v))$ ，$d$ 是两点间距离，$h$ 是到根节点的距离

# 倍增算法

- 倍增算法是最经典的 $LCA$ 算法
- 基础是我们先找深度较大的那个点，然后向上跳直到与另外一个点深度相同，然后再同时向上跳，它们最后一定会相遇，相遇点就是最近公共祖先
- 我们通过预处理 $fa_{x,i}$ 数组来快速移动，大幅减少移动跳跃次数，$fa_{x,i}$ 表示点 $x$ 的第 $2^i$ 个祖先，它可以通过 $DFS$ 预处理出来
- 我们先将 $x$ 设为深度较大的那个节点，然后让其上跳直到与 $y$ 接近
- 然后再同时上跳 $x$ 和 $y$ ，直到跳到同一点

## 代码

```cpp
int dep[N],fa[N][32];

void dfs(int x,int Fa){
	dep[x]=dep[Fa]+1;
	fa[x][0]=Fa;
	for(int i=1;(1<<i)<=dep[x];i++){
		fa[x][i]=fa[fa[x][i-1]][i-1];
	}
	for(int i=head[x];i;i=edge[i].nxt){
		if(edge[i].v!=Fa)
		dfs(edge[i].v,x);
	}
}

int lca(int x,int y){
	if(dep[x]<dep[y]) swap(x,y);
	for(int i=31;i>=0;i--){
		if(dep[fa[x][i]]>=dep[y]) x=fa[x][i];
		if(x==y) return x;
	}
	for(int i=31;i>=0;i--){
		if(fa[x][i]!=fa[y][i]){
			x=fa[x][i];
			y=fa[y][i];
		}
	}
	return fa[x][0];
}
```

