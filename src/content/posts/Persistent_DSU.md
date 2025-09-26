---
title: 可持久化数据结构
published: 2025-07-04
tags: [DS]
category: Knowledge
draft: false
---

# 可持久化数据结构

- 指支持版本存储、查询、修改的数据结构
- 可持久化数据结构是一种维护二维信息的数据结构，第一维度一般是具体信息，第二维度可以是时间、区间等，所以它支持平面上的可减信息，支持矩形查询，常用于强制在线的二维数点问题
- 有什么问题呢，比如说可以查询某一时间点的信息，或者以任意某一个版本为基础就行修改，或者查询长度或者时间区间信息，既矩形查询
- 如下图如果我们要查询黄色区间信息，就是用类似前缀和将右上角减去左下角信息

<center> <img src=https://cdn.luogu.com.cn/upload/image_hosting/7xkc6l5b.png></center>

# 可持久化线段树

- 主席树用于维护平面可减信息，支持矩形查询，常见于强制在线的二维数点问题
- 如果我们对于一颗线段树 $T_{0}$ 我们要进行 $q$ 次单点修改，得到了 $T_{1},T_{2},...,T_{q}$ ，对于得到的新的线段树我们如果朴素地存储这是不能接受的
- 注意到我们每次修改单点信息时只会更改 $O(\log n)$ 个线段树节点信息，那么我们就可以新开节点来存储这个新的信息，既我们新开节点来存储被修改的那一侧节点，对于另外一侧则直接接上原来的节点的儿子
- 下既是一个简单的维护区间和线段数的可持久化版本

```cpp
void modify(int pre,int &x,int l,int r,int p,int v){
	if(x==0) x=++node;
	d[x]=d[pre];
	ls[x]=ls[pre];
	rs[x]=rs[pre];
	if(l==r){
		d[x]=v;
		return ;
	}
	if(p<=mid) modify(ls[pre],ls[x],l,mid,p,v);
	else modify(rs[pre],rs[x],mid+1,r,p,v);
	push_up(x);
}
```

## 应用

- 基于这个思想我们可以衍生出许多操作，并且不仅限于线段树，还可以有 $Trie$ 树并查集等等

### 求区间第 $k$ 小

- 对于每个元素我们有两个值，下标和权值，我们在权值线段树上二分可以求出构成该权值线段树的所有元素的第 $k$ 小值，那么求区间最小值就是我们在 $a_{i\sim j}$ 构成的权值线段树上二分，我们可以通过 $a_{1\sim j}$ 的权值线段树减去 $a_{1\sim i-1}$ 的权值线段树来得到，从而二分

```cpp
int n,q;
int rt[N];
int d[N<<5],ls[N<<5],rs[N<<5];
int a[N],b[N];
int node=0;

void modify(int pre,int &x,int l,int r,int pos){
    x=++node;
    d[x]=d[pre]+1;
    ls[x]=ls[pre];
    rs[x]=rs[pre];
    if(l==r) return ;
    if(pos<=mid) modify(ls[pre],ls[x],l,mid,pos);
    else modify(rs[pre],rs[x],mid+1,r,pos);
    return ;
}

int query_rk(int x,int y,int l,int r,int rk){
    if(l==r) return b[l];
    int v=d[ls[y]]-d[ls[x]];
    if(rk<=v) return query_rk(ls[x],ls[y],l,mid,rk);
    else return query_rk(rs[x],rs[y],mid+1,r,rk-v);
}

void solve(){
    cin>>n>>q;
    f(i,1,n) cin>>a[i],b[i]=a[i];
    sort(b+1,b+n+1);
    f(i,1,n) a[i]=lower_bound(b+1,b+n+1,a[i])-b;
    f(i,1,n) modify(rt[i-1],rt[i],1,n,a[i]);
    while(q--){
        int l,r,k;
        cin>>l>>r>>k;
        cout<<query_rk(rt[l-1],rt[r],1,n,k)<<"\n";
    }
    return ;
}
```

### 求区间最大异或和

- 使用可持久化 $Trie$ 树，然后运用类似求区间第 $k$ 大的方法来求解

