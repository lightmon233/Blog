---
title: "2024江苏省大学生程序设计竞赛JSCPC题解（E、I、K、H）"
date: 2024-05-24T11:13:01+08:00
draft: false
description: ""
---
{{< katex >}}


## E. Divide

首先，将一个数字\(x\)不断进行\(/2\)操作最终会变成\(0\), 这个操作只会执行\(log(x)\)次。因此可以将原数组\(a_1, a_2, ..., a_n\)分解为\(a_1, a_1/2, a_1/4, ..., 0, a_2, a_2/2, a_2/4, ..., 0, ..., a_n, a_n/2, a_n/4, ..., 0\)。

考虑将每个数的下标作为第一维度，其真实数值作为第二维度。则以上数组再转化为\((1, a_1), (1, a_1/2), ..., (1, a_1/4), (1, 0), (2, a_2), (2, a_2/2), (2, a_2/4), ..., (2, 0), ..., (n, a_n), (n, a_n/2), (n, a_n/4), ..., (n, 0)\)。

可以发现，求\([l, r]\)中每次将最大值\(/2\)，执行\(k\)次后的最大值等价于求上述二维度数组的第二维度的第\(k + 1\)大值（可重集，即倒序排完序后的第\(k + 1\)个元素），这可以用主席树维护。

设\(V\)为原数组\(a\)中的最大元素的值，即\(1e5\), 则时间复杂度为\(O(nlog^2V)\)。

```cpp
int n, m;
int a[N];
struct Node {
    int lson, rson;
    int cnt;
}tr[N * 17 * 18];
int root[N];
int c[N];
int idx;
 
int insert(int p, int l, int r, int x) {
    int q = ++idx;
    tr[q] = tr[p];
    if (l == r) {
        tr[q].cnt ++;
        return q;
    }
    int mid = l + r >> 1;
    if (x <= mid) tr[q].lson = insert(tr[p].lson, l, mid, x);
    else tr[q].rson = insert(tr[p].rson, mid + 1, r, x);
    tr[q].cnt = tr[tr[q].lson].cnt + tr[tr[q].rson].cnt;
    return q;
}
 
int query(int q, int p, int l, int r, int k) {
    if (l == r) return l;
    int cnt = tr[tr[q].lson].cnt - tr[tr[p].lson].cnt;
    int mid = l + r >> 1;
    if (k <= cnt) return query(tr[q].lson, tr[p].lson, l, mid, k);
    else return query(tr[q].rson, tr[p].rson, mid + 1, r, k - cnt);
}
 
void solve() {
    cin >> n >> m;
    for (int i = 1; i <= n; i ++) {
        cin >> a[i];
    }
    for (int i = 1; i <= n; i ++) {
        c[i] = c[i - 1];
        int flag = 0;
        while (1) {
			// 这里注意要把所有第一维度为i的点都一次性加入到主席树中，并且顺着一个根节点root[i]加入。
            if (flag == 0) {
                root[i] = insert(root[i - 1], 0, 1e5, a[i]);
                flag = 1;
            }
            else {
                root[i] = insert(root[i], 0, 1e5, a[i]);
            }
            c[i] ++;
            if (a[i] == 0) break;
            a[i] = a[i] / 2;
        }
    }
    while (m --) {
        int l, r, k;
        cin >> l >> r >> k;
        if (k > c[r] - c[l - 1]) {
            cout << 0 << '\n';
            continue;
        }
        cout << query(root[r], root[l - 1], 0, 1e5, c[r] - c[l - 1] - k) << '\n';
    }
}
```

## I. Integer Reaction

这题给了个惊醒。看到求最小值最大或者最大值最小一般可以往二分上考虑。虽说知道这一点但是赛时一直认为不能二分。。。

考虑二分时check里面是判断操作后集合\(S_2\)中的最小元素能否\(>=x\)或者\(<=x\)。

如果是\(>=\)，则二分序列如下所示：

![image](/img/cnblogs/3349274-20240525135141250-744106107.png)

如果是\(<=\)，则二分序列如下所示：

![image](/img/cnblogs/3349274-20240525135842123-1740741632.png)

至于check内部的贪心，我们只需在每次\(c[i]\)与\(S_1\)中颜色不同的时候，贪心地选择\(S_1\)中\(>=x-a[i]\)的最小元素与其反应即可。因为这样可以尽可能地保证最后最小值\(>=x\)并且\(S_1\)中较大的元素可以留给后面的元素与之反应。

