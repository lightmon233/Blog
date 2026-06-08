---
title: "Codeforces Round 949题解（A、B、C、D）"
date: 2024-06-04T08:41:01+08:00
draft: false
description: ""
---
{{< katex >}}


## A. Turtle and Piggy Are Playing a Game

首先\(p\)选\(2\)的话除得最慢，得的分多。考虑二进制表示，如果\(x = (1000000000)_{bin}\)，则每次除以\(2\)都是相当于右移一位，除完之后仍然是\(2\)的倍数，变成\(1\)的步数就是把最高位的\(1\)移动到\(0\)位的步数。

因为\(2l \le r\)，所以\(l \le \lfloor \frac{r}{2} \rfloor\)，若\(r = (1011101)_{bin}\)，则\(r >> 1 = (101110)_{bin}\)，可以发现\((1000000)_{bin}\)一定是在\([l, r]\)区间内部，因此我们可以直接选择\(x = 1 << (lg(r))\)。

```cpp
int l, r;
 
void solve() {
    cin >> l >> r;
    cout << lg(r) << '\n';
}
```

## B. Turtle and an Infinite Sequence

写下几组数据发现第\(m\)次改变后\(a_n\)的值变为\([n - m, n + m]\)这个区间内所有\(a_i\)的按位或的值。

令\(l = n - m\)，\(r = n + m\)。

```
l: 01011 0 11011
r: 01011 1 00110
```
设从左到右第一个不相等的位为\(bit\)，则\(ans\)的\(bit\)位左边的值就是\(l\)或\(r\)的这个位的值。对\(bit\)位及其右边的位，发现`01011 0 11111`在\([l, r]\)内，因此后面的位每一位或出来都是\(1\)，则加上\((1 << (bit + 1)) - 1\)即可。

```cpp
int n, m;
 
void solve() {
    cin >> n >> m;
    int l = max(0, n - m);
    int r = n + m;
    int ans = 0;
    for (int i = 31; i >= 0; i --) {
        if ((l >> i & 1) != (r >> i & 1)) {
            ans += (1 << (i + 1)) - 1;
            break;
        }
        else {
            ans |= (l >> i & 1) << i;
        }
    }
    cout << ans << '\n';
}
```

## C. Turtle and an Incomplete Sequence

考虑所有非\(-1\)的位置，我们要保证这些位置中每相邻的两个位置上的数要在他们的间隔允许的步数范围内实现相互转化。

比如\(3, -1, -1, -1, 9, -1\)，我们要保证\(3\)可以在\(4\)步之内转化到\(9\)，通过\(/2\)或者\(*2\)或者\(*2+1\)的方式。

既然如此，我们可以考虑最少需要多少步，可以将\(a\)转化成\(b\)。

```
3: 0011
9: 1001
```

首先我们可以把\(3\)的前缀一给删掉。

```
3: 1 1
9: 1 0 01
```

然后，这些操作可以抽象为二进制序列右移一位、左移一位补\(0\)、左移一位补\(1\)三种。

因为操作都是从尾部进行的，所以\(3\)和\(9\)的公共前缀部分可以不用考虑，从第一个不相同的位开始后面的所有位都会被操作（需要或者不需要修改的位都会在修改者第一个不相同的位的时候被顺便操作到），所以这个最小步数就是\(3\)和\(9\)的第一个不相同的位及其后面的位数之和的和。

因为从\(a\)变成\(b\)，位数的变化是一定的，二每个操作都是增加一个位或者减少一个位，所以所有操作方案都和最小方案具有相同的奇偶性。

因此我们只要判断每两个非\(-1\)位置的间距是否大于等于最小方案并且与其具有相同奇偶性即可。

对于第一个非\(-1\)位置的左边和最后一个非\(-1\)位置的右边，我们连续地\(*2\)，\(/2\)即可。

