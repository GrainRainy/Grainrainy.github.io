---
title: 可持久化线段树 | 学习笔记
date: 2023-3-4 18:45:41
categories: 数据结构
tags: 数据结构
author: GrainRain
cover: https://pic.imgdb.cn/item/6483059e1ddac507ccf14f08.jpg
single_column: true
---


# 可持久化线段树 (主席树)

一个非常上流的英文名：$Persistable\ Segment\ Tree$

$Persistable$ 意为可读写的，动态的，持久的

顾名思义，基于线段树维护区间的基本功能，可持久化线段树保存了每一个版本的信息，以便于对任意一个版本的信息进行修改

因此我们需要在线段树的基础之上多开节点来存储版本信息。然而如果对于每一个版本都存储一棵完整的树形结构，空间复杂度为 $O(m * 4n)$。尝试思考：是否有更优的存储版本信息方式呢？

注意到区间修改的操作，每一次最多改变 $log_2n$ 个节点，然而大部分节点的信息并没有发生改变，我们尝试仍利用这些节点的信息，在线段树外新建节点存储变化信息，为变化的信息仍指向原树

## 空间复杂度

对于 $m$ 次修改，单次单点修改最多改变 $log_2 n$ 个节点，因此可持久化线段树空间复杂度为 $m·log_2 n$

可持久化线段树一般应用于维护区间问题，当维护 $1e9$ 值域而数据量比较小时，尝试用离散化的技巧存储以优化空间复杂度：

当读入数据时，将数据存入 $vector$ 去重

```cpp
int a[N];
vector<int> nums;

for (int i = 1; i <= n; i ++) {
    cin >> a[i];
    nums.push_back(a[i]);
}

sort(nums.begin(), nums.end());
nums.erase(unique(nums.begin(), nums.end()), nums.end());
```

利用 $stl$ 查找离散化值

```cpp
int find(int x) {
    return lower_bound(nums.begin(), nums.end()) - nums.begin();
}
```

## 应用场景

由于 $k$ 个版本信息会产生 $k$ 个根节点，因此节点左右子节点无法满足 `idx * 2` 和 `idx * 2 + 1` 的性质，因此我们需要在节点信息中存储左右儿子信息而非存储左右边界，并在查询或修改函数中传入左右边界信息 

另外，由于可持久化线段树会有多个节点表示同一区间，因此难以处理懒标记，也就难以进行区间修改的操作


### 1. [Acwing255 - 第k小数](https://www.acwing.com/problem/content/257/)

静态区间第 $k$ 小解法：
1. 归并树 $O(n·log^3n)$
2. 划分树 时间复杂度 $O(n·logn)$，空间复杂度 $O(n·logn)$
3. 树套树 线段树套平衡树 支持修改 $O(n·log^2n)$ 空间复杂度 $O(n·logn)$
4. 可持久化线段树 时间复杂度 $O(n·logn)$，空间复杂度 $O(n·logn)$

可持久化线段树 $Solution$：

1. 离散化，减小维护值域
2. 类似于权值线段树，该题在数值上建立线段树，维护每个数值区间中数的个数
3. 线段树上二分

每次递归一边，logn层，最多访问logn个节点，时间复杂度logn

[1, r] 找到插入 r 时的版本
[l, r] 插入 r 时的版本减去插入 l 时的版本

插入操作，同时新建节点存储新版本信息

```cpp
int insert(int cur, int l, int r, int k) {
    int to = ++ idx;
    tree[to] = tree[cur];
    if (l == r) cnt(to) ++;
    else {
        int mid = l + r >> 1;
        if (k <= mid) idxl(to) = insert(idxl(cur), l, mid, k);
        else idxr(to) = insert(idxr(cur), mid + 1, r, k);
        cnt(to) = cnt(idxl(to)) + cnt(idxr(to));
    }
    return to;
}
```

查询版本信息，为访问到原树中未修改的信息，需要同时遍历原树节点和新节点

```cpp
int query(int to, int cur, int l, int r, int k){
    if (l == r) return r;
    int cnt = cnt(idxl(to)) - cnt(idxl(cur));
    int mid = l + r >> 1;
    if (k <= cnt) return query(idxl(to), idxl(cur), l, mid, k);
    else return query(idxr(to), idxr(cur), mid + 1, r, k - cnt);
}
```

与线段树不同的是，可持久化线段树不需要建树，而是随着插入操作动态开点并维护节点关系

在主函数中，我们需要将原数组离散化后去重，并插入可持久化线段树中

```cpp
signed main() {
	cin >> n >> m;
	for (int i = 1; i <= n; i ++) { 
		cin >> a[i];
		nums.push_back(a[i]); // start from 0
	}
	
	sort(nums.begin(), nums.end());
	nums.erase(unique(nums.begin(), nums.end()), nums.end());
	
	for (int i = 1; i <= n; i ++)
		root[i] = insert(root[i - 1], 0, nums.size() - 1, find(a[i]));
	
	while (m --) {
		int l, r, k;
		cin >> l >> r >> k;
		cout << nums[query(root[r], root[l - 1], 0, nums.size() - 1, k)] << endl;
        // 注意需要输出离散化的原始值
	}
	return 0;
}
```