---
title: 最短路
published: 2025-01-06
tags: [GRAPH]
category: Knowledge
draft: false
---

# 前言

- 字面意思，最短路算法就是求最短路径的算法，一般有**Dijkstra**、**Spfa**和**Floyd**三种

# Floyd

- **Floyd**算法主要用动态规划的思想来求多源最短路问题，它可以求出任意一点 $x$ 到任意一点 $y$ 的最短距离，它同样可以处理负权边
- 它的时间复杂度是 $O(n^3)$ 
- 他要运用邻接矩阵，对每两个点，进行 $n$ 次迭代更新，找出最短距离，它的状态方程是
$$f_{i,j}=\min(f_{i,j},f_{i,k}+f_{k,i})$$

## 代码
```cpp
void Floyd(){
	for(int i=1;i<=n;i++){
		for(int j=1;j<=n;j++){
			if(i==j) dis[i][j]=0;
			else if(!dis[i][j]) dis[i][j]=INF;
		}
	}

	for(int k=1;k<=n;k++){
		for(int i=1;i<=n;i++){
			for(int j=1;j<=n;j++){
				dis[i][j]=min(dis[i][j],dis[i][k]+dis[k][j]);
			}
		}
	}
}
```

# Dijkstra

- **Dij**主要贪心加点来求最短路
- 时间复杂度 $O(n^2)$
- 我们从起点出发，依次遍历与起点相连的每一个点，更新他们与起点的距离
- 然后再选择一个与起点距离最短的点，再去更新与之相邻的点
- 这样由于每次都选择的是最优解（即最短路）所以，每个点都遍历完之后，每个点上面的数值，就是与起点的最短距离

## 代码
```cpp
void Dij(){
	for(int i=1;i<=n;i++) dis[i]=INF,vis[i]=0;
	dis[1]=0;
	while(1){
		int k=-1,len=INF;
		for(int i=1;i<=n;i++){
			if(!vis[i]&&len>dis[i]){
				len=dis[i];
				k=i;	
			}
		}
		if(k==-1) break;
		vis[k]=1;
		for(int i=head[k];i;i=edge[i].nxt){
			int v=edge[i].v;
			if(!vis[v]&&dis[v]>len+edge[i].w) dis[v]=len+edge[i].w;
		}
	}
}
```

## 优化

- 使用优先队列优化，使时间复杂度达到 $O(m\log m)$ 
```cpp
struct node{
	int dis,u;
	bool operator > (const node& a) const{return dis>a.dis;}
};

int dis[N],vis[N];
priority_queue <node,vector<node>,greater<node>> q;

void Dij(){
	for(int i=1;i<=n;i++) dis[i]=0x3f,vis[i]=0;
	dis[s]=0;
	q.push({0,s});
	while(!q.empty()){
		int u=q.top().u;
		q.pop();
		if(vis[u]) continue;
		vis[u]=1;
		for(int i=head[u];i;i=edge[i].nxt){
			int v=edge[i].v,w=edge[i].w;
				if(dis[v]>dis[u]+w){
					dis[v]=dis[u]+w;
					q.push({dis[v],v});
				}
		}
	}
}
```

# Spfa

- **Spfa**主要用队列的性质
- 时间复杂度 $O(n\log n)$ - $O(n^2)$ 
- 从起点开始，每次将与他相邻的点遍历，判断这个点是不是再队列中，如果不在就加入队列，在队列中就只更新最小值

## 代码
```cpp
void Spfa(){
	queue <int> q;
	for(int i=1;i<=n;i++) dis[i]=INF;
	q.push(1);
	dis[1]=0;
	inq[1]=1;
	while(!q.empty()){
		int u=q.front();
		q.pop();
		inq[u]=0;
		for(int i=head[u];i;i=edge[i].nxt){
			int v=edge[i].v;
			if(dis[v]>dis[u]+edge[i].w){
				dis[v]=dis[u]+edge[i].w;
				if(inq[v]==0){
					q.push(v);
					inq[v]=1;
				}
			}
		}
	}
}
```


# Johnson

- **Johnson**与**Floyd**一样，是一种能求出无负环图上任意两点间最短路径的算法
- 一般来说我们可以跑 $n$ 次**Dijkstra**来求全源最短路，时间复杂度为 $O(nm\log m)$ ，但是**Dijkstra**不能处理带负权边的最短路，所以我们要对图中的边进行预处理，来使所有边权不为负
- 我们新建一个虚拟节点，从这个点向其他点连一条边权为 $0$ 的边，然后我们用**SPFA**算出所有点到这个虚拟节点的最短路，记为 $h_i$ 
- 然后我们把从 $u$ 到 $v$ 的边权为 $w$ 的边的边权改为 $w+h_u-h_v$ 
- 然后我们以每一个点为起点跑一边**Dijkstra**
- [例题 洛谷P5905 全源最短路](https://www.luogu.com.cn/problem/P5905)