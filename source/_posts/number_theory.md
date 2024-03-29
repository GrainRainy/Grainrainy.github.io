---
title: 数论总述 | 一文从入门到毕业
date: 2023-4-27 23:47:52
categories: 数学
tags: 数学
author: GrainRain
cover: https://pic.imgdb.cn/item/64b0162b1ddac507cc913e29.jpg
single_column: true
---


# 数论总述

包含质数, 约数, 同余, 乘法逆元, 同余方程, 积性函数与数论中的重要函数. 

## 质数

### 千奇百怪的质数定理

- $1\sim n$ 中共有 $\frac{n}{\ln n}$ 个质数. 

### 判质数

#### 1. 试除法 $O(\sqrt n)$

```cpp
bool isPrime(int u){
	if (u < 2) return false;
	for (int i = 2; i <= n / i; i ++)
		if (n % i == 0) return true;
	return false;
}
```

#### 2. 埃氏筛 $O(n \cdot \log_2 \log_2 n)$

基本思想是筛去质数的倍数. 

自己思考下就会发现埃氏筛很对, 若一个数是合数, 其必然含有小于它本身的因子, 同样地, 若一个数是质数, 其因子仅有 $1$ 和它本身. 当筛法指针指向 $i$ 时, 若它没被筛到, 则必然为质数, 反之亦然. 

由于在筛 $i$ 时, $2 \sim i - 1$ 都已被筛过, 因此代码中 $1$ 处从 $i \cdot i$ 开始筛即可. 

时间复杂度 $O(n \cdot \log_2 \log_2 n)$

```cpp
vector<int> prime;
// Stacks for storing prime numbers
void Eratosthenes(int n) {
	np[0] = np[1] = true;
	for (int i = 2; i <= n; ++i) {
		if (!np[i]) {
			prime.push_back(i);
			for (int j = i * i; j <= n; j += i) // 1 
				np[j] = true;
		}
	}
	return;
}
```

#### 3. 线性筛 / 欧拉筛 $O(n)$

尝试优化埃式筛法, 发现在埃氏筛中一个合数会被多次标记, 可以尝试从此处进行优化, 诞生了线性筛的基本思想：一个合数 $n$ 只会被最小质因子筛出. 

我们发现, 埃氏筛的实现方式是对于每个质数依靠遍历自然数的方式来标记自身倍数. 线性筛则在遍历自然数的同时, 对于每个自然数遍乘质数表中的所有数.  

标记非素数的过程相当于对于每一个素数之上建立一个剩余系, 随着 $i$ 的推进, 通过 $\times i$ 逐步扩大剩余系规模. 当 `i % prm[j] == 0` 则表示如果继续向下遍乘质数, 那么当前数不为下一个筛到的数的最小质因子, 直接退出即可. 

例如：如果 $i$ 是 $4$, 第一个质数是 $2$, 那么 $4$ % $2$ == $0$. 如果继续向后遍历, 筛掉的数将是 $4 \times 3 = 12$, 而 $12$ 的最小质因子是 $2$, 不应由 $4$ 来筛掉 $12$, 而应在遍历到 $6$ 时筛掉 $6 \times 2 = 12$, 避免重复标记, 因此 `break` 掉. 

对于上述情况的处理也成为实现 $O(n)$ 预处理质数表的关键. 

```cpp
void Euler(int n) {
	np[0] = np[1] = true; // np -> not prime
	for (int i = 2; i <= n; ++ i) {
		if (!np[i]) prm[++ top] = i; // prm -> primes 
		for (int j = 1; j <= top and prm[j] <= n / i; ++ j) {
			np[prm[j] * i] = true;
			if (!(i % prm[j])) break;
		}
	}
}
```

进行以上算法之后, 我们即得到了一张质数表 `prm`. 

另外, 还可以在线性筛的过程中处理一些私货. 比如预处理每个数的最小质因子, 稍微改一下上面的代码即可. 

