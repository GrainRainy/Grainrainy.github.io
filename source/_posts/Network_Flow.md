---
title: 网络流基础模板
date: 2023-6-19 21:20:18
categories: 图论
tags: 学习笔记
author: GrainRain
cover: https://pic.imgdb.cn/item/64bfccda1ddac507cc5bf040.png
single_column: true
---


# 网络流 $Network\ Flow$

~~网络瘤~~

## 基本概念

### 符号约定

$S$ 表示源点,  $T$ 表示汇点

容量一般用 $c$ 表示, $u$, $v$ 间的流量一般用 $f_{(u, v)}$ 表示, 整张图的 $f$ 被称为流函数. 

对 $f$ 的限制有以下两点：
1. 容量限制： $0 \le f_{(u, v)} \le c_{(u, v)}$
2. 流量守恒：对于 $\forall v \ne S, v \ne T$ 流入流出流量满足 $\sum \limits_{(a, v) \in E} f_{(a, v)} = \sum \limits_{(v, b) \in E} f_{(v, b)}$. 因此除源点与汇点的其他各个节点不储存流量. 

满足以上两个性质即为**可行流**

### 可行流

定义 $\left | f \right |$ 为总流量大小, 则 $\left | f \right | = \sum\limits_{(s, v) \in E} f_{(s, v)} - \sum\limits_{(v, s) \in E} f_{(v, s)}$

我们称 $\left | f \right |$ 取得最大值的情况为最大流（最大可行流）

## 最大流

事实上, 所有最大流算法都基于 $Ford-Fulkerson$ 增广算法和 $Push-Relabel$ 预流推进算法. 

### $Ford-Fulkerson$ 增广算法

#### $Edmonds-Karp$ 单路增广

- 时间复杂度 $O(n \cdot m^2)$, 一般可通过 $1e3 \sim 1e4$ 规模的图. 

包含重边优化

##### 前置数组

```cpp
int n, m, s, t, u, v, f;
int head[N], idx = 1;
struct Node { int nxt, to; LL f; }edge[M << 1];
// graph 
int pre[N], pos[N][N];
// pre 用于存储增广路中一点的前驱, 用于寻找增广路 
// pos 作为重边优化存储链表位置
LL incf[N], maxFlow;
bool vis[N];
```

##### 寻找增广路

```cpp
bool bfs() {
	for (int i = 1; i <= n; ++ i) vis[i] = false; 
	queue<int> q;
	q.push(s), vis[s] = true, incf[s] = INF;
	while (q.size()) {
		auto u = q.front(); q.pop();
		for (int i = head[u], j; ~i; i = edge[i].nxt) {
			if (!edge[i].f) continue;
			j = edge[i].to;
			if (vis[j]) continue;
			incf[j] = min(incf[u], edge[i].f);
			pre[j] = i; // 记录当前点前驱, 用于倒推增广路, 注意存储的是 nxt 指针
			q.push(j), vis[j] = true;
			if (j == t) return true;
		}
	}
	return false;
}
```

##### 使用 $bfs$ 搜到的增广路更新剩余流量与答案 $maxFlow$

增广路可行流 $f$ 为增广路中所有边剩余流量最小值, 即为 $incf_t$

```cpp
void update() {
	int x = t, i;
	while (x != s) {
		i = pre[x];
		edge[i].f -= incf[t];
		edge[i ^ 1].f += incf[t]; // 修改反向边剩余流量
		x = edge[i ^ 1].to;
	}
	maxFlow += incf[t];
}
```

注意连入反向边, 并将反向边限制流量设为 $0$. 

```cpp
int main() {
	input(n), input(m), input(s), input(t);
	memset(head, -1, sizeof(int) * (n + 10));
	for (int i = 1; i <= m; ++ i) {
		input(u), input(v), input(f);
		if (pos[u][v]) edge[pos[u][v]].f += f;
		else add(u, v, f), pos[u][v] = idx, add(v, u, 0);
        // 重边优化 
	}
	while (bfs()) update();
	cout << maxFlow << endl;
	return 0;
}
```

##### 时间复杂度 $O(n \cdot m^2)$ 证明

显然, 单轮 BFS 增广的时间复杂度是 $O(m)$. 

