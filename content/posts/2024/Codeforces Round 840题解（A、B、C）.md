---
title: "Codeforces Round 840题解（A、B、C）"
date: 2024-06-08T01:51:40+08:00
draft: false
description: ""
---
{{< katex >}}


## A. Absolute Maximization

我们可以选择两个位置\(i, j\)来存放最大值\(a_i\)和最小值\(a_j\)，对每一位，如果从\(a_{[1, n]}\)的这一位有\(1\)，我们就可以把\(1\)挪到\(a_i\)里，如果这意味有\(0\)，我们就可以把\(0\)挪到\(a_j\)里，这样就可以构造出最大的\(a_i\)和最小的\(a_j\)。

```cpp
int n;
int a[550];

void solve() {
    cin >> n;
    for (int i = 1; i <= n; i ++) {
        cin >> a[i];
    }
    int max_v = 0, min_v = 1023;
    for (int r = 9; r >= 0; r --) {
        for (int i = 1; i <= n; i ++) {
            if (a[i] >> r & 1) {
                max_v |= 1 << r;
            }
            else {
                min_v &= ~(1 << r);
            }
        }
    }
    cout << max_v - min_v << '\n';
}
```

## B. Incinerate

将怪物按照\(h\)从小到大排序，每次打出\(k\)的伤害就相当于把所有\(h \le\)前\(i\)次伤害总和的怪物击倒。那么我们每次可以二分找出剩下的怪物的起始位置。

当某一次\(k\)降到\(\le 0\)后，就不可能再打倒新的怪物，如果此时还没有把所有怪物都打倒的话，那么就不可能打倒所有的怪物了。

模拟这个过程，最坏情况是\(k\)每次都减一，因此时间复杂度最坏为\(klogn+nlogn\)。

```cpp
int n, k;
PII a[N];
int min_v[N];

void solve() {
    cin >> n >> k;
    for (int i = 1; i <= n; i ++) {
        cin >> a[i].first;
    }
    for (int i = 1; i <= n; i ++) {
        cin >> a[i].second;
    }
    sort(a + 1, a + 1 + n);
    min_v[n + 1] = INF;
    for (int i = n; i >= 1; i --) {
        min_v[i] = min(min_v[i + 1], a[i].second);
    }
//    for (int i = 1; i <= n; i ++) {
//        cout << min_v[i] << ' ';
//    }
//    cout << '\n';
    int cur = k;
    while (1) {
        int pos = lower_bound(a + 1, a + 1 + n, PII(cur + 1, 0)) - a - 1;
//        cout << pos << '\n';
        if (pos >= n) {
            cout << "YES" << '\n';
            return;
        }
        int cnt = k - min_v[pos + 1];
//        cout << cnt << '\n';
        cur += cnt;
        k = cnt;
        if (cnt <= 0) {
            cout << "NO" << '\n';
            return;
        }
    }
}
```

## C. Another Array Problem

~~有点恶心的一道题，我最后\(15\)分钟才突然产生思路，但是分类有点小问题加没能调出来。。。结束后看分数，纳尼！竟然是\(2000\)分的题。。。~~

观察到如果最大值所在位置左边或者右边有\(\ge 2\)个元素，我们就可以把这个方向上所有元素都变成最大值。

例如：\(2, 5, 3, 4\) -> \(2, 5, 1, 1\) -> \(2, 5, 0, 0\) -> \(2, 5, 5, 5\)。

实际上，如果最大值所在位置左边或右边有\(\ge 2\)个元素，我们可以把数组中所有元素都变成最大值。

例如：\(2, 5, 3, 4\) -> \(2, 5, 5, 5\) -> \(3, 3, 5, 5\) -> \(0, 0, 5, 5\) -> \(5, 5, 5, 5\)。

那么就只剩下两种特别情况了，当\(n = 2\)时，只有两种方案，换或不换，直接输出比较即可。

当\(n = 3\)并且最大值在中间时~~比较恶心~~，我们注意到在换的过程中出现在最左边或者最右边的数都可以被用来覆盖整个数组。我们可以发现出现在左边的数最大是\(a[1]\)或者\(a[2] - a[1]\)，出现在右边的数最大是\(a[3]\)或\(a[2] - a[3]\)，因为只要换一次，原本的最大值肯定就不保了，所以把这四个情况和不换的情况综合起来取\(max\)就是答案。但是如果要求具体详细严谨的证明，比较麻烦。~~蒟蒻想不出来，想出来也表达不清楚啊。。。~~

```cpp
int n;
int a[M];

void solve() {
    cin >> n;
    int max_v = 0;
    for (int i = 1; i <= n; i ++) {
        cin >> a[i];
        max_v = max(max_v, a[i]);
    }
    vector<int> bag;
    for (int i = 1; i <= n; i ++) {
        if (a[i] == max_v) {
            bag.push_back(i);
        }
    }
    int ans = 0;
    for (int i = 1; i <= n; i ++) {
        ans += a[i];
    }
    if (n == 2) {
        cout << max(ans, abs(a[1] - a[2]) * 2) << '\n';
        return;
    }
    if (n >= 4) {
        cout << n * max_v << '\n';
        return;
    }
    if (a[1] == max_v || a[3] == max_v) {
        cout << n * max_v << '\n';
        return;
    }
    cout << max(ans, max({a[2] - a[1], a[2] - a[3], a[1], a[3]}) * 3) << '\n';
}
```