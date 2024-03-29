---
title: 最短路算法 | 学习笔记
date: 2023-1-17 15:42:34
categories: 图论
tags: 图论
author: GrainRain
cover: https://pic.imgdb.cn/item/63d53e8aface21e9efc274a0.jpg
single_column: true
---


# 单源最短路 $Single\ Source\ Shortest\ Path$

源点指起点, 单源最短路径问题即为求一个点到另一个点的最短路. 

## $\rm Dijkstra$ 

- 基于贪心的思想. 
- 仅适用于正数边权. 

使用 $n$ 表示点集, $m$ 表示边集, 朴素版 $Dijkstra$ 时间复杂度为 $O(n^2)$ , 二叉堆优化的 $Dijkstra$ 时间复杂度为 $O(m \cdot log_2n)$. 

由于朴素版 $Dijkstra$ 时间复杂度与边数没有关系, 因此**稠密图适合使用朴素版 $Dijkstra$**. 

### 朴素版 $O(n^2)$ 

$Dijkstra$ 算法的基本思想有以下几步：

1. 初始化距离 $dist$ `dist[1] = 0, dist[i] = INF`
1. 循环 $n$ 次重复找到当前位置能确定的 (且未被标记的), 距源点最近的节点 $x$, 放入标记最短距离的点 $s$ 数组, 再用该点更新与之最近的点. 

```cpp
int map[N][N];
// 邻接矩阵存图
int n, m;
int dist[N];
// 存储点的距离 
bool st[N]; 
// 存储已经确定最短距离的点 

int dijkstra() {
	memset(dist, 0x3f, sizeof dist);
	dist[1] = 0;
	for (int i = 0; i < n; i ++) { // 迭代 n 次求最短路 
		int t = -1;
		for (int j = 1; j <= n; j ++)
			if (!st[j] && (t == -1 or dist[t] > dist[j]))
				t = j; // 更新可扩展的最短路径 
		st[t] = true; // t 已确定最小值, 标记为最短
		for (int j = 1; j <= n; j ++)
			dist[j] = min(dist[j], dist[t] + map[t][j]);
			// 使用 t 对新的最短边进行更新
	}
	if (dist[n] == 0x3f3f3f3f) return -1; // 特判无解 
	return dist[n]; 
}
```

### 二叉堆优化

在上述算法中, 我们枚举了所有点 $n$ 以及与该点相连的其他点, 因此 $\rm Dijkstra$ 未优化时的时间复杂度为 $O(n^2)$

在此基础上, 如果使用 $\rm STL$ 二叉堆对 $dis$ 数组进行维护, 可以达到 $O(m \cdot \log _2 n)$ 的时间复杂度

```cpp
int dist[N];
bool st[N]; 
priority_queue<PII, vector<PII>, greater<PII> > heap;
// 小根堆, 每次取出距离源点最近的点
// 双关键字, 节点大小以及节点编号 

void dijkstra() {
	memset(dist, 0x3f, sizeof dist);
	dist[1] = 0;
	heap.push({0, 1});
	while (heap.size()) {
		PII t = heap.top(); heap.pop();
		int cur = t.second; // 节点编号
		if (st[cur]) continue; // 当已经确定该点的最短路径时, 代表当前队列中存储的元素冗余, 直接跳过
		st[cur] = true;
		for (int i = head[cur], j; ~i; i = edge[i].nxt) {
			j = edge[i].to;
			if (dist[j] > dist[cur] + edge[i].w) {
				dist[j] = dist[cur] + edge[i].w;
				heap.push({dist[j], j});
			}
		}
	}
}
```

## $\rm Bellman - Ford$ 

- 适用于对经过的边数有限制的最短路问题. 
- 可适用于负权边. 
- 时间复杂度 $O(m \cdot n)$. 

### 基本过程

1. 进行 $n$ 次迭代, 迭代 $k$ 次表示经过不超过 $k$ 条边所能到达的点的最短路. 
2. 遍历所有边 $a$, $b$, $w$ 表示从 $a$ 到 $b$ 存在边权为 $w$ 的边. 
3. 松弛操作 `dist[b] = min(dist[b], dist[a] + w)`. 

另外, 如果在迭代 $n$ 次之后再次迭代仍可更新, 则代表图中存在负环. 

```cpp
int n, m, k;
int dist[N], last[N];
struct node{
	int a, b, w;
}edge[M];
// 表示从 a 到 b 有一条边权为 w 的边 

void bellman_ford() {
	memset(dist, 0x3f, sizeof dist);
	dist[1] = 0;

	for (int i = 0; i < k; i ++) {
		memcpy(last, dist, sizeof dist);
		// 备份 dist, 存储上一次迭代结果
		for (int j = 0; j < m; j ++) {
			int a = edge[j].a, b = edge[j].b, w = edge[j].w;
			dist[b] = min(dist[b], last[a] + w);
			// 使用上一次迭代结果更新, 防止前面的结果影响后面 
		}
	}
}

int main() {
	cin >> n >> m >> k;
	for (int i = 0, a, b, w; i < m; i ++) {
		cin >> a >> b >> w;
		edge[i] = {a, b, w};
	}
	bellman_ford();
	if (dist[n] > 0x3f3f3f3f / 2) puts("impossible");
	else cout << dist[n] << endl;
	return 0;
}
```

## $\rm spfa$

### 算法思路