增广轮数最多为 $O(n \cdot m)$, 证明详见 [OI-Wiki](https://oi-wiki.org/graph/flow/max-flow/#%E6%97%B6%E9%97%B4%E5%A4%8D%E6%9D%82%E5%BA%A6%E5%88%86%E6%9E%90) ~~太长了, 不想写~~

因此总时间复杂度为 $O(n \cdot m^2)$

尝试思考 $EK$ 算法进行了哪些不必要的搜索操作, 是增广路算法优化的关键所在：

发现




#### $Dinic$ 多路增广

```cpp
int n, m, s, t, u, v, f, maxFlow;
int head[N], now[N], idx = 1;
struct Node { int nxt, to, f; } edge[M << 1];
// graph 
int dis[N], pos[N][N]; // pos 用于重边优化 
```

```cpp
bool bfs() {
	for (int i = 1; i <= n; ++ i) dis[i] = INF;
	queue<int> q;
	q.push(s), dis[s] = 0, now[s] = head[s];
	while (q.size()) {
		int u = q.front(); q.pop();
		for (int i = head[u], j; ~i; i = edge[i].nxt) {
			if (edge[i].f and dis[j = edge[i].to] == INF) {
				q.push(j), dis[j] = dis[u] + 1, now[j] = head[j];
				if (j == t) return true;
			}
		}
	}
	return false;
}
```

```cpp
int dfs(int u, int limit) {
	if (u == t) return limit;
	int k, res = limit;
	for (int i = now[u], j; ~i and res; i = edge[i].nxt) {
		now[u] = i, j = edge[i].to;
		if (edge[i].f and dis[j] == dis[u] + 1) {
			k = dfs(j, min(res, edge[i].f));
			if (!k) dis[j] = INF;
			edge[i].f -= k;
			edge[i ^ 1].f += k;
			res -= k;
		}
	}
	return limit - res;
}
```

```cpp
int main() {
	input(n), input(m), input(s), input(t);
	memset(head, -1, sizeof(int) * (n + 10));
	for (int i = 1; i <= m; ++ i) {
		input(u), input(v), input(f);
		if (pos[u][v]) edge[pos[u][v]].f += f;
		else add(u, v, f), pos[u][v] = idx, add(v, u, 0);
	}
	while (bfs()) maxFlow += dfs(s, INF);
	cout << maxFlow << endl;
	return 0;
}
```

## 最小费用最大流 

$Minimum\ Cost\ Maximum\ Flow$ 问题

[$\rm Luogu\ P3381$ - 最小费用最大流](https://www.luogu.com.cn/problem/P3381)

给定一个网络 $G = (V, E)$, 每条边除了有容量限制 $f(u, v)$, 还有一个单位流量的费用 $c(u, v)$. 要求在取得最大流量的同时付出最小代价

所有的 $MCMF$ 算法都基于 $SSP$ 的思想, $SSP$（$Successive\ Shortest\ Path$）算法是一个贪心的算法. 通过每次寻找单位费用最小的增广路进行增广, 直到图上不存在增广路为止. 

~~事实上就是在最大流上套一个最短路~~

为了维护反悔边的负权, 有两种解决方案：

### 1. 使用 $spfa$

使用普通快读, 洛谷 $\text{p3381}$ 模板题用时 $\text{1.10s}$. 

#### 前置数组

$edge$ 中 $f$ （$flow$） 表示限制流量, $c$ （$cost$）表示单位流量费用. 

```cpp
int n, m, s, t, u, v, f, c, maxFlow, minCost;
// bases 
int head[N], now[N], idx = 1;
struct Node { int nxt, to, f, c; }edge[M << 1];
// graph 
int dis[N]; bool st[N];
// spfa 
```

#### 找到费用最小的增广路

```cpp
bool spfa() {
	memset(st, 0, sizeof st);
	for (int i = 1; i <= n; ++ i) dis[i] = 0x3f3f3f3f;
	queue<int> q;
	q.push(s), dis[s] = 0, st[s] = true, now[s] = head[s];
	while (q.size()) {
		int u = q.front(); q.pop(); st[u] = false;
		for (int i = head[u], j; ~i; i = edge[i].nxt) {
			j = edge[i].to;
			if (edge[i].f and dis[j] > dis[u] + edge[i].c) {
				dis[j] = dis[u] + edge[i].c;
				now[j] = head[j];
				if (!st[j]) q.push(j), st[j] = true;
			}
		}
	}
	return dis[t] != INF;
}
```

#### 更新增广路

```cpp
int dfs(int u, int limit) {
	if (u == t) return limit;
	int k, res = limit;
	st[u] = true;
	for (int i = now[u], j; ~i and res; i = edge[i].nxt) {
		now[u] = i;
		j = edge[i].to;
		if (!st[j] and edge[i].f and dis[j] == dis[u] + edge[i].c) {
			k = dfs(j, min(res, edge[i].f));
			if (!k) dis[j] = INF;
			edge[i].f -= k;
			edge[i ^ 1].f += k;
			res -= k;
			minCost += edge[i].c * k;
		}
	}
	st[u] = false; 
	return limit - res;
}
```

#### 建图及统计答案

```cpp
int main() {
	memset(head, -1, sizeof head);
	input(n), input(m), input(s), input(t);
	for (int i = 1; i <= m; ++ i) {
		input(u), input(v), input(f), input(c);
		add(u, v, f, c), add(v, u, 0, -c);
	}
	while (spfa()) maxFlow += dfs(s, INF);
	cout << maxFlow << ' ' << minCost << endl;
	return 0;
}
```

但众所周知, $\sout{\text{Spfa 他死了}}$ $Spfa$ 的运行效率不稳定, 在稀疏图中效率仍可接受, 而在稠密图中实际运行时间差强人意. 而相比之下 $Dijkstra$ 实际运行时间方差远远小于 $Spfa$, 因此为了使用 $Dijkstra$, 我们就需要解决反向边的负权问题. 

### 2. $Primal-Dual$ 原始对偶算法



## 最小割

> 最小割定理：
