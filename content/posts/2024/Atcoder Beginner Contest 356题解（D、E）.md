---
title: "Atcoder Beginner Contest 356题解（D、E）"
date: 2024-06-03T22:22:16+08:00
draft: false
description: ""
categories: ["算法竞赛"]
tags: ["atcoder", "组合", "数学", "调和级数"]
---
{{< katex >}}


## D.  Masked Popcount

按位考虑 + 排列组合

考虑\(M = 10110111001\)
\(i\)从\(0\)循环到\(N\)

因为求的是所有\(i \& M\)的二进制表示中1的个数，所以可以按位考虑，考虑有多少个\(i\)的\(bit\)位与\(M\)的\(bit\)位\(\&\)出来是\(1\)。

首先如果两个数\(bit\)位\(\&\)出来是\(1\)，那么这两个位一定都是\(1\)。

假设当前\(bit\)是\(7\)
```
bit: 10 9 8 7 6 5 4 3 2 1 0
M:    1 0 1 1 0 1 1 1 0 0 1
i:    _ _ _ 1 _ _ _ _ _ _ _
```

如果\(bit\)前面的位填`1 0 1`，则\(bit\)后面的位只能填`0`到`0 1 1 1 0 0 1`；方案数是\(1 * (M \& 0x11110000000 + 1)\)
如果\(bit\)前面的位填`0`到`1 0 0`，则\(bit\)后面的位可以填`0`到`1 1 1 1 1 1 1`。方案数是\((i >> (bit + 1)) * (1 << bit)\)

写代码时注意取模操作，传入函数中的参数也应该取模。

```cpp
int n, m;

void solve() {
//    cout << ((1 << 9) + (1 << 7) + (1 << 6) + (1 << 4) + 1) << '\n';
    cin >> n >> m;
    int ans = 0ll;
    for (int r = 60ll; r >= 0ll; r --) {
        if (m >> r & 1ll) {
            if (n >> r & 1ll) {
                plut(ans, plu(mul((n >> r + 1ll) % mod, (1ll << r) % mod), (n & ((1ll << r) - 1ll)) % mod + 1ll));
            }
            else {
                plut(ans, mul((n >> r + 1ll) % mod, (1ll << r) % mod));
            }
        }
    }
    cout << ans % mod << '\n';
}
```

## E. Max/Min

考虑如果a数组中的元素各不相同。

观察到\(a[i] \le 1e6\)，可以开个桶来存储每个\(a[i]\)出现的次数，可以对这个桶数组再求个前缀和，则可以用这个前缀和数组\(O(1)\)地求出某一连续值域中出现的\(a[i]\)的个数。

现在如果我们（先将\(a\)排序以方便理解）从前往后枚举\(a\)中的每一个元素，求当前位置\(i\)后面有多少个\(j\)使得\(\lfloor \frac{a_j}{a_i} \rfloor\)等于某一个值\(x\)，我们发现这个x的最大值就是\(\frac{max(a[i])}{a[i]}\)。对于每个x，我们可以用\(cnt[a[i] * x + a[i] - 1] - cnt[a[i] - 1]\)来得出\(a[i]\)对答案的贡献，假设\(m = max(a[i])\)，那么我们这样操作的时间复杂度就是\(\sum\limits_{i=1}{n} \frac{m}{a[i]} <= \sum\limits_{i=1}{n} \frac{m}{i}\)，因为后者是调和级数，所以时间复杂度就是\(O(mlogn)\)。

在代码实现中要注意特判\(a[j]\)是\(a[i]\)的\(1\)倍的情况并且注意把相同的\(a[i]\)内部的贡献加上。

```cpp
int n;
int cnt[(int)2e6 + 10];
int s[(int)2e6 + 10];

void solve() {
    cin >> n;
    memset(cnt, 0, sizeof cnt);
    vector<int> a;
    int max_a = 0;
    for (int i = 0; i < n; i ++) {
        int x;
        cin >> x;
        a.push_back(x);
        cnt[x] ++;
        max_a = max(max_a, x);
    }
    sort(a.begin(), a.end());
    a.erase(unique(a.begin(), a.end()), a.end());
    int ans = 0;
    memset(s, 0, sizeof s);
    for (int i = 1; i <= (int)1e6; i ++) {
        s[i] = s[i - 1] + cnt[i];
    }
//    for (int i = 1; i <= 4; i ++) {
//        cout << s[i] << ' ';
//    }
//    cout << '\n';
//    for (int i = 0; i < a.size(); i ++) {
//        cout << a[i] << ' ';
//    }
//    cout << '\n';
//    for (int i = 0; i < a.size(); i ++) {
//        cout << cnt[a[i]] << ' ';
//    }
//    cout << '\n';
    for (int i = 0; i < a.size(); i ++) {
        int m = max_a / a[i];
        for (int j = 1; j <= m; j ++) {
//            cout << j << ' ' << j * a[i] + a[i] - 1 << ' ' << j * a[i] << '\n';
//            int ct = (s[j * a[i] + a[i] - 1] - s[(j == 1 ? j * a[i] : j * a[i] - 1)]) * cnt[a[i]] * j;
//            cout << ct << '\n';
            ans += (s[min(j * a[i] + a[i] - 1, (int)1e6)] - s[min((int)1e6, (j == 1 ? j * a[i] : j * a[i] - 1))]) * cnt[a[i]] * j;
        }
        ans += cnt[a[i]] * (cnt[a[i]] - 1) / 2;
    }
    cout << ans << '\n';
}
```
