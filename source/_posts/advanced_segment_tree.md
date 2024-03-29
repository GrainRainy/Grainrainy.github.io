---
title: 线段树进阶应用
date: 2023-7-4 15:07:21
categories: 数据结构
tags: 学习笔记
author: GrainRain
cover: https://pic.imgdb.cn/item/64bfd2cc1ddac507cc65b86d.jpg
single_column: true
---

# 线段树进阶应用

## 动态开点线段树

多用于维护权值线段树, 省去离散化的时空复杂度. 

节点个数 $\mathcal{O}(n \cdot \log S)$. （$n$ 为插入次数, $S$ 为维护值域）

### 前置数组

```cpp
int root, tTol, lson[N << 1], rson[N << 1];
```

$root$ 表示线段树根节点. 
$tTol$ 维护栈顶位置, 达到动态开点的目的. 
$lson[i]\ /\ rson[i]$ 表示线段树节点 $i$ 的左右子节点. 

下文以维护区间和 $sum$ 为例, 附带懒标记处理. 需要注意的是, 与普通线段树一样, 懒标记已经对当前区间进行了处理. 

### 下传懒标记

```cpp
void pushDown(int u, int tl, int tr) {
	if (!lson[u]) lson[u] = ++ tTol;
	if (!rson[u]) rson[u] = ++ tTol;
	tag[lson[u]] += tag[u], tag[rson[u]] += tag[u];
	int mid = tl + tr >> 1;
	sum[lson[u]] += tag[u] * (mid - tl + 1);
	sum[rson[u]] += tag[u] * (tr - mid);
	tag[u] = 0;
}
```

### 插入 / 单点修改

不难发现每一层单次修改最多插入一个节点, 对于 $logn$ 层的动态开点线段树, 单次插入时间复杂度为 $logn$

```cpp
void insert(int &u, int tl, int tr, int pos, int val) {
// u 表示当前子树根节点, tl tr 表示当前节点区间, pos 表示插入位置, val 表示插入值
	if (!u) u = ++ tTol; // 动态开点 
	sum[u] += val; // 处理信息 
	if (tl == tr) return;
	int mid = tl + tr >> 1;
	if (pos <= mid) insert(lson[u], tl, mid, pos, val);
	else insert(rson[u], mid + 1, tr, pos, val);	
}
```

注意这里的线段树根节点传入引用值. 

### 区间修改

```cpp
void modifyRange(int &u, int tl, int tr, int l, int r, int val) {
	if (!u) u = ++ tTol;
	int len = min(tr, r) - max(tl, l) + 1; // 区间重合长度 
	sum[u] += val * len;
	if (tl >= l and tr <= r) return tag[u] += val, void(); // 打入懒标记并直接返回 
	int mid = tl + tr >> 1; // 继续递归处理 
	if (l <= mid) modifyRange(lson[u], tl, mid, l, r, val);
	if (r > mid) modifyRange(rson[u], mid + 1, tr, l, r, val);
}
```

### 单点 / 区间查询

```cpp
LL query(int u, int tl, int tr, int l, int r) {
// root, tl, tr 代表当前线段树节点信息 l, r 代表查询区间边界 
	if (tl >= l and tr <= r) return sum[u];
	pushDown(u, tl, tr);
	int mid = tl + tr >> 1; LL res = 0;
	if (l <= mid) res += query(lson[u], tl, mid, l, r);
	if (r > mid) res += query(rson[u], mid + 1, tr, l, r);
	return res;
}
```

### 应用：

#### 前缀和 动态开点权值线段树：[寒假作业](https://www.luogu.com.cn/problem/P2717)

##### 简化题意

给定序列和一个整数 $k$, 求区间平均数大于等于 $k$ 的区间个数. 

##### $Solutions$

首先将所有数 $-k$, 问题转化为求 $sum_{l, r} \geqslant 0$ 的区间个数. 再对区间前缀和处理, 转化为求 $s_r - s_{l - 1}$ 的书数对个数. 

以上都是很经典且一般的套路. 

尝试固定一个边界（左）, 查询另一边界（右）. 

当固定了一个边界 $x$, 只需求 $r$ 使得 $sum_r \geqslant sum_{x - 1}$. 即求当前插入的元素中大于等于 $sum_{x - 1}$ 的数的个数. 使用权值线段树 / 离散化后树状数组 维护都是可以的. 