此时我们再观察 $\rm Bellman\ Ford$ 算法的公式 `dist[b] = min(dist[b], dist[a] + w[i])` , 会发现只有在 `dist[a]` 减小之后, `dist[b]` 才有可能进行更新, 因此我们可以将与 `dist[]` 减小点相连的点全部入队, 每次进行松弛操作, 可以达到减少循环的效果. 

**具体实现可以分为以下几步：**

1. 将起点放入队列. 
2. 当队列不空时, 取出队头 $t$. 
3. 更新 $t$ 的所有出边. 
4. 如果 $t$ 更新到的点不在队列中, 将该点距离更新并加入队列. 

### $\rm spfa$ 特点

- 可适用于负数边权. 
- 不可用于负环. 
- 平均时间复杂度 $O(m)$, 由于常数问题, 最坏情况下时间复杂度为 $O(n \cdot m)$, 经常被~~毒瘤~~出题人卡常. 

```cpp
void spfa() {
	memset(dist, 0x3f, sizeof dist);
	dist[1] = 0;
	
	queue<int> q; // 存储所有待更新的点 
	q.push(1); st[1] = true; // st 存储当前点是否在队列中
	while (q.size()) {
		int t = q.front();
		q.pop(), st[t] = false;
		for (int i = head[t], j; ~i; i = edge[i].nxt) {
			j = edge[i].to;
			if (dist[j] > dist[t] + edge[i].w) {
				dist[j] = dist[t] + edge[i].w;
				if (!st[j]) q.push(j), st[j] = true;
			}
		}
	}
}
```

### $\rm spfa$ 判负环

在使用 $spfa$ 判断负环时, 只需要多存储从**源点到该点的边数** `cnt[]`, 当 `cnt[x] >= n` 时, 表示经过了大于 $n$ 条边, 由抽屉原理可得, 图中必定存在负环. 

使用 `cnt[]` 数组存储最短路径长度, 在 `dist[j] = dist[t] + edge[i].w` 更新时记录路径长度 `cnt[j] = cnt[t] + 1` , 大于等于 $n$ 返回即可. 

或存储入队次数, 若入队大于 $n$ 此次则存在负环. 

```cpp
typedef pair<int, int> PII;
int n, m;
int dist[N], cnt[N];
bool st[N];

bool spfa() {
	queue<int> q;
	for (int i = 1; i <= n; i++) st[i] = true, q.push(i);
	while (q.size()) {
		int t = q.front(); q.pop();
		st[t] = false;
		for (int i = head[t], j; ~i; i = edge[i].nxt) {
			j = edge[i].to;
			if (dist[j] > dist[t] + edge[i].w) {
				dist[j] = dist[t] + edge[i].w;
				cnt[j] = cnt[t] + 1;
				if (cnt[j] >= n) return true;
				if (!st[j]) q.push(j), st[j] = true;
			}
		}
	}
	return false;
}
```

### $\rm spfa$ 与差分约束系统

差分约束系统, 指若干个形如 $x_i - x_j \leqslant w$ 的不等式. 

可以用 $\rm spfa$ 求对于给定的一组差分约束系统中各变量的一个特解. 

尝试将 $x_i - x_j \leqslant a$ 转化为图论模型. 原式等价于 $x_i \leqslant x_j + w$ 将 $x_i$ 视为从起点到 $i$ 的最短路, 则原式的实际意义为点 $j$ 到点 $i$ 的距离不超过 $w$, 即从 $j$ 向 $i$ 连一条边权为 $w$ 的边, $\rm spfa$ 最终跑出最短路的 $\rm dist_i$ 即为原式的一组特解. 

另外, 由这样的不等式组成的图可能出现负环, 当一个点入队 $n + 1$ 次时, 根据抽屉原理, 必然出现一条边被经过两次, 即出现负环. 

具体实现上需要注意的是, 原图也可能出现不连通的情况. 因此需要建立超级源点或初始时将所有点入队. 

```cpp
bool spfa() {
	queue<int> q;
	for (int i = 1; i <= n; ++ i) q.push(i), st[i] = true;
	while (q.size()) {
		int u = q.front(); q.pop(); st[u] = false;
		for (int i = head[u], j; ~i; i = edge[i].nxt) {
			j = edge[i].to;
			if (dis[j] > dis[u] + edge[i].w) {
				dis[j] = dis[u] + edge[i].w;
				if (!st[j]) st[j] = true, q.push(j), cnt[j] ++;
				if (cnt[j] > n) return false;
			}
		}
	}
	return true;
}

int main() {
	rd(n), rd(m);
	memset(head, -1, sizeof(int) * (n + 10));
	for (int i = 1, x, y, z; i <= m; ++ i)
		rd(x), rd(y), rd(z), add(y, x, z);
	if (!spfa()) return puts("NO") & 0;
	for (int i = 1; i <= n; ++ i) ot(dis[i], ' ');
	return 0;
}
```

#### 应用

##### [赛车游戏](https://www.luogu.com.cn/problem/P5590)



-----------

# 多源汇最短路径问题

## $\rm Floyd$

- 基于动态规划的思想
- 时间复杂度 $O(n^3)$

### 算法思路

基于动态规划, 三维状态 $d_{k, i, j}$ 表示**从点 $i$ 经过 $1$ 到 $k$ 这些中间点到达 $j$ 的最短距离**. 使用三层循环枚举中间点, 每层循环内尝试更新最短路径, 因此 $Floyd$ 算法的时间复杂度是 $O(n^3)$

注意枚举顺序. 

```cpp
void floyd() {
	for (int k = 1; k <= n; k ++)
		for (int i = 1; i <= n; i ++)
			for (int j = 1; j <= n; j ++)
				dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);
}
```
