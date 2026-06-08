---
title: "第十届山东省大学生程序设计竞赛题解（A、F、M、C）"
date: 2024-05-10T02:25:46+08:00
draft: false
description: ""
categories: ["算法竞赛"]
tags: ["cpp", "xcpc", "省赛"]
---
{{< katex >}}


~~部分代码define了long long，请记得开long long~~

## A. Calandar

把年份、月份、单个的天数全都乘以对应的系数转化成单个的天数即可，注意最后的结果有可能是负数，要转化成正数。发现技巧是：`(ans % 5 + 5) % 5`。<font color=red>？</font>

还有注意不能这样写，答案不正确。或许是因为取模运算没有这样的性质？<font color=red>？</font>
```cpp
ans = plu(sub(plu(plu(mul(sub(y2, y), 365), mul(sub(m2, m), 30)), d2), d), x);
```

```cpp
int y, m, d;
int y2, m2, d2;

map<string, int> mp = {
        {"Monday", 0},
        {"Tuesday", 1},
        {"Wednesday", 2},
        {"Thursday", 3},
        {"Friday", 4},
};

map<int, string> mpr = {
        {0, "Monday"},
        {1, "Tuesday"},
        {2, "Wednesday"},
        {3, "Thursday"},
        {4, "Friday"},
};

void solve() {
    cin >> y >> m >> d;
    string s;
    cin >> s;
    int x = mp[s];
    cin >> y2 >> m2 >> d2;
    int ans = (y2 - y) * 365 + (m2 - m) * 30 + d2 - d + x;
    if (ans >= 0) cout << mpr[ans % 5] << '\n';
    else cout << mpr[(ans % 5 + 5) % 5] << '\n';
}
```

## M. Sekiro

很简单的签到，但是把我害惨了，记得以前校某次比赛用过这题，当时就让我大脑宕机了一次。

注意到每次操作都会使\(n\)减半，因此最多只会操作\(log(n)\)次，但是有可能减半后和减半前值相同，造成死循环，因此这种情况要特判结束循环。

```cpp
int n, k;

void solve() {
    cin >> n >> k;
    while (k --) {
        if (n == (n + 1) / 2) break;
        n = (n + 1) / 2;
    }
    cout << n << '\n';
}
```

## F. Stones in the bucket

![image](/img/cnblogs/3349274-20240510020715761-689260659.png)

我们可以先将序列排个序，然后假设我们最后要把序列的元素全部变成\(x\)，那么我们可以把所有\(>x\)的区块都一个个的撒给\(<x\)的区块，就像撒沙子那样。可以证明，只要大于\(x\)的量足够填满小于\(x\)的量，则一定是可以有方案实现这样填满，填满后剩下的部分用操作1扔掉即可。

故二分答案即可。

时间复杂度\(O(log(1e9)n)\)

```cpp
int n;
int a[N];

bool check(int x) {
    int sum0 = 0;
    int sum1 = 0;
    for (int i = 1; i <= n; i ++) {
        if (a[i] < x) sum0 += x - a[i];
        else sum1 += a[i] - x;
    }
    return sum1 >= sum0;
}

void solve() {
    cin >> n;
    for (int i = 1; i <= n; i ++) {
        cin >> a[i];
    }
    sort(a + 1, a + 1 + n);
    int l = 0, r = 1e9 + 1;
    while (l < r) {
        int mid = l + r + 1ll >> 1ll;
        if (check(mid)) l = mid;
        else r = mid - 1;
    }
    int ans = 0;
    for (int i = 1; i <= n; i ++) {
        if (a[i] > l) ans += a[i] - l;
    }
    cout << ans << '\n';
}
```

## C. Wandering robot

最让我头疼的一题，看完题解最想锤头的一题。

我刚开始一直在画图，想着算出前\(k-1\)次移动到的位置加上最后一次可以触及的最大曼哈顿距离即可。首先是分类讨论的头疼，接下来是发现可以统一为一种情况但是一直WA2的头疼。。。

正解是推导表达式看函数图像取最值。

设第\(i\)步之后坐标是\((x_i, y_i)\)，则\((tn + i)\)步之后坐标是\((tx_n + x_i, ty_n + y_i)\)，距离原点的曼哈顿距离是\(|tx_n + x_i| + |ty_n + y_i|\)。

函数\(f(t) = |tx_n + x_i|\)的图像是\(V\)形，两个叠在一起可以通过画图得出。

这里引用官方题解的图。

![image](/img/cnblogs/3349274-20240510021952750-1751243022.png)

可见该函数在左端点或者右端点取得最大值，因此我们只需要考虑\(t = 0\)和\(t = k - 1\)即可。

时间复杂度\(O(n)\)。

```cpp
int n, k;
string s;

void solve() {
    cin >> n >> k;
    cin >> s;
    int X = 0, Y = 0;
    for (int i = 0; i < n; i ++) {
        if (s[i] == 'L') {
            X --;
        }
        else if (s[i] == 'R') {
            X ++;
        }
        else if (s[i] == 'U') {
            Y ++;
        }
        else {
            Y --;
        }
    }
    int x = 0, y = 0;
    int ans = 0;
    for (int i = 0; i < n; i ++) {
        if (s[i] == 'L') {
            x --;
        }
        else if (s[i] == 'R') {
            x ++;
        }
        else if (s[i] == 'U') {
            y ++;
        }
        else {
            y --;
        }
        ans = max({ans, abs(x) + abs(y), abs((k - 1) * X + x) + abs((k - 1) * Y + y)});
    }
    cout << ans << '\n';
}
```
