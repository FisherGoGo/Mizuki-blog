---
title: 线段树
published: 2025-07-29
tags: [DS]
category: Knowledge
draft: false
---

# 前言

-  线段树是用来维护**区间信息**的数据结构
- 线段树可以在 $O(\log N)$ 的时间复杂度之内实现单点修改、区间修改、区间查询、区间求和、求最大值、最小值等操作
- [学习来源 : alex-wei佬的blog](https://www.cnblogs.com/alex-wei/p/segment_tree_yyds.html)  

# 结构

- 我们用 $d_i$ 表示线段树上节点为 $i$ 的值
- $d_i$ 的左儿子是 $d_{2\times i}$ ，右儿子是 $d_{2\times i +1}$ ，如果父节点表示的是区间 $\left[s,t\right]$ ，那么 左儿子节点表示的区间是 $\left[s,\frac{s+t}{2} \right]$ ，右儿子表示的区间就是 $\left[\frac{s+t}{2}+1,t\right]$ 
- 因此，我们考虑用递归建树

# 操作

## 建树

```cpp
#define ll long long
ll d[maxn*4],num[maxn*4]; \\d为线段树，num为原数组

void push_up(int x){
	d[x]=d[x*2]+d[x*2+1]; \\这里可以根据线段树不同的用途来修改，比如储存区间积、异或
}

void build(int x,int l,int r){
	if(l==r){
		d[x]=num[l];
		return ;
	}
	build(x*2,l,(l+r)/2); \\递归左儿子
	build(x*2+1,(l+r)/2+1,r); \\递归右儿子
	push_up(x);
}
```

## 区间修改

- 如果要修改区间 $\left[l,r\right]$ ，把其中所有节点都遍历一遍的话时间复杂度太高，所以我们引入**懒惰标记**
- 简单来说，就是我们延迟对这个节点的修改，先把修改的信息打上标记，但是不更新其子节点，直到下一次访问到子节点时才去更新信息

```cpp
ll lazy[maxn*4];

void push_down(int x,int l,int r){ \\懒标记下传
	int mid=(l+r)/2;
	d[x*2]+=(mid-l+1)*lazy[x];
	d[x*2+1]+=(r-mid)*lazy[x];
	lazy[x*2]+=lazy[x];
	lazy[x*2+1]+=lazy[x];
	lazy[x]=0;
}

void add(int x,int l,int r,int ql,int qr,ll num){
	if(ql<=l&&r<=qr){
		lazy[x]+=num;
		d[x]+=(r-l+1)*num;
		return ;
	}
	push_down(x,l,r);
	int mid=(l+r)/2;
	if(ql<=mid) add(x*2,l,mid,ql,qr,num);
	if(qr>mid) add(x*2+1,mid+1,r,ql,qr,num);
	push_up(x);
}
```

## 区间查询

```cpp
ll query(int x,int l,int r,int ql,int qr){ \\类似区间修改
	if(ql<=l&&r<=qr){
		return d[x];
	}
	ll ans=0;
	int mid=(l+r)/2;
	push_down(x,l,r);
	if(ql<=mid) ans+=query(x*2,l,mid,ql,qr);
	if(qr>mid) ans+=query(x*2+1,mid+1,r,ql,qr);
	return ans;
}
```

# 权值线段树 | 动态开点

- **权值线段树**是一种以序列的数值为下标，节点统计的是对应区间 $\left[l,r\right]$ 中，这个值域所有数出现的次数
- 每个**叶子节点**的值： 代表 这个值的出现次数。
- **非叶子节点**的值：代表了某一个值域内，所有值出现次数的和
- 如果按照普通的线段树构造方法，对于一个取值范围为 $w$ 的序列，我们需要 $O(w\log w)$ 的空间，这对于许多题来说都是不够用的，所以我们需要**动态开点**
- 这很简单易懂，普通的线段树的左右儿子关系是 $2\times x$ 与 $2\times x+1$ ，那么动态开点就用 $ls_x$ 与 $rs_x$ 来替代，只有在遍历到这个节点时才给它新建节点值

## Code
```cpp
#include<iostream>
#include<algorithm>
#define mid ((l)+(r)>>1)
using namespace std;
typedef long long ll; 

const int maxn=1e7+8;
const ll lim=1e9;  

ll d[maxn];
int ls[maxn],rs[maxn],node=0;
int rt=0;
int n;

void push_up(int x){
    d[x]=d[ls[x]]+d[rs[x]];
    return ;
} 

void update(int &x,ll l,ll r,ll col){
    if(x==0) x=++node;
    if(l==r){
        d[x]++;
        return ;
    }  

    if(col<=mid) update(ls[x],l,mid,col);
    else update(rs[x],mid+1,r,col);
  
    push_up(x);
}

ll query(int x,ll l,ll r,ll val){
    if(x==0) return 0;
    if(l>val) return d[x];
  
    ll res=0;
    if(val<=mid) res+=query(ls[x],l,mid,val);
    res+=query(rs[x],mid+1,r,val);  

    return res;
}

int main(){
    cin>>n; 

    ll ans=0;
    for(int i=1;i<=n;i++){
        ll x;
        cin>>x;
        update(rt,1,lim,x);
        ans+=query(rt,1,lim,x);
    }  

    cout<<ans;
    return 0;
}
```

# 标记永久化

- 懒标记不再下发，变成只属于某个范围的标记信息，上下级的标记之间不在互相影响
- 从上往下的遍历过程中，维护标记的叠加信息，即可完成查询
- 有了标记永久化我们可以减少空间占用，因为没有懒标记下发的过程我们就可以避免去健许多新节点，一般在可持久化线段树使用
- 以区间取最大值，区间求最大值为例，对每个区间我们维护两个信息，一是 $val$ 表示子树最大值，二是 $laz$ 表示当前区间的懒标记的最大值。也就是说，对于一个区间，它的信息的真实值为它的 $val$ 值与从根到当前区间的路径 $P$ 上所有节点的 $laz$ 值的最大值，因为能够下传懒标记到当前区间的节点恰好也全都在路径 $P$ 上
- 那我们现在想知道那些标记可以永久化
- 首先：区间修改后可以快速更新，区间取最大值只需要修改并取 $\max$ ；其次：区间修改必须与顺序无关
- 但是它有局限性，它只适用于可叠加性质的数据
# 线段树分治

- 线段树分治是一种**离线**的按时间分治的技巧，主要是在时间 $t$ 内，有多个时间段 $\left[l_i,r_i\right]$ 进行了不同的操作，然后询问每个时间点内的状态
- 它适用于仅支持**插入**，不支持**删除**的数据结构，任何难以快速删除的操作（从若干次操作带来的影响当中撤销某一次操作，得到新的影响），均可以通过线段树分治将删除转化为撤销
- 对于一个操作，在离线后我们知道它影响的时间段是 $\left[l,r\right]$ ，我们在线段树走到此位置并添加此操作，之后我们进行中序遍历，我们可以知道：向儿子走表示加入子节点的操作贡献，向父亲走表示撤销当前节点的所有操作。所以时刻 $t$ 内的操作就是从根节点走到区间 $\left[t,t\right]$ 的操作
- 时间复杂度为： $O(Tk\log T)$ ，$T$ 是时间范围， $k$ 一般是 $\log n$ 
## Code
```cpp
#include<iostream>
#include<algorithm>
#include<cstring>
#include<vector>
#define mid ((l)+(r)>>1)
#define ls ((l+r)*2)
#define rs ((l+r)*2+1)
using namespace std;  
const int maxn=1e7;

int n,m,k;
int fa[maxn],height[maxn];

struct Edge{
    int u,v;
}e[maxn];
struct stk{
    int u,v,add;
}st[maxn];
int top=0;
vector <int> t[maxn];

int find(int x){
    while(x!=fa[x]) x=fa[x];
    return fa[x];
}

void merge(int x,int y){
    int fx=find(x),fy=find(y);
    if(height[fx]>height[fy]) swap(fx,fy);
    st[++top]={fx,fy,height[fx]==height[fy]};
    fa[fx]=fy;
    if(height[fx]==height[fy]) height[fy]++;
}

void update(int x,int l,int r,int ql,int qr,int val){
    if(ql<=l&&r<=qr){
        t[x].push_back(val);
        return ;
    }

    if(ql<=mid) update(ls,l,mid,ql,qr,val);
    if(qr>mid) update(rs,mid+1,r,ql,qr,val);
}

void query(int x,int l,int r){
    int ans=1;
    int lasttop=top;
    for(int i=0;i<t[x].size();i++){
        int fu=find(e[t[x].at(i)].u);
        int fv=find(e[t[x].at(i)].v);
        if(fu==fv){
            for(int k=l;k<=r;k++) cout<<"No\n";
            ans=0;
            break;
        }
        merge(e[t[x].at(i)].u,e[t[x].at(i)].v+n);
        merge(e[t[x].at(i)].v,e[t[x].at(i)].u+n);
    }
    if(ans){
        if(l==r) cout<<"Yes\n";
        else {
            query(ls,l,mid);
            query(rs,mid+1,r);
        }
    }
    while(top>lasttop){
        height[fa[st[top].u]]-=st[top].add;
        fa[st[top].u]=st[top].u;
        top--;
    }
    return ;
}

int main(){
    cin>>n>>m>>k;
    for(int i=1;i<=m;i++){
        cin>>e[i].u>>e[i].v;
        int l,r;
        cin>>l>>r;
        l++;
        update(1,1,k,l,r,i);
    }
    for(int i=1;i<=2*n;i++) fa[i]=i,height[i]=1;
    
    query(1,1,k);

    return 0;
}
```

# 线段树合并

- 线段树合并指的是将两棵线段树进行合并，一般是对于**动态开点**的树进行合并，如果对于普通的线段树，他们的节点已经满了，那我们去进行合并不如更新每一个节点后去重新建树
- 对于两颗线段树 $T_x$ 和 $T_y$ ，设当前区间为 $x$ 和 $y$ 
- 若 $x$ 和 $y$ 至少有一个为空，就返回另一个
- 否则新开 $z$ 节点，令 $z$ 的左儿子为 $x$ 的左儿子和 $y$ 的左儿子合并后的结果，右儿子同理
- 如果递归到叶子节点就直接合并节点数据并返回，因为区间长度为 $1$ 的区间合并很简单
- 如果合并 $x$ 和 $y$ 后节点 $y$ 的数据不会用到，那么我们可以直接将 $y$ 的信息合并到 $x$ 上，既用 $x$ 来取代 $z$ 
- 时间复杂度分析：每次合并的复杂度为两颗线段树**重合**的节点个数，既**删去**的节点个数，因此总复杂度为所有线段树节点数之和。若一开始有 $n$ 颗树，每棵树仅在一个节点有值，那么合并他们的复杂度为 $O(n\log n)$ 

## Code
```cpp
int merge(int x,int y,int l,int r){
    if(!x||!y) return x|y;
    int z=++node;
    if(l==r){
        d[z]=d[x]+d[y];
        return z;
    }
    ls[z]=merge(ls[x],ls[y],l,mid);
    rs[z]=merge(rs[x],rs[y],mid+1,r);
    return z;
}
```

# 树套树

- 有时候，我们可以用**DS**套**DS**的结构来维护一些信息，比如说树状数组套线段树或者线段树套线段树
- 用树状数组套线段树可以用来维护比如二维数点或者动态第 $k$ 小，一般来说需要维护的数据的第一维值域不大，不然的话要换成动态开点线段树
- 对于维护二维数点来说，我们用树状数组来维护 $x$ 这一维，用线段树来维护每一个 $x$ 对应的纵轴，既 $y$ 轴，意味着我们将平面分割成 $n$ 条直线，每一条用动态开点线段树来维护。因为我们需要单点修改和区间查询，所以外层**DS**也要写一个对应的修改查询函数
- 比如对于矩阵查询，查询的内容是一段区间 $\left[x_{left},x_{right}\right]$ 内所有线段树在一段区间 $\left[y_{left},y_{right}\right]$ 的取值之和



# 应用

- 线段树本身很简单，有时候难的是维护各种信息，下边就放一些我见过的信息维护与运用

## 信息合并

### 杭州 2022 B

- 有 $n$ 个数字，第 $i$ 个数字为 $c_{i}$ ，有权值 $d_{i}$ ，有单点修改，每次询问对于哪一个 $k$ ，使得对于数对 $(i,j)$ ，$c_{i}+c_{j}$ 在第 $k$ 位发生进位且 $d_{i}+d_{j}$ 最大
- 我们发现，$c_{i}+c_{j}$ 在第 $k$ 位发生进位当且仅当 $c_{i}\pmod{2^{k}}+c_{j}\pmod{2^{k}}\ge 2^{k}$ ，我们变化式子，得到 $c_{i}\pmod{2^{k}}\ge 2^{k}-c_{j}\pmod{2^{k}}$ ，此时我们令 $x=c_{i}\pmod{2^{k}},y=2^{k}-c_{j}\pmod{2^{k}}$ ，令 $A_{x}$ 表示所有 $x$ 的最大的 $d$ ，令 $B_{y}$ 代表所有的 $y$ 的最大的 $d$ 
- 那么就变成了求 $\max\{A_{x}+B_{y}|x\ge y\}$ ，用权值线段树维护然后枚举每个 $k$ 求最大即可
## 打标记

## 二分

- 线段树的特殊性质使得其可以二分，当题目涉及到找到一个特殊性质的数时，我们可以维护区间整体性质来二分找满足条件的数

## 三角形区域

- 求一个等腰直角三角形区域内的数点数，给定左下角坐标 $(A,B)$ 与边长 $d$ 的三角形区域
- 我们会发现就是求这样一个区域：
$$\begin{cases}x\ge A\\ y\ge B\\x+y\le A+B+d\end{cases}=\begin{cases}x\ge A\\y\ge B\end{cases}-\begin{cases} x+y>A+B+d\end{cases}+\begin{cases}x+y>A+B+d\\x<A\end{cases}+\begin{cases}x+y>A+B+d\\y<B\end{cases}$$
- 那我们维护一个点对 $(x+y,x)$ 与 $(x+y,y)$ 即可
## 区间

- 假如我们要求一个区间与几个不同的区间重叠，有以下几种
### 求数量

- 我们定义：
	- $D_i$ ：维护 $i$ 端点左边有几个区间的开头，即左端点
	- $E_{i}$ ：维护 $i$ 端点左边有几个区间的结尾，即右端点
- 那么对于区间 $[l,r]$ ，答案就是 $D_{r}-E_{l-1}$ ，画图模拟一下即可
### 求权值

- 假如每个区间都有一个权值，求与给定区间重合的权值运算和
- 一个区间与给定区间 $[L,R]$ 重合的条件是 $l\le R$ 与 $r\ge L$ 
- 我们把 $l$ 视为 $x$ ，把 $r$ 视为 $y$ ，那么重合区间是一个矩形区域 $[1,R]\times[L,\inf]$ 
- 我们用可持久化线段树维护这个平面矩形信息即可，或者扫描线维护

## 复杂度分析

- 对于一些难以叠加的操作，如区间开根，区间取模，区间取 $\log$ 或者维护有一些特殊性质的条件，我们可以难以使用懒标记维护，这时候我们可以分析其复杂度
- 比如对于区间开根这些操作我们明显发现对于一个数最多只能进行有限次，假如最多进行 $K$ 次，那么我们暴力修改的复杂度就是 $O(KN)$ ，一般来说这个 $K$ 很小，所以我们可以维护一个区间标致值，来决定这个区间还要不要被修改，从而降低单次修改复杂度
- 又比如我们要维护这样一个信息，有没有一个下标使得 $a_i=pre_{i-1}$ ，由于 $a_{i}$ 有上界，符合条件的位置很少，所以我们大可维护区间 $a_{i}-pre_{i-1}$ 的最大值，然后暴力二分查找即可
## 最长子段问题

- 一般这个问题是求区间最长连续满足某一条件的子端，如最长连续 $1$ 等
- 我们一般要维护这些信息：区间最大值（$max$），包括左端点的最大值（$lmax$），包括右端点的最大值（$rmax$）
- 在向上合并时我们有：
$$lmax_{x}=\begin{cases}lmax_{ls}\quad 左儿子左前缀未占满整个左区间\\\\ lmax_{ls}+lmax_{rs}\quad 左儿子左前缀占满了整个左区间\end{cases}$$
- $rmax$ 同理
- 对于 $max$ ，有：
$$max_{x}=\max \begin{cases}max_{ls}\\max_{rs}\\rmax_{ls}+lmax_{rs}\end{cases}$$
- 最后一种情况就是最大区间跨越左右儿子
- 如果有时候有更多条件的话我们就在取值时加上更多判断即可
- 还有一种变种，思想也类似，比如求 $\max\{a_{i}-a_{j}|i>j\}$ ，
## DDP

- 给 $n$ 列点，每列有 $k$ 个点，第 $i$ 列的点只会向第 $i+1$ 列点连边，问两点之间最短路
- 我们考虑快速合并信息来维护这个模型
### 定义

- 我们将每一列的 $k$ 个点看作一个整体，从第 $i$ 列到第 $i+1$ 列的连接关系可以被看作**变换**或者**转移**，我们的目标是求出从第 $i$ 列到第 $j$ 列**总变换** 
- 对于管理 $[L,R]$ 区间的节点，我们储存一个 $k\times k$ 的矩阵 $dist$ ，其中 $dist[u][v]$ 表示从第 $L$ 列的第 $u$ 个节点出发，到达第 $R$ 列的第 $v$ 个节点的最短路长度
### 节点合并

- 假如一个父节点代表区间 $[L,R]$ ，它的左子节点代表区间 $[L,M]$ ，右子节点代表区间 $[M+1,R]$ ，我们如何合并上去，我们发现我们还要 $[M,M+1]$ 的变化矩阵，所以我们把线段树建立在列与列的间隙上，一共有 $n-1$ 个间隙
- 那么对于叶子节点 $i$ 表示第 $i$ 列到第 $i+1$ 列的变换，他储存的矩阵就是 $dist[u][v]$ ，既从第 $i$ 列的点 $u$ 到第 $i+1$ 列的点 $v$ 的边权，如果没有就是正无穷
- 那么左儿子就是 $[L,M]$ 的变换，右儿子就是 $[M,R]$ 的变换，那么要从 $L$ 列的第 $i$ 点走到 $R$ 列的第 $j$ 点，就要枚举所有中间可能点取最小，既 **Min-Plus 矩阵乘法**
$$C[i][j]=\min_{p=0}^{k-1}(A[i][p]+B[p][j])$$

### 推广

- 当然，上边是一个简单的例子，DDP 的思想就是用矩阵来维护答案，一般是矩阵来维护 DP，且 DP 一般是高维 DP，来几个例题

### CF750E

- 我们设 $f_{i,j}$ 表示前 $i$ 个数字与 $2017$ 的前 $j$ 位匹配，且不与 $j+1$ 位匹配，不与 $2016$ 匹配所需要删除元素的最小数量，先考虑 $f_{i,0}$
$$f_{i,0}=\begin{cases}f_{i-1,0}+1\;(s_{i}=2)\\\\f_{i-1,0}\;(s_{i}\ne 2)\end{cases}$$
- 接着考虑 $f_{i,1}$ 
$$f_{i-1,1}=\min\left(\begin{cases}f_{i-1,0}\;(s_{i}=2)\\\\ \inf\;(s_{i}\ne 2)\end{cases},\begin{cases}f_{i-1,1}+1\;(s_{i}=0)\\\\ f_{i-1,1}\;(s_{i}\ne0)\end{cases}\right)$$
- 这里取 $\inf$ 是取正无穷来做特判，我们来解释一下这个式子：
	- 若我们从 $f_{i-1,0}$ 转移时，说明前边已经得到一个与空集匹配的字符串，如果这时候来一个 $2$ ，那么就可以与 $2$ 匹配，否则不存在答案
	- 若我们从 $f_{i-1,1}$ 转移时，说明前边已经得到一个与 $2$ 匹配的字符串，若来一个 $0$ ，则违反了定义，说明我们要删去，否则不用删
- 依次类推，我们得到剩下的式子
$$\begin{cases}f_{i,2}=\min\left(\begin{cases}f_{i-1,2}+1\;(s_{i}=1)\\\\ f_{i-1,2}\;(s_{i}\ne1)\end{cases},\begin{cases}f_{i-1,1}\;(s_{i}=0)\\\\\inf\;(s_{i}\ne0)\end{cases}\right)\\\\ \\
f_{i,3}=\min\left(\begin{cases}f_{i-1,3}+1\;(s_{i}=7\lor s_{i}=6)\\\\ f_{i-1,3}\;(s_{i}\ne7\land s_{i}\ne 6)\end{cases},\begin{cases}f_{i-1,2}\;(s_{i}=1)\\\\\inf\;(s_{i}\ne1)\end{cases}\right)\\\\ \\
f_{i,4}=\min\left(\begin{cases}f_{i-1,4}+1\;(s_{i}=6)\\\\ f_{i-1,4}\;(s_{i}\ne6)\end{cases},\begin{cases}f_{i-1,3}\;(s_{i}=7)\\\\\inf\;(s_{i}\ne7)\end{cases}\right)\\\\\end{cases}$$
- 其中边界为 $f_{0}=[0,\inf,\inf,\inf,\inf]$ 
- 这里就用矩阵优化DP，可以得到
$$\begin{cases}\vec{v_{i}}=[f_{i,0},f_{i,1},f_{i,2},f_{i,3},f_{i,4}]\\\\ \vec{v_{i}}=\vec{v_{i-1}}\times M_{i}\\\\  
M_{i}= \begin{vmatrix}(s_{i}=2)?1:0&(s_{i}=2)?0:\inf&\inf&\inf&\inf\\\inf&(s_{i}=0)?1:0&(s_{i}=0)?0:\inf&\inf&\inf\\\inf&\inf&\inf&(s_{i}=7\lor s_{i}=6)?1:0&(s_{i}=7)?0:\inf\\\inf&\inf&\inf&\inf&(s_{i}=6)?1:0\end{vmatrix}  \end{cases}$$
- 用线段树维护区间矩阵即可，这里用的也是 Min-Plus 矩阵乘法，其实可以发现 $M_{l}\cdot M_{l+1}\cdot \ldots\cdot M_{r}$ 第一列的最后一项既答案，[附上记录](https://codeforces.com/contest/750/submission/331212462) 

