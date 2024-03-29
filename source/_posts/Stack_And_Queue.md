---
title: 栈和队列 | 学习笔记
date: 2022-10-08 14:16:25
categories: 数据结构
tags: 学习笔记
author: GrainRain
cover: https://pic.imgdb.cn/item/63ba9fe4be43e0d30e10f62c.jpg
single_column: true
---


# 栈

## 特点：

元素遵循先进后出的原则
`tt`存储栈顶元素下标

## 前置数组

```cpp
int stk[N]; // 栈
int tt = 0; // 指向栈顶元素所在位置
```

## 基本操作

```cpp
stk[++ tt] = x; // 插入，从 1 开始存储

tt --; // 删除 

tt > 0 // 判空 true 则非空，false 则为空
```

## 单调栈

- 分为单调递增栈和单调递减栈，栈中元素具有单调性

- 栈中的元素是单调递增或单调递减的

- 对于栈中的 $a_x \geqslant a_y$,  $a_x$ 永远不会被作为答案输出，那么就在队列中删除 $a_x$ 以达到优化时间复杂度的目标

基本应用：求每一个数 左侧 / 右侧 离它最近且比他小的数，能将 $O(n^2)$ 的问题优化为 $O(n)$

[acwing830 - 单调栈](https://www.acwing.com/problem/content/832/)

```cpp
int n;
int stk[N], tt;
int main() {
	cin >> n;
	for (int i = 0; i < n; i ++) {
		int tmp;
		cin >> tmp;
		while (tt && stk[tt] >= tmp) tt --;
		if (!tt) cout << "-1" << ' ';
		else cout << stk[tt] << ' ';
		stk[++ tt] = tmp;
	}
}
```

# 队列

- 元素前进先出，在队尾插入元素，在队头弹出元素

## 前置数组

```cpp
int q[N]; // 队列
int hh; // 队头
int tt = -1; // 队尾
```

## 基本操作

```cpp
q[++ tt] = x; // 插入

hh ++; // 弹出

hh <= tt // 队列判空 true 则非空，false则为空
```

## 循环队列

节省空间复杂度

## 单调队列 / 滑动窗口

能够将 $O(n^2)$ 的时间复杂度优化成 $O(n)$

[acwing154 - 滑动窗口](https://www.acwing.com/problem/content/156/)

```cpp
int nums[N], q[N];
int hh = 0, tt = -1;

for (int i = 0; i < n; i ++) {
	while (hh <= tt && i - k + 1 > q[hh]) hh ++;
	//判断队头是否滑出窗口 
	while (hh <= tt && nums[q[tt]] >= nums[i]) tt --;
	// 对队列进行严格单调性处理
	// 如果要求最大值则将 >= 改为 <= 即可
	q[++ tt] = i; // 读入队列 
	if (i >= k - 1) cout << nums[q[hh]] << ' ';
	// 判断 i 是否小于区间长度，如果是则跳过 
}
```

## 例题

[COCI2009-2010#4 - OGRADA](https://www.luogu.com.cn/problem/P7697)

