---
title: 线段树 | 学习笔记
date: 2023-3-4 18:30:22
categories: 数据结构
tags: 数据结构
author: GrainRain
cover: https://pic.imgdb.cn/item/64036deaf144a0100735d9b7.jpg
---

# 线段树 $Segment\ Tree$

线段树是一种数据结构，可维护区间信息，与树状数组不同的是，线段树可以维护更多信息，因而功能也更强大，但缺点是递归操作和建树的常数较大

线段树的主要思想是将整个区间分成若干小段，以二倍关系向上合成区间（实际代码实现时使用递归从上向下建树），将数组信息构建出树形结构，通过**子结点向父节点传递信息**，从而达到 $log$ 级别的复杂度

特别地，我们规定左子结点的下标为 `idx << 1`，右子结点的下标为 `idx << 1 | 1`，从而相对稠密地存储，因此对于一个长度为 $n$ 的区间，需要**开 $4n$ 大小的数组存储**线段树。在动态开点线段树中，抛弃了这一规则，改为与插入操作同步新建节点，在线段树节点中记录左右子结点下标，进一步优化了空间复杂度

## 模板

### 存储方式（以维护区间和为例）

```cpp
#define sonl (idx << 1)
#define sonr (idx << 1 | 1)

int n, m;
int nums[N];

struct SegmentTree
{
	int l, r, val;
	int lazy; // 延迟标记 
	
	#define l(x) (tree[(x)].l)
	#define r(x) (tree[(x)].r)
	#define val(x) (tree[(x)].val)
	#define lazy(x) (tree[(x)].lazy)
}tree[N << 2]; // 4 倍空间
```

### 递归建树

```cpp
void build(int idx, int l, int r)
// 当前节点编号，区间左端点，区间右端点
{
	l(idx) = l, r(idx) = r;
	if (l == r) return val(idx) = nums[l], void();
	
	int mid = l + r >> 1;
	build(sonl, l, mid), build(sonr, mid + 1, r);
	pushup(idx);
}

/*调用时使用*/

build(1, 1, n);
```

### 向父节点上传信息

```cpp
void pushup(int idx)
{
	val(idx) = val(sonl) + val(sonr);
}
```

### 单点修改（无需懒标记）

```cpp
void modify(int idx, int x, int v)
// 将位置为 x 的点的值修改为 v
{
	if (l(idx) > r || r(idx) < l) return;
	if (l(idx) == x and r(idx) == x) 
		return val(idx) = v, void();

	modify(sonl, x, v), modify(sonr, x, v);
	pushup(idx);
}
```

对于单点修改，每次修改操作最多遍历 $log_2n$ 个节点，而如果对区间中每一个点都进行单点修改，修改时间复杂度会退化到 $O(n)$。

类比于区间查询，如果查询区间包含整个线段树区间，我们可以在更新信息后直接返回，因此时间复杂度仍会控制到 $log_2n$ 级别，常数控制在 $4$ 左右。

我们将存储该信息的操作称为**懒标记**。当遍历到一个区间时，对被完全包含的区间暂时不进行修改，而将需要修改的信息暂时赋给懒标记，**需要向下查询时再下传一层懒标记**。显然，为保证当前访问节点信息的正确性，在一个区间被遍历到时，它和它祖先的懒标记需要被下放完成。

值得注意的是，懒标记一般不包含当前节点信息。

利用懒标记，线段树可以支持区间修改与区间查询的操作，下面是下放懒标记的 $pushdown$ 操作，需要在 $query$ 和 $modify$ 操作时分别调用

```cpp
void pushdown(int idx)
{
	lazy(sonl) += lazy(idx);
	lazy(sonr) += lazy(idx);
	// 更新子节点延迟标记信息 lazy 
	val(sonl) += (r(sonl) - l(sonl) + 1) * lazy(idx);
	val(sonr) += (r(sonr) - l(sonr) + 1) * lazy(idx);
	// 更新子节点 value
	lazy(idx) = 0;
}
```

### 区间修改

```cpp
void modify(int idx, int l, int r, int v)
{
	if (l(idx) > r || r(idx) < l) return;
	if (l <= l(idx) && r >= r(idx))
	{
		lazy(idx) += v;
		val(idx) += (r(idx) - l(idx) + 1) * v;
		return;
	}
	// 否则当前区间与赋值区间有交集, 需要先下传懒标记
	pushdown(idx);
	modify(sonl, l, r, v), modify(sonr, l, r, v);
	pushup(idx);
}
```

### 查询

