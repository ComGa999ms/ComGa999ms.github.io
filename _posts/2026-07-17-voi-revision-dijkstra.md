---
title: "VOI Revision - Dijkstra"
date: 2026-07-17 20:27:00 +0700
categories: [Competitive Programing, VOI Revision - Dijkstra]
tags: [cp, graph, dijkstra]
math: true
image:
  path: /assets/img/posts/voi-revision-dijkstra.png
  alt: VOI Revision - Dijkstra
---

Link contest: [https://oj.vnoi.info/contest/vr_dijkstra](https://oj.vnoi.info/contest/vr_dijkstra)

## SGraph

### Tóm tắt

Cho đồ thị gồm n đinh, m cạnh. Với mỗi đỉnh $i$ tính độ dài ngắn nhất từ $1 \to i$ và sau đó từ $i \to 1$. Vói $1 \leq n, m \leq 10^{5}, w \leq 10^{9}$.

### Ý tưởng

Bài đầu tiên thì để tính tổng đường đi từ $1 \to i \to 1$ thì chiều đầu tiên khá dễ nó là bài dijkstra cơ bản, còn chiều ngược lại thì ý tưởng tương tự nhưng dijkstra trên đồ thị ngược chiều với đồ thị gốc. 

### Code AC

```cpp
#include <bits/stdc++.h>
#define pii pair <int, int>
#define int long long
#define fi first
#define se second
#define endl "\n"

using namespace std;

const int N = 1e6 + 5;
const int mod = 1e9 + 7;
const int inf = 1e18 + 7;

int n, m;
int d[N], rd[N];
vector <pii> g[N], rg[N];

void dijk(int sta, int d[], vector <pii> adj[]) {
    for (int i = 1; i <= n; ++i) d[i] = inf;
    d[sta] = 0;

    priority_queue <pii, vector <pii>, greater <pii>> q;
    q.push(pii(0, sta));

    while (q.size()) {
        int u = q.top().se, du = q.top().fi;
        q.pop();

        for (auto [v, uv] : adj[u]) {
            if (d[v] > d[u] + uv) {
                d[v] = d[u] + uv;
                q.push(pii(d[v], v));
            }
        }
    }
}

void solve() {
    cin >> n >> m;
    for (int i = 1; i <= m; ++i) {
        int u, v, c;
        cin >> u >> v >> c;

        g[u].push_back(pii(v, c));
        rg[v].push_back(pii(u, c));
    }

    dijk(1, d, g);
    dijk(1, rd, rg);

    for (int i = 2; i <= n; ++i) {
        if (d[i] == inf || rd[i] == inf) cout << -1 << endl;
        else cout << d[i] + rd[i] << endl;
    }
}

signed main() {
    ios_base::sync_with_stdio(0); cin.tie(0); cout.tie(0);
    solve();
    return 0;
}
```

## PColor
### Tóm tắt
Cho $1 \leq n, C_{i} \leq 10^5$ và $1 \leq L, R, C \leq 10^9$ điểm từ $1$ đến $n$ trên trục $Ox$. Điểm thứ $i$ có màu $C_{i}$. Tại điểm $i$ có thể:

- Đi tới điểm $i + 1$ (nếu $i \neq n$), tốn $R$ giây.

- Đi tới điểm $i - 1$ (nếu $i \neq 1$), tốn $L$ giây.

- Tốc biến tới điểm $j$ (nếu $C_{i} = C_{j}$), tốn $C$ giây.

Cho 2 điểm $st$ và $en$, hãy tính thời gian ngắn nhất đi từ $st$ đến $en$. 
### Ý tưởng

Đối với bài này thì 2 điều kiện đầu khá dễ chỉ đơn giản là build cạnh theo như đề bài yêu cầu. Bài này khó ở điều kiện 3 vì nếu build như theo đề thì sẽ tốn đâu đó $n^2$ cạnh với TH là toàn $C_{i}$ có cùng một giá trị. 

Ý tưởng bài này là đối với các đỉnh trùng giá trị $i$ thì mình sẽ build một đỉnh ảo $hub_i$. Thì lúc này thay vì nối từ $i \to j$ thì mình sẽ nối như sau:

- $i \to hub_i$ trọng số là $C$ 
- $hub_i \to i$ trọng số là $0$ 
- $j \to hub_i$ trọng số là $C$ 
- $hub_i \to j$ trọng số là $0$ 

Thế thì lúc này thay vì đi từ $i \to j$ trọng số là $C$. Thì lúc này từ $i$ sẽ đi qua $hub_i$ trọng số là $C$ và đi qua $j$ trọng số là $0$. Làm như này với stress test ban đầu ví dụ thì sẽ tạo đâu đó $2n$ cạnh. Thì vẫn có thể AC được.

### Code AC

```cpp
#include <bits/stdc++.h>
#define pii pair <int, int>
#define int long long
#define fi first
#define se second
#define endl "\n"

using namespace std;

const int N = 4e5 + 5;
const int mod = 1e9 + 7;
const int inf = 1e18 + 7;

int n;
int L, R, C, sta, en;
int c[N], d[N], hub[N];

vector <pii> adj[N];

void Add(int u, int v, int w) {
    adj[u].push_back(pii(v, w));
}

void solve() {
    cin >> n >> L >> R >> C >> sta >> en;

    int mx = -inf;
    for (int i = 1; i <= n; ++i) {
        cin >> c[i];
        hub[c[i]] = n + c[i];
        mx = max(mx, c[i] + n);
    }   
    
    for (int i = 1; i < n; ++i) Add(i, i + 1, R);
    for (int i = 2; i <= n; ++i) Add(i , i - 1, L);

    for (int i = 1; i <= n; ++i) {
        int h = hub[c[i]];
        Add(i, h, C);
        Add(h, i, 0);
    }

    priority_queue <pii, vector<pii>, greater<pii>> q;
    for (int i = 0; i <= mx + 1; ++i) d[i] = inf;
    d[sta] = 0;
    q.push(pii(0, sta));

    while (q.size()) {
        auto [du, u] = q.top(); q.pop();
        if (du != d[u]) continue;
        
        for (auto [v, w] : adj[u]) {
            if (d[v] > d[u] + w) {
                d[v] = d[u] + w;
                q.push(pii(d[v], v));
            }
        }
    }
    cout << d[en];
}   

signed main() {
    ios_base::sync_with_stdio(0); cin.tie(0); cout.tie(0);
    solve();
    return 0;
}
```

## Beuty Roads

### Tóm tắt

Cho đồ thị vô hướng gồm $n$ đỉnh, $m$ cạnh. Mỗi cạnh có độ dài $l$ và độ đẹp $c$. Tìm đường đi từ đỉnh $1$ đến đỉnh $n$ sao cho tổng độ dài là nhỏ nhất. Nếu có nhiều đường đi cùng độ dài ngắn nhất, chọn đường có tổng độ đẹp lớn nhất.

In ra tổng độ dài ngắn nhất $L$ và tổng độ đẹp lớn nhất $B$. Nếu không tồn tại đường đi từ $1$ tới $n$ thì in $-1$. Với $1 \le n, m \le 2 \cdot 10^5$, $1 \le l, c \le 10^9$.

### Ý tưởng

Bài này dijkstra như bình thường nhưng ngoài mảng $d_u$ lưu độ dài ngắn nhất tới $u$, ta lưu thêm $beauty_u$ là tổng độ đẹp lớn nhất trong các đường đi có độ dài $d_u$.

Khi xét cạnh $u \to v$:

- Nếu tìm được đường có độ dài nhỏ hơn $d_v$ thì cập nhật cả $d_v$ và $beauty_v$.
- Nếu độ dài mới bằng $d_v$ nhưng có tổng độ đẹp lớn hơn $beauty_v$ thì chỉ cập nhật $beauty_v$.

Cuối cùng in ra $d_n$ là độ dài ngắn nhất và $beauty_n$ là tổng độ đẹp lớn nhất.

### Code AC

```cpp
#include <bits/stdc++.h>
#define pii pair <int, int>
#define p2i pair <int, pii>
#define int long long
#define fi first
#define se second
#define endl "\n"

using namespace std;

const int N = 1e6 + 5;
const int mod = 1e9 + 7;
const int inf = 1e18 + 7;

int n, m;
vector <p2i> adj[N];
int dis[N], beauty[N];

void solve() {
    cin >> n >> m;
    for (int i = 1; i <= m; ++i) {
        int u, v, l, c;
        cin >> u >> v >> l >> c;

        adj[u].push_back(p2i(l, pii(c, v)));
        adj[v].push_back(p2i(l, pii(c, u)));
    }

    for (int i = 1; i <= n; ++i) {
        dis[i] = inf;
        beauty[i] = -1;
    }

    priority_queue<pii, vector<pii>, greater<pii>> q;

    dis[1] = 0;
    beauty[1] = 0;
    q.push(pii(0, 1));

    while (q.size()) {
        int du = q.top().fi, u = q.top().se;
        q.pop();

        if (du > dis[u]) continue;
        for (int i = 0; i < adj[u].size(); ++i) {
            int l = adj[u][i].fi;
            int c = adj[u][i].se.fi;
            int v = adj[u][i].se.se;

            int new_d = dis[u] + l;
            int new_beauty = beauty[u] + c;

            if (new_d < dis[v]) {
                dis[v] = new_d;
                beauty[v] = new_beauty;
                q.push(pii(dis[v], v));
            }
            else if (new_d == dis[v] && new_beauty > beauty[v]) {
                beauty[v] = new_beauty;
                q.push(pii(dis[v], v));
            }
        }
    }

    if (dis[n] == inf) cout << -1;
    else cout << dis[n] << " " << beauty[n];
}

signed main() {
    ios_base::sync_with_stdio(0); cin.tie(0); cout.tie(0);
    solve();
    return 0;
}
```

## Nhà Máy Điện

### Tóm tắt

Cho đồ thị vô hướng gồm $n$ thành phố, $m$ con đường. Mỗi con đường có độ dài bằng $1$. Có $k$ dự án nhà máy điện, dự án thứ $i$ đặt nhà máy tại thành phố $p_i$ và có thể cung cấp điện cho các thành phố cách nó không quá $r_i$ cạnh.

In ra xâu nhị phân độ dài $n$, ký tự thứ $i$ là $1$ nếu thành phố $i$ được cấp điện, ngược lại in $0$. Với $1 \le n, m \le 2 \cdot 10^5$, $0 \le k \le 5 \cdot 10^5$.

### Ý tưởng

Bài này thì không thể BFS từng nhà máy vì số nhà máy nhiều và sẽ bị TLE.

Ý tưởng bài này thì xem bán kính $r_i$ là trọng số ban đầu của nhà máy. Mỗi lần đi qua một cạnh thì trọng số giảm $1$. Bài này dijkstra từ các nhà máy và duy trì luôn xét thành phố đang có trọng số còn lại lớn nhất. Gọi $P_u$ là lượng trọng số lớn nhất còn lại khi có thể đi tới thành phố $u$.

- Ban đầu với mỗi dự án $(p_i, r_i)$, gán $P_{p_i} = \max(P_{p_i}, r_i)$.

- Khi từ $u$ sang đỉnh kề $v$, trọng số còn lại là $P_u - 1$.

- Nếu $P_u - 1 > P_v$ thì cập nhật $P_v$ và đưa $v$ lại vào dijkstra tiếp.

Cuối cùng thì thành phố $i$ được cấp điện khi $P_i \ge 0$.

### Code AC

```cpp
#include <bits/stdc++.h>
#define pii pair <int, int>
#define p2i pair <int, pii>
#define int long long
#define fi first
#define se second
#define endl "\n"

using namespace std;

const int N = 1e6 + 5;
const int mod = 1e9 + 7;
const int inf = 1e18 + 7;

int n, m, k;
vector <int> adj[N];
int P[N];

void solve() {
    cin >> n >> m >> k;
    for (int i = 1; i <= m; ++i) {
        int u, v;
        cin >> u >> v;

        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    for (int i = 1; i <= n; ++i) P[i] = -1;

    priority_queue<pii> q;
    for (int i = 1; i <= k; ++i) {
        int p, r;
        cin >> p >> r;

        if (r > P[p]) {
            P[p] = r;
            q.push(pii(r, p));
        }
    }

    while (q.size()) {
        int cur = q.top().fi, u = q.top().se;
        q.pop();

        if (cur > P[u] || cur == 0) continue;
        for (int v : adj[u]) {
            if (P[v] < cur - 1) {
                P[v] = cur - 1;
                q.push(pii(P[v], v));
            }
        }
    }

    for (int i = 1; i <= n; ++i) {
        if (P[i] >= 0) cout << 1;
        else cout << 0;
    }
}

signed main() {
    ios_base::sync_with_stdio(0); cin.tie(0); cout.tie(0);
    solve();
    return 0;
}
```

## Tunnel

### Tóm tắt

Cho đồ thị vô hướng có trọng số. Buổi sáng Aaron đi từ $A$ đến $B$ theo một đường đi có tổng chi phí nhỏ nhất và kích hoạt các cạnh trên đường đi đó. Buổi chiều Hasun đi từ $C$ đến $D$. Những cạnh Aaron đã kích hoạt thì Hasun đi qua miễn phí, các cạnh khác vẫn phải trả chi phí.

Tìm chi phí nhỏ nhất Hasun cần trả. Với $2 \le n \le 10^5$, $1 \le m \le 2 \cdot 10^5$.


### Ý tưởng

Bài này đầu tiên là dijkstra từ $A, B, C, D$ để có các mảng $d_{i}$ tương ứng. Ta có nhận xét là một cạnh $u \to v$ nằm trên một đường đi ngắn nhất từ $A$ đến $B$ khi:
$$
d_{A}​[u] + w(u, v) + d_{B}​[v] = d_{A}​[B]
$$
Các cạnh thỏa điều kiện này tạo thành DAG theo thứ tự tăng của $d_A$. Gọi $f_v$ là chi phí nhỏ nhất để đi từ $C$ tới một đỉnh nào đó trên đường ngắn nhất, sau đó đi miễn phí theo các cạnh đã kích hoạt để tới $v$.

- Ban đầu: $f_v = d_C[v]$.

- Với cạnh $u \to v$ thuộc DAG: $f_{v} ​ = min(f_{v}​,f_{u}​)$

Khi đang ở $v$ thì Hasun có thể đi tiếp tới $D$ với chi phí $d_D[v]$, nên xét $f_{v} + d_{D}[v]$. Cần làm thêm chiều ngược lại, vì Hasun có thể đi miễn phí ngược hướng với đường Aaron đi. Tức là đổi vai trò $C$ và $D$ rồi tính tương tự.

### Code AC

```cpp
#include <bits/stdc++.h>
#define pii pair <int, int>
#define p2i pair <int, pii>
#define int long long
#define fi first
#define se second
#define endl "\n"

using namespace std;

const int N = 1e6 + 5;
const int mod = 1e9 + 7;
const int inf = 1e18 + 7;

int n, m;
int A, B, C, D;

vector<pii> adj[N];

int dA[N], dB[N], dC[N], dD[N];
int f[N], g[N];

void dijk(int s, int d[]) {
    for (int i = 1; i <= n; ++i) d[i] = inf;
    priority_queue<pii, vector<pii>, greater<pii>> q;
    d[s] = 0;
    q.push(pii(0, s));

    while (q.size()) {
        int du = q.top().fi, u = q.top().se;
        q.pop();

        if (du > d[u]) continue;
        for (auto [v, uv] : adj[u]) {
            if (d[v] > d[u] + uv) {
                d[v] = d[u] + uv;
                q.push(pii(d[v], v));
            }
        }
    }
}

void solve() {
    cin >> n >> m >> A >> B >> C >> D;

    for (int i = 1; i <= m; ++i) {
        int u, v, w;
        cin >> u >> v >> w;
        adj[u].push_back(pii(v, w));
        adj[v].push_back(pii(u, w));
    }

    dijk(A, dA);
    dijk(B, dB);
    dijk(C, dC);
    dijk(D, dD);

    vector <int> ord;
    for (int i = 1; i <= n; ++i) {
        ord.push_back(i);
        f[i] = dC[i];
        g[i] = dD[i];
    }

    sort(ord.begin(), ord.end(), [&](int u, int v) {
        return dA[u] < dA[v];
    });

    int ans = dC[D];
    for (int u : ord) {
        ans = min(ans, f[u] + dD[u]);
        ans = min(ans, g[u] + dC[u]);

        for (auto [v, c] : adj[u]) {
            // u -> v nam tren duong di ngan nhat A -> B
            if (dA[u] + c + dB[v] == dA[B]) {
                f[v] = min(f[v], f[u]);
                g[v] = min(g[v], g[u]);
            }
        }
    }

    cout << ans;
}

signed main() {
    ios_base::sync_with_stdio(0); cin.tie(0); cout.tie(0);
    solve();
    return 0;
}
```