```cpp
for (int i = n; i; -- i) {
	insert(rt, 1, INF << 1, a[i] + INF);
	ans += query(rt, 1, INF << 1, a[i - 1] + INF, INF << 1);
}
```

单次插入与单次查询时间复杂度均为 $O(logn)$, 总时间复杂度 $O(n \cdot logn)$

当然, 以上枚举的是左边界, 换用右边界枚举也是一样的. 

---------------

## 标记永久化

对于动态开点类的线段树上标记的修改, 如果每次下方都将当前节点的两个子节点全部建出, 虽然复杂度仍是对的, 但将常数翻倍了. 这让本就常数大的线段树雪上加霜. 是否有办法不新建节点也能将当前查询到的节点类加上修改标记呢? 

于是标记永久化应运而生. 考虑到查询时的递归路径是一条从线段树根节点到查询区间的路径, 如果将这条路上每一个点的修改标记都进行修改, 那么最终查询到的值就是修改完成的值了. 

---------------

## 单侧递归线段树: [楼房重建](https://www.luogu.com.cn/problem/P4198)

### 简化题意

动态维护整个序列每一个元素大于前一项的必选, 小于等于前一项的必不选, 所的得到的序列长度, $n \le 10^5$. 

### $Solutions$

由于包含区间修改, 因此尝试放在线段树上去做. 重点在于如何将两个子区间的答案合并为父区间答案. 

对于一个固定的序列, 其每个子区间答案是一定的(由于是必选, 不具有决策). 因此合并两个区间时需要找到右区间中首项大于左区间末项的位置, 父区间的答案即为该长度与左区间答案之和. 因此在线段树上维护区间答案和区间首项值即可. 

```cpp
int query(int u, int tl, int tr, double lim) {
	if (mx(u) <= lim) return 0;
	if (tl == tr) return mx(u) > lim;
	int mid = tl + tr >> 1;
	if (mx(lson) <= lim) return query(rson, mid + 1, tr, lim);
	else return query(lson, tl, mid, lim) + len(u) - len(lson);
}

void modify(int u, int tl, int tr, int pos, double val) {
	if (tl == tr) return mx(u) = val, len(u) = 1, void();
	int mid = tl + tr >> 1;
	if (pos <= mid) modify(lson, tl, mid, pos, val);
	else modify(rson, mid + 1, tr, pos, val);
	mx(u) = max(mx(lson), mx(rson));
	len(u) = len(lson) + query(rson, mid + 1, tr, mx(lson));
}
```

单次区间修改至多向右区间递归一次, 不难发现时间复杂度 $O(n \cdot logn)$. 

---------------

## 线段树合并：[雨天的尾巴](https://www.luogu.com.cn/problem/P4556)

### 合并节点 $merge$

复杂度证明：

1. 若当前合并的两颗线段树此处均无节点, 则退出
2. 若当前合并的两颗线段树此处只有一个有节点, 则直接传回节点编号, 并退出
3. 若当前合并的两颗线段树此处均有节点, 则继续递归左右儿子, 并在递归返回后 $pushUp$

不难发现, $1$ 和 $2$ 单次操作复杂度均为 $O(1)$, 而没有子节点的节点数量级是 $O(n \cdot logn)$ 的. 而 $2$, $3$ 操作结束后线段树的总节点个数会 $-1$, 线段树总结点个数最多为 $n \cdot logn$, 所以总复杂度为线段树节点的个数同阶, 复杂度为 $n \cdot logn$. （这里的 $n$ 表示两棵线段树节点重合部分）

函数返回合并后的节点编号. 

```cpp
int merge(int a, int b, int tl, int tr) {
	if (!a or !b) return a | b;
	if (tl == tr) {	sum[a] += sum[b]; return a;	}
	int mid = tl + tr >> 1;
	lson[a] = merge(lson[a], lson[b], tl, mid);
	rson[a] = merge(rson[a], rson[b], mid + 1, tr);
	update(a);
	return a;
}
```

注意：以上写法在合并过程中丢失了 $a$ 节点的信息, 因此需要将所有询问离线. 而在某些特定题目中需要新建节点存储. 

在模板题中, 将路径修改转化为树上差分, 进行单点修改, 最后 $dfs$ 合并每个节点的权值线段树即可. 

### 空间复杂度分析