```cpp
int n,m;
int d[N<<6];
int a[N];
int rt[N<<5];
int node=0;
int son[N<<6][2];

void modify(int pre,int &x,int val,int dep){
    x=++node;
    d[x]=d[pre]+1;
    son[x][0]=son[pre][0];
    son[x][1]=son[pre][1];
    if(!dep) return;
    dep--;
    int nx=(val>>dep)&1;
    modify(son[pre][nx],son[x][nx],val,dep);
}

int query(int x,int y,int val,int dep){
    if(!dep) return 0;
    dep--;
    int nx=(val>>dep)&1;
    if(d[son[y][nx^1]]-d[son[x][nx^1]]) return (1<<dep)+query(son[x][nx^1],son[y][nx^1],val,dep);
    else return query(son[x][nx],son[y][nx],val,dep);
}

void solve(){
    cin>>n>>m;
    modify(rt[0],rt[0],0,24);
    int res=0;
    f(i,1,n){
        cin>>a[i];
        modify(rt[i-1],rt[i],res^=a[i],24);
    }

    int cnt=n;
    while(m--){
        char A;
        int l,r,x;
        cin>>A;
        if(A=='A'){
            cin>>x;
            cnt++;
            modify(rt[cnt-1],rt[cnt],res^=x,24);
        }
        else {
            cin>>l>>r>>x;
            if(l==1) l=0;
            else l=rt[l-2];
            r=rt[r-1];
            cout<<query(l,r,x^res,24)<<"\n";
        }
    }
}
```

### DSU On TREE

- 其实和其他的并没有什么不同，一般会有两个问法
	- 求 $X$ 的子树的信息
	- 求 $U$ 到 $V$ 的路径的信息
- 对于第一种问法，我们用欧拉序将树拍平，然后就变成了区间操作询问
- 对于第二种问法，我们一般建前缀主席树，既 $\mathbf{s[u]}$ 为 $\mathbf{u}$ 到根节点的信息和，然后用以下公式即可得到路径信息
$$\mathbf{s[u]+s[v]-s[lca(u,v)]-s[fa[lca(u,v)]]}$$

# 可持久化并查集

- 我们平常用的并查集是路径压缩的，但是你要可持久化，就不能做到路径压缩，但是如果随意合并，会怎么样？复杂度会来到 $O(n)$ ，我们可以看以下例子：
	- 初始情况我们有 $(1),(2),(3)...$ 
	- 我们依次执行 $\mathbf{unio}(i,i+1)$ 
	- 这样这个关系网会成退化成为一个链，那么查询的时候复杂度就来到了 $O(n)$ 
- 再来讲讲为什么不能路径压缩：
	- 我们每一次进行一个合并的空间与时间代价是 $O(\log n)$ ，既修改一个 $fa$ 指针的代价
	- 那如果我们要路径压缩的话，对于节点 $x$ 到根节点 $\mathbf{root}$ ，我们对于路上每一个点我们都需要修改，复杂度会来到 $O(k\log n)$ 
- 我们可以按照**大小**或者按照**深度**合并，下边我按集合大小合并 
```cpp
int n,m;
int node=0;
int rt[N<<1],ls[N<<6],rs[N<<6],fa[N<<6],sz[N<<6];

void build(int &x,int l,int r){
    x=++node;
    if(l==r){
        fa[x]=l;
        sz[x]=1;
        return ;
    }
    build(ls[x],l,mid);
    build(rs[x],mid+1,r);
}

void modify(int pre,int &x,int l,int r,int pos,int v,int v2){
    x=++node;
    ls[x]=ls[pre];rs[x]=rs[pre];
    if(l==r){
        fa[x]=v;
        sz[x]=v2;
        return ;
    }
    if(pos<=mid) modify(ls[pre],ls[x],l,mid,pos,v,v2);
    else modify(rs[pre],rs[x],mid+1,r,pos,v,v2);
}

pair<int,int> query(int x,int l,int r,int pos){
    if(l==r){
        return make_pair(fa[x],sz[x]);
    }
    if(pos<=mid) return query(ls[x],l,mid,pos);
    else return query(rs[x],mid+1,r,pos);
}

int find(int x,int v){
    int f=query(rt[v],1,n,x).first;
    return x==f?f:find(f,v);
}


void solve(){
    cin>>n>>m;
    build(rt[0],1,n);//先建一课普通的树，父亲为自己，集合大小为1
    f(i,1,m){
        int typ,a,b;
        cin>>typ>>a;
        if(typ==1){
            rt[i]=rt[i-1];//可持久化关键，继承
            cin>>b;
            int x=find(a,i),y=find(b,i);
            if(x==y) continue;
            int sa=query(rt[i],1,n,x).second;
            int sb=query(rt[i],1,n,y).second;
            if(sa<sb) swap(sa,sb),swap(x,y);//按大小合并
            modify(rt[i],rt[i],1,n,y,x,sb);//修改
            modify(rt[i],rt[i],1,n,x,x,sa+sb);       
        }
        if(typ==2) rt[i]=rt[a];
        if(typ==3){
            rt[i]=rt[i-1];
            cin>>b;
            int x=find(a,i),y=find(b,i);
            cout<<(x==y)<<"\n";
        }
    }
}
```
