# 小话随机数

随机数的重要性不言而喻：加密密钥、赌博、抽奖、蒙特卡洛模拟。计算机是怎么生成随机数序列的？随机性来自何处？我们需要完全的随机吗？

## 真伪随机数

什么是随机？这几乎是个哲学问题。随机生成的序列应该满足：

1. 统计随机：生成的序列能够通过统计学的随机性验证
2. 不可预测：给定算法和部分序列，无法有效推断下一个值
3. 不可重现：给定算法和起始种子，无法重新生成相同序列

完全满足三个条件就能叫***真随机数***，但是生成真随机数又贵又难又慢，比如需要观察量子或辐射效应[^1]。大多数时候，我们不需要如此高质量的序列。通过计算生成的随机数序列叫***伪随机数***，并根据是否可以预测，分为以下两种：

## 统计学伪随机数

> 序列A：010101010101010101010101010101  
> 序列B：110010000110000111011110111011
>
> 哪个是真随机？

想象你要生成一串只包含`0`和`1`的序列，可以用掷硬币的方式，每次的结果正面记`1`，反面记`0`。当然你也可以作弊，胡乱写一行数字，大致保证它们的个数相等。如何分辨哪一串是乱写的呢？其实我们有一系列统计学工具[^2]可以帮助我们，并且序列越长，把握越大。

```cpp
#include <cstdlib>
#include <ctime>

int main() {
    srand((unsigned)time(NULL));
    auto pseudorandom_number = rand();
}
```

C++中的`rand()`采用***线性同余法***生成随机数，它简单、快速、易于实现，几乎是最古老、最常用的一种伪随机数生成算法，思想最早由庞加莱在19世纪末提出。

> 生成 $X_i ∈ [0, m-1]$的随机数序列 $X$，可以用以下线性迭代公式：
> 
> $X_{n+1} = (aX_n+c) \mod m$

其中 $a$， $c$， $m$和 $X_0$都是常数，如在`GCC`中

- $a = 1103515245$
- $c = 12345$
- $m = 2^{31}$
- 种子 $X_0$在`srand()`中设置，如使用`time()`返回的Unix时间戳
- $X_i$在`rand()`取低31位后返回，保证int类型变量非负

不同的常数组合在计算效率和随机性分布上有区别，特别对 $a$和 $m$的选择尤其敏感，因此注定不能随机选取他们的值。但不论如何选、是否随机选，对 $m$取模的操作，都让这个随机序列有不大于 $m$的周期（当且仅当Hull–Dobell定理[^3]满足时，周期等于 $m$），生成的随机数质量也因此被限制。如果需要高质量、大周期的随机数序列用于蒙特卡洛过程，可以考虑使用梅森旋转算法[^4]或PCG家族[^5]。

其次，这类算法生成的序列还是是确定的。尽管这种特性为调试软件提供了方便，也能在结果上达到统计学上的随机性，但由于 $a$、 $c$、 $m$一般公开，攻击者只需要获得序列中连续的两个值，就能确定整个序列，所以此类算法生成的随机数，无法用于密码学领域。

## 密码学伪随机数

指不仅能满足统计学随机性，还能在给定算法和一部分序列时，无法在多项式时间内，对下一个生成的值有显著大于50%的把握，即无法猜出下一个值。这类随机数广泛应用于对安全性有要求的密钥生成、密码加盐等过程，如HTTPS的TLS握手、信用卡的CVV码、数据库存储密码的过程等。

满足多项式时间内不可预测的算法，如何做到呢？现代操作系统能够从环境噪音中获取熵：键盘敲击、磁盘I/O、网络活动等等，这些信息既不确定，又难以被外界观察，可以看做一种真随机源（有争议）或一个个“熵池”。但是前文提到，这种外部的随机信息通常比较昂贵，或者数量无法满足要求。

因此算法会从一个难以预测的种子开始，再计算生成后续的随机数。算法首先把种子作为私钥，并维护一个`V`值。随机数生成的过程中，有两种不断向算法引入熵值的做法：一种是获取熵值作为新的种子，即改变私钥；另一种是把获取的熵值同`V`值打包，生成新的`V`值。在每次生成新的随机数时，结合密钥、`V`值、上一个随机数，通过哈希函数生成下一个随机值，即

    new_value = hash(concatenate(old_value, private_key, V))

可以看出`V`值和私钥（种子）的作用是相似的，都是向算法中增加随机性，以抵抗攻击。更新它们的频率由安全性要求决定，只要没有完全泄露（由于哈希函数的特性，知道了随机数序列的一段，也无法倒推出种子），生成的新随机数值就无法预测。

[^1]: [RANDOM.ORG](https://random.org)和[ANU QRNS](https://qrng.anu.edu.au/random-hex/)分别通过大气噪音和量子波动提供真随机数生成服务

[^2]: Statistical tests by [NIST](https://csrc.nist.gov/Projects/random-bit-generation/Documentation-and-Software/Guide-to-the-Statistical-Tests)

[^3]: [Hull–Dobell定理](https://chagall.med.cornell.edu/BioinfoCourse/PDFs/Lecture4/random_number_generator.pdf)：$m$与$c$互素，且$a-1$能被$m$的所有因子整除，且如果$m$能被4整除，$a-1$也要能被4整除

[^4]: [梅森旋转算法](http://www.math.sci.hiroshima-u.ac.jp/~m-mat/MT/ARTICLES/mt.pdf)：是Python中默认的随机数生成算法，提供了比线性同余法大得多的周期

[^5]: [PCG家族](https://www.pcg-random.org/index.html)：2011年提出的一种现代随机数生成算法，它修正了一些古典随机数生成器的缺陷，提供了更好的计算性能、难以预测性、统计随机性等，是目前综合表现最好的伪随机数生成器之一