```cpp
int n;
int a[N];
int c[N];
 
bool check(int x) {
    multiset<PII> s;
    for (int i = 1; i <= n; i ++) {
        if (!s.empty() && c[i] != (*s.begin()).second) {
            auto pos = s.lower_bound(PII(x - a[i], -1));
            if (pos == s.end()) return false;
            else s.erase(pos);
        }
        else s.insert({a[i], c[i]});
    }
    return true;
}
 
void solve() {
    cin >> n;
    for (int i = 1; i <= n; i ++) {
        cin >> a[i];
    }
    for (int i = 1; i <= n; i ++) {
        cin >> c[i];
    }
    int l = 2, r = 2e8;
    while (l < r) {
        int mid = l + r + 1 >> 1;
        if (check(mid)) l = mid;
        else r = mid - 1;
    }
    cout << l << '\n';
}
```

## K. Number Deletion Game

考虑什么情况下当前局面者必赢。如果当前只有一个数，那么当前局面者可以选择\(y = 0\)，这样数组中删除掉这一个数，并且不会有新的数字加入，当前局面者就必赢。

因为每次都只能选\(x\)为序列最大值，不妨考虑最大值，每次操作后最大值为多少，最后可以使最大值变成\(0\)，即将数组清空的玩家获胜。

如果当前最大值\(x\)数量是奇数，如果最大值数量\(>1\)，那么我可以选择\(y = 0\)使得最大值\(x\)数量变成偶数，把这个偶数的局面丢给对方玩家。如果最大值数量只有\(1\)，那么我可以根据次大值的数量来选择\(y\)，如果次大值有奇数个，我就选择\(y = x - 1\)，如果次大值有偶数个，我就选择\(y != x - 1\)，总是可以把最大值数量为偶数的局面丢给对方玩家。

而对方玩家由于只能删除一个\(x\)，所以他丢给我的永远是最大值数目为奇数的局面。

故如果先手最大值数目为奇数，他必胜；否则必输。

```cpp
int n;
int a[(int)1e3 + 10];

void solve() {
    cin >> n;
    int max_v = 0;
    for (int i = 1; i <= n; i ++) {
        cin >> a[i];
        max_v = max(max_v, a[i]);
    }
    int cnt = 0;
    for (int i = 1; i <= n; i ++) {
        if (a[i] == max_v) cnt ++;
    }
    if (cnt & 1) cout << "Alice\n";
    else cout << "Bob\n";
}
```

## H. Real Estate Is All Around

小蓝卖出一套房子不贬值，小红卖出一套房子贬值1元，小绿卖出一套房子贬值\(\lceil \frac{a_i}{10} \rceil\)元，那么对于所有的房子，我们最后应该是让他们尽量全被小蓝卖出，如果不能全被小蓝卖出，再抽取部分让小红卖出，如果不能全被小红、小蓝卖出，再抽取部分让小绿卖出（卖出是指先存在助理手里并且最终会被卖掉）。

如果所有的房子再被小红、小蓝小绿卖出部分后，还剩一部分，那么我们其实可以把这剩下的一部分房子都存在小绿手中，因为放在小红或者小蓝手中的话可能会影响他们的卖房策略，但是放在小绿手里的话，因为小绿每次买房狂潮时会优先卖出自己手中房子中价值最大的，所以新放入的剩下的房子再小绿手中并不会被卖掉，因此不会影响小绿的卖房策略。

我们可以把不卖出理解为贬值\(a_i\)，这样每个房子都会有贬值，求最大利润可以转化为求\(a\)的总和\(-\)最小贬值的和。

因为每个房子都会有自己的去向，不管贬值多少肯定是都要减去一个贬值的，如果不减去贬值的话就不符合题意。所以本问题就可以转化成一个最小费用最大流的模型了。