对于 $N$ 次操作, 若每次操作修改 $x$ 个节点, 每次修改的值域为 $T$, 假设每次都新开点, 则空间复杂度为 $N \cdot x \cdot ( \lceil \log_2 T \rceil + 1 )$. 

### 多写些模板总是好的：[永无乡](https://www.luogu.com.cn/problem/P3224)

并查集维护连通性, 同步合并线段树. 查询第 $k$ 大时线段树上二分即可. 

#### 需要注意的几点：

- 并查集合并方向要和线段树一致, 可以在 $\text{merge}$ 函数传参时使用引用
    ```cpp
    void mergeDsu(int &a, int &b) {
        a = find(a), b = find(b);
        if (a == b) return;
        if (siz[a] < siz[b]) swap(a, b);
        fa[b] = a, siz[a] += siz[b];
        return;
    }
    ```
    这样在合并线段树时仅需
    ```cpp
    mergeDsu(a, b);
    rt[a] = rt[b] = merge(rt[a], rt[b], 1, n);
    ```
    而无需担心按秩合并时交换 $a$ $b$ 造成的影响. 

### 应用

#### 合并时需要新开节点: [[湖南集训] 更为厉害](https://www.luogu.com.cn/problem/P3899)

##### 简化题意

给定一棵树和若干形如 $(a, b)$ 的询问, 询问有多少二元组 $(x, y)$ 满足 $y \in \text{son}(a) \wedge y \in \text{son}(x) \wedge \text{dis}(a, x) \le b$. 

##### 在线: 线段树合并



##### 离线: 二维偏序

对节点 $x$ 的分类讨论同上, 仅需考虑 $x \in \text{son}(a)$ 的情况, 不难发现这种情况的贡献为 $\text{siz}(x) - 1$. 

考虑如何判断 $x$ 是否在 $a$ 子树内. 仅需判断 $\text{dfn}(x) \in [\text{dfn}(a), \text{dfn}(a) + \text{siz}(a) - 1]$ 即可. 

如果将树中每个节点视为坐标为 $(\text{dfn}(u), \text{dep}(u))$, 贡献为 $\text{siz}(u) - 1$ 的点, 则每次询问等价于询问 $([])$

-----------

## 有合并自然有分裂: [线段树分裂](https://www.luogu.com.cn/problem/P5494)

当维护需要拆分的集合时, 可使用线段树分裂维护. 

线段树分裂一般有两种写法: 

### 1. 按数量分裂

分裂后 $x$ 树中包含 $rk$ 个元素,  $y$ 传入新分裂出的子树的根, 因此需要传入引用值, 便于在下一层递归新建节点. 

当递归至一个节点 $x$, 按照 $rk$ 与 $x$ 左子树大小 $cnt(l(x))$ 分类: 

1. $rk > cnt(l(x))$: 这时 $x$ 节点左子树仍属于 $x$, 更新 $rk$ 后递归分裂右子树即可. 
2. $rk = cnt(l(x))$: 这时 $x$ 左子树属于 $x$, 右子树属于 $y$, 交换 $x$ 的右子树与 $y$ 的右子树 (此时为空) 即可. 
3. $rk < cnt(l(x))$: 这时 $x$ 右子树属于 $y$, 递归分裂左子
4. 树. 

最后别忘了更新线段树维护的 $cnt$ 以及其他值, 就成功将 $y$ 从 $x$ 中分离啦. 

```cpp
void split(int x, int &y, LL rk) {
	if (!x) return;
	y = newNode(), tree[y].init();
	if (rk > cnt(l(x))) split(r(x), r(y), rk - cnt(l(x)));
	else swap(r(x), r(y));
	if (rk < cnt(l(x))) split(l(x), l(y), rk);
	cnt(y) = cnt(x) - rk, cnt(x) = rk;
}
```

#### 复杂度分析

不难发现以上操作每次递归最多向一棵子树递归, 因此时间复杂度即为树高 $O(\log n)$. 

### 2. 按权值分裂

将 $x$ 在 $[l, r]$ 部分的元素分裂给 $y$, 基本上同区间修改, 不详细展开. 

```cpp
void split(int &x, int &y, int tl, int tr, int l, int r) {
	if (!x or l > tr or r < tl) return;
	if (l <= tl and tr <= r) return swap(x, y), void();
	int mid = tl + tr >> 1;
	split(l(x), l(y), tl, mid, l, r);
	split(r(x), r(y), mid + 1, tr, l, r);
	cnt(x) = cnt(l(x)) + cnt(r(x)), cnt(y) = cnt(l(y)) + cnt(r(y));
}
```

#### 时间复杂度分析

复杂度分析同线段树的区间修改, 就不再写一遍了. 

------------

## 线段树分治：[二分图](https://www.luogu.com.cn/problem/P5787)

一张图有 $n$ 个节点的图,  在 $k$ 时间中会出现 $m$ 条边, 表示有一条连接 $x,y$ 的边在 $l$ 时刻出现 $r$ 时刻消失, 求问在第 $i$ 个时间段中图是否为二分图. 

```cpp
int n, m, k;
struct Edge { int a, b; }edge[M];
struct Save { int fa, fb; bool add; }st[M]; int top;
vector<int> tree[N << 2]; // SegmentTree 
int fa[N << 1], h[N << 1];

inline int find(int a) {
	while (fa[a] ^ a) a = fa[a];
	return a;
}

void merge(int x, int y) {
	x = find(x), y = find(y);
	if (h[x] > h[y]) swap(x, y); // x to y
	st[++ top] = {x, y, h[x] == h[y]};
	fa[x] = y;
	h[y] += (h[x] == h[y]);
}

void modify(int u, int tl, int tr, int l, int r, int val) {
	if (tl > r or tr < l) return;
	if (tl >= l and tr <= r) return tree[u].push_back(val), void();
	int mid = tl + tr >> 1;
	modify(ls, tl, mid, l, r, val);
	modify(rs, mid + 1 ,tr, l, r, val);
}

void dfs(int u, int tl, int tr) {
	bool f = true; int ptr = top;
	int tmpa, tmpb;
	for (int t : tree[u]) {
		tmpa = find(edge[t].a), tmpb = find(edge[t].b);
		if (tmpa == tmpb) {
			for (int i = tl; i <= tr; ++ i) puts("No");
			f = false;
			break;
		}
		merge(edge[t].a, edge[t].b + n);
		merge(edge[t].a + n, edge[t].b);
	}
	if (f) {
		if (tl == tr) puts("Yes");
		else {
			int mid = tl + tr >> 1;
			dfs(ls, tl, mid), dfs(rs, mid + 1, tr);
		}
	}
	for (; top > ptr; top --) {
		h[fa[st[top].fa]] -= st[top].add;
		fa[st[top].fa] = st[top].fa;
	}
	return;
}

int main() {
	input(n), input(m), input(k);
	for (int i = 1, x, y, l, r; i <= m; ++ i) {
		input(x), input(y), input(l), input(r);
		edge[i] = {x, y};
		modify(1, 1, k, l + 1, r, i);
		// note that max Range is k 
	}
	for (int i = 1; i <= (n << 1); ++ i) fa[i] = i, h[i] = 1; // init dsu
	dfs(1, 1, k);
	return 0;
}
```

-----------

## 李超线段树：[Segment](https://www.luogu.com.cn/problem/P4097)

### 简化题意：

~~已经很简单了, 自己去看~~

维护不同横坐标下若干个一次函数的最大值 / 最小值

### $Solutions$

#### 前置数组与函数

类型名 `D` 定义为 `double` 类型

类型名 `PDI` 定义为 `pair<D, int>` 类型

`SX` 与 `SY` 为横纵坐标值域

`eps` 用于精度比较

`n` 表示线段（一次函数）个数, 由于一条线段最多分成 $logn$ 个线段, 因此总规模为 $n \cdot logn$. 

`a` 存储函数属性（斜率 $k$, 纵截距 $b$. 

`tree` 存储当前线段树节点对应函数位置

```cpp
using D = double;
using PDI = pair<D, int>;
const int N = 1e5 + 10;
const int SX = 39989, SY = 1e9;
const int T = 4e4;
const D eps = 1e-9;
int n, lastans, op, qxa, qya, qxb, qyb;
struct Segment { D k, b; }a[N];
int tree[N + (T << 1)], top;
```

##### 浮点数比较函数

为防止浮点数误差, 自行写函数进行比较

```cpp
int cmp(D x, D y) {
// bigger return 1, smaller return -1, equal return 0
	if (x - y > eps) return 1;
	if (y - x > eps) return -1;
	return 0;
}
```

##### $pair$ 取 $max$

```cpp
PDI pMax(PDI a, PDI b) {
	if (cmp(a.fi, b.fi) == -1) return b;
	if (cmp(a.fi, b.fi) == 1) return a;
	return a.se < b.se ? a : b;
}
```

##### 计算某一横坐标 $x$ 下的函数值

```cpp
D calc(int i, int x) { return a[i].k * x + a[i].b; }
```

#### 添加一次函数

```cpp
void add(int xa, int ya, int xb, int yb) {
	if (xa == xb) a[++ top].k = 0, a[top].b = max(ya, yb);
	else {
		a[++ top].k = (D)(yb - ya) / (xb - xa), 
		a[top].b = ya - a[top].k * xa;	
	}
	return;
}
```

#### 更新最优斜率

```cpp
void pushDown(int rt, int tl, int tr, int u) {
	int &v = tree[rt], mid = tl + tr >> 1;
	if (cmp(calc(u, mid), calc(v, mid)) == 1) swap(u, v);
	int bl = cmp(calc(u, tl), calc(v, tl));
	int br = cmp(calc(u, tr), calc(v, tr));
	if (bl == 1 or (!bl and u < v)) pushDown(lson, tl, mid, u);
	if (br == 1 or (!br and u < v)) pushDown(rson, mid + 1, tr, u);
}
```

#### 将线段分裂, 挂载到树上

```cpp
void cover(int rt, int tl, int tr, int l, int r, int u) {
	if (l <= tl and r >= tr) return pushDown(rt, tl, tr, u);
	int mid = tl + tr >> 1;
	if (l <= mid) cover(lson, tl, mid, l, r, u);
	if (r > mid) cover(rson, mid + 1, tr, l, r, u);
}
```

#### 查询某个横坐标 $x$ 下所有函数最大值

```cpp
PDI query(int rt, int l, int r, int x) {
	if (l > x or r < x) return {0, 0};
	int mid = l + r >> 1;
	D res = calc(tree[rt], x);
	if (l == r) return {res, tree[rt]};
	return pMax({res, tree[rt]}, pMax(query(lson, l, mid, x), query(rson, mid + 1, r, x)));
}
```

另外, 注意本题强制在线. 

### 应用: [李超线段树优化 dp](https://www.luogu.com.cn/problem/P4655)

### 扩展: [李超线段树合并](https://www.luogu.com.cn/problem/CF932F)

#### 简化题意



#### $Solutions$



-----------

## 吉司机线段树 $\rm Segment Beats$: [线段树 3](https://www.luogu.com.cn/problem/P6242)

### 复杂度证明



-----------

## 线段树优化建图：[Legacy](https://www.luogu.com.cn/problem/CF786B)

### 简化题意：

要求实现以下几个操作：
1. 由 $u$ 向 $v$ 连边权为 $w$ 的边
2. 由 $u$ 向区间 $l \sim r$ 连边权为 $w$ 的边
3. 由区间 $l \sim r$ 向 $v$ 连边权为 $w$ 的边

求起始位置 $s$ 到其他所有点的最短距离. 

### $Solutions$

最暴力的想法是依照题意连边, 复杂度 $O(n^2)$

尝试进行优化, 我们知道线段树的一个节点代表一个区间, 因此由一个点向区间连边可视为向线段树上的最多 $n \cdot \log n$ 个节点连边. 由区间向点连边的情况与上述类似. 

线段树上大区间和子区间之间转移也是可以的, 因此需要在线段树节点之间连权值为 $0$ 的边. 同理, 小区间向大区间转移也是可以的, 但显然不能在同一棵线段树之间连双向权值为 $0$ 的边, 因此考虑建两棵线段树, 一棵连由子节点向上的边, 一棵连由父节点向下的边. 同时, 也要在代表同一个区间的叶节点连权值为 $0$ 的边. 

![](https://pic.imgdb.cn/item/64b1138e1ddac507cc55ba2a.jpg)

最后从起始节点跑一遍最短路即可, 时间复杂度 $O(n \cdot \log n)$

记得算好空间 $4 \cdot n + n \cdot \log n$

### 建树与连边 $Code$

代码中两棵树的距离 $GAP$ 为 $4n$

```cpp
void build(int u, int tl, int tr) {
	if (tl == tr) {
		leaf[tl] = u;
		add(u, u + GAP, 0), add(u + GAP, u, 0);
		return;
	}
	add(u, ls, 0), add(u, rs, 0);
	add(ls + GAP, u + GAP, 0), add(rs + GAP, u + GAP, 0);
	int mid = tl + tr >> 1;
	build(ls, tl, mid), build(rs, mid + 1, tr);
}

void connect(int u, int tl, int tr, int l, int r, int v, int w, bool type) {
// type == 0 代表由 v 点向区间 l, r 连边 
// type == 1 代表由区间 l, r 向 v 点连边 
	if (tl >= l and tr <= r) {
		if (type) add(u + GAP, leaf[v], w);
		else add(leaf[v], u, w);
		return;
	}
	int mid = tl + tr >> 1;
	if (r <= mid) connect(ls, tl, mid, l, r, v, w, type);
	else if (l > mid) connect(rs, mid + 1, tr, l, r, v, w, type);
	else connect(ls, tl, mid, l, mid, v, w, type), 
		 connect(rs, mid + 1, tr, mid + 1, r, v, w, type);
}
```



-----------

## 扫描线

当维护矩形面积并时, 也可以尝试使用线段树优化. 

![](https://pic.imgdb.cn/item/63ff17e5f144a01007cc4b7b.jpg)

如图, 建立平面直角坐标系, 如果对于 $n$ 个矩形左右边界作为分割线, 可以将整张图分为 $2n - 1$ 个区间, 左边界赋为 $+1$, 右边界赋为 $-1$, 需要维护面积时, 只需要将当前边上**值为正数的部分乘横坐标之差**即可. 因此我们将问题转化成了**维护区间正值部分**. 

考虑解决问题需要的信息, 在线段树节点上维护 $cnt$, 表示当前区间**整体**被覆盖的次数, $len$ 表示值大于 $0$ 的区间长度. 考虑到仍不是很好维护, 我们尝试从中挖掘一些性质：

1. 区间值必定先加后减, 不存在负值情况. 
2. 查询时只需要根节点信息, $query$ 不需要 $pushDown$. 
3. 所有操作均是成对出现. 

综上可以发现, 在树中进行维护操作时不需要 $pushDown$ 操作, 因此保证了时间复杂度 $n \cdot logn$. 

另外值得注意的是, 线段树中维护的是区间而不是点位, 因此右边界值需要在适当位置 $-1$. 

### $Code$

```cpp
int n;
struct Segment{
    double x, y1, y2;
    int k;
    bool operator < (const Segment t) const { return x < t.x; }
}seg[N << 1];

struct SegmentTree{
    int l, r, cnt;
    double len;
    #define l(x) tree[x].l
    #define r(x) tree[x].r
    #define cnt(x) tree[x].cnt
    #define len(x) tree[x].len
}tree[N << 3];

vector<double> lis;

int find(double y) {
    return lower_bound(lis.begin(), lis.end(), y) - lis.begin();
}

void PushUp(int u) {
    if (cnt(u)) len(u) = lis[r(u) + 1] - lis[l(u)];
    else if (l(u) != r(u)) len(u) = len(lson) + len(rson);
    else len(u) = 0;
}

void build(int u, int l, int r) {
    tree[u] = {l, r, 0, 0};
    if (l != r) {
        int mid = l + r >> 1;
        build(lson, l, mid), build(rson, mid + 1, r);
    }
}

void modify(int u, int l, int r, int k) {
    if (l(u) > r or r(u) < l) return;
    if (l(u) >= l and r(u) <= r) {
        cnt(u) += k;
        PushUp(u);
        return;
    }
    modify(lson, l, r, k), modify(rson, l, r, k);
    PushUp(u);
}

int main() {
	int n; cin >> n;
	for (int i = 0, j = 0; i < n; i ++) {
		double x1, x2, y1, y2;
		cin >> x1 >> y1 >> x2 >> y2;
		seg[j ++] = { x1, y1, y2, 1 };
		seg[j ++] = { x2, y1, y2, -1 };
		lis.push_back(y1), lis.push_back(y2);
	}
	sort(lis.begin(), lis.end());
	lis.erase(unique(lis.begin(), lis.end()), lis.end()); // 离散化
	sort(seg, seg + (n << 1));
	build(1, 0, lis.size() - 2); // 建树
	double ans = 0;
	for (int i = 0; i < n << 1; i ++) {
		if (i > 0) ans += len(1) * (seg[i].x - seg[i - 1].x);
		modify(1, find(seg[i].y1), find(seg[i].y2) - 1, seg[i].k);
	}
	printf("%d\n", ans);
    return 0;
}
```