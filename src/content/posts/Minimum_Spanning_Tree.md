---
title: 最小生成树
published: 2025-01-06
tags: [GRAPH]
category: Knowledge
draft: false
---

# 最小生成树 MST

- **MST**指边权合最小的生成树，而**生成树**指对于一个有 $n$ 个点的图，有 $n-1$ 条无向边，只有连通图有最小生成树
- 我们有两个算法求最小生成树，分别是 **Prim** 和 **kruskal**

# Prim

- **Prim**是一个**加点**的算法，从一个节点开始，不断扩展加点
- 每次从未加入的点中找到一个离已加入的点距离最短的点并加入
- 时间复杂度：暴力 $O(n^2+m)$ ，二叉堆优化 $O((n+m)\log n)$ 

## 代码
```cpp
struct S{
	int u,dis; //存点
};

bool operator < (const S &x,const S &y) {return x.dis>y.dis;} //小根堆

priority_queue <S> q;
int dis[Maxn],vis[Maxn];
int cnt=0,res=0;

void Prim(){
	memset(dis,0x3f3f,sizeof(dis));
	dis[1]=0;
	q.push({1,0});
	while(!q.empty()){
		if(cnt>=n) break;
		int u=q.top().u,d=q.top().dis;
		q.pop();
		if(vis[u]) continue;
		vis[u]=1;
		++cnt;
		res+=d;
		for(int i=head[u];i;i=edge[i].nxt){
			int v=edge[i].v,w=edge[i].w;
			if(w<dis[v]){
				dis[v]=w;
				q.push({v,w});
			}
		}
	}
}
```

# Kruskal

- 与**Prim**相反，**Kruskal**是一个加边的算法
- 不断找出最小的并且这条边所连的两个点是**不连通的**并加入，直到找到 $n-1$ 个边
- 使用 $O(m\log m)$ 的排序算法和 $O(m\log n)$ 的并查集，那么它的时间复杂度就是 $O(m\log m)$

## 代码
```cpp
struct edge{
	int u,v,w;
}

int fa[N];

int find(int x){
	return fa[x]==x?x:fa[x]=find(fa[x]);
}

void unio(int x,int y){
	x=find(x);
	y=find(y);
	if(x==y) return ;
	fa[y]=x;
}

bool cmp(edge A,edge B){
	return A.w<B.w;
}

void Kruskal(){
	int tot=0;
	int ans=0;
	for(int i=1;i<=n;i++) fa[i]=i;
	for(int i=1;i<=m;i++){
		int x=find(edge[i].u),y=find(edge[i].v);
		if(x!=y){
			unio(x,y);
			tot++;
			ans+=edge[i].w;
		}
		if(tot>=n-1) return ;
	}
}
```

# Kruskal 重构树

- 我们在跑**Kruskal**时从小到大加入若干条边，现在我们依旧按照这个顺序加入
- 不过我们在加入时，我们新建一个点，并把两个点都连到这个点上，并且把点权设为边权
- 这样子我们就构建了一棵新树，并且对于图上任意两点，他们的简单路径中**最大边权的最小值**就是这颗重构树上两点 $LCA$ 的点权
- 如果时求**最小边权的最大值**，我们就从大到小加入边

## 例题 [网络稳定性](https://www.luogu.com.cn/problem/P9235)
```cpp
#include<iostream>
#include<algorithm>
using namespace std;
const int N=300005;

int n,m,q;
int fa[N*2];
int point[N*2];
int newnode=0;

int find(int x){
    if(x!=fa[x]) return fa[x]=find(fa[x]);
    else return x;
}
void unio(int x,int y){
    x=find(x);
    y=find(y);
    fa[y]=x;
}

struct Edge{
    int u,v,nxt;
}edge[N*4];
int head[N*2],cnt;
void addedge(int u,int v){
    edge[++cnt]={u,v,head[u]};
    head[u]=cnt;
}

struct EEdge{
    int u,v,w;
}E[N];
bool cmp(EEdge A,EEdge B){
    return A.w>B.w;
}

int dep[N],Fa[N][32];

void dfs(int x,int f){
	dep[x]=dep[f]+1;
	Fa[x][0]=f;
	for(int i=1;(1<<i)<=dep[x];i++){
		Fa[x][i]=Fa[Fa[x][i-1]][i-1];
	}
	for(int i=head[x];i;i=edge[i].nxt){
		if(edge[i].v!=f) dfs(edge[i].v,x);
	}
}

int lca(int x,int y){
	if(dep[x]<dep[y]) swap(x,y);
	for(int i=31;i>=0;i--){
		if(dep[Fa[x][i]]>=dep[y]) x=Fa[x][i];
		if(x==y) return x;
	}
	for(int i=31;i>=0;i--){
		if(Fa[x][i]!=Fa[y][i]){
			x=Fa[x][i];
			y=Fa[y][i];
		}
	}
	return Fa[x][0];
}

int main(){
    cin>>n>>m>>q;
    newnode=n;
    for(int i=1;i<=2*n;i++) fa[i]=i;
    for(int i=1;i<=m;i++){
        cin>>E[i].u>>E[i].v>>E[i].w;
    }

    sort(E+1,E+m+1,cmp);

    for(int i=1;i<=m;i++){
        int u=E[i].u;
        int v=E[i].v,w=E[i].w;
        int rootu=find(u),rootv=find(v);
        if(rootu!=rootv){
            newnode++;
            // cout<<rootu<<" "<<rootv<<endl;
            addedge(rootu,newnode);
            addedge(newnode,rootu);
            addedge(rootv,newnode);
            addedge(newnode,rootv);
            unio(newnode,rootv);
            unio(newnode,rootu);
            point[newnode]=w;
        }
    }

    dep[0]=1;
    for(int i=newnode;i>n;i--){
        if(dep[i]==0){
            dfs(i,0);
        }
    }

    for(int i=1;i<=q;i++){
        int x,y;
        cin>>x>>y;
        if(find(x)!=find(y)) cout<<-1<<endl;
        else {
            int z=lca(x,y);
            cout<<point[z]<<endl;
        }
    }
}
```