---
title: 二叉搜索树
published: 2024-11-26
tags: [DS]
category: Knowledge
draft: false
---

# 介绍
- **二叉搜索树**是一种二叉树的数据结构，它满足以下特点：
	- 空树是一颗二叉搜索树
	- 其左子树上的所有节点的权值都小于根节点，右子树上的所有节点的权值都小于根节点
	- 符合上一个条件的树，其子树也为二叉搜索树

# 功能
- 二叉搜索树可以实现诸如 **插入 删除 查询最大/最小值 查询排名 查询第k大值...** 
	>以下代码是我对 [oi-wiki](https://oi-wiki.org/ds/bst/)中用指针写法~~我讨厌指针~~的改写法，没有实际运行过，所以无法保证正确性，建议去看Fhq Treap部分
- **定义**
	```cpp
	struct Tree{
		int l,r;
		int val;
		int size;
		int count;
	}t[maxn];
	int cnt,root;

	int newnode(int val){
		tree[++cnt].val=val;
		tree[cnt].size=1;
		tree[cnt].count=1;
		return cnt;
	}
	```
- **插入**
	- 每插入一个值，从根节点开始，如果比这个节点小，进去左子树，否则进右子树，如果进入的子树为空，则新建节点并放入此处
	```cpp
	int insert(int now,int val){
		if(!now) return newnode(val);
		if(val<t[now].val){
			t[now].l=insert(t[now].l,val);
		}
		else if(val>t[now].val){
			t[now].r=insert(t[now].r,val);
		}
		else {
			t[now].count++;
		}
		t[now].size=t[t[now].l].size+t[t[now].r].size+t[now].count;
	}
	```
- **删除**
	- 在以$root$为节点的二叉树先搜索大小为指定值的节点
		- 如果$root$为叶节点，直接删除
		- 如果$root$只有一个儿子，返回这个儿子
		- 如果$root$有两个非空子节点，一般是用它左子树的最大值（左子树最右的节点）或右子树的最小值（右子树最左的节点）代替它，然后将它删除
		>有一说一我还是喜欢fhq treap，所以这部分就不写了，fhq treap有更简单的写法
- **查找最大/最小值**
	- 最小值为左链端点，最大值为右链条端点
		>简单 略
- **搜索元素**
	- 如果在以$root$为根节点的树上寻找为$val$的值
		- 如果$root$为空，返回$false$
		- 如果$root$的值等于$val$，返回$true$
		- 如果$root$的值大于$val$，寻找左子树
		- 反之寻找右子树
		```cpp
		bool search(int &now,int val){
			if(!now) return false;
			if(t[now].val==val) return true;
			else if(val<t[now].val){
				return search(t[now].l,val);
			}
			else {
				return search(t[now].r,val);
			}
		}
		```
- **求元素排名**
	- 从根节点开始寻找元素，如果进入右子树，答案加上左子树的大小并加当前节点重复个数，最后找到元素后再加上终点左子树的大小加一
		```cpp
		int rank(int now,int val){
			if(!now) return 0;
			if(t[now].val==val) return(t[t[now].l].size+1);
			if(t[now].val>val) return rank(t[now].l,val);
			else return rank(t[now].r,val)+t[t[now].l].size+t[now].size;
		}
		```
- **求第k大元素**
	- 元素的排名取决于他所在的根节点的左子树大小
	- 因此如果左子树的大小大于等于k，那么此元素在左子树
	- 如果左子树的大小区间在$[k-count,k-1]$之中，那么此元素为此根节点
	- 如果左子树大小小于$k-count$，那么元素在右子树中
	```cpp
	int getnum(int now,int k){
		if(!now) return -1;
		if(t[now].l){
			if(t[t[now].l].size>=k) return getnum(t[now].l,k);
			if(t[t[now].l].size+t[now].count>=k) return t[now].val;
		}
		else {
			if(k==1) return t[now].val;
		}
		return getnum(t[now].r,k-t[t[now].l].size-t[now].count);
	}
	```

# 平衡树
## 内容
- 我们发现，如果我们按照$1，2，3，4，5，6 ...$的顺序依次插入元素，二叉搜索树会退化成链表，那么他所拥有的复杂度优势也会失去，因为二叉搜索树的时间复杂度取决于它的高度$h$ 
- 我们理想的二叉搜索树是左右子树大小差不多，也就是平衡的
## 平衡树双子星
- **Fhq Treap**和**Splay**是人称oi界的平衡树双子星
- 前者通过随机法来维护数组平衡
- 后者通过Splay不断旋转来维持平衡
  ~~有一说一我觉得学这两个一个就够用了，特别是fhq treap神中神~~