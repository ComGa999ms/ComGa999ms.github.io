---
title: "Lời giải đề thi thử PTNK 2026: HUB05 - HUB08"
date: 2026-05-27 00:00:00 +0700
categories: [Events, PTNK]
tags: [ptnk, hub05, hub06, hub07, hub08, cpp, solution]
---

Bài viết này ghi lại lời giải và code C++ cho bốn bài HUB05, HUB06, HUB07 và HUB08 trong đề thi thử tuyển sinh lớp 10 PTNK 2026.

## HUB05 - Đèn Giáng Sinh

Ta cần tìm một đoạn con sao cho đoạn đó là một hoa văn có độ dài `len` được lặp lại đúng `k` lần. Cách làm trực tiếp là duyệt vị trí bắt đầu `l`, duyệt độ dài hoa văn `len`, rồi kiểm tra `k` khối liên tiếp có giống nhau không.

Nếu tìm được đoạn hợp lệ đầu tiên, in ra `len` và hoa văn tương ứng. Nếu không có đoạn nào thỏa mãn thì in `-1`.

```cpp
#include <bits/stdc++.h>
#define int long long
#define rep(i, l, r) for (int i = l; i <= r; ++i)
#define sz(v) (int)(v).size()
#define pb push_back
#define endl '\n'

using namespace std;

const int N = 55;

int n, k;
int a[N];

bool ok(int l, int len) {
    rep(t, 1, k - 1) {
        rep(i, 0, len - 1) {
            if (a[l + i] != a[l + t * len + i]) return false;
        }
    }
    return true;
}

void solve(void) {
    cin >> n >> k;
    rep(i, 1, n) cin >> a[i];

    rep(l, 1, n) {
        rep(len, 1, n) {
            int r = l + len * k - 1;
            if (r > n) break;

            if (ok(l, len)) {
                cout << len << endl;
                rep(i, l, l + len - 1) cout << a[i] << ' ';
                return;
            }
        }
    }

    cout << -1;
}

signed main() {
    #define task "HUB05"
    ios_base::sync_with_stdio(0); cin.tie(0); cout.tie(0);

    freopen(task".inp", "r", stdin);
    freopen(task".out", "w", stdout);

    solve();
    return 0;
}
```

## HUB06 - Vi khuẩn chia đôi

Ý tưởng là dựng xâu ban đầu bằng cách chia thành nhiều khối nhị phân có cùng độ dài. Ta chọn độ dài khối `block_len = 2^m` nhỏ nhất sao cho số khối `block_cnt = 2^(n-m)` không vượt quá số xâu nhị phân khác nhau có độ dài `block_len`.

Sau đó, với mỗi số từ `0` đến `block_cnt - 1`, viết nó thành chuỗi nhị phân đúng `block_len` bit rồi ghép lại. Cách dựng này tạo ra nhiều khối khác nhau nhất có thể ở mức đang xét.

```cpp
#include <bits/stdc++.h>
#define int long long
#define rep(i, l, r) for (int i = l; i <= r; ++i)
#define endl '\n'

using namespace std;

int n;

string bin(int x, int len) {
    string s;
    for (int i = len - 1; i >= 0; --i) {
        s += char('0' + ((x >> i) & 1));
    }
    return s;
}

void solve(void) {
    cin >> n;

    int m = 0;
    while (m <= n) {
        int block_len = 1LL << m;
        int block_cnt = 1LL << (n - m);
        if (block_cnt <= (1LL << block_len)) break;
        ++m;
    }

    int block_len = 1LL << m;
    int block_cnt = 1LL << (n - m);

    string ans = "";
    rep(i, 0, block_cnt - 1) ans += bin(i, block_len);

    cout << ans;
}

signed main() {
    #define task "HUB06"
    ios_base::sync_with_stdio(0); cin.tie(0); cout.tie(0);

    freopen(task".inp", "r", stdin);
    freopen(task".out", "w", stdout);

    solve();
    return 0;
}
```

## HUB07 - Chuỗi số

Vì các giá trị trùng nhau trên bảng chỉ dùng được một lần, trước hết ta sort và xóa trùng. Gọi `need = m - s`, tức là số giá trị cũ cần xuất hiện trong dãy liên tiếp độ dài `m`.

Mỗi nhóm `need` giá trị liên tiếp trong mảng đã nén sẽ tạo ra một khoảng vị trí bắt đầu hợp lệ `[L, R]`. Sau đó chỉ cần gộp các khoảng giao nhau hoặc kề nhau rồi cộng tổng độ dài.

Trường hợp `need = 0`, mọi dãy liên tiếp độ dài `m` đều hợp lệ, đáp án là `n - m + 1`.

