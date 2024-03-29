---
title: 最小生成树 | 学习笔记
date: 2023-1-20 21:16:14
categories: 图论
tags: 图论
author: GrainRain
cover: https://pic.imgdb.cn/item/64b010071ddac507cc7679e3.jpg
single_column: true
---


# 最小生成树 $MST$

最小生成树指在无向图中找出任意两点都相互连通的树结构的最小边权

值得注意的是，一张无向图的最小生成树可能有很多棵，而如果一张图无法获得一个最小生成树，则得到的是最小生成森林（很形象化的名字）

## $Prim$

### 朴素版 $O(n^2)$

- 适用于稠密图（时间复杂度与边的个数无关）

#### 算法思想

类似于 $Dijkstra$ 算法，先定义一个集合存储最小生成树，之后进行以下几步：

1. 将所有距离初始化为正无穷
2. 进行 $n$ 次迭代，每次迭代中找到集合外**距离集合**最近的点 $t$ 
3. 用 $t$ 更新其他点到**集合**的距离

#### 模板

```cpp
int g[N][N];
int dist[N];
bool st[N];

int n ,m;

int prime(){
	memset(dist, 0x3f, sizeof dist);
	int ans = 0;
	// 用于存储最小生成树中所有边权之和 
	for (int i = 0; i < n; i ++){
		int t = -1;
		for (int j = 1; j <= n; j ++)
			if (!st[j] && (t == -1 || dist[t] > dist[j]))
				t = j;
		// 取出集合外距离集合最近的点 
		if (i && dist[t] == INF) return INF;
		// 如果不是第一个点并且距离为正无穷，说明该图不连通
		// 则没有最小生成树，判无解
		if (i) ans += dist[t];
		// 如果不是第一个点，则将该点边权加入最小生成树 
		
		for (int j = 1; j <= n; j ++)
			dist[j] = min(dist[j], g[t][j]);
		// 使用该点进行更新 
		
		st[t] = true;
		// 将该点加入集合 
	}
	return ans;
}
```

在 $n$ 次的迭代中循环查找，因此时间复杂度为 $O(n^2)$，类比于 $Dijkstra$ 的二叉堆优化，使用 `priority_queue` 可以将循环中查找的时间复杂度降到 $O(m·log_2n)$，相比之下，$Kruskal$ 在时间复杂度相同的情况下思路更简单，因此更多地使用 $Kruskal$ ，请接着向下阅读

## $Kruskal$ $O(m \cdot log_2m)$

- 适用于稀疏图（时间复杂度随边数变化而变化）

### 算法思想

1. 将所有边按权重从小到大排序 $O(m \cdot log_2m)$（常数很小）
2. 枚举每条边，设这条边从 $a$ 指向 $b$ ，权重为 $c$。如果当前 $a$ 和 $b$ 所在的集合不连通（使用并查集维护），就将这条边加入集合中

### 模板

```cpp
int p[N]; // 并查集 father 数组
int find(int x) {
	if (p[x] != x) p[x] = find(p[x]);
	return p[x];
}

void kruskal() {
	sort(edge, edge + m);
	for (int i = 1; i <= n; i ++) p[i] = i;
	// 初始化并查集 
	
	int res = 0, cnt = 0;
	// res 存储最小生成树中边权之和
	// cnt 存储最小生成树中边数 
	for (int i = 0; i < m; ++ i) {
		int a = edge[i].a, b = edge[i].b, w = edge[i].w;

		a = find(a), b = find(b);
		if (a != b){
			p[a] = b; 
			res += w;
			cnt ++;
		}
	}
	if (cnt < n - 1) puts("impossible");
	else cout << res << endl;
}
```

## 拓展：[严格次小生成树](https://www.luogu.com.cn/problem/P4180)

可以在学完 [树链剖分]() 之后继续以下阅读

