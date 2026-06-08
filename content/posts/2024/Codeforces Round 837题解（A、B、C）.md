---
title: "Codeforces Round 837题解（A、B、C）"
date: 2024-06-10T13:50:03+08:00
draft: false
description: ""
categories: ["算法竞赛"]
tags: ["codeforces", "堆"]
---
{{< katex >}}


## A. Hossam and Combinatorics

\(|a_i - a_j|\)最大的就是最大值和最小值，注意要开`long long`。

```cpp
int n;
int a[N];
 
void solve() {
    cin >> n;
    int min_v = INF, max_v = 0;
    for (int i = 1; i <= n; i ++) {
        cin >> a[i];
        min_v = min(min_v, a[i]);
        max_v = max(max_v, a[i]);
    }
    int cnt_min = 0, cnt_max = 0;
    if (min_v == max_v) {
        cout << n * (n - 1) << '\n';
        return;
    }
    for (int i = 1; i <= n; i ++) {
        if (a[i] == min_v) cnt_min ++;
        if (a[i] == max_v) cnt_max ++;
    }
    cout << cnt_min * cnt_max * 2 << '\n';
}
```

## B. Hossam and Friends

考虑对每一个\(l\)，有那些\(r\)满足\([l, r]\)中的朋友都互相认识。

设共有\(m\)各互相不认识的对\((l, r)\)，则所有小于\(l\ge\)当前枚举到的\(i\)的\((l, r)\)对中最小的\(r\)的下标都可以作为当前\(i\)的\(r\)。

那么我们开个优先队列维护即可。

```cpp
 
int n, m;
vector<int> bag[N];
priority_queue<PII, vector<PII>, greater<>> pq;
 
void solve() {
    cin >> n >> m;
    for (int i = 1; i <= n; i ++) bag[i].clear();
    while (!pq.empty()) pq.pop();
    for (int i = 1; i <= m; i ++) {
        int l, r;
        cin >> l >> r;
        if (l > r) swap(l, r);
        bag[l].push_back(r);
    }
    int ans = 0;
    for (int i = n; i >= 1; i --) {
        for (auto j : bag[i]) pq.push({j, i});
        if (pq.empty()) {
            ans += n - i + 1;
            continue;
        }
        ans += pq.top().first - i;
    }
    cout << ans << '\n';
}
```

## C. Hossam and Trainees

如果\(x\)可以同时整除\(a[i]\)和\(a[j]\)，那么\(x\)的一个质因子\(p\)就也可以整除\(a[i]\)和\(a[j]\)。一种想法是枚举\(\le 1e9\)的的所有质数，判断是否有一个质数可以同时整除\(a[i]\)和\(a[j]\)，但是\(\le 1e9\)的质数的数量是\(5.08e7\)，就算打表，综合起来的时间复杂度也不够用。

考虑对每个数分解质因数，那么如果\(a[i]\)和\(a[j]\)分解质因数的质因子序列有重合的话，就说明\(a[i]\)和\(a[j]\)可以被这两个重合的质因子整除。直接这样做的时间复杂度是\(O(n \sqrt{1e9})\)，且质因子的跨度太大，开桶难以存下。

我们可以先预处理出\(\le \sqrt{1e9}\)的所有质数，这样分解每个\(a[i]\)的质因数的时间复杂度可以优化到\(O(\frac{\sqrt{1e9}}{log \sqrt{1e9}})\)。然后由于\(n\)最多只会有一个\(> \sqrt n\)的质因数，所以\(> \sqrt{a[i]}\)的质因数可以用`map`来存，\(\le \sqrt{a[i]}\)的质因数用普通的桶来存即可。

```cpp
int primes[N], cnt, st[N];
int a[N];
int n;
int cnt_prime[N];
map<int, int> mp;

void getPrimes(int n) {
    for (int i = 2; i <= n; i ++) {
        if (!st[i]) primes[cnt++] = i;
        for (int j = 0; primes[j] <= n / i; j ++) {
            st[primes[j] * i] = 1;
            if (i % primes[j] == 0) break;
        }
    }
}

void solve() {
    cin >> n;
    for (int i = 1; i <= n; i ++) {
        cin >> a[i];
    }
    int flag = 0;
    mp.clear();
    memset(cnt_prime, 0, (cnt + 2) * sizeof(int));
    for (int i = 1; i <= n; i ++) {
        for (int j = 0; j < cnt && primes[j] <= a[i]; j ++) {
            if (a[i] % primes[j] == 0) {
                cnt_prime[j] ++;
                while (a[i] % primes[j] == 0) a[i] /= primes[j];
            }
        }
        if (a[i] > 1) {
            if (mp[a[i]]) {
                flag = 1;
                break;
            }
            mp[a[i]] = 1;
        }
    }
    for (int i = 0; i < cnt; i ++) {
        if (cnt_prime[i] > 1) {
            flag = 1;
            break;
        }
    }
    cout << (flag ? "YES" : "NO") << '\n';
}

bool Med;
signed main() {
    fprintf(stderr, "%.3lf MB\n", (&Med - &Mbe) / 1048576.0);
    // setIO();
    int T = 1;
    cin >> T;
    getPrimes(3.4e4);
    while (T --) solve();
    cerr << 1e3 * clock() / CLOCKS_PER_SEC << " ms\n";
    return 0;
}
```
