---
title: 树上启发式合并 | 学习笔记
date: 2023-05-31 21:20:30
categories: 图论
tags: 学习笔记
author: GrainRain
cover: https://pic.imgdb.cn/item/64b0176c1ddac507cc96f72b.jpg
single_column: true
---

# 树上启发式合并 $dsu\ on\ tree$

~~和 并查集 ($\sout{Disjoint\ Set\ Union}$) 一点儿关系没有~~

## 基本原理

利用遍历中的重复信息达到减少时间复杂度的效果。

考虑暴力计算子树信息的方式：

```cpp
void dfs(int u, int fa) {
	for (int i = head[u], j; ~i; i = edge[i].nxt) {
		j = edge[i].to;
		if (j != fa) dfs(u);
	}
	addSubtree(u);
	/* Calculate the Answers */
	delSubtree(u);
}
```

发现在上面的 $dfs$ 遍历过程中，最后一个子树的信息在计算当前节点父节点的子树中仍需要重新计算，因此遍历最后一棵子树时可以不删除存储信息。

保留哪棵子树的信息好呢？当然是子节点规模最大的一棵（这种~~玄学~~优化方式被称为启发式算法），因此我们想到保留重儿子。

```cpp
void dfs(int u, int fa, bool kp = 1) {
// kp -> keep, 传入子树是否保留信息 
	for (int i = head[u], j; ~i; i = edge[i].nxt) {
		j = edge[i].to;
		if (j != fa and j != hs[u]) dfs(j, u, false);
	}
    if (hs[u]) dfs(hs[u], u, true);
    add(u);
	for (int i = head[u], j; ~i; i = edge[i].nxt) {
		j = edge[i].to;
		if (j != fa and j != hs[u]) addSubtree(v, u);
	}
    /* Calculate the Answers */
    if (!kp) delSubtree(u, fa);
}
```

不过需要注意的是，由于不需要保证重链上的 $dfn$ 序连续，只需要保证子树 $dfn$ 序连续即可，因此无需像树链剖分一样写成两个 $dfs$，维护重子节点的同时进行 $dfn$ 序遍历即可

## 模板

以维护子树颜色个数为例：

### 前置数组

```cpp
int col[N], cnt[N], tCol;
int ans[N];
int siz[N], hs[N], node[N], dfn[N], stamp;
```

$tCol$ 维护颜色种类. 
$hs$ 表示当前节点的重儿子. 
$node$ 存储 $dfn$ 到树上节点编号的映射. 

### 维护颜色个数桶

```cpp
inline void add(int u) {
	if (!cnt[col[u]]) tCol ++;
	cnt[col[u]] ++;
}
inline void del(int u) {
	cnt[col[u]] --;
	if (!cnt[col[u]]) tCol --;
}
```

### 初始化 $hs$ 重子节点与 $dfn$ 序

由于不需要保证所有重链 $dfn$ 连续，只保证子树 $dfn$ 连续即可. 因此写在同一个 $dfs$. 

```cpp
void init(int u, int fa) {
	dfn[u] = ++ stamp, siz[u] = 1, node[stamp] = u;
	for (int i = head[u], j; ~i; i = edge[i].nxt) {
		j = edge[i].to;
		if (j == fa) continue;
		init(j, u);
		siz[u] += siz[j];
		if (siz[hs[u]] < siz[j]) hs[u] = j;
	}
}
```

### 预处理子树颜色个数

```cpp
void dfs(int u, int fa, bool kp) {
	for (int i = head[u], j; ~i; i = edge[i].nxt) {
		j = edge[i].to;
		if (j != fa and j != hs[u]) dfs(j, u, false); 
		// 判掉重儿子保证时间复杂度 很重要
	}
	if (hs[u]) dfs(hs[u], u, true); // 最后遍历重子节点 
	for (int i = head[u], j; ~i; i = edge[i].nxt) {
		j = edge[i].to;
		if (j == fa or j == hs[u]) continue;
		for (int k = dfn[j]; k <= dfn[j] + siz[j] - 1; k ++) 
			add(node[k]);
	}
	add(u);
	ans[u] = tCol;
	if (!kp) for (int i = dfn[u]; i <= dfn[u] + siz[u] - 1; i ++) 
		del(node[i]);
}
```

## 例题

### 树上 $k$ 级表亲：[Blood Cousins](https://www.luogu.com.cn/problem/CF208E)

考虑到直接找表亲不好找，于是先从当前点 $u$ 跳到 $k$ 级祖先，再在其祖先的子树中寻找 $k$ 级子节点（可表示为 $dep[fa] + k$），开桶计数即可

将询问离线处理，树剖跳 $k$ 级祖先，$dsu\ on\ tree$ 询问子树 $k$ 级子节点。时间复杂度 $O(n \cdot logn)$

```cpp
int rt[N], rTop;
vector<PII> q[N]; // vertex and kth 
// basis 
int fa[N], dep[N], siz[N], hs[N], top[N], dfn[N], stamp;
int node[N];
// cuttings 
int cnt[N], ans[N];
// dsu on tree 

inline void add(int u) { cnt[dep[u]] ++; }
inline void del(int u) { cnt[dep[u]] --; }

void dfs1(int u, int depth) {
	dep[u] = depth, siz[u] = 1;
	for (int i = head[u], j; ~i; i = edge[i].nxt) {
		j = edge[i].to;
		dfs1(j, depth + 1);
		siz[u] += siz[j];
		if (siz[j] > siz[hs[u]]) hs[u] = j;
	}
	return;
}

void dfs2(int u, int Top) {
	dfn[u] = ++ stamp, top[u] = Top, node[stamp] = u;
	if (!hs[u]) return; dfs2(hs[u], Top);
	for (int i = head[u], j; ~i; i = edge[i].nxt) {
		j = edge[i].to;
		if (!top[j]) dfs2(j, j);
	}
	return;
}

void dfs3(int u, bool kp) {
	for (int i = head[u]; ~i; i = edge[i].nxt)
		if (edge[i].to != hs[u]) dfs3(edge[i].to, false);
	if (hs[u]) dfs3(hs[u], true);
	for (int i = head[u], j; ~i; i = edge[i].nxt) {
		j = edge[i].to; if (j == hs[u]) continue;
		for (int k = dfn[j]; k <= dfn[j] + siz[j] - 1; ++ k) add(node[k]);
	}
	add(u);
	for (auto t : q[u]) ans[t.second] = cnt[t.first + dep[u]];
	if (!kp) for (int i = dfn[u]; i <= dfn[u] + siz[u] - 1; ++ i) del(node[i]);
	return;
}

int query(int u, int kth) {
	while (kth >= dfn[u] - dfn[top[u]] + 1 and u) {
		kth -= dfn[u] - dfn[top[u]] + 1;
		u = fa[top[u]];
	}
	return node[dfn[u] - kth];
}

int main() {
	memset(head, -1, sizeof head);
	input(n);
	for (int i = 1; i <= n; ++ i) {
		input(fa[i]);
		if (fa[i]) add(fa[i], i);
		else rt[++ rTop] = i;
	}
	
	for (int i = 1; i <= rTop; ++ i) dfs1(rt[i], 1), dfs2(rt[i], rt[i]);
	
	input(m);
	for (int i = 1, x, y, z; i <= m; ++ i) {
		input(x), input(y);
		z = query(x, y);
		q[z].push_back({y, i});
	}
	for (int i = 1; i <= rTop; ++ i) dfs3(rt[i], 0);
	for (int i = 1; i <= m; ++ i) output(max(ans[i] - 1, 0), " \n"[i == m]);
	return 0;
}
```