```cpp
int n;
int a[M];
int b[M];

int get_times_l(int x, int y) {
    int l = lg(x), r = lg(y);
    int i, j;
    for (i = l, j = r; i >= 0 && j >= 0; i --, j --) {
        if ((x >> i & 1) != (y >> j & 1)) {
            return i + 1;
        }
    }
    return i + 1;
}

int get_times_r(int x, int y) {
    int l = lg(x), r = lg(y);
    int i, j;
    for (i = l, j = r; i >= 0 && j >= 0; i --, j --) {
        if ((x >> i & 1) != (y >> j & 1)) {
            return j + 1;
        }
    }
    return j + 1;
}

void solve() {
    cin >> n;
    for (int i = 1; i <= n; i ++) {
        b[i] = 1;
    }
    for (int i = 1; i <= n; i ++) {
        cin >> a[i];
    }
    for (int i = 1; i <= n; i ++) {
        if (a[i] != -1) {
            b[i] = a[i];
            int j;
            for (j = i + 1; j <= n; j ++) {
                if (a[j] != -1) {
                    break;
                }
            }
            if (j > n) {
                break;
            }
            int l = get_times_l(a[i], a[j]);
            int r = get_times_r(a[i], a[j]);
//            cout << l << ' ' << r << '\n';
//            cout << i << ' ' << j << '\n';
            if (j - i >= l + r && (j - i - l - r) % 2 == 0) {
                for (int k = i + 1; k <= i + l; k ++) {
                    b[k] = b[k - 1] / 2;
                }
                int rr = r;
                for (int k = i + l + 1; k <= i + l + r; k ++) {
                    b[k] = b[k - 1] * 2 + ((a[j] >> (rr-- - 1)) & 1);
                }
                int cnt = 0;
                for (int k = i + l + r + 1; k < j; k ++) {
                    if (cnt++ & 1) b[k] = b[k - 1] / 2;
                    else b[k] = b[k - 1] * 2;
                }
            }
            else {
                cout << "-1\n";
                return;
            }
            i = j - 1;
        }
    }
    int pos_l = -1, pos_r = -1;
    for (int i = 1; i <= n; i ++) {
        if (a[i] != -1) {
            pos_l = i;
            break;
        }
    }
    for (int i = n; i >= 1; i --) {
        if (a[i] != -1) {
            pos_r = i;
            break;
        }
    }
    int cnt = 0;
    if (pos_l != -1) {
        for (int i = pos_l - 1; i >= 1; i --) {
            if (cnt++ & 1) b[i] = b[i + 1] / 2;
            else b[i] = b[i + 1] * 2;
        }
    }
    cnt = 0;
    if (pos_r != -1) {
        for (int i = pos_r + 1; i <= n; i ++) {
            if (cnt++ & 1) b[i] = b[i - 1] / 2;
            else b[i] = b[i - 1] * 2;
        }
    }
    if (pos_l == -1 && pos_r == -1) {
        b[1] = 1;
        cnt = 0;
        for (int i = 2; i <= n; i ++) {
            if (cnt++ & 1) b[i] = b[i - 1] / 2;
            else b[i] = b[i - 1] * 2;
        }
    }
    for (int i = 1; i <= n; i ++) {
        cout << b[i] << ' ';
    }
    cout << '\n';
}
```

## D. Turtle and Multiplication

\(a_i \cdot a_{i+1} = a_j \cdot a_{j+1}\)的必要条件是无序对\((a_i, a_{i+1})\)和无序对\(a_j, a_{j+1}\)相同。实际上，当\(a_i\)都是质数时，这个必要条件会变成充要条件。如果两个质数对中有一个质数是相同的，那么乘积相同则另一个质数也是相同的；如果两个质数对中的质数两两不同，由分解质因数可知他们的乘积为\(2^a 3^b 5^c ...\)，其中\(a, b, c, ...\)中只能有且只有两个为1，其他全为0，那么排列组合的话也就只有这一种组合，故无序对相同是充要条件。

我们可以把\((a_i, a_{i+1})\)看作是一条边，则问题变为找到点数最少得无向完全图（每个点还有一个自环），是的这个完全图存在一条经过\(n - 1\)条边且不经过重复边的路径。

