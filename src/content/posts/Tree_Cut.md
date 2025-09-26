---
title: 树链剖分
published: 2025-01-10
tags: [DS,GRAPH]
category: Knowledge
draft: false
---

# 前言

- 树链剖分用于将树分割成若干条链的形式，以维护树上路径的信息
- 具体来说，将整棵树剖分为若干条链，使它组合成线性结构，然后用其他的数据结构维护信息
- **树链剖分**有多种形式，如**重链剖分**、**长链剖分**或者用于**Link-Cut-Tree**的剖分，大多数情况下如没有特别说明，都是指**重链剖分**
- **重链剖分**可以将树上任意一条路径划分为不超过 $O(\log n)$ 条连续的链，每条链上的点深度互不相同，链上所有点的 $LCA$ 为链的一个端点
- 还可以保证划分出来的每一条链上的节点 $DFS$ 序连续，因此可以方便地用一些维护序列的数据结构（如线段树）来维护树上路径的信息
- 比如：修改树上两点之间路径上所有点的值、查询树上两点之间路径上节点权值的**和/极值/等等**
- 它还可以在 $O(\log n)$ 求$LCA$ 

# 重链剖分

## 定义

- 重子节点：子结点中子树最大的节点，如果有多个就取一个
- 轻子节点：子节点中除重子节点外其他节点
- 重边：节点到它重子节点的边
- 轻边：节点到它轻子节点的边
- 重链：若干条重边首位衔接，落单的节点也被当成重链

## 实现

- 用两次 $DFS$ 

## 代码&定义

- $fa_x$ 节点 $x$ 的父亲
- $dep_x$ 节点 $x$ 在树上的深度
- $siz_x$ 节点 $x$ 子树节点个数
- $son_x$ 节点 $x$ 的重儿子
- $top_x$ 节点 $x$ 所在重链顶部节点
- $dfn_x$ 节点 $x$ 的$DFS$ 序
- $rnk_x$ 节点 $x$ 的 $DFS$ 序所对应的节点编号，有 $rnk(dfn(x))=x$ 

### 第一次DFS

- 记录每个结点的父节点、深度、子树大小、重子节点
```cpp
void dfs1(int x,int f,int deep){
    dep[x]=deep;
    fa[x]=f;
    son[x]=-1;
    siz[x]=1;
    for(int i=head[x];i;i=edge[i].nxt){
        int v=edge[i].v;
        if(v==f) continue;
        // cout<<x<<" "<<v<<endl;
        dfs1(v,x,deep+1);
        siz[x]+=siz[v];
        if(son[x]==-1||siz[v]>siz[son[x]]) son[x]=v;
    }
    return ;
}
```

### 第二次DFS

- 记录所在链链顶、重边优先遍历时的 $DFS$ 序、$DFS$ 序所对应编号
```cpp
void dfs2(int x,int t){
    top[x]=t;
    Cnt++;
    rnk[x]=Cnt;
    num[Cnt]=a[x];
    if(son[x]==-1) return ;
    dfs2(son[x],t);
    for(int i=head[x];i;i=edge[i].nxt){
        int v=edge[i].v;
        if(v!=son[x]&&v!=fa[x]) dfs2(v,v);
    }
    return ;
}
```

## 性质

- 树上每个节点都属于且仅属于一条重链
- 重链开头的节点不一定是重子节点
- 所有重链将树完全剖分
- 在剖分时重边优先排序，最后树的 $DFS$ 序上重链的 $DFS$ 序是连续的，按 $DFN$ 排序后的序列即位剖分后的链
- 一颗子树内的 $DFS$ 序是连续的
- 当我们向下经过一条轻边时，子树大小至少除以二，因此树上每条路都会被拆分成不超过 $O(\log n)$ 条重链

# 应用

## 路径上维护

- 求树上两点间路径权值和
```cpp
int qRange(int x,int y){
	int ans=0;
	while(top[x]!=top[y]){
		if(dep[top[x]]<dep[top[y]]) swap(x,y);
		res=0;
		res=query(1,1,n,rnk[top[x]],rnk[x]);
		ans+=res;
		ans%=mod;
		x=fa[top[x]];
	}
	if(dep[x]>dep[y]) swap(x,y);
	res=0;
	res=query(1,1,n,rnk[x],rnk[y]);
	ans+=res;
	return ans%mod
}
```

- 修改路径上权值
```cpp
void updRange(int x,int y,int k){
	k%=mod;
	while(top[x]!=top[y]){
		if(dep[top[x]]<dep[top[y]]) swap(x,y);
		update(1,1,n,rnk[top[x]],rnk[x],k);
		x=fa[top[x]];
	}
	if(dep[x]>dep[y]) swap(x,y);
	update(1,1,n,rnk[x],rnk[y],k);
}
```

## 子树维护

- 求子树权值和
```cpp
int qSon(int x){
	return query(1,1,n,rnk[x],rnk[x]+siz[x]-1);
}
```

- 修改子树
```cpp
void updSon(int x,int k){
	update(1,1,n,rnk[x],rnk[x]+siz[x]-1,k);
}
```

## 求LCA

- 不断向上跳重链，当跳到同一条重链上时，深度较小的结点即为 $LCA$ 
```cpp
int lca(int x,int y){
	while(top[x]!=top[y]){
		if(dep[top[x]]>dep[top[y]]) x=fa[top[x]];
		else y=fa[top[y]];
	}
	return dep[x]>dep[y]?y:x;
}
```