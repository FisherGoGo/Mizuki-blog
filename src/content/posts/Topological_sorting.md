---
title: 拓扑排序
published: 2025-01-07
tags: [GRAPH]
category: Knowledge
draft: false
---


# 概念

- 在一个[DAG](https://oi.wiki/graph/dag/) 中，我们将图中的顶点线性排序，使得对于任何顶点 $u$ 到 $v$ 的有向边，都可以有 $u$ 在 $v$ 的前边
- 在 $DAG$ 中，如果 $i$ 到 $j$ 有边连接，那么 $j$ 依赖于 $i$ ，如果有路径抵达，那么称 $j$ 间接依赖于 $i$ ，那么拓扑排序就是使排在前边的节点不依赖于后边的节点

# AOV网

- 在日常生活中，一个大工程可以用若干个小工程表示，比如建房子要先画草图后打地基等，他们之间存在先后顺序，必须得完成一些子工程才能开始另外一些子工程
- 如果我们用有向图来表示他们的关系，就形成一张**AOV网**，**AOV网**必定是一张有向无环图
- **AOV网**用顶点表示活动，弧线表示先后关系，有向边的起点称为终点的**前驱活动**，终点称为起点的**后驱活动**

## AOE网

- **AOE网**是一个带权的有向无环图，它的边权经常用来表示活动的持续的时间，用**AOE网**可以用来估计事件的完成时间

# Kahn算法

- 我们用一个集合 $S$ 装所有**入度**为 $0$ 的节点，用 $L$ 来存拓扑排序结果
- 每次从 $S$ 中取出一个点 $u$ ，然后把点 $u$ 所有边 $(u,v_i)$ 删除，删除后将 $v_i$ 的入度减 $1$ ，如果操作后 $v_i$ 的入度为 $0$ ，那么将其放入集合 $S$ 
- 不断重复上述过程，直到集合 $S$ 为空，如果图中还有边，那么此图就不是一个DAG，否则集合 $L$ 中的放入顺序就是拓扑排序结果

## 代码
```cpp
void Kahn(){
	queue <int> q;
	int ans[N],tot=0;
	for(int i=1;i<=n;i++) if(!in[i]) q.push(i);
	while(!q.empty()){
		int u=q.front();
		q.pop();
		ans[++tot]=u;
		for(int i=head[u];i;i=edge[i].nxt){
			edge[i].del=1;
			in[edge[i].v]--;
			if(!in[edge[i].v]) q.push(edge[i].v);
		}
	}
	int flag=1;
	for(int i=1;i<=cnt;i++) if(edge[i].del==0) flag=0; //检查有没有未被删去的边
	if(flag==1){
		for(int i=1;i<=n;i++) cout<<ans[i];
	}
	return ;
}
```