---
title: 并查集 | 学习笔记
date: 2022-11-11 15:30:52
categories: 数据结构
tags: 学习笔记
author: GrainRain
cover: https://pic.imgdb.cn/item/63ba9aa8be43e0d30e07fe0f.jpg
single_column: true
---


# 并查集 $Union\ \text{-} Find\ Sets$

在近似于 $O(1)$ 的时间内**将两个集合合并**或**询问两个元素是否在一个集合中**

## 基本思想：<!--  -->

每个集合用一棵树表示，用树根编号代表集合。对于每一个子节点都存储它的父节点 `fa[x]`来解决问题

1. 判断两个元素是否属于同一个集合：
   不断查找该节点的父节点 `while (fa[x] != x) x = fa[x]` ，直到 `fa[x] == x`为止，即查找到了根节点
2. 合并两个集合：
   只需将 1 集合的根节点连向集合 2 的根节点
   ![](https://pic.imgdb.cn/item/63c66b0fbe43e0d30ef4afaa.jpg)


## 模板

### 初始化
```cpp
int fa[N];
// fa 数组，用于指向每个节点的父节点
for (int i = 1; i <= n; i ++) fa[i] = i;
// 将每个节点的父节点指向自己
```

### 查找根节点

```cpp
int find(int x) {
	if (fa[x] != x) fa[x] = find(fa[x]);
	// 如果 x 不为根节点，则向上查找
	// 同时将 x 的父节点指向根节点，处理路径压缩优化
	return fa[x];
	// 返回 fa[x] 此时的父节点 ( 即为根节点 ) 
}
```

一般简写为：

```cpp
int find(int x) {
	return x ^ fa[x] ? fa[x] = find(fa[x]) : x;
}
```

## 并查集优化

树形结构的查找操作时间复杂度为 $O(h)$ ($h$ 指树的深度)。因此优化的总思路是将并查集的树形结构深度尽可能缩小. 这一算法通常由主观感受而来，被称为启发式算法.

### 优化1：按秩合并

秩定义为树的高度 $h$ ，当两棵树秩不相等时，将秩小的树合并到秩大的树，减少查找祖先次数.

```cpp
void Union(int x,int y) {
	if (x == y) return;
	if(siz[x] >= siz[y]) fa[y] = x, siz[x] += siz[y];
	else fa[x] = y, siz[y] += siz[x];
}
```

### 优化2：路径压缩

![](https://pic.imgdb.cn/item/63c61bcabe43e0d30e604469.jpg)

我们会发现，以上的两种树形结构在并查集算法中是**完全等效**的，而第二种树形结构 $h$ 明显优于第一种树形结构，查找效率更高。因此对树的高度进行压缩，能够达到优化时间复杂度的效果，对于一个优化完全的并查集，不难发现查询的时间复杂度降到了 $O(1)$.

```cpp
int find(int x) {
	return fa[x] == x ? x : fa[x] = find(fa[x]);
}
```

### 优化3：空间复杂度优化

利用[哈希表]()的基本思想，将二维图映射到一维进行并查集维护.

[洛谷p5877 - 棋盘游戏](https://www.luogu.com.cn/problem/P5877)


## 并查集变种

   
### 1. 种类并查集

在维护如 `朋友的朋友是朋友，朋友的敌人是敌人，敌人的敌人是朋友` 一类的复杂关系

### 2. 带权并查集

[洛谷p2024 - 食物链](https://www.luogu.com.cn/problem/P2024)

### 3. 扩展域并查集



## 常见套路

### 动态维护集合元素数量：[Acwing837](https://www.acwing.com/problem/content/839/)

用集合来维护一个连通块，当在两个数字间连一条边时，即将两个集合合并。

```cpp
int siz[N];
// 表示每个集合中点的数量，只保证根节点的 size 有意义 
int a, b;
char op[3];
// operation 

for (int i = 1; i <= n; i ++) {
	fa[i] = i;
	siz[i] = 1;
}
// 将每一个点都建立一个集合

while (m --) {
	cin >> op;
	if (op[0] == 'C') {
		scanf("%d%d", &a, &b);
		if (find(a) == find(b)) continue;
		siz[find(b)] += siz[find(a)];
		// 在合并集合的同时，将元素个数同时合并 
		fa[find(a)] = find(b);
	}
	if (op[1] == '1') {
		scanf("%d%d", &a, &b);
		if (find(a) == find(b)) puts("Yes");
		else puts("No");
	}
	else {
		scanf("%d", &a);
		printf("%d\n", siz[find(a)]);
	}
}
```

### 并查集求最小环：[信息传递](https://www.luogu.com.cn/problem/P2661)

### 压缩二维状态：[棋盘游戏](https://www.luogu.com.cn/problem/P5877)