一种建图策略是：
1. 建立源点\(S\)和汇点\(T\)，建立所有房子，\(S\)向所有房子连一条容量为\(1\)，费用为\(0\)的边，表示每个房子最多被卖\(1\)次；每个房子向\(T\)连一条容量为\(1\)，费用为\(a_i\)的边，表示每个房子如果不卖的话贬值为\(a_i\)。这里建立了\(O(n)\)个点，\(O(2n)\)条边。
2. 建立每个助理的分时点\((i, j)\)，其中\(i\)代表当前助理所在时间点，\(j\)代表当前助理是谁，对每个\((i, j)\)，向\((i + 1, j)\)连一条容量为\(+ \infty\)，费用为\(0\)的边，表示这个助理手中的所有房子可以顺着时间推移传到下一个时间点。这里建立了\(O(3n)\)个点，\(O(3n)\)条边。
3. 对每次储存机会，从\(a_i\)房子向\((i, 1)\)、\((i, 2)\)、\((i, 3)\)连接容量都为\(1\)、费用分别为\(1\)，\(\lceil \frac{a_i}{10} \rceil\)、\(0\)的边，表示这个房子最多只能由一个助理卖掉，贬值分别为\(1\)或\(\lceil \frac{a_i}{10} \rceil\)或\(0\)。这里建立了\(O(3n)\)条边。
4. 对每次买房狂潮，对当前时刻的每个助理\((i, j)\)向\(T\)连一条容量为\(1\)、费用为\(0\)的边，表示每个助理可以卖出最多一个房子。这里建了\(O(3n)\)条边。

最终点开\(4n\)个，边开\(11n\)个即可。

注意，虽然我们策略里面是不卖的房子交给小绿，但是实际上建图时是把不卖的房子连向了\(T\)，因为我们通过这样的网络流算出的策略中直接给\(T\)的房子，在我们实际操作中，可以给小绿，所以这样连边其实和连向小绿其实是等价的。

```cpp
const int N = 210 * 4, M = N * 11;

int n;
int a[N];
int h[N], e[M * 2], ne[M * 2], f[M * 2], w[M * 2], idx;
int q[N], incf[N], d[N], pre[N], st[N];
int S, T;

void addEdge(int a, int b, int c, int d) {
    e[idx] = b, f[idx] = c, w[idx] = d, ne[idx] = h[a], h[a] = idx ++;
    e[idx] = a, f[idx] = 0, w[idx] = -d, ne[idx] = h[b], h[b] = idx ++;
}

int getID(int x, int y) {
    return 3 * (x - 1) + y;
}

bool spfa() {
    int hh = 0, tt = 0;
    memset(d, 0x3f, sizeof d);
    memset(incf, 0, sizeof incf);
    memset(st, 0, sizeof st);
    q[tt ++] = S, d[S] = 0, incf[S] = INF;
    while (hh != tt) {
        int t = q[hh ++];
        if (hh == N) hh = 0;
        st[t] = 0;
        for (int i = h[t]; ~i; i = ne[i]) {
            int ver = e[i];
            if (f[i] && d[ver] > d[t] + w[i]) {
                d[ver] = d[t] + w[i];
                pre[ver] = i;
                incf[ver] = min(incf[t], f[i]);
                if (!st[ver]) {
                    q[tt ++] = ver;
                    if (tt == N) tt = 0;
                    st[ver] = 1;
                }
            }
        }
    }
    return incf[T] > 0;
}

void EK(int &flow, int &cost) {
    flow = cost = 0;
    while (spfa()) {
        int t = incf[T];
        flow += t, cost += t * d[T];
        for (int i = T; i != S; i = e[pre[i] ^ 1]) {
            f[pre[i]] -= t;
            f[pre[i] ^ 1] += t;
        }
    }
}

void solve() {
    cin >> n;
    memset(h, -1, sizeof h);
    idx = 0;
    int cnt = 0;
    int tot = 3 * n;
    S = ++tot, T = ++tot;
    int sum = 0;
    for (int i = 1; i <= n - 1; i ++) {
        for (int j = 1; j <= 3; j ++) {
            addEdge(getID(i, j), getID(i + 1, j), INF, 0);
        }
    }
    for (int i = 1; i <= n; i ++) {
        int op;
        cin >> op;
        if (op == 1) {
            cin >> a[++cnt];
            sum += a[cnt];
            addEdge(S, ++tot, 1, 0);
            addEdge(tot, T, 1, a[cnt]);
            addEdge(tot, getID(i, 1), 1, 1);
            addEdge(tot, getID(i, 2), 1, (a[cnt] + 9) / 10);
            addEdge(tot, getID(i, 3), 1, 0);
        }
        else {
            for (int j = 1; j <= 3; j ++) {
                addEdge(getID(i, j), T, 1, 0);
            }
        }
    }
    int flow, cost;
    EK(flow, cost);
    cout << sum - cost << '\n';
}
```