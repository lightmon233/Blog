---
title: "莫队算法（基础莫队）小结"
date: 2024-05-09T19:21:13+08:00
draft: false
description: ""
categories: ["算法竞赛"]
tags: ["cpp", "莫队", "分块", "数据结构"]
---
{{< katex >}}


# 莫队

## 基础莫队

本质是通过排序优化了普通尺取法的时间复杂度。

考虑如果某一列询问的右端点是递增的，那么我们更新答案的时候，右指针只会从左往右移动，那么i指针的移动次数是\(O(n)\)的。

当然，我们不可能让左右端点都单调来做到总体\(O(n)\)。

考虑对左端点进行分块。

莫队排序：
左端点按照分块的编号来排，如果分块编号不同的话编号较小的靠前，如果相同的话右端点小的在前。可以证明这样排完序的话时间复杂度可以做到\(O(n \sqrt n)\)。

这样我们把区间分成了\(\sqrt n\)块，每块的长度都是\(\sqrt n\)，在每一块内部，所有查询的右端点是递增的。

右指针：在每一块内部，右端点递增，所以右端点走的总数不会超过\(n\)（注意每一块内部放的是左端点，右端点是完全有可能超出这个块的范围的，因此这里为\(n\)），一共有\(\sqrt n\)块，所以右端点总共走的次数不会超过\(n \sqrt n\)。

左指针：先考虑每一次询问：

1. 左指针在块内部移动，块的长度是\(\sqrt n\)，因此最多只会移动\(\sqrt n\)次。

2. 左指针在相邻两块之间移动，最坏是从第一个块的左端点移动到第二个块的右端点，因此最坏移动\(2 \sqrt n\)次。

因为有q次询问，所以1是\(q \sqrt n\)，2是\(2n\)。
因为一共有\(\sqrt n\)个块，我们从前往后要跨过\(\sqrt n - 1\)次，每次最多是\(2 \sqrt n\)，所以时间复杂度是\(2n\)。

所以总时间复杂度为\(O(q \sqrt n)\)。

```c++
// 代码为统计一段区间上是否有不相同的数，没有输出yes
int n, q;
int a[N];
vector<array<int, 3>> v;
int cnt[210];
int ans[N];
int len;

int get(int x) {
    return x / len;
}

void adds(int x, int &res) {
    if (!cnt[x]) res ++;
    cnt[x] ++;
}

void del(int x, int &res) {
    cnt[x] --;
    if (!cnt[x]) res --;
}

void solve() {
    cin >> n >> q;
    len = max(1, (int)sqrt((double)n * n / q));
    for (int i = 1; i <= n; i ++) {
        cin >> a[i];
        a[i] += 100;
    }
    for (int i = 1; i <= q; i ++) {
        int l, r;
        cin >> l >> r;
        v.push_back({i, l, r});
    }
    auto cmp = [&](array<int, 3> &a, array<int, 3> &b) {
        int i = get(a[1]), j = get(b[1]);
        if (i != j) return i < j;
        return a[2] < b[2];
    };
    sort(v.begin(), v.end(), cmp);
    // i是右指针，j是左指针
    for (int k = 0, i = 0, j = 1, res = 0; k < q; k ++) {
        int id = v[k][0], l = v[k][1], r = v[k][2];
        while (i < r) adds(a[++ i], res);
        while (i > r) del(a[i --], res);
        while (j < l) del(a[j ++], res);
        while (j > l) adds(a[-- j], res);
        ans[id] = res;
    }
    for (int i = 1; i <= q; i ++)
        if (ans[i] == 1) cout << "YES\n";
        else cout << "NO\n";
}
```
