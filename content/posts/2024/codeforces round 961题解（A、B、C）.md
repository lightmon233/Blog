---
title: "codeforces round 961题解（A、B、C）"
date: 2024-06-07T10:22:27+08:00
draft: false
description: ""
categories: ["算法竞赛"]
tags: ["codeforces", "位运算", "构造"]
---
{{< katex >}}


## A. Guess the Maximum

因为\(i < j\)，所以所有的\([i, j]\)区间中都至少包含两个相邻元素，所以只要求出所有相邻元素中较大值的最小值即可。

```cpp
int n;
int a[N];

void solve() {
    cin >> n;
    int min_v = 1e9 + 1;
    for (int i = 1; i <= n; i ++) {
        cin >> a[i];
    }
    for (int i = 1; i <= n - 1; i ++) {
        min_v = min(min_v, max(a[i], a[i + 1]));
    }
    cout << min_v - 1 << '\n';
}
```

## B. XOR Sequences

观察结论，发现样例\(4\)的答案是\(2^{25} = 33554432\)，猜测所有答案都是\(2\)的次方。

以样例\(3\)为例：
```
    5432 10        5432 10
57: 1110 01    37: 1001 01
    0100 00        0011 00
    0100 01        0011 01
    0100 10        0011 10
    0100 11        0011 11
```
发现\(x\)和\(y\)的最长公共后缀对应的位可以从\(0\)开始连续地填，从\(00\)填到\(11\)就走完了这两位可以提供的所有连续数值。如果从\(000\)填到\(111\)的话，因为更高位\(x\)和\(y\)的值不同，所以异或出来的值不是连续的。

再看\(5432\)位，我们要保证\(x\)和\(y\)都不能填\(0000\)，因为\(0000\)会和后面两位\(00\)组成\(0\)，但是题目要求是从\(1\)开始。假设\(x\)填\(0001\)，如果\(y\)必须填\(0000\)才能保证前缀异或相同，那么我们可以把\(x\)改填\(0011\)，因为异或的性质，原本第\(3\)位取的是\(x\)的第三位，现在我们改成\(1\)，就是取\(x\)的第三位取反，那么\(y\)的第三位就也必须取反，那么\(y\)就得填\(0010\)。这样，我们总可以不用选\(0000\)去填。

```cpp
int x, y;

void solve() {
    cin >> x >> y;
    int i;
    for (i = 0; i <= 30; i ++) {
        if ((x >> i & 1) != (y >> i & 1)) {
            cout << (1 << i) << '\n';
            return;
        }
    }
    cout << (1 << i) << '\n';
}
```

## C. Earning on Bets

吐槽：忘了删刚开始猜的判断\(-1\)的情况，导致赛时一直\(WA \ 8\)。

<font color=red>更新：刚开始猜的是对的，在判断\(flag == 1\)时应该这样写：</font>
```cpp
flag > 1 && fabs(flag - 1) < 1e-6
```

设\(x\)的总和为\(s\)。

因为\(k_i * x_i > s\)，所以\(x_i >= s / k_i + 1\)，然后我们要保证所有的\(s / k_i + 1\)加起来小于等于\(s\)。因为这样我们可以在每个\(s / k_i + 1\)上加若干值使得他们的总和等于\(s\)，且仍然满足\(k_i * x_i > s\)。

那么我们可以二分查找这个\(s\)，找不到就输出\(-1\)。

```cpp
int n;
int k[55];
int a[55];

bool check(int x) {
    int sum = x ;
    for (int i = 1; i <= n; i ++) {
        sum -= x / k[i];
    }
    return sum >= n;
}

void solve() {
    cin >> n;
    for (int i = 1; i <= n; i ++) {
        cin >> k[i];
    }
//    double flag = 0;
//    for (int i = 1; i <= n; i ++) {
//        flag += (double)1 / (double)k[i];
//    }
//    if (flag > 1 || fabs(flag - 1) < 1e-6) {
//        cout << -1 << '\n';
//        return;
//    }
    int l = n - 1, r = n * (int)1e9 + 1;
    while (l < r) {
        int mid = l + r >> 1;
        if (check(mid)) r = mid;
        else l = mid + 1;
    }
    int s = l;
    if (s == n - 1 || s == n * (int)1e9 + 1) {
        cout << -1 << '\n';
        return;
    }
//    cout << l << '\n';
    int sum = 0;
    for (int i = 1; i <= n; i ++) {
        a[i] = s / k[i] + 1;
        sum += a[i];
    }
    int cnt = l - sum;
    a[1] += cnt;
    for (int i = 1; i <= n; i ++) {
        cout << a[i] << ' ';
    }
    cout << '\n';
//    sum = 0;
//    for (int i = 1; i <= n; i ++) {
//        sum += a[i];
//    }
//    for (int i = 1; i <= n; i ++) {
//        cout << a[i] * k[i] - sum << ' ';
//    }
//    cout << '\n';
}
```
