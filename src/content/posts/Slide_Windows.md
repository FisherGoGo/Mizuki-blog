---
title: 滑动窗口
published: 2024-11-18
tags: [Algorithm]
category: Knowledge
draft: false
---

# 作用
- 长度为k的窗口在一个数列中滑动
- 处理长度为k的序列中的数据问题，比如求连续区间内最大最小值

# 实现
- 用一个 *Q* 队列存储当前窗口里的数，Num最左侧的元素既当前窗口最大（小）值
  - 如果是求最大值，*Q* 队列的元素不递增，反之不递减
 <br>
- 例子
  - 一个 *1 3 -1 -3 5 3 6 7* 的数列，窗口大小为 3，我们要找窗口内最大值
  - *1* 先进来，数组内没有比*1* 大的值，直接压入*1* 
  - 然后 *3* 进来，*3* 比 *1* 大，*1* 永远不可能成为窗口内最大值，所以 *1* 弹出
  - *-1* 进来，如果 *3* 出去了，*-1* 还有可能成为最大值，所以 *-1* 入队
  - *-3* 同上，入队
  - 现在队列是 *3 -1 -3* ，开始考虑 *5* 入队，但是别忘记窗口大小为 **3** ，所以在窗口之外的元素要出队，比如 *3* 就要出队，然后 *5* 进来就同上
 <br>
- 具体实现
  - 使用双指针，一个 *head* 代表队列的头，一个 *tail* 表示队列的尾部 
  - 同时用一个数组 *pos* 存元素的下标
  - 记住 $pos_{tail}-pos_{head}$ 既窗口长度小于给定的长度
 <br>

- 具体代码

<details>
  <summary>滑动窗口</summary>

```cpp
//m为窗口长度

//区间最大值
for(int i=1;i<=n;i++){
	while(head<=tail&&pos[head]+m<=i) head++;
	while(head<=tail&&a[i]>a[pos[tail]]) tail--;
}

//区间最小值
for(int i=1;i<=n;i++){
	while(head<=tail&&pos[head]+m<=i) head++;
	while(head<=tail&&a[i]<a[pos[tail]]) tail--;
}
```
</details>



# 例题

- [洛谷 P1886](https://www.luogu.com.cn/problem/P1886)  **模板题**
- [洛谷 P2251](https://www.luogu.com.cn/problem/P2251)