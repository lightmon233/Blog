---
title: "Codeforces Round 836题解（A、B、C）"
date: 2024-06-15T14:30:54+08:00
draft: false
description: ""
categories: ["算法竞赛"]
tags: ["数论", "质数", "分解质因数", "异或"]
---
{{< katex >}}


## A. SSeeeeiinngg DDoouubbllee

直接将原字符串翻转一下拼到原字符串的后面就构成了回文串。

```cpp
string s;

void solve() {
    cin >> s;
    cout << s;
    reverse(s.begin(), s.end());
    cout << s << '\n';
}
```

## B. XOR = Average

分\(n\)的奇偶性考虑，若\(n\)为奇数，我们可以让所有的\(a_i\)相等，这样\(a_1 \bigoplus a_2 \bigoplus \cdots \bigoplus a_n = a_1 = \frac{n \cdot a_1}{n}\)。若\(n\)为偶数，我们可以考虑可否将\(n\)为奇数时的某个\(a_i\)拆成两个\(a_{i_1}\)和\(a_{i_2}\)，使得\(a_{i_1} \bigoplus a_{i_2} = a_i\)并且\(\frac{a_{i_1} + a_{i_2}}{2} = a_i\)，这样就仍然满足\(a_1 \bigoplus a_2 \bigoplus \cdots \bigoplus a_n = a_1 = \frac{n \cdot a_1}{n}\)。容易发现\(1\)和\(3\)就刚好满足这样的限制，\(1 \bigoplus 3 = 2\)并且\(\frac{1 + 3}{2} = 2\)。

```cpp
int n;

void solve() {
    cin >> n;
    if (n & 1) {
        for (int i = 1; i <= n; i ++) {
            cout << 1 << ' ';
        }
        cout << '\n';
    }
    else {
        cout << 1 << ' ' << 3 << ' ';
        for (int i = 1; i <= n - 2; i ++) {
            cout << 2 << ' ';
        }
        cout << '\n';
    }
}
```

## C. Almost All Multiples

如果\(1\)这个位置放的是\(n\)，那么一定是可以构造出来一个排列的，并且字典序最小的排列就是除了\(1\)和\(n\)这两个位置外的其他位置\(i\)都放\(i\)。

排放形式如下：
`n 2 3 4 5 6 7 8 9 10 11 1`(\(n\)取\(12\)的情况)

如果\(1\)这个位置放的不是\(n\)，譬如说是\(x\)，那么我们就必须把\(x\)这个位置后面的是\(x\)的倍数的数放到\(x\)这个位置，并在那个数的位置上放上新的数。

那么如果\(n\)不是\(x\)的倍数，则\(n\)也不会是\(x * 2, x * 3, \cdots\)这些数的倍数，则我们按照上述的规则移动了一些数的位置后，最后\(n\)就无法用来补上最后移动的那个数所空缺下来的位置，所以这种情况下应该输出\(-1\)。

贪心地考虑，当\(n\)是\(x\)的倍数时，我们可以选择形如\(x, x * 2, x * 2  * 3, x * 2 * 5, n\)这些位置，将这些位置上的数向左移动一位，最后把\(n\)补到最后一位。显然，将\(\frac{n}{x}\)分解质因数，可以使乘的倍数的次数最大化，即让字典序较小的数字更多地靠前，且按照质因数从小到大的顺序乘，即让字典序越小的数越靠前。这样得到的排列就是最优的。

对于\(n = 12\)举例，\(x = 4\)的话，所求排列就是这样的：

`4 2 3 12 5 6 7 8 9 10 11 1`

```cpp
int n, x;

void solve() {
    cin >> n >> x;
    if (n % x != 0) {
        cout << -1 << '\n';
        return;
    }
    int y = n / x;
    vector<int> bag;
    for (int i = 2; i <= y / i; i ++) {
        if (y % i == 0) {
            while (y % i == 0) {
                bag.push_back(i);
                y /= i;
            }
        }
    }
    if (y > 1) bag.push_back(y);
    vector<int> ans(n + 1, 0);
    for (int i = 1; i <= n; i ++) {
        ans[i] = i;
    }
    ans[1] = x, ans[n] = 1;
    int cur = x;
    for (int i = 0; i < (int)bag.size(); i ++) {
        int t = cur;
        cur *= bag[i];
        ans[t] = cur;
    }
    for (int i = 1; i <= n; i ++) {
        cout << ans[i] << ' ';
    }
    cout << '\n';
}
```