考虑若完全图点数确定，我们如何计算这个完全图的最长不经过重复边的路径长度。

设完全图点数为\(m\)，若\(m\)是奇数则每个点的度数都是偶数，所以这个图存在欧拉路径，路径长度等于边数等于\(\frac{m(m+1)}{2}\)。

若\(m\)是偶数那么每个点的度数都是奇数，我们需要删除一些边使得这个图存在欧拉路径。可以发现删掉一条边最多可使奇度数的点的数量减少\(2\)，所以我们至少需要删除\(\frac{m}{2} - 1\)条边。发现有一种方案：删除\((2, 3), (4, 5), ..., (m - 2, m - 1)\)这些边即可。路径长度为\(\frac{m(m-1)}{2} - \frac{m}{2} + 1 + m = \frac{m^2}{2} + 1\)。

当\(n = 10^6\)时最小的\(m\)是\(1415\)，第\(1415\)小的质数是\(11807\)，符合\(a_i \le 3 \cdot 10^5\)。

我们可以二分求出最小的\(m\)，再用Fleury等算法求出一个无向图的欧拉路径。

时间复杂度：每个测试样例\(O(n)\)。

注意，我们在添加欧拉回路的路径时，只把当前边的\(v\)节点（这个变到达的那个点）加到了栈中，所以最后要记得再把\(1\)号节点（初始的起点）也加到栈中。数组大小的\(3N\)是为了防止越界所设，实际上接近\(2N\)即可。

```cpp
int n;
int h[N], e[N * 3], ne[N * 3], idx;
int st[N], primes[N], cnt;
int used[N * 3], tot;
int ans[N * 3];

void getPrimes(int n) {
    for (int i = 2; i <= n; i ++) {
        if (!st[i]) primes[cnt ++] = i;
        for (int j = 0; primes[j] <= n / i; j ++) {
            st[primes[j] * i] = 1;
            if (i % primes[j] == 0) break;
        }
    }
}

void addEdge(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++;
}

bool check(int x) {
    if (x & 1) return x * (x + 1) / 2 >= n - 1;
    else return x * x / 2 + 1 >= n - 1;
}

void dfs(int u) {
    for (int i = h[u]; ~i; i = h[u]) {
        if (used[i]) {
            h[u] = ne[i];
            continue;
        }
        used[i] = 1;
        used[i ^ 1] = 1;
        h[u] = ne[i];
        dfs(e[i]);
        ans[++tot] = e[i];
    }
}

void solve() {
    cin >> n;
    memset(h, -1, (n + 2) * 4);
    idx = 0;
    tot = 0;
    int l = 1, r = 10000;
    while (l < r) {
        int mid = l + r >> 1;
        if (check(mid)) r = mid;
        else l = mid + 1;
    }
    int m = l;
    for (int i = 1; i <= m; i ++) {
        for (int j = i; j <= m; j ++) {
            if (!(m & 1) && j == i + 1 && !(i & 1))
                continue;
            addEdge(i, j);
            addEdge(j, i);
        }
    }
    for (int i = 0; i < idx; i ++) {
        used[i] = 0;
    }
    dfs(1);
    ans[++tot] = 1;
    reverse(ans + 1, ans + 1 + tot);
//    for (int i = 1; i <= tot; i ++) {
//        cout << ans[i] << " \n"[i == tot];
//    }
    for (int i = 1; i <= n; i ++) {
        cout << primes[ans[i] - 1] << " \n"[i == n];
    }
}

bool Med;
int main() {
    fprintf(stderr, "%.3lf MB\n", (&Med - &Mbe) / 1048576.0);
    // setIO();
    int T = 1;
    cin >> T;
    getPrimes(1e6);
//    for (int i = 0; i < 100; i ++) {
//        cout << primes[i] << " \n"[i == 99];
//    }
    while (T --) solve();
    cerr << 1e3 * clock() / CLOCKS_PER_SEC << " ms\n";
    return 0;
}
```