```cpp
void Euler(int n) {
	for (int i = 2; i <= n; ++ i) {
		if (!f[i]) prm[++ top] = i, f[i] = i;
		for (int j = 1; j <= top and prm[j] <= n / i and prm[j] <= f[i])
			f[i * prm[j]] = prm[j];
	}
}
```

进行以上算法之后, 我们同时得到了每个数的最小质因子 $f_i$

然而, 线性筛中使用了大量~~模法~~模运算, 常数比埃氏筛略大. 在小数据范围时, 埃氏筛的 $\log$ 会很小, 因此数据范围不大时埃氏筛可能不失为一种更好的选择. 

### 分解质因数

#### 试除法 $O(\sqrt n)$

```cpp
void decompose(int u) {
	for (int i = 2; ; ++ i) {
		while (u % i == 0) cnt[i] ++, u /= i;
		if (u == 1) return;
	}
}
```

-----------

## 约数

### 求所有约数

#### 试除法 $O(\sqrt n)$

```cpp
vector<int> getDivisors(int n){
	vector<int> ans;
	for (int i = 1; i <= n / i; i ++){
		if (n % i == 0){
			ans.push_back(i);
			if (i != n / i) ans.push_back(n / i);
		}
	}
	sort(ans.begin(), ans.end());
	return ans;
}
```

值得注意的是, $n$ 的约数个数约为 $\ln n$, 因此排序不是时间复杂度瓶颈. 

### 约数个数

一个数 $n$ 的约数个数常用函数 $d(n)$ 表示. 

依据算术基本定理计算约数个数：

对于一个数 $n$, 若对其分解质因数可表示为若干个质因数相乘的形式, 即将 $n$ 写成标准分解式的形式

$$n = p_1^{c_1} \times p_2^{c_2} \times ... \times p_k^{c_k}. $$

不难发现, 约数个数相当于在若干个约数中选取若干个约数相乘, 组成一个新的约数, 计入答案, 因此使用乘法原理即可. 

则约数个数为

<div>
$$
\begin{aligned}
	d(n) & = (c_1 + 1) \times (c_2 + 1) \times ... \times (c_k + 1) \\
		 & = \prod \limits_{i = 1}^{k} (c_i + 1). 
\end{aligned}
$$
</div>

### 约数之和

一个数 $n$ 的约数之和的 $k$ 次方常用函数 $\sigma_k(n)$ 表示. 

首先考虑简单情况, 即 $k = 1,\ \sigma_1(n)$, 常简记为 $\sigma(n)$. 

同样, 对于一个数 $n$ 写成它的标准分解式的形式

$$n = p_1^{c_1} \times p_2^{c_2} \times ... \times p_k^{c_k}.$$

同理, 考虑 $n$ 的一个因数是如何组成的：在每个质因子中选择 $0 \sim c_i$ 个相乘得到一个因数. 若求所有因数和, 根据乘法原理, 即得

<div>
$$
\begin{aligned}
	\sigma_1(n) & = (p_1^0 + p_1^1 + ... + p_1^{c_1}) \times ... \times (p_k^0 + p_k^1 + ... + p_k^{c_k}) \\ 
			  & = \prod\limits_{i = 1}^{k} \sum\limits_{j = 0}^{c_i} p_i^j. 
\end{aligned}
$$
</div>

推广可得

$$\sigma_k(n) = \prod\limits_{i = 1}^{k} \sum\limits_{j = 0}^{c_i} p_i^{j \times k}.$$

特别地, 当 $k = 0$, $\sigma_k(n) = d(n)$.

### 最大公约数

#### 辗转相除法 / 欧几里得算法

易证：

若 $d \mid a$ 且 $d \mid b$, 则 $d \mid ax + by$. 

因此 $\gcd(a, b) = \gcd(b, a \bmod b)$. 

-----------

## 同余

对于两个整数 $a$, $b$, 若满足 $a \bmod m = b \bmod m$, 则称 $a$ 与 $b$ 在 $mod m$ 的意义下**同余**, 记作 $a \equiv b \pmod m$. 

### 乘法逆元

#### 定义

