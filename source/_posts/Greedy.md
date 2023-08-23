---
title: 贪心 | 学习笔记
date: 2022-12-21 09:04:34
categories: 基础算法
tags: 学习笔记
author: GrainRain
cover: https://pic.imgdb.cn/item/6480a72f1ddac507cc6d4f37.jpg
single_column: true
---


# 贪心

> 从局部最优推向全局最优

## 基本思想

始终做出当前最优选择，从而期望全局达到最优。值得注意的是，如果当前最优选择无法保证推出全局最优解，我们可以更换策略如**动态规划**

## 贪心证明方式

> 贪心不证明，亲人两行泪

1. 反证法: 对于当前的贪心策略，否定当前的选择来证明是否能达到最优解
2. 构造法: 将问题构造成已知的算法或数据结构从而证明贪心策略正确
3. 数学推导

### 反证法的应用实例：[田忌赛马](https://www.luogu.com.cn/problem/P1650)

贪心策略：

1. 如果田忌目前的最快🐎快于齐王的最快马，则两者比
2. 如果田忌的最快马慢于齐王的最快马，则用田忌的最慢马与齐王的最快马比
3. 如果两者的最快马速度相等，则：
    1. 若田忌的最慢马快于齐王的最慢马，则两个最慢马比
    2. 若田忌的最慢马慢于齐王的最慢马或者两者相等，则用田忌的最慢马与齐王的最快马比

#### $Solutions$

```cpp
int hs1[N], hs2[N];
int n, mny;

cin >> n;
for (int i = 1; i <= n; ++ i)
    cin >> hs1[i];
for (int i = 1; i <= n; ++ i)
    cin >> hs2[i];

sort (hs1 + 1, hs1 + n + 1, greater<int>());
sort (hs2 + 1, hs2 + n + 1, greater<int>());

int l1 = 1, r1 = n, l2 = 1, r2 = n;
// 1 refers to TianJi, 2 refers to the King

while (l1 <= r1 && l2 <= r2)
    if (hs1[l1] > hs2[l2]) l1 ++, l2 ++, mny ++;
    else if (hs1[l1] < hs2[l2]) r1 --, l2 ++, mny --;
    else
        if (hs1[r1] > hs2[r2]) r1 --, r2 --, mny ++;
        else {
            if (hs1[r1] < hs2[l2]) mny --;
            r1 --, l2 ++;
        }

cout << mny * 200 << endl;
```

## 经典套路

### 工序安排问题


#### 双工序 第二工序需等待：[工序安排](https://www.luogu.com.cn/problem/P2751)



#### 双工序 第二工序无需等待：[午餐](https://www.luogu.com.cn/problem/P2577)

仅需一步贪心转化：第二工序时间长的先进行第一工序

剩下的是 $dp$ ~~虚晃一枪，没想到吧~~

### 区间问题

#### 选择不相交的区间

> 数轴上有 $n$ 个区间，每条线段都有起点和终点，选择最多的不相交的线段个数。

最优的贪心策略为：

对右端点进行排序，一次选择左端点大于前一个已经选择的右端点的区间

#### 区间选点 完全覆盖：[雷达安装](https://www.luogu.com.cn/problem/P1325)

> 数轴上有 $n$ 个闭区间 $[a, b]$ 。取尽可能少的点，使得每个区间内都至少有一个点 （不同区间内含的点可以是同一个）。

最优的贪心策略为：

右端点升序排列，右端点相等的左端点降序，当有区间的左端点大于当前右端点时，则增加一个新的点

##### $Solutions$

```cpp
struct range {
	double l, r;
	
	bool operator < (const range &t) const {
		if (r != t.r) return r < t.r;
		else return l > t.l;
	}
}nums[N];

int ans;
int n, d;

cin >> n >> d;
int x, y;
for (int i = 1; i <= n; ++ i) {
	cin >> x >> y;
	if (y <= d) {
		nums[i].l = x - sqrt(d * d - y * y);
		nums[i].r = x + sqrt(d * d - y * y);
	}
	else {
		puts("-1");
		return 0;
	}
}

sort(nums + 1, nums + n + 1);
// input

int tmpr = nums[1].r;
ans = 1;
for (int i = 2; i <= n; ++ i)
	if (nums[i].l > tmpr)
		tmpr = nums[i].r, ans ++;

cout << ans << endl;
```


#### 选择区间 完全覆盖：[区间覆盖（加强版）](https://www.luogu.com.cn/problem/P2082)

> 数轴上有n个闭区间 `[a, b]` ，选择尽量少的区间覆盖一条指定线段 `[s,t]` 。

##### $Soluitons$

```cpp
struct ababaab {
	LL l, r;
	
	bool operator < (const ababaab &t) const {
		return l < t.l;
	}
}nums[N];

LL len, n;
	
cin >> n;
for (LL i = 1; i <= n; ++ i)
	cin >> nums[i].l >> nums[i].r;

sort(nums + 1, nums + n + 1);

LL tmpl = nums[1].l, tmpr = nums[1].r;
len = tmpr - tmpl + 1;
for (LL i = 2; i <= n; ++ i) {
	if (nums[i].l <= tmpr && nums[i].r > tmpr) {
		len += nums[i].r - tmpr;
		tmpr = nums[i].r;
	}
	else if (nums[i].l > tmpr) {
		tmpl = nums[i].l, tmpr = nums[i].r;
		len += tmpr - tmpl + 1;
	}
}
cout << len << endl;
```

#### 区间并集：[区间](https://www.luogu.com.cn/problem/P2434)

给定 $n$ 个闭区间 $[a, b]$ ，求能够覆盖的长度。

##### 贪心策略

1. 按区间左端点排序
2. 动态维护某一区间 $x$，考虑对下一个区间 $y$ 分类讨论：
   1. $l_x \leqslant l_y$ 且 $r_x \geqslant r_y$
   2. $l_x \leqslant l_y$ 且 $r_x < r_y$
   3. $r_x < l_y$

对于第一种情况：维持原来的起始点 $st$ 和结束点 $ed$ 即可
对于第二种情况：将尾节点更新成 $2$ 区间的尾节点
对于第三种情况：将维护的此区间存入答案，将 $3$ 区间作为新的动态维护空间

```cpp
using PII = pair<int, int>;
int n;
vector<PII> segs; // 表示所有区间 

void merge (vector<PII> &segs) {
	vector<PII> ans; // 用于存储合并区间后的结果 
	sort(segs.begin(), segs.end());
	// sort 对 pair 进行排序时，先根据first关键字排序，再根据second关键字排序
	int st = -2e9, ed = -2e9;
	// 边界值，依据题目数据范围而定
	for (auto seg : segs) {
		if (ed < seg.first) {
            if (st != -2e9) ans.push_back({st, ed});
			st = seg.first, ed = seg.second;
        }
		else ed = max(ed, seg.second);
	}
	// merge  
	if (st != -2e9) ans.push_back({st, ed}); // 判空 
	segs = ans; // 更新区间 
}

int main() {
	cin >> n;
	for (int i = 0; i < n; ++ i) {
		int l, r;
		cin >> l >> r;
		segs.push_back({l, r});
	}
	merge(segs);
	cout << segs.size() << endl;
	return 0; 
}
```