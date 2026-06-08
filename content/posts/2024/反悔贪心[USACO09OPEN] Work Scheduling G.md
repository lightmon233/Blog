---
title: "反悔贪心[USACO09OPEN] Work Scheduling G"
date: 2024-05-25T15:51:03+08:00
draft: false
description: ""
categories: ["算法竞赛"]
tags: ["堆", "贪心", "反悔贪心"]
---
{{< katex >}}


```cpp
int n;
int ans = 0;
PII a[N];

// 定义priority_queue的比较函数
struct cmp {
    bool operator() (PII a, PII b) {
        return a.second > b.second;
    }
};

priority_queue<PII, vector<PII>, cmp> pq;

void solve() {
    cin >> n;
    for (int i = 1; i <= n; i ++) {
        cin >> a[i].first >> a[i].second;
    }
    sort(a + 1, a + 1 + n);
    for (int i = 1; i <= n; i ++) {
        if (a[i].first > pq.size()) {
            ans += a[i].second;
            pq.push(a[i]);
        }
        else {
            if (a[i].second > pq.top().second) {
                ans -= pq.top().second;
                pq.pop();
                pq.push(a[i]);
                ans += a[i].second;
            }
        }
    }
    cout << ans << '\n';
}
```
