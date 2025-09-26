---
title: 网络流
published: 2025-05-12
tags: [GRAPH]
category: Knowledge
draft: false
---

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=2608101502&auto=0&height=66"></iframe>

# 前置准备

- [学习来源](https://www.cnblogs.com/alex-wei/p/network_flow_bipartite_graph_and_graph_matching.html)
- 一个网络是一张**有向图** $G=(V,E)$ ，对于每条有向边 $(u,v)\in E$ ，存在流量限制 $c(u,v)$ ，如果 $(u,v)\notin E$ ，那么 $c(u,v)=0$ 
- 我们再引入**流函数** $f(u,v)$ ，其中 $(u,v)\in E$ ，成为其边的流量
- 有一下性质：
	-  $f(u,v)\le c(u,v)$ ，若 $f(u,v)=c(u,v)$ 我们称其满流
	- $f(u,v)=-f(v,u)$ ，既**斜对称性**
	- $\sum\limits f(u,i)=\sum\limits f(i,v)$ ，对于所有 $i$ ，既**流量守恒性**
- 对于网络流还有以下定义：
	- **残量网络**：若已有流量在网络 $G$ 上，我们定义残量网络为 $G_{f}=(V,E_{f})$ 为容量函数等于 $c_{f}=c-f$ 的网络，如果一条边的容量为 $0$ ，那么在网络中删去这条边
	- **增广路**：在残量网络 $G_{f}$ 上从源点 $S$ 到汇点 $T$ 的一条路径
	- 将 $V$ 分成为互不相交的点集 $A,B$ ，其中 $S\in A,T\in B$ ，这种方式叫做**割**，定义割的**容量**为 $\sum\limits_{u\in A}\sum\limits_{v\in B}c(u,v)$ ，同理**流量**为 $\sum\limits_{u\in A}\sum\limits_{v\in B} f(u,v)$ ，如果一条边 $(u,v)\;\; u\in A,v\in B$ ，那么称其为**割边**
# 网络最大流

- 既给定网络 $G=(V,E)$ ，求**最大流量**，既
$$\max \sum\limits f(u,v)$$

## 增广

- 网络流算法主要用到了**不断寻找增广路**与**能流满就流满**的算法
- 我们在残量网络中找到一条增广路 $P$ ，并为 $P$ 上每一条边增加 $c_{f}(P)=\min\limits_{(u,v)\in P}c_{f}(u,v)$ 的流量
- 我们在增广时尽量流满一条增广路，同时每条边的流量在增广时不会减少，我们怎么确保贪心正确
- 我们在给 $(u,v)\in P$ 加流量 $c_{f}(P)$ 时，需要给其反边的容量加上 $c_{f}(P)$ ，这样我们就可以**支持反悔**
- 以上的过程称为一次增广

## 最大流最小割定理

- **最大流等于最小割**
- 证明：
- 任意一组流的流量**不大于**任意一组割的容量：
	- 对于每单位流量，设其经过 $u\in A,v\in B$ 的割边 $u\rightarrow v$ 的次数为 $to$ ，经过 $v\rightarrow u$ 的割边次数为 $back$ ，那么必有 $to=back+1$ ，否则不能从 $S$ 流到 $T$ 
	- 每单位流量对割边流量之和的定义为 $to-back=1$ ，既网络流总量等于割边流量之和
	- 我们也可以推出： $流的流量 \le 割的容量$ 
- 存在一组流的流量**等于**割的容量：
	- 当最大流存在时，残量网络不连通，否则我们可以继续增广

## $\mathrm{Edmonds-Karp}$

- 核心是用 $\mathrm{bfs}$ 寻找长度最短的增广路
- 我们记录流向每个点的边的编号，然后从汇点不断反推到源点
- 时间复杂度 $O(nm^2)$ 

```cpp
struct flow{
    ll fl[N],limit[M*2];
    int cnt=1,head[N],nxt[M*2],to[M*2],fr[N];
    void add(int u,int v,int w){
        nxt[++cnt]=head[u],head[u]=cnt,to[cnt]=v,limit[cnt]=w;
        nxt[++cnt]=head[v],head[v]=cnt,to[cnt]=u,limit[cnt]=0;
    }
    ll maxflow(int s,int t){
        ll flow=0;
        while(1){
            queue <int> q;
            memset(fl,-1,sizeof(fl));
            fl[s]=1e18;
            q.push(s);
            while(!q.empty()){
                int u=q.front();
                q.pop();
                for(int i=head[u];i;i=nxt[i]){
                    int v=to[i];
                    if(limit[i]&&fl[v]==-1){
                        fl[v]=min(limit[i],fl[u]);
                        fr[v]=i;
                        q.push(v);
                    }
                }
            }
            if(fl[t]==-1) return flow;
            flow+=fl[t];
            for(int u=t;u!=s;u=to[fr[u]^1]){
	            limit[fr[u]]-=fl[t];
	            limit[fr[u]^1]+=fl[t];
            }
        }
    }
};
```

## $\mathrm{Dinic}$

- 核心思想是**分层图**和**相邻层之间增广**，通过 $\mathrm{bfs}$ 和 $\mathrm{bfs}$ 实现
- 我们首先给 $\mathrm{bfs}$ 图分层，然后从 $S$ 开始 $\mathrm{dfs}$ **多路增广**，维护当前节点和剩余流量，向下一层节点继续流
- 给图分层的目的是将网络转化成一个 $\mathrm{DAG}$ ，防止流成一个环
- 还有一个重要的**当前弧优化**：
	- 增广时，容量等于流量，既满流的边无用，可以直接跳过，所以我们不需要每次搜索到同一个点时都从邻接表头开始遍历
	- 所以我们记录每个点出发第一条没有流满的边，称为**当前弧**，每次搜索到一个节点就从当前弧开始增广
	- 注意，每次多路增广时每个点的当前弧应初始化为邻接表头，因为并非一但满流，这条边就完全无用，因为反向边流量的增加会让它重新出现在参量网络中
- 加了当前弧优化后的复杂度为 $O(n^2m)$ ，否则会退化至 $O(nm^2)$ 

```cpp
struct flow{
    ll fl[N],limit[M*2];
    int cnt=1,head[N],nxt[M*2],to[M*2],fr[N];
    void add(int u,int v,int w){
        nxt[++cnt]=head[u],head[u]=cnt,to[cnt]=v,limit[cnt]=w;
        nxt[++cnt]=head[v],head[v]=cnt,to[cnt]=u,limit[cnt]=0;
    }
    int T,dis[N],cur[N];
    ll dfs(int u,ll res){
        if(u==T) return res;
        ll flow=0;
        for(int i=cur[u];i&&res;i=nxt[i]){
            cur[u]=i;
            int c=min(res,(ll)limit[i]);
            int v=to[i];
            if(dis[u]+1==dis[v]&&c){
                int k=dfs(v,c);
                flow+=k;
                res-=k;
                limit[i]-=k;
                limit[i^1]+=k;
            }
        }
        if(!flow) dis[u]=-1;
        return flow;
    }
    ll maxflow(int s,int t){
        T=t;
        ll flow=0;
        while(1){
            queue <int> q;
            memcpy(cur,head,sizeof(head));
            memset(dis,-1,sizeof(dis));
            q.push(s);
            dis[s]=0;
            while(!q.empty()){
                int u=q.front();
                q.pop();
                for(int i=head[u];i;i=nxt[i]){
                    if(dis[to[i]]==-1&&limit[i])
                        dis[to[i]]=dis[u]+1,q.push(to[i]);
                }
            }
            if(dis[t]==-1) return flow;
            flow+=dfs(s,1e18);
        }
    }
};
```

# 费用流

- 一般是最小费用最大流，既在**保证最大流**的情况下，求出下式最小值
$$\sum\limits_{(x,y)\in E}f(x,y)\times w(x,y)$$

## $\mathrm{SSP}$

- 连续最短路算法
- 核心是每次找到**长度最短的增广路**进行增广，且仅在网络**初始无负环时**能得到正确答案
- 一般基于 $\mathrm{EK}$ 或者 $\mathrm{Dinic}$ 实现，两者都要求将 $\mathrm{bfs}$ 换成 $\mathrm{SPFA}$ ，每条边的长度为 $w$ ，且 $\mathrm{Dinic}$ 的 $\mathrm{dfs}$ 多路增广仅在 $dis_x+w(x,y)=dis_y$ 之间的边进行
- 要退掉反边的费用，既对于每条边 $(x,y)$ ，其反边的权值应为 $w(y,x)=-w(x,y)$
- 时间复杂度为 $O(nmf)$ ， $f$ 为最大流量，并且实际中增广次数远达不到 $f$ ，同时 $\mathrm{SPFA}$ 的复杂度也远达不到 $nm$
- 一般基于 $\mathrm{EK}$ 来实现 $\mathrm{SSP}$
- 注意，$\mathrm{SPFA}$ 在队首为 $T$ 时不能直接 $break$，因为第一次取出 $T$ 时 $dis[T]$ 不一定取到最短路。

```cpp
struct flow {
    int cnt=1,head[N],nxt[M*2],to[M*2],limit[M*2],cst[M*2];
    void add(int u,int v,int w,int c){
        nxt[++cnt]=head[u],head[u]=cnt,to[cnt]=v,limit[cnt]=w,cst[cnt]=c;
        nxt[++cnt]=head[v],head[v]=cnt,to[cnt]=u,limit[cnt]=0,cst[cnt]=-c;
    }
    int fr[N],fl[N],in[N],dis[N];
    pair<int,int> mincost(int s,int t){
        int flow=0,cost=0;
        while(1){
            queue<int> q;
            memset(dis,0x3f,sizeof(dis));
            memset(in,0,sizeof(in));
            fl[s]=1e9,dis[s]=0,q.push(s);
            while(!q.empty()){
                int u=q.front();
                q.pop();
                in[u]=0;
                for(int i=head[u];i;i=nxt[i]){
                    int v=to[i],d=dis[u]+cst[i];
                    if(limit[i]&&d<dis[v]){
                        fl[v]=min(limit[i],fl[u]),fr[v]=i,dis[v]=d;
                        if(!in[v]) in[v]=1,q.push(v);
                    }
                }
            }
            if(dis[t]>1e9) return make_pair(flow,cost);
            flow+=fl[t],cost+=dis[t]*fl[t];
            for(int u=t;u!=s;u=to[fr[u]^1]){
                limit[fr[u]]-=fl[t];
                limit[fr[u]^1]+=fl[t];
            }
        }
    }
};
```

# 上下界网络流

- 相对于原始网络 $G$ ，我们多了一个属性：流量下届 $b(u,v)$ ，既我们每条边的流量要满足： $b(u,v)\le f(u,v)\le c(u,v)$

## 无源汇可行流

- 既对于一张没有源点和汇点的图，我们找出一个流函数 $f$ 使得满足所有流量限制、斜对称遗迹流量守恒限制
- 解决这类问题的核心是**先满足下界在尝试调整**
- 具体就是我们先让每条边都**流满下界**，在算出每个点的净流量 $w_{i}=\sum\limits f(u,i)-\sum\limits f(i,u)$ ，若 $w_{i}>0$ 时，说明流到点 $i$ 的流量太多了，还要再流出去 $w_{i}$ 才能守恒，反之就是要流进 $w_{i}$ 才能守恒。因为 $\sum\limits w_{i}=0$ ，所以有 $\Delta=\sum\limits_{w_{i}>0} w_{i}=\sum\limits_{w_{i}<0} -w_{i}$
- 对于此问题，我们新建一个网络 $G'\approx G$ ，其每条边的流量限制变为 $c'=c-f=c-b$ ，我们再建立超级源点 $TT$ 和超级汇点 $SS$ ；若 $w_i>0$ 我们连一条容量为 $w_{i}$ 的边 $SS\rightarrow i$ ，反之连一条容量为 $w_{i}$ 的边 $i\rightarrow TT$ ，不难发现从 $SS$ 流出了 $\Delta$ 的容量，流入 $TT$ 的容量为 $\Delta$ 
- 若 $SS\rightarrow TT$ 的最大流不等于 $\Delta$ ，既无解；反之说明 $f_{cur}(u,v)=b(u,v)+f'(u,v)$ 合法，且满足 $b\le f_{cur}\le c$
- 若题目是**有源汇可行流**，我们从 $T\rightarrow S$ 连一条容量为 $+\infty$ 的边即可，转化为无源汇可行流

```cpp
struct flow{
    int cnt=1,head[N],nxt[M*2],to[M*2],limit[M*2];
    void add(int u,int v,int w){
        nxt[++cnt]=head[u],head[u]=cnt,to[cnt]=v,limit[cnt]=w;
        nxt[++cnt]=head[v],head[v]=cnt,to[cnt]=u,limit[cnt]=0;
    }
    int cur[N],dis[N],T;
    int dfs(int u,int res){
        if(u==T) return res;
        int flow=0;
        for(int i=cur[u];i&&res;i=nxt[i]){
            cur[u]=i;
            int c=min(res,limit[i]),v=to[i];
            if(dis[u]+1==dis[v]&&c){
                int k=dfs(v,c);
                flow+=k,res-=k,limit[i]-=k,limit[i^1]+=k;
            }
        }
        if(!flow) dis[u]=-1;
        return flow;
    }
    int maxflow(int s,int t){
        T=t;
        int flow=0;
        while(1){
            queue <int> q;
            memcpy(cur,head,sizeof(head));
            memset(dis,-1,sizeof(dis));
            q.push(s),dis[s]=0;
            while(!q.empty()){
                int u=q.front();
                q.pop();
                for(int i=head[u];i;i=nxt[i]){
                    if(dis[to[i]]==-1&&limit[i]){
                        dis[to[i]]=dis[u]+1;
                        q.push(to[i]);
                    }
                }
            }
            if(dis[t]==-1) return flow;
            flow+=dfs(s,1e9);
        }
    }
};

struct network_flow{
    int cnt=0,u[M],v[M],limit[M],low[M];
    void add(int _u,int _v,int w,int b){
        u[++cnt]=_u,v[cnt]=_v;
        limit[cnt]=w;
        low[cnt]=b;
    }
    flow g;
    int maxflow(int n,int ss,int tt){
        int w[N];
        memset(w,0,sizeof(w));
        for(int i=1;i<=cnt;i++){
            w[u[i]]-=low[i],w[v[i]]+=low[i];
            g.add(u[i],v[i],limit[i]-low[i]);
        }
        int tot=0;
        for(int i=1;i<=n;i++){
            if(w[i]>0) g.add(ss,i,w[i]),tot+=w[i];
            else if(w[i]<0) g.add(i,tt,-w[i]);
        }
        return g.maxflow(ss,tt)-tot;
    }
}f;


void solve(){
    cin>>n>>m;
    for(int i=1;i<=m;i++){
        int u,v,w,b;
        cin>>u>>v>>b>>w;
        f.add(u,v,w,b);
    }
    if(f.maxflow(n,0,n+1)!=0) cout<<"NO";
    else {
        cout<<"YES\n";
        for(int i=1;i<=m;i++) cout<<f.low[i]+f.g.limit[(i*2)^1]<<"\n";
    }
}
```

## 有源汇最大流

- 此算法基于一个重要结论：
- 给定**任意**一组**可行流**，对其进行最大流算法，我们总能得到正确的最大流
- 因为最大流算法本身**支持撤销**，所以无论初始流函数如何，只要**合法**，我们总能正确求出最大流
- 所以进行以下步骤：
- 先对于网络 $G$ 求出有源汇**可行流**，新建出网络 $G'$ ，然后我们撤去 $SS,TT$ 和 $T\rightarrow S$ 的容量为 $+\infty$ 的边，那么此时我们得到了一个有源可行流，如果无有源可行流，说明无解；若要获取当前流量， $T\rightarrow S$ 的反边 $S\rightarrow T$ 即位所求
- 接下来**二次调整**，我们只要以 $S$ 为源 $T$ 为汇在 $G'$ 跑一边最大流，再将可行流流量与最大流流量相加即位答案
- 如果是求**有源汇最小流**，我们根据 $S\rightarrow T$ 的最小流等于 $T\rightarrow S$ 的最大流的相反数这一结论，我们用可行流流量减掉 $G'$ 上 $T\rightarrow S$ 的最大流

 ```cpp
struct graph{
    int cnt=1,hd[N],nxt[M*2],to[M*2],limit[M*2];
    void add(int u,int v,int w){
        nxt[++cnt]=hd[u],hd[u]=cnt,to[cnt]=v,limit[cnt]=w;
        nxt[++cnt]=hd[v],hd[v]=cnt,to[cnt]=u,limit[cnt]=0;
    }
    int cur[N],dis[N],T;
    int dfs(int u,int res){
        if(u==T) return res;
        int flow=0;
        for(int i=cur[u];i&&res;i=nxt[i]){
            cur[u]=i;
            int c=min(res,limit[i]),v=to[i];
            if(dis[u]+1==dis[v]&&c){
                int k=dfs(v,c);
                flow+=k,res-=k,limit[i]-=k,limit[i^1]+=k;
            }
        }
        if(!flow) dis[u]=-1;
        return flow;
    }
    int maxflow(int s,int t){
        T=t;
        int flow=0;
        while(1){
            queue<int> q;
            memcpy(cur,hd,sizeof(hd));
            memset(dis,-1,sizeof(dis));
            q.push(s);
            dis[s]=0;
            while(!q.empty()){
                int u=q.front();
                q.pop();
                for(int i=hd[u];i;i=nxt[i]){
                    if(dis[to[i]]==-1&&limit[i]){
                        dis[to[i]]=dis[u]+1;
                        q.push(to[i]);
                    }
                }
            }
            if(dis[t]==-1) return flow;
            flow+=dfs(s,1e9);
        }
    }
};
struct network_flow{
    int cnt=0,u[M],v[M],limit[M],low[M];
    void add(int _u,int _v,int w,int b){
        u[++cnt]=_u,v[cnt]=_v,limit[cnt]=w,low[cnt]=b;
    }
    graph g;
    int maxflow(int n,int s,int t,int ss,int tt){
        int w[N];
        memset(w,0,sizeof(w));
        for(int i=1;i<=cnt;i++){
            w[u[i]]-=low[i],w[v[i]]+=low[i];
            g.add(u[i],v[i],limit[i]-low[i]);
        }
        int tot=0;
        for(int i=1;i<=n;i++){
            if(w[i]>0) g.add(ss,i,w[i]),tot+=w[i];
            else if(w[i]<0) g.add(i,tt,-w[i]);
        }
        g.add(t,s,1e9);
        if(g.maxflow(ss,tt)!=tot) return -1;
        int flow=g.limit[g.hd[s]];
        g.hd[s]=g.nxt[g.hd[s]];
        g.hd[t]=g.nxt[g.hd[t]];
        return flow+g.maxflow(s,t);
    }
}f;
```

## 有源汇费用流

- 只需要最大流算法换成费用流算法即可，所有 $SS,TT$ 相关的连边代价均为 $0$ 
- 初始费用为 $\sum\limits b(u,v)w(u,v)$ ，进行初步调整时需要加上 $SS\rightarrow TT$ 调整的费用，既 $SS\rightarrow TT$ 的最小费用最大流的费用，进行二次调整时也要加上产生的费用 
- [例题P4553](https://www.luogu.com.cn/problem/P4553)

```cpp
struct graph {
    int cnt=1,hd[N],nxt[M*2],to[M*2],limit[M*2],cst[M*2];
    void add(int u,int v,int w,int c){
        nxt[++cnt]=hd[u],hd[u]=cnt,to[cnt]=v,limit[cnt]=w,cst[cnt]=c;
        nxt[++cnt]=hd[v],hd[v]=cnt,to[cnt]=u,limit[cnt]=0,cst[cnt]=-c;
    }
    int fr[N],fl[N],in[N],dis[N];
    int mincost(int s,int t){
        int flow=0,cost=0;
        while(1){
            queue<int> q;
            memset(dis,0x3f,sizeof(dis));
            memset(in,0,sizeof(in));
            fl[s]=1e9,dis[s]=0,q.push(s);
            while(!q.empty()){
                int u=q.front();
                q.pop();
                in[u]=0;
                for(int i=hd[u];i;i=nxt[i]){
                    int v=to[i],d=dis[u]+cst[i];
                    if(limit[i]&&d<dis[v]){
                        fl[v]=min(limit[i],fl[u]),fr[v]=i,dis[v]=d;
                        if(!in[v]) in[v]=1,q.push(v);
                    }
                }
            }
            if(dis[t]>1e9) return cost;
            flow+=fl[t],cost+=dis[t]*fl[t];
            for(int u=t;u!=s;u=to[fr[u]^1]){
                limit[fr[u]]-=fl[t];
                limit[fr[u]^1]+=fl[t];
            }
        }
    }
};
struct network_flow{
    int cnt=0,u[M],v[M],limit[M],low[M],cst[M];
    void add(int _u,int _v,int w,int b,int c){
        u[++cnt]=_u,v[cnt]=_v,limit[cnt]=w,low[cnt]=b,cst[cnt]=c;
    }
    graph g;
    int mincost(int nn,int s,int t,int ss,int tt){
        int w[N];
        memset(w,0,sizeof(w));
        int cost=0;
        for(int i=1;i<=cnt;i++){
            cost+=cst[i]*low[i];
            w[u[i]]-=low[i],w[v[i]]+=low[i];
            g.add(u[i],v[i],limit[i]-low[i],cst[i]);
        }
        int tot=0;
        for(int i=0;i<=nn;i++){
            if(w[i]>0) g.add(ss,i,w[i],0),tot+=w[i];
            else if(w[i]<0) g.add(i,tt,-w[i],0);
        }
        g.add(t,s,1e9,0);
        cost+=g.mincost(ss,tt);
        g.hd[s]=g.nxt[g.hd[s]];
        g.hd[t]=g.nxt[g.hd[t]];
        return cost+g.mincost(s,t);
    }
}fl;

int n,m;
int S=0,T=0;

void solve(){
    cin>>n>>m;
    T=2*n;
    fl.add(S,1,m,0,0);
    f(i,2,n) fl.add(1,i*2-1,m,0,0),fl.add(i*2,T,m,0,0);
    f(i,1,n){
        int x;
        cin>>x;
        fl.add(i*2-1,i*2,x,x,0);
    }
    f(i,1,n){
        f(j,i+1,n){
            int x;
            cin>>x;
            if(x!=-1) fl.add(i*2,j*2-1,m,0,x);
        }
    }

    cout<<fl.mincost(2*n,S,T,T+1,T+2)<<"\n";
}
```

# 网络流

## 贪心

- 网络流是一种特殊的贪心，也是**可反悔贪心**，因此我们不需要为每一个贪心寻找策略，只要建出图后跑一边模板就行

## 技巧

- 网络流的题目，核心在于发现**题目的每一种方案与一种流或割对应**
- 为此，我们应用 **最小割等于最大流** 这一结论，考虑如何 **用一组割来表示一种意见方案**，最终得到解法

## 求方案注意点

- 最大流中并不是所有满流的边都是最小割的边，应该是对于边 $(u,v)$ 有 $u\in A,v\in B$
- 因此，如果一组割对应了题目的一种方案，在求解最大流之后，一定不能将所有流满的边视作割边，而是将两端所在集合不同的边视作割边。在解决集合划分模型相关问题时需要格外注意这一点

## 反悔性质

- 求解最大流过程中，因为网络流自带返回操作，所以我们可以**动态加边**
- 但对于费用流，因为其正确性依赖于每一步增广路均为最短路，所以一旦给网络加入新边，就必须重新跑费用流才能得到正确费用



# 应用与模型

## 求最小割点

- 如果题目变成删去每个点 $i$ 有代价 $w_{i}$ ，求使 $S,T$ 不连通的最小代价
- 我们把点转化为边，把每个点拆成入点 $i_{in}$ 和出点 $i_{out}$ ，从 $i_{in}$ 向 $i_{out}$ 连一条容量为 $w_{i}$ 的边，对于原图的每一条边，比如对 $(u,v)$ ，我们连一条 $u_{out}\rightarrow v_{in}$ 容量为 $\infty$ 的边
- 所以 $S_{out}\rightarrow T_{in}$ 的最小割即位所求

## 集合划分模型

$$\min_{x_1,x_2,..,x_n}\sum\limits_{(u,v)\in E}c_{u,v}x_u\overline{x_{v}}+\sum\limits_{u}a_{u}x_{u}+b_{u}\overline{x_{u}}$$
- 其中 $x_{i}=0/1$ ， $\overline{x_i}$ 表示将 $x_{i}$ 取反得到的结果
- 那么这个式子就是为 $x_{i}$ 选择合适的值，使整个式子的和最小
- 这个式子也可以被赋予以下实际意义：我们有 $n$ 个物品和 $A,B$ 两个集合，在 $A$ 集合有代价 $a_i$ ，在 $B$ 集合有代价 $b_{i}$ ，此外还有限制 $(u,v,c_{u,v})$ 表示若 $u,v$ 在不同集合有 $c_{u,v}$ 的代价
- 我们开始建模：
- 将 $i$ 与 $S,T$ 连边，容量分别为 $b_{i},a_{i}$ ，我们再将限制 $(u,v,c_{u,v})$ 建为连接 $u,v$ 容量为 $c_{u,v}$ 的双向边，得到网络 $G$ ，那么上述问题与最小割等价：
	- $i$ 与 $S$ 相连，此时割开 $u\rightarrow T$ ，表示将其划分到 $A$ ，有 $a_{i}$ 的代价
	- $i$ 与 $T$ 相连，此时割开 $S\rightarrow i$ ，表示将其划分到 $B$ ，有 $b_{i}$ 的代价 
	- 若 $u,v$ 属于不同的集合，那么 $u\rightarrow v$ 和 $v\rightarrow v$ 必有一条边被断开
- 如果题目出现若若干点都处于集合 $A$ 有额外成本 $d_{i}$ ，我们建一个辅助节点 $P$ ，连接 $P\rightarrow T$ 容量为 $d_{i}$ ，连接 $u_1\rightarrow P,u_{2}\rightarrow P,...$ 容量为 $+\infty$ ，若此时点都在 $S$ 侧，我为了分割我必须割掉 $P\rightarrow T$ 这条边，反之则不用
- 如果题目是求最大收益，那么就是把所有收益加起来，关于收益的边反向来连，然后减去最小割即可
- 那么最小割即位所求
- 如果有如下情况：
	- 当 $a_{i},b_{i}$ 为负数时，普通的最大流算法无法解决。我们考虑将 $a_{i},b_{i}$ 同时加上 $\delta_i$ ，最后再在最小割中减去 $\sum\limits\delta_{i}$ ，因为 $i$ 必须在选或者不选中选一个状态，所以为 $a_{i},b_{i}$ 同时加上 $\delta_{i}$ 的影响为 $\delta_i$ 而不是 $2\delta_{i}$ 
	- 若 $c_{u,v}$ 出现负值，除非所有 $c$ 均为负值且要求代价最大化，此时我们可以将所有边权取反，否则问题不可做
	- 如果题目有诸如 $x$ 在集合 $A$ 且 $y$ 在集合 $B$ 的代价为 $w$ ，那么我们连一条 $x\rightarrow y$ 容量为 $w$ 的边

## 最大权闭合子图

- 若有图 $G'\in G,V'\in V$ ，且 $V'$ 内所有出边都指向 $V'$ 的点，我们称 $G'$ 为 $G$ 的闭合子图
- 最大权闭合子图问题即为：每个点有点权 $w_i$ ，求点集最大权和
- 我们考虑**集合划分模型**，既对于每个点划入**选**或**不选**的集合中，在建图上体现为：
	- $S$ 与 $i$ 相连表示选，那么断开 $i\rightarrow T$ ，贡献为 $w_i$ ，因此 $i\rightarrow T$ 的容量为 $w_{i}$ ，同理 $S\rightarrow i$ 的容量为 $0$ ，对于原图的边 $u,v$ ，说明若 $u$ 被分到选的集合内，那么 $v$ 也被分入要选的集合内，既我们对他们连一条 $u\rightarrow v$ 容量为 $-\infty$ 的边，我们对上图求**最大割**
- 我们考虑权值取相反数来求最小割从而求得最大割：
	- 对于 $w_{i}\le0$ 的点， $i\rightarrow T$ 的容量 $-w_{i}\ge 0$ ，但对于 $w_{i}> 0$ 的点， $-w_{i}<0$ ，我们不允许最大流过程中出现负容量边
	- 我们借鉴集合划分模型中的扩展问题，将 $S\rightarrow i$ 和 $i\rightarrow T$ 同时加上 $w_{i}$ ，那么 $S\rightarrow i$ 容量为 $w_{i}$ ， $i\rightarrow T$ 容量为 $0$ ，最后减去所有正权点权值之和，并对所求结果取相反数
	- 上述操作等价于先将所有正权点纳入闭合子图，在考虑去掉不选正权点的贡献，如果某个 $w_i>0$ 的 $i$ 和 $T$ 连通说明 $i$ 不选，割掉 $S\rightarrow i$ 后有 $w_i$ 的代价，所以容量为 $w_{i}$
- 综上我们得到一个求解最大权闭合子图的算法：
	- 对于 $w_{i}\ge0$ ，$S\rightarrow i$ 连容量为 $w_{i}$ 的边；对于 $w_{i}<0$ ， $i\rightarrow T$ 连容量为 $-w_{i}$ 的边；对于 $(u,v)\in E$ ，连一条 $u\rightarrow v$ 容量为 $+\infty$ 的边。若我们得到的网络为 $G'$ ，那么最终答案为
$$\left(\sum\limits_{w_{i}\ge 0}w_{i}\right)-\mathrm{Minimum\;Cut}(G')$$

## 有负环的费用流

- 我们考虑运用上下界网络使负权边强制满流，并令反边 $b(u,v)=0,c(v,u)=c(u,v)$ 表示退流，此时因为 $b(u,v)=c(u,v)$ ，不会出现在残量网络中，所以无负环

```cpp
struct flow{
    int cnt=1,hd[N],nxt[M*2],to[M*2],limit[M*2],cst[M*2];
    void add(int u,int v,int w,int c){
        nxt[++cnt] = hd[u], hd[u] = cnt, to[cnt] = v, limit[cnt] = w, cst[cnt] = c;
        nxt[++cnt] = hd[v], hd[v] = cnt, to[cnt] = u, limit[cnt] = 0, cst[cnt] = -c;
    }
    int fl[N],fr[N],dis[N],in[N];
    pair<int,int> mincost(int s,int t){
        int flow=0,cost=0;
        while(1){
            queue<int> q;
            memset(dis,0x3f,sizeof(dis));
            q.push(s);
            fl[s]=1e9,dis[s]=0;
            while(!q.empty()){
                int u=q.front();
                q.pop();
                in[u]=0;
                for(int i=hd[u];i;i=nxt[i]){
                    int v=to[i],d=dis[u]+cst[i];
                    if(limit[i]&&d<dis[v]){
                        dis[v]=d;
                        fl[v]=min(fl[u],limit[i]);
                        fr[v]=i;
                        if(!in[v]) in[v]=1,q.push(v);
                    }
                }
            }
            if(dis[t]>1e9) return make_pair(flow,cost);
            flow+=fl[t],cost+=dis[t]*fl[t];
            for(int u=t;u!=s;u=to[fr[u]^1]) limit[fr[u]]-=fl[t],limit[fr[u]^1]+=fl[t];
        }
    }
};

struct bounded_flow{
    int cnt=0,u[M],v[M],lo[M],hi[M],cst[M];
    void add(int _u,int _v,int w,int c){
        if(c<0){
            u[++cnt]=_u,v[cnt]=_v,lo[cnt]=w,hi[cnt]=w,cst[cnt]=c;
            u[++cnt]=_v,v[cnt]=_u,lo[cnt]=0,hi[cnt]=w,cst[cnt]=-c;
        }
        else u[++cnt]=_u,v[cnt]=_v,lo[cnt]=0,hi[cnt]=w,cst[cnt]=c;
    }
    flow g;
    pair<int,int> mincost(int n,int s,int t,int ss,int tt){
        int w[N];
        memset(w,0,sizeof(w));
        int flow=0,cost=0,tot=0;
        for(int i=1;i<=cnt;i++){
            w[u[i]]-=lo[i];
            w[v[i]]+=lo[i];
            cost+=lo[i]*cst[i];
            g.add(u[i],v[i],hi[i]-lo[i],cst[i]);
        }
        for(int i=1;i<=n;i++){
            if(w[i]>0) g.add(ss,i,w[i],0),tot+=w[i];
            else if(w[i]<0) g.add(i,tt,-w[i],0);
        }
        g.add(t,s,1e9,0);
        pair<int,int> res=g.mincost(ss,tt);
        cost+=res.second;
        flow+=g.limit[g.hd[s]];
        g.hd[s]=g.nxt[g.hd[s]];
        g.hd[t]=g.nxt[g.hd[t]];
        res=g.mincost(s,t);
        return make_pair(flow+res.first,cost+res.second);
    }
}f;
```