```cpp
#include <bits/stdc++.h>
#define int long long
#define all(v) (v).begin(), (v).end()
#define rep(i, l, r) for (int i = l; i <= r; ++i)
#define sz(v) (int)(v).size()
#define pb push_back
#define endl '\n'

using namespace std;

const int N = 1e5 + 5;

int n, m, s;
int a[N];
int L[N], R[N];

void solve(void) {
    cin >> n >> m >> s;

    vector<int> v;

    rep(i, 1, m) {
        cin >> a[i];
        v.pb(a[i]);
    }

    sort(all(v));
    v.erase(unique(all(v)), v.end());

    int cnt = sz(v);
    rep(i, 1, cnt) a[i] = v[i - 1];

    int need = m - s;
    int lastStart = n - m + 1;

    if (need == 0) {
        cout << lastStart;
        return;
    }

    int dem = 0;

    rep(i, 1, cnt - need + 1) {
        int j = i + need - 1;

        int l = max(1LL, a[j] - m + 1);
        int r = min(a[i], lastStart);

        if (l <= r) {
            ++dem;
            L[dem] = l;
            R[dem] = r;
        }
    }

    int ans = 0;

    if (dem == 0) {
        cout << 0;
        return;
    }

    int curL = L[1], curR = R[1];

    rep(i, 2, dem) {
        if (L[i] > curR + 1) {
            ans += curR - curL + 1;
            curL = L[i];
            curR = R[i];
        } else {
            curR = max(curR, R[i]);
        }
    }

    ans += curR - curL + 1;

    cout << ans;
}

signed main() {
    #define task "HUB07"
    ios_base::sync_with_stdio(0); cin.tie(0); cout.tie(0);

    freopen(task".inp", "r", stdin);
    freopen(task".out", "w", stdout);

    solve();

    return 0;
}
```

## HUB08 - Mã hoán vị

Ta cần biến xâu độ dài `2n` thành dạng `T + T` bằng số phép đổi chỗ hai kí tự liên tiếp ít nhất. Với mỗi chữ cái, lưu toàn bộ vị trí xuất hiện của nó. Nếu một chữ cái xuất hiện `cnt` lần, ghép nửa đầu danh sách vị trí với nửa sau danh sách vị trí.

Sau khi có các cặp, sort theo vị trí đầu tiên. Cặp thứ `i` sẽ đi tới hai vị trí đích `i + 1` và `n + i + 1`. Khi đó bài toán trở thành đếm số nghịch thế của mảng vị trí đích, dùng Fenwick Tree để tính nhanh.

```cpp
#include <bits/stdc++.h>
#define int long long
#define rep(i, l, r) for (int i = l; i <= r; ++i)
#define sz(v) (int)(v).size()
#define pb push_back
#define pii pair<int,int>
#define fi first
#define se second
#define endl '\n'

using namespace std;

const int N = 2e5 + 5;

int n;
string s;
int bit[N];
vector<int> pos[26];

void update(int i, int val) {
    for (; i < N; i += i & -i) bit[i] += val;
}

int get(int i) {
    int res = 0;
    for (; i > 0; i -= i & -i) res += bit[i];
    return res;
}

void solve(void) {
    cin >> n >> s;
    int len = 2 * n;
    s = " " + s;

    rep(i, 1, len) {
        pos[s[i] - 'a'].pb(i);
    }

    vector<pii> pairs;

    rep(c, 0, 25) {
        int cnt = sz(pos[c]);
        int half = cnt / 2;

        rep(i, 0, half - 1) {
            pairs.pb({pos[c][i], pos[c][i + half]});
        }
    }

    sort(pairs.begin(), pairs.end());

    vector<int> target(len + 1);

    rep(i, 0, n - 1) {
        int l = pairs[i].fi;
        int r = pairs[i].se;

        target[l] = i + 1;
        target[r] = n + i + 1;
    }

    int ans = 0;

    rep(i, 1, len) {
        ans += i - 1 - get(target[i]);
        update(target[i], 1);
    }

    cout << ans;
}

signed main() {
    #define task "HUB08"
    ios_base::sync_with_stdio(0); cin.tie(0); cout.tie(0);

    freopen(task".inp", "r", stdin);
    freopen(task".out", "w", stdout);

    solve();
    return 0;
}
```

## Tổng kết

Bốn bài trên có thể gom lại thành vài ý chính:

- HUB05: duyệt đoạn và kiểm tra hoa văn lặp.
- HUB06: dựng xâu bằng các khối nhị phân phân biệt.
- HUB07: nén giá trị, tạo khoảng hợp lệ, rồi gộp khoảng.
- HUB08: quy về đếm nghịch thế bằng Fenwick Tree.

Đây là một bộ bài khá tốt để luyện tư duy duyệt, dựng cấu hình, xử lý khoảng và đếm nghịch thế.
