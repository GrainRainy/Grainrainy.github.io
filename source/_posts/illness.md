---
title: 传染病 | 2023.10.17 鲜花
date: 2023-10-17 23:51:12
categories: 鲜花
tags: 鲜花
author: GrainRain
cover: https://pic.imgdb.cn/item/652eadb6c458853aef34363d.jpg
single_column: true
---


刺眼的白炽灯, 照的没有阴影的地面, 门和窗开在同一侧的奇怪屋子, 50cm 规格的老式纤维吊顶上的一小块缺口是夹在目之所及的白色之间唯一的一点黑色, 我能盯着它看一整天, 并顺便脑补出吊顶上面密密麻麻的虫子掉下来的恐怖景象. 

我似乎染上了一种怪病. 

医生给我开了副药, 上面写着: 温喉颗

"它的名字为什么不叫 xx 颗粒?" 这句话刚想说出口, 就被我噎了回去. 

我十分清楚, 这个时间, 这种状况下遵医嘱或许是更好的选择. 但在这种医院的这种地方, 哪个人不是绝命赌徒呢. 既然是赌局, 那也得有筹码, 还得有永远相信自己下一把, 下下一把能赢的狂热. 

呆在那样的地方几乎使我窒息, 我并没有拿那副药, 慌忙跑走了. 

</br>

几天前, gk 跟我说: "只要我每天学 whk 30 分钟, 若干天后我的 whk 水平就是正无穷, 吊打你们."

于是我反击: "那我一天学 whk 1 分钟, 相同时间后我的水平也是正无穷. 于是我每天学 1 分钟就可以吊打你学 30 分钟. "

于是他开始扯皮. 

不过我发现我并没有每天学 whk 1 分钟, 而据我观察, 他也没有每天学 whk 30 分钟. 

我想起来停课前我对自己说: 停课后每天中午吃完饭到睡觉中间有 30 分钟, 拿来练字不出几个月就能练到能看的水平. 

我又想起来, 那本字帖已经快两个月没摸过了. 

这是我得的另一重怪病. 

</br>

在鲜花中写做题记录的卷王那么多, 我认为我也是一个 juanwang, 所以我也要在鲜花里写做题记录. 

但显然写鲜花的时候是卷不动的, 这就要向 GCY_XZT 取取经了. 

于是我去除了正经且严谨的公式化题解, 而保留了一部分杂谈形式的做题总结, 希望不要噎到大家. 

给定两个字符集相同的序列, 可以对其中一个序列的任意两个相邻位置进行任意次交换, 求使得这两个序列相同的最小交换次数. 

事实上对于其中一个序列可交换的约束是假的, 因为对于另一个序列的交换都可以唯一地对应到当前序列进行的一次反交换, 于是一个序列可交换和两个序列可交换是等价的. 

不失一般性的, 我们将目标序列视为有序, 则问题等价于对当前序列进行冒泡排序的交换次数. 

而冒泡排序有一个经典结论: 交换次数为序列逆序对数. 

因此若建立由当前位置向目标位置的映射, 则问题转化为求该映射数组的逆序对. 