![](https://pic.imgdb.cn/item/63fef113f144a010078e4dbd.jpg)

查询时间复杂度 $n·log_2n$，常数大概为4，与之相比树状数组常数较小，稳定为 $1$

#### 查询时间复杂度证明

我们尝试分类讨论

![](https://pic.imgdb.cn/item/63fef41cf144a01007944626.jpg)

不难发现，对于任意一种情况都只会递归一次，或在第二层递归终止

```cpp
int query(int idx, int l, int r)
{
	if (l(idx) > r || r(idx) < l) return 0;
	if (l(idx) >= l && r(idx) <= r) return val(idx);
	pushdown(idx); // 下传懒标记
	return query(sonl, l, r) + query(sonr, l, r);
}
```


## $Tricks$

区间加单点查询可以在线段树中存储差分数组，转化为单点修改区间查询，无需使用懒标记每次询问单点x查询[1, x] 即可

## 例题

### 1. [Acwing246 - 区间最大公约数](https://www.acwing.com/activity/content/problem/content/1609/)

核心公式

gcd(a, b, c) = gcd(a, b - a, c - b)

因此线段树维护原序列差分序列即可

### 2. [洛谷p3373 - 线段树2](https://www.luogu.com.cn/problem/P3373)

一个同时维护区间乘和区间加的模板，下放懒标记时先下放乘法标记再下放加法标记即可 

## 线段树扩展 - 扫描线

当维护矩形面积的并时，也可以尝试使用线段树优化

![](https://pic.imgdb.cn/item/63ff17e5f144a01007cc4b7b.jpg)

如图，建立平面直角坐标系，如果对于 $n$ 个矩形左右边界作为分割线，可以将整张图分为 $2n - 1$ 个区间，左边界赋为 `+1`，右边界赋为 `-1`，需要维护面积时，只需要将当前边上**值为正数的部分乘横坐标之差**即可。因此我们将问题转化成了**维护区间正值部分**。

考虑解决问题需要的信息，在线段树节点上维护 $cnt$，表示当前区间**整体**被覆盖的次数， $len$ 表示值大于 $0$ 的区间长度。考虑到仍不是很好维护，我们尝试从中挖掘一些性质：

1. 区间值必定先加后减，不存在负值情况
2. 查询时只需要根节点信息，$query$ 不需要 $pushdown$
3. 所有操作均是成对出现

综上可以发现，在树中进行维护操作时不需要 $pushdown$ 操作，因此保证了时间复杂度 $n·logn$

另外值得注意的是，线段树中维护的是区间而不是点位，因此右边界值需要在适当位置 `- 1`

### 模板

```cpp
#define sonl (idx << 1)
#define sonr (idx << 1 | 1)
using LL = long long;

int n;
struct Segment
{
    double x, y1, y2;
    int k;

    bool operator < (const Segment t) const
    {
        return x < t.x;
    }
}seg[N << 1];

struct SegmentTree
{
    int l, r;
    int cnt;
    double len;

    #define l(x) tree[x].l
    #define r(x) tree[x].r
    #define cnt(x) tree[x].cnt
    #define len(x) tree[x].len
}tree[N * 8];

vector<double> lis;

int find(double y)
{
    return lower_bound(lis.begin(), lis.end(), y) - lis.begin();
}

void pushup(int idx)
{
    if (cnt(idx)) len(idx) = lis[r(idx) + 1] - lis[l(idx)];
    else if (l(idx) != r(idx)) len(idx) = len(sonl) + len(sonr);
    else len(idx) = 0;
}

void build(int idx, int l, int r)
{
    tree[idx] = {l, r, 0, 0};
    if (l != r)
    {
        int mid = l + r >> 1;
        build(sonl, l, mid), build(sonr, mid + 1, r);
    }
}

void modify(int idx, int l, int r, int k)
{
    if (l(idx) > r or r(idx) < l) return;
    if (l(idx) >= l and r(idx) <= r)
    {
        cnt(idx) += k;
        pushup(idx);
        return;
    }
    modify(sonl, l, r, k), modify(sonr, l, r, k);
    pushup(idx);
}

int main()
{
    int t; input(t);
    while (t --)
    {
		int n; input(n);
        for (int i = 0, j = 0; i < n; i ++)
        {
            double x1, x2, y1, y2;
            cin >> x1 >> y1 >> x2 >> y2;
            seg[j ++] = {x1, y1, y2, 1};
            seg[j ++] = {x2, y1, y2, -1};
            lis.push_back(y1), lis.push_back(y2);
        }

        sort(lis.begin(), lis.end());
        lis.erase(unique(lis.begin(), lis.end()), lis.end());
		// 离散化

        sort(seg, seg + 2 * n);

        build(1, 0, lis.size() - 2);
		// 建树

        double ans = 0;
        for (int i = 0; i < 2 * n; i ++)
        {
            if (i > 0) ans += len(1) * (seg[i].x - seg[i - 1].x);
            modify(1, find(seg[i].y1), find(seg[i].y2) - 1, seg[i].k);
        }

        cout << ans << endl;
    }
    return 0;
}
```