对于两个互质的正整数 $a$, $b$, 若满足 $ax \equiv 1 \pmod b$, 则称 $x$ 为 $a$ 在 $mod\ b$ 意义下的**逆元**, 记作 $a^{-1}$

> 另外, 对于逆元还有另一种定义：
> 若 $x$ 满足 $m / a \equiv m * x \pmod b$, 则称 $x$ 为 $a$ 在 $mod\ b$ 意义下的逆元
> (其中 $b$, $m$ 互质, $b\mid a$)
> 
> (摘自李煜东《算法竞赛进阶指南》)

形式化地, 我们可以将逆元理解为 在模运算下的倒数, 即：

$$a * a^{-1} \bmod b = 1$$

#### 求一个数 $a$ 在 模 $b$ 意义下的逆元

##### 1. 扩展欧几里得算法

对于同余式 $ax \equiv 1 \pmod b$, 设 $ax - 1$ 是 $b$ 的 $-y$ 倍, 则有：

$$ax + by = 1$$

根据 $Bézout$ 定理（裴蜀定理）, 对于以上的一个线性同于方程有解, 当且仅当 $\gcd(a, b) = 1$, 即 $a$ 与 $b$ 互质. 我们直接应用 $exgcd$ 求出一组可行解, 再将解通过~~模法~~模运算偏移至 $[1, b - 1]$ 的区间即可

这是我们可爱的扩展欧几里得算法板子：

```cpp
LL exgcd(LL a, LL b, LL &x, LL &y){
	if (!b) { x = 1, y = 0; return a; }
	LL d = exgcd(b, a % b, x, y);
	LL tmp = x;
	x = y, y = tmp - (a / b) * y;
	return d;
}

x = (x % b + b) % b;
```

##### 2. 费马小定理

当模数 $b$ 为质数时（此时经常写作 $p$）, 我们可以根据费马定理, 对以上同余式进行变形, 得到：

$$b \times b^{p - 2} \equiv 1 \pmod p$$

因此, 我们使用快速幂在 $O(log_2 p)$ 的时间内求出 $b^{p - 2}$ 对 $p$ 取模的模数即可

```cpp
LL fPow(LL a, LL b, LL p) {
	LL res = 1;
	while (b) {
		if (b & 1) res = res * a % p;
		a = a * a % p;
		b >>= 1;
	}
	return res;
}
```

##### 3. 线性求逆元

当解决形如：

> 给定一个正整数 $n$, 求 $1$ ~ $n$ 之间所有数 $\bmod\ p$  意义下的逆元（保证 $p$ 为质数）

的问题时, 可以考虑在线性时间内递推逆元. 

设 $p = k \times i + r$, $(1 < r < i < p )$

因此 $k \times i + r \equiv 0 \pmod p$

两侧同时乘 $r^{-1}, i^{-1}$ 可得

$$i^{-1} \equiv -k \times r^{-1} \pmod p$$

即

$$i^{-1} \equiv -\lfloor \frac{p}{i} \rfloor \times (p \bmod i)^{-1} \pmod p$$

因此, 我们获得了在线性时间内递推区间模 $p$ 的逆元的方法

```cpp
inv[0] = inv[1] = 1;
for(int i = 2; i < p; ++ i)
	inv[i] = (p - p / i) * inv[p % i] % p;
```

### 同余方程

依据未知数次数的不同我们将同余方程分为以下两种：

#### 1. 线性同余方程



#### 2. 高次同余方程



#### 解同余方程组

##### $situation\ 1$ : 模数两两互质

中国剩余定理($crt$)

##### $situation\ 2$ : 模数不保证互质

扩展中国剩余定理 ($excrt$)

对于

## 特殊函数通识

除了上文介绍过的 $d(n)$ 与 $\sigma_k(n)$ 以外, 下面再介绍几种数论中的常见函数. 

### 下标函数

$$\text{id}(n) = n.$$

### 单位函数

<div>
$$\epsilon(n) = \begin{cases} 1 & x = 1 \\ 0 & x \ne 1 \end{cases} = [n = 1].$$
</div>

