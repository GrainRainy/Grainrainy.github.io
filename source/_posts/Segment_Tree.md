---
title: 线段树 | 学习笔记
date: 2023-3-4 18:30:22
categories: 数据结构
tags: 数据结构
author: GrainRain
cover: https://pic.imgdb.cn/item/648307cf1ddac507ccf3f688.jpg
single_column: true
---


# 线段树 $Segment\ Tree$

线段树是一种数据结构, 可维护区间信息, 与树状数组不同的是, 线段树可以维护更多信息, 因而功能也更强大, 但缺点是递归操作和建树的常数较大

线段树的主要思想是将整个区间分成若干小段, 以二倍关系向上合成区间（实际代码实现时使用递归从上向下建树）, 将数组信息构建出树形结构, 通过**子结点向父节点传递信息**, 从而达到单次查询 $log$ 级别的复杂度

特别地, 我们规定左子结点的下标为 `cur << 1`, 右子结点的下标为 `cur << 1 | 1`, 从而相对稠密地存储, 因此对于一个长度为 $n$ 的区间, 需要**开 $4n$ 大小的数组存储**线段树. 在动态开点线段树中, 抛弃了这一规则, 改为与插入操作同步新建节点, 在线段树节点中记录左右子结点下标, 进一步优化了空间复杂度, 如需直接学习动态开点线段树, 请跳至后文. 

## 模板

### 存储方式（以维护区间和为例）

```cpp
#define lson (cur << 1)
#define rson (cur << 1 | 1)

int n, m;
int nums[N];

struct SegmentTree{
	int l, r, val;
	int tag;
	
	#define l(x) (tree[(x)].l)
	#define r(x) (tree[(x)].r)
	#define val(x) (tree[(x)].val)
	#define tag(x) (tree[(x)].tag)
}tree[N << 2]; // 4 倍空间
```

### 递归建树 $Build$

```cpp
void build(int cur, int l, int r) {
// 当前节点编号, 区间左端点, 区间右端点
	l(cur) = l, r(cur) = r;
	if (l == r) return val(cur) = nums[l], void();
	
	int mid = l + r >> 1;
	build(lson, l, mid), build(rson, mid + 1, r);
	PushUp(cur);
}
```

```cpp
build(1, 1, n);
```

### 向父节点上传信息 $PushUp$

```cpp
void PushUp(int cur) {
	val(cur) = val(lson) + val(rson);
}
```

### 单点修改 $Modify$

```cpp
void modify(int cur, int x, int v) {
// 将位置为 x 的点的值修改为 v
	if (l(cur) > r or r(cur) < l) return;
	if (l(cur) == x and r(cur) == x) 
		return val(cur) = v, void();

	modify(lson, x, v), modify(rson, x, v);
	PushUp(cur);
}
```

对于单点修改, 每次修改操作最多遍历 $log_2n$ 个节点, 而如果对区间中每一个点都进行单点修改, 单次修改时间复杂度会退化到 $O(n \cdot logn)$. 

类比于区间查询, 如果查询区间包含整个线段树区间, 我们可以在更新信息后直接返回, 因此时间复杂度仍会控制到 $log_2n$ 级别, 常数控制在 $4$ 左右. 

我们将存储该信息的操作称为**懒标记**. 当遍历到一个区间时, 对被完全包含的区间暂时不进行修改, 而将需要修改的信息暂时赋给懒标记, **需要向下查询时再下传一层懒标记**. 显然, 为保证当前访问节点信息的正确性, 在一个区间被遍历到时, 它和它祖先的懒标记需要被下放完成. 

值得注意的是, 懒标记一般不包含当前节点信息. 

### 下传懒标记 $PushDown$

利用懒标记, 线段树可以支持区间修改与区间查询的操作, 下面是下放懒标记的 $PushDown$ 操作, 需要在 $query$ 和 $modify$ 操作时分别调用

```cpp
void PushDown(int cur) {
	tag(lson) += tag(cur);
	tag(rson) += tag(cur);
	// 更新子节点延迟标记信息 tag 
	val(lson) += (r(lson) - l(lson) + 1) * tag(cur);
	val(rson) += (r(rson) - l(rson) + 1) * tag(cur);
	// 更新子节点 value
	tag(cur) = 0;
}
```

### 区间修改

```cpp
void modify(int cur, int l, int r, int v) {
	if (l(cur) > r or r(cur) < l) return;
	if (l <= l(cur) and r >= r(cur)){
		tag(cur) += v;
		val(cur) += (r(cur) - l(cur) + 1) * v;
		return;
	}
	// 否则当前区间与赋值区间有交集, 需要先下传懒标记
	PushDown(cur);
	modify(lson, l, r, v), modify(rson, l, r, v);
	PushUp(cur);
}
```

### 查询

![](https://pic.imgdb.cn/item/63fef113f144a010078e4dbd.jpg)

查询时间复杂度 $n \cdot log_2n$, 常数大概为4, 与之相比树状数组常数较小, 稳定为 $1$

#### 查询时间复杂度证明

我们尝试分类讨论

![](https://pic.imgdb.cn/item/63fef41cf144a01007944626.jpg)

不难发现, 对于任意一种情况都只会递归一次, 或在第二层递归终止

```cpp
int query(int cur, int l, int r) {
	if (l(cur) > r or r(cur) < l) return 0;
	if (l(cur) >= l and r(cur) <= r) return val(cur);
	PushDown(cur); // 下传懒标记
	return query(lson, l, r) + query(rson, l, r);
}
```

## $Other\ Tricks$

区间加单点查询可以在线段树中存储差分数组, 转化为单点修改区间查询, 无需使用懒标记每次询问单点 $x$ 查询 $[1, x]$ 即可

-----------

## 例题

### [区间最大公约数](https://www.acwing.com/activity/content/problem/content/1609/)

核心公式：

$$gcd(a, b, c) = gcd(a, b - a, c - b)$$

因此线段树维护原序列差分序列即可

### [区间乘 区间加](https://www.luogu.com.cn/problem/P3373)

一个的模板, 下放懒标记时先下放乘法标记再下放加法标记即可 

### [区间求和 区间异或](https://codeforces.com/problemset/problem/242/E)

考虑对每一位二进制分别处理, 维护 $20$ 棵线段树, 时间复杂度 $O(n \cdot logn \cdot logs)$（s为值域）

#### $Solutions$ 

```cpp
int tl[N << 2], tr[N << 2], tag[N << 2];
int sum[N << 2][LOGN + 10];
// Segment Tree 

void PushUp(int u) {
    for (int i = 0; i < LOGN; ++ i) 
        sum[u][i] = sum[lson][i] + sum[rson][i];
    return;
}

void PushDown(int u) {
    if (!tag[u]) return;
    for (int i = 0; i < LOGN; ++ i) {
        if ((tag[u] >> i) & 1) {
            sum[lson][i] = (tr[lson] - tl[lson] + 1) - sum[lson][i];
            sum[rson][i] = (tr[rson] - tl[rson] + 1) - sum[rson][i];
        }
    }
    tag[lson] ^= tag[u], tag[rson] ^= tag[u];
    tag[u] = 0;
    return;
}

void build(int u, int l, int r) {
    tl[u] = l, tr[u] = r;
    if (l == r) {
        for (int i = 0; i < LOGN; ++ i) 
            sum[u][i] = (a[l] >> i) & 1;
        return;
    }
    int mid = l + r >> 1;
    build(lson, l, mid), build(rson, mid + 1, r);
    PushUp(u);
}

void modify(int u, int l, int r, int val) {
    if (l > tr[u] or r < tl[u]) return;
    if (l <= tl[u] and r >= tr[u]) {
        for (int i = 0; i < LOGN; ++ i) 
            if ((val >> i) & 1) 
                sum[u][i] = (tr[u] - tl[u] + 1) - sum[u][i];
        tag[u] ^= val;
        return;
    }
    PushDown(u);
    modify(lson, l, r, val), modify(rson, l, r, val);
    PushUp(u);
}

LL query(int u, int l, int r) {
    if (l > tr[u] or r < tl[u]) return 0;
    if (l <= tl[u] and r >= tr[u]) {
        LL res = 0, p = 1;
        for (int i = 0; i < LOGN; ++ i) {
            res += p * sum[u][i];
            p <<= 1;
        }
        return res;
    }
    PushDown(u);
    return query(lson, l, r) + query(rson, l, r);
}
```

### [单点修改 区间异或和](https://www.luogu.com.cn/problem/P6098)

异或运算满足交换律和结合律, 因此直接维护就行了, 而且无需维护懒标记, 甚至更简单. 

~~虚晃一枪, 没想到吧~~

### [动态查询空区间数量](https://www.luogu.com.cn/problem/P4215)

题意：维护序列. 给定一些区间, 每次修改单点值（只会减少, 且保证为正整数）, 在每次修改后输出空区间数量. 强制在线. 

#### $Solutions$

把询问的区间挂到线段树区间上 (要求必须完全覆盖), 线段树区间开 `vector` 存储询问区间 $id$. 对每个询问区间维护 $cnt$ 表示被分成了多少个小区间, 当 $cnt$ 减至 $0$ 时 $ans ++$ 即可. 

### [区间开根 区间加 区间和](https://uoj.ac/problem/228)

事实上这题并不难, 只不过复杂度需要进行势能分析. 

记线段树区间最小值为 $mn(u)$, 区间最大值为 $mx(u)$, 不难发现当 $mn(u) - \sqrt{mn(u)} = mx(u) - \sqrt{mx(u)}$ 时, 区间开根操作可转化为区间修改, 因此使用线段树维护区间和 $sum$, 区间最小值 $mn$, 区间最大值 $mx$ 和懒标记即可. 

#### 时间复杂度分析

难点在于时间复杂度分析

其他操作时间复杂度显然为 $O(n \cdot \log n)$, 考虑区间开根的复杂度即可. 

设值域为 $T$, 考虑最坏情况, $a_i = 2^{\lfloor \log T \rfloor}$, 此时对 $a_i$ 开根, 指数每次最少减少一半, 因此至多 $\log \log (mx - mn)$ 次区间开根之后可以转化为区间修改. 

由此, 设线段树节点势能为 $\log \log (mx - mn)$, 那么每次区间开根操作即将 $l \sim r$ 区间内势能非 $0$ 的节点的势能减小 $1$, 每次修改对于单个线段树节点势能最多恢复到  $\log \log (mx - mn)$, 总时间复杂度即为势能总量  $O((n + m \cdot \log n) \cdot \log \log T)$