### 莫比乌斯函数



### 积性函数

定义：设 $f(n)$ 为定义在正整数上的函数, 如果有 $f(1) = 1$, 且满足 $\forall a, b \in \text{N}^*, \gcd(a, b) = 1 \Rightarrow f(ab) = f(a) \times f(b)$, 则称 $f$ 函数为积性函数. 

更一般性地, 若不要求 $\gcd(a, b) = 1$, 仍满足 $f(ab) = f(a) \times f(b)$ 则称该函数为 **完全积性函数**. 

可以证明, $d(n)$ 为积性函数, $\text{id}(n)$, $\epsilon(n)$, 与 $\sigma_k(n)$ 均为完全积性函数. 

#### 积性函数线性递推

利用积性函数性质, 若有关于 $n$ 的标准分解式 

$$n = \prod\limits_{i = 1}^k p_i^{c_i}$$

则有

$$f(n) = \prod\limits_{i = 1}^k f(p_i^{c_i})$$

有了以上递推式, 我们可以使用线性筛在 $O(n)$ 时间内预处理 $1 \sim n$ 内的积性函数值, 下文以求约数函数 $d(n)$ 为例. 



```cpp

```

##### 简单应用: [Sum of Divisors](https://atcoder.jp/contests/abc172/tasks/abc172_d)

### 欧拉函数

$\varphi(n)$ 表示 $1$ ~ $n - 1$ 中与 $n$ 互质的数的个数. 

对于任意正整数 $n$, 有

$$\varphi(n) = n \times \prod\limits_{p \mid n} (1 - \frac{1}{p})$$

且

$$\sum\limits_{d \mid n} \varphi(d) = n$$

对于任意两个正整数 $a$, $b$. 

$$\varphi(a \times b) = \varphi(a) \times \varphi(b) \times \frac{gcd(a, b)}{\varphi(\gcd(a, b))}$$

#### 感性理解

先考虑一种感性理解：

对于 $$n = p_1^{c_1} \times p_2^{c_2} \times ... \times p_k^{c_k}$$

将 $n$ 视为数轴上的一个线段, 将 $1 \sim n$ 分为若干长度为 $p_i$ 的小段, 则每一小段中只有最后一个数与 $n$ 不互质, 即总的互质概率为

$$\prod\limits_{p \mid n} \frac{p - 1}{p}$$

则 $1 \sim n$ 中与 $n$ 互质的数的个数为 $$n \times \prod\limits_{p \mid n} \frac{p - 1}{p}$$asdfasdf

#### 严格证明

考虑容斥. 

$$n - \frac{n}{p_1} - \frac{n}{p_2} ... + \frac{n}{p_1 \times p_2} ... - \frac{n}{p_1 \times p_2 \times p_3} ...$$

将原式打开发现

$$\varphi(n) = n \times (1 - \frac{1}{p_1}) \times (1 - \frac{1}{p_2}) \times (1 - \frac{1}{p_3}) ...$$

即与上式等价, 证毕. 

#### 线性筛欧拉函数

上文说到, 欧拉函数 $\varphi(n)$ 也是积性函数, 因此也可以通过线性筛在 $O(n)$ 时间内预处理 $1 \sim n$ 内的 $\varphi(n)$.

```cpp

```

## $\text{Lucas}$ 定理

```cpp
LL Lucas(LL n, LL m) {
  if (m == 0) return 1;
  return C(n % p, m % p) * Lucas(n / p, m / p) % p;
}

```

## $\text{Dirichlet}$ 卷积

对于两个数论函数 $f(x)$ 和 $g(x)$, 定义两个函数的狄利克雷卷积 

$$h(x) = \sum\limits_{d \mid n} f(n) \times g(n / d) = \sum\limits_{a \times b = n} f(a) \times g(b).$$

记作

$$h = f \ast g.$$

## 莫比乌斯反演

## 杂项

### $\text{Legengre' Law}$

### $\text{Kummer's Law}$

$p$ 在组合数中的幂次等于 p 进制下 n - m 需要借位的次数. 

证明:

