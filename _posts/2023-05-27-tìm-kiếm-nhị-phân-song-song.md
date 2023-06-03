---
title: Tìm kiếm nhị phân song song
date: 2023-05-27 11:00 +0700
categories: [CP]
tags: [Binary Search, Parallel Binary Search]
pin: true
math: true
---

# TÌM KIẾM NHỊ PHÂN SONG SONG

## PREREQUISITES
- Tìm kiếm nhị phân
- Các CTDL như Segment Tree, Fenwick Tree, DSU, ... 



## Tìm kiếm nhị phân song song là gì?

Chắc hẳn nhiều người đọc bài viết này đều biết "tìm kiếm nhị phân" là kĩ thuật quen thuộc trong CP, nhưng cụm từ "song song" mang ý nghĩa gì? Có thể hiểu rằng là ta có thể được cho nhiều truy vấn khác nhau, và sẽ tìm kiếm nhị phân đáp án **đồng thời** thay vì riêng lẻ từng truy vấn.

Hãy cùng xem qua một vài ví dụ để hiểu rõ hơn về **Tìm kiếm nhị phân song song** nhé.

## BÀI TOÁN 1

### Statement
Cho một dãy số gồm $n$ phần tử $a_1, a_2, ..., a_n$. Bạn phải viết chương trình để trả lời $q$ câu hỏi có dạng $(l$, $r$, $k)$ là in ra số lớn thứ $k$ trong các số thuộc đoạn $a_l$, $a_{l+1}$,$...$, $a_r$.
### Input
- Dòng đầu tiên gồm số nguyên dương $n$ $(1 \le n \le 50000)$.
- Dòng thứ hai gồm $n$ số nguyên dương $a_1$, $a_2$,$...$, $a_n$ $(1 \le a_i \le 10^9)$.
- Dòng thứ ba gồm số nguyên dương $q$ $(1 \le q \le 50000)$.
- Trong $q$ dòng tiếp theo, dòng thứ $i$ gồm $3$ số $l_i, r_i, k_i$ $(1 \le l_i \le r_i \le n, 1 \le k_i \le r_i - l_i+1)$.
### Output
- Gồm $q$ dòng, dòng thứ $i$ ($1 \le i \le q)$ là đáp án của truy vấn thứ $i$.

### Example
#### Sample Input
``` cpp
7
1 5 2 6 3 7 4
3
2 5 3
4 4 1
1 7 3

```
#### Sample Output
``` cpp
3
6
5
```

### Solution
**Lưu ý:** Bài này có thể được giải bằng nhiều cách, tuy nhiên mình sẽ sử dụng **Tìm kiếm nhị phân song song** để cho các bạn cái này trực quan về thuật toán này.

Ta sẽ bắt đầu bằng một thuật toán ngây thơ nhất: với mỗi truy vấn, ta sẽ tìm kiếm nhị phân để tìm số thứ $k$. Với mỗi lần tìm kiếm nhị phân, ta cần biết được trong đoạn $[l, r]$ có bao nhiêu số $\geq mid$ trong $O(n)$ $(1)$. Độ phức tạp cho mỗi truy vấn là $O(n \times logn)$. Độ phức tạp tổng của lời giải sẽ là $O(q \times n \times logn)$. 

Như đã nói, vì là một thuật toán ngây thơ nên độ phức tạp trên sẽ không thể giải quyết bài toán trong thời gian cho phép. Ta sẽ cần cải tiến thuật toán trên.

Mỗi truy vấn sẽ cần tìm kiếm nhị phân trong độ phức tạp tối đa là $O(logn)$. Giả sử ở lần tìm kiếm nhị phân thứ $i$ của $q$ truy vấn, gọi $[low_1, high_1]$, $[low_2, high_2]$,$...$, $[low_q, high_q]$ là khoảng cần tìm kiếm nhị phân, $mid_1$, $mid_2$,$...$, $mid_q$ là giá trị ta cần xử lí ở bài toán $(1)$ ở các truy vấn, tương đương với mỗi $mid_j$ $(1 \le j \le q)$, có bao nhiêu số trong đoạn $[l_j, r_j] \geq mid_j$. Ta đưa về có một bài toán con như sau:

Cho một dãy số $a_1, a_2, ..., a_n$ và $q$ truy vấn có dạng $(l, r, k)$ là hãy đếm xem có bao nhiêu số $\geq k$ trong đoạn $[l, r]$.

Đây chính là bài toán [KQUERY](https://oj.vnoi.info/problem/kquery) và được giải trong độ phức tạp $O(nlogn)$ sử dụng CTDL Fenwick Tree hoặc Segment Tree. 

Sau khi xác định được số phần tử $\geq mid_j$ trong đoạn $[l_j, r_j]$, ta sẽ xác định được khoảng cần tìm kiếm nhị phân mới $[low'_j, high'_j]$ và tiếp tục ở lần tìm kiếm nhị phân thứ $i+1$.

**Độ phức tạp:** $O((N + Q) \times log^2(N))$

### Code
```cpp
#include <bits/stdc++.h>
#define int long long
#define ii pair<int, int>
#define st first
#define nd second
#define endl "\n"
#define db long double
#define all(v) v.begin(), v.end()

using namespace std;

const int MAXN = 50005;

int a[MAXN], q, n, l[MAXN], r[MAXN], res[MAXN];

vector<int> b;

struct Q{
    int l, r, k;
} query[MAXN];

vector<int> candidate[MAXN], vl[MAXN];

struct FenwickTree{
    int fen[MAXN];

    void init(){
        memset(fen, 0, sizeof(fen));
    }

    void update (int u, int val){
        for (; u < MAXN; u += (u & (-u))) fen[u] += val;
    }

    int get (int u){
        int ans = 0;
        for (; u; u -= (u & (-u))) ans += fen[u];
        return ans;
    }

    int get (int l, int r){
        return get(r) - get(l - 1);
    }
} d;

void PROGRAM(int iTest){
    cin >> n;

    for (int i = 1; i <= n; i++){
        cin >> a[i];
        b.push_back(a[i]);
    }
    
    /*
     * Nén số (ta chỉ quan tâm đến quan hệ 
     * lớn nhỏ của các số nên sẽ không ảnh hưởng tính tương đối)
    */
    sort(all(b));
    b.erase(unique(all(b)), b.end());

    for (int i = 1; i <= n; i++){
        a[i] = lower_bound(all(b), a[i]) - b.begin() + 1;
        vl[a[i]].push_back(i);
    }
    
   

    cin >> q;

    int N = b.size();

    for (int i = 1; i <= q; i++){
        int ll, rr, k;
        cin >> ll >> rr >> k;
        query[i] = {ll, rr, k};
        
        /*
         * khoảng tìm kiếm nhị phân [l, r] của truy vấn thứ
         * i, giá trị sẽ thuộc khoảng a[1..n]
        */
        l[i] = 1;
        r[i] = N;
        res[i] = -1;

    }
    // mỗi truy vấn có tìm kiếm nhị phân trong độ phức tạp tối đa O(logn)
    
    for (int LOOP = 1; LOOP <= 16; LOOP++){ 
        for (int i = 1; i <= N; i++) candidate[i].clear();
        for (int i = 1; i <= q; i++){
            if (l[i] > r[i]) continue;
            int mid = (l[i] + r[i]) >> 1;
            candidate[mid].push_back(i);
        }
        
        // Đoạn xử lí bài toán KQUERY
        d.init();
        for (int i = N; i >= 1; i--){
            for (auto v : vl[i]) d.update(v, 1);

            for (auto v : candidate[i]){
                int ll = query[v].l, rr = query[v].r, k = query[v].k;
                if (d.get(ll, rr) >= k){
                    res[v] = i;
                    l[v] = i + 1;
                }
                else{
                    r[v] = i - 1;
                }
            }
        }
    }

    for (int i = 1; i <= q; i++){
        cout << b[res[i] - 1] << endl;
    }


}

signed main(){

    ios_base::sync_with_stdio(false);
    cin.tie(0);

    int test = 1;
    for (int iTest = 1; iTest <= test; iTest++){
        PROGRAM(iTest);
    }
}
```

## Bài toán 2

### Nguồn: [CSES - New Road Queries](https://cses.fi/problemset/task/2101)

### Statement
Cho một đồ thị gồm $n$ đỉnh. Trong $m$ thời điểm, tại thời điểm $i$ ta sẽ nối $2$ cạnh $u$ và $v$ với nhau (Dữ liệu đảm bảo cạnh $u$ và $v$ chưa được nối với nhau trước đó). 

Cho $q$ truy vấn, mỗi truy vấn gồm cặp số $(u, v)$, bạn cần trả lời sau thời điểm nào thì $u$ và $v$ có thể đi đến nhau, có thể qua một cạnh hoặc nhiều cạnh.

### Input
- Dòng đầu tiên gồm ba số nguyên dương $n, m, q$ $(1 \le n, m, q \le 2 \times 10^5)$.
- Dòng thứ $2$ $...$ $m + 1$: mỗi dòng gồm 2 số nguyên dương $u$ và $v$ $(1 \le u, v \le n)$.
- $q$ dòng cuối cùng, mỗi dòng gồm 2 số nguyên dương $u$, $v$ $(1 \le u, v \le 10^5)$.

### Output
- Gồm $q$ dòng, dòng thứ $i$ là đáp án của truy vấn thứ $i$. Nếu sau $m$ ngày $(u, v)$ không đi được đến nhau thì in ra $-1$.

### Example
#### Sample Input
```cpp
5 4
1 2 
1 3
2 4
3 4
3
2 3
1 4
4 5
```
#### Sample Output
```cpp
2
3
-1
```
### Solution
Ta sẽ sử dụng **Tìm kiếm nhị phân song song** kết hợp với CTDL **Disjoint Set Union** để giải bài tập này.

Ta sẽ tìm kiếm nhị phân, với mỗi truy vấn $(u, v)$ và $mid$, ta cần được biết đến ngày $mid$ cặp đỉnh $(u, v)$ đã liên thông chưa. Tương tư như bài toán $1$, kết hợp sử dụng tìm kiếm nhị phân song song, ta sẽ có bài toán con sau:

Cho $n$ đỉnh và $m$ ngày, ngày thứ $i$ sẽ nối $2$ đỉnh $u$ và $v$. Cho $q$ truy vấn $(u, v, k)$, cần trả lời đến ngày $k$ cặp đỉnh $(u, v)$ đã cùng thuộc một thành phần liên thông hay chưa?

Để giải bài toán con, ta sẽ đi qua từng ngày một và sử dụng **Disjoint Set Union** để nối các đỉnh lại. Tại ngày thứ $i$, ta sẽ xử lí các truy vấn có $mid = i$ và kiểm tra xem $(u, v)$ đã liên thông chưa.

**Độ phức tạp:** $O((m + q) \times log(m) \times log(n))$.

### Code
```cpp
#include <bits/stdc++.h>
#define int long long
#define ii pair<int, int>
#define st first
#define nd second
#define endl "\n"
#define all(v) v.begin(), v.end()
#define Unique(v) v.erase(unique(all(v)), v.end())

using namespace std;

const int MAXN = 2e5 + 5;

int n, m, q;

struct Edge{
    int u, v;
} e[MAXN];

struct Q{
    int u, v, id;
} query[MAXN];

int l[MAXN], r[MAXN], res[MAXN];    

struct DSU{
    int n;
    int par[MAXN], sz[MAXN];

    void init(int _n){
        n = _n;        
        for (int i = 1; i <= n; i++){
            par[i] = i;
            sz[i] = 1;
        }
    }

    int find(int u){
        return (u == par[u]) ? u : par[u] = find(par[u]);
    }

    void join (int u, int v){
        u = find(u), v = find(v);
        if (u == v) return;
        if (sz[u] < sz[v]) swap(u, v);
        par[v] = u;
        sz[u] += sz[v];
    }
};

vector<int> candidate[MAXN];

void PROGRAM(int _){
    cin >> n >> m >> q;

    for (int i = 1; i <= m; i++){
        cin >> e[i].u >> e[i].v;
    }

    for (int i = 1; i <= q; i++){
        cin >> query[i].u >> query[i].v;
        query[i].id = i;
        l[i] = 0, r[i] = m;
        res[i] = -1;
    }
    DSU d;
    for (int LOOP = 1; LOOP <= 18; LOOP++){
        d.init(n);
        
        for (int i = 1; i <= m; i++){
            candidate[i].clear();
        }

        for (int i = 1; i <= q; i++){
            if (l[i] > r[i]) continue;
            int mid = (l[i] + r[i]) >> 1;
            candidate[mid].push_back(i);
        }

        for (int i = 0; i <= m; i++){
            if (i > 0) d.join(e[i].u, e[i].v);
            for (auto V : candidate[i]){
                int u = query[V].u, v = query[V].v, id = query[V].id;
                u = d.find(u), v = d.find(v);
                if (u == v){
                    res[id] = i;
                    r[id] = i - 1;
                }
                else{
                    l[id] = i + 1;
                }
            }
        }
    }

    for (int i = 1; i <= q; i++) cout << res[i] << endl;


}

signed main(){
    ios_base::sync_with_stdio(false);
    cin.tie(0);

    int test = 1;

    for (int _ = 1; _ <= test; _++){
        PROGRAM(_);
    }

    return 0;
}
```

## Bài toán 3

### Nguồn: [LQDOJ Cup Round 3 - Shopping](https://lqdoj.edu.vn/problem/lqdoj2022r3shopping) ([Group 2022 Contest Practice VOI](https://lqdoj.edu.vn/organization/372-2022-contest-practice-voi))

### Tóm tắt đề bài
Cho dãy gồm $n$ cửa hàng, nếu mua đồ tại cửa hàng thứ $i$ sẽ mất $a_i$ đồng. Cho $q$ truy vấn có dạng $(l, u, v, k)$, đưa ra số cửa hàng có thể đi qua khi bắt đầu mua sắm tại vị trí $l$, chỉ mua ở những cửa hàng có $u \le a_i \le v$ và sử dụng tối đa $k$ đồng. Nếu đi đến một cửa hàng có $a_i < u$ hoặc $a_i > v$ thì sẽ chỉ đi qua mà không mua, còn nếu đi đến cửa hàng có $u \le a_i \le v$ mà không đủ tiền thì sẽ dừng việc mua sắm.

### Input
- Dòng đầu tiên gồm hai số nguyên $n$ và $q$ $(1 \le n, q \le 10^5)$.
- Dòng thứ hai gồm $n$ số nguyên dương $a_1, a_2, ..., a_n$ $(1 \le a_i \le 10^9)$.
- Trong $q$ dòng cuối cùng, mỗi dòng gồm bốn số nguyên dương $l, u, v, k$ $(1 \le l \le n, 1 \le u \le v \le 10^9, 1 \le k \le 10^9)$.

### Output
- Gồm $q$ dòng, dòng thứ $i$ là kết quả của truy vấn thứ $i$.

### Example
#### Sample Input
```cpp
7 3
4 6 8 2 10 5 1
4 1 5 7
1 2 3 5
1 1 10 15
```

#### Sample Output
```cpp
3
7
2
```

### Solution
Ta có thuật toán ngây thơ: với mỗi truy vấn $i$ $(1 \le i \le q)$, ta sẽ tìm kiếm nhị phân vị trí $r_i$ mà có thể đi qua các cửa hàng thuộc $[l_i, r_i]$, kiểm tra trong $O(n)$.
Sử dụng kĩ thuật tìm kiếm nhị phân song song để cải tiến. Với khoảng $[l_i, r_i]$, ta sẽ cần biết tổng các $a[l_i...r_i]$ thỏa mãn $u_i \le a_j \le v_i$ $(1 \le j \le n)$. Bài toán con sẽ là như sau:

Cho dãy $a_1, a_n, ..., a_n$. Cho $q$ truy vấn có dạng $(l, r, u, v)$, hãy tìm tổng của các phần tử $a[l..r]$ tỏa mãn $u \le a_i \le v$ $(2)$.


Tổng $(2)$ sẽ được tính bằng hiệu của tổng các phần tử $a[l..r]$ thỏa mãn $a_i \le v$ $(3)$ và tổng các phần tử $a[l..r]$ thỏa mãn $a_i \le u - 1$ $(4)$. Lần lượt update các phần tử thuộc dãy $a$, đồng thời duyệt toàn bộ các truy vấn $(3)$ và $(4)$ theo thứ tự giá trị tăng dần, cộng $(3)$ và trừ $(4)$ cho mỗi truy vấn.

**Độ phức tạp:** $O((N + Q) \times log^2(N))$ 

### Code
```cpp
#include <bits/stdc++.h>
#define int long long
#define ii pair<int, int>
#define st first
#define nd second
#define endl "\n"
#define all(v) v.begin(), v.end()
using namespace std;
const int MAXN = 1e5 + 5;
const int LOG = 17;
int n, q;
ii a[MAXN];
struct Q{
    int l, u, v, k;
} query[MAXN];
struct F{
    int type, val, idx;
    bool op;
    bool operator < (const F &other){
        return (
            val != other.val ? val < other.val :
            type < other.type
        );
    }
};
int fen[MAXN];
int l[MAXN], r[MAXN], ans[MAXN];
vector<F> candidate;
void update(int u, int v){
    for (; u <= n; u += (u & (-u))) fen[u] += v;
}
int get(int u, int ans = 0){
    for (; u >= 1; u -= (u & (-u))) ans += fen[u];
    return ans;
}
signed main(){
    ios_base::sync_with_stdio(false);
    cin.tie(0);
    if (fopen("SHOPPING.INP", "r")){
        freopen("SHOPPING.INP", "r", stdin);
        freopen("SHOPPING.OUT", "w", stdout);
    }

    cin >> n >> q;
    for (int i = 1; i <= n; i++) cin >> a[i].st, a[i].nd = i;

    for (int i = 1; i <= q; i++){
        cin >> query[i].l >> query[i].u >> query[i].v >> query[i].k;
        l[i] = query[i].l - 1;
        r[i] = n + 1;
    }
    for (int i = 1; i <= n; i++){
        candidate.push_back((F){1, a[i].st, i, 0});
    }
    for (int i = 1; i <= q; i++){
        candidate.push_back((F){2, query[i].u - 1, i, 0});
        candidate.push_back((F){2, query[i].v, i, 1});
    }

    sort (all(candidate));

    for (int loop = 1; loop <= LOG; loop++){
        memset(fen, 0, sizeof(fen));
        memset(ans, 0, sizeof(ans));
        // Duyệt các giá trị theo thứ tự tăng dần
        for (int i = 0; i < candidate.size(); i++){
            if (candidate[i].type == 1){
                update(candidate[i].idx, a[candidate[i].idx].st);
            }
            else{
                int val = candidate[i].val, idx = candidate[i].idx;
                bool op = candidate[i].op;
                int le = query[idx].l, ri = (l[idx] + r[idx]) >> 1;
                int ttt = (get(ri) - get(le - 1));
                if (op == 0) ans[idx] -= ttt;
                else ans[idx] += ttt;
            }
        }
        for (int i = 1; i <= q; i++){
            if (ans[i] <= query[i].k) l[i] = (l[i] + r[i]) >> 1;
            else r[i] = (l[i] + r[i]) >> 1; 
        }
    }
    for (int i = 1; i <= q; i++) cout << l[i] - query[i].l + 1 << endl;
}

```
## Bài toán 4
### Nguồn: [CodeChef - Evil Chris](https://www.codechef.com/problems/MCO16504?tab=statement)

### Tóm tắt đề bài
Cho đồ thị gồm có $n$ đỉnh, $m$ cạnh hai chiều, đỉnh thứ $i$ hiện tại có $a_i$ học sinh. Độ hạnh phúc của một học sinh được tính bằng số học sinh khác có thể gặp được bằng cách đi qua các cạnh của đồ thị (tính cả chính học sinh đó). Có $D$ ngày gồm $D$ sự kiện diễn ra, vào ngày thứ $t$ có thể xảy ra một trong hai sự kiện sau:
- Đường đi giữa hai đỉnh $(u, v)$ bị phá hủy $(*)$
- $x$ học sinh của đỉnh $i$ sẽ bị loại bỏ. $(**)$

Cho $q$ truy vấn, mỗi truy vấn gồm 2 đỉnh $(u, v)$, hãy tìm ngày $d$ lớn nhất thỏa mãn trước ngày $d+1$ rằng tổng độ hạnh phúc của $(u, v)$ có giá trị ít nhất là $k$.

### Input
- Dòng đầu tiên gồm bốn số nguyên dương $n, m, D, q$ $(1 \le n, m, D, q \le 2 \times 10^5)$.
- Dòng thứ hai gồm $n$ số nguyên dương $a_1, a_2,..., a_n$ $(1 \le a_i \le 10^9)$.
- $m$ dòng tiếp theo, mỗi dòng gồm hai số nguyên dương $(u, v)$ $(1 \le u, v \le n)$.
- $D$ dòng tiếp theo, mỗi dòng gồm ba số nguyên dương:
    - $1$ $u$ $v$: Đường đi giữa $(u, v)$ bị phá hủy.
    - $2$ $i$ $x$: Có $x$ học sinh của đỉnh $i$ bị loại bỏ.
- $q$ dòng tiếp theo, mỗi dòng gồm ba số nguyên dương $(u, v, k)$ $(1 \le u, v \le n,$ $1 \le k \le 10^9)$.

### Output
- Gồm $q$ dòng, mỗi dòng chứa đáp án trả lời cho từng truy vấn. Nếu tổng độ hạnh phúc của hai học sinh không bao giờ $\geq k$, kể cả trước ngày diễn ra sự kiện đầu tiên, in ra $-1$.

### Example
#### Sample Input
```cpp
4 5 3 4
1 2 3 4
1 2
2 3
1 3
3 4
2 4
1 2 4
1 3 4
2 3 2
1 4 21
1 4 20
1 3 9
2 2 2
```
#### Sample Output
```cpp
-1
1
2
3
```

### Solution
Với mỗi truy vấn, tìm kiếm nhị phân song song ngày $mid$ xem tổng hạnh phúc hai học sinh $(u, v)$ có $\le k$ không. Tương tư các bài tập trên, ta có một bài toán con:

Cho đồ thị gồm $n$ đỉnh, $q$ truy vấn $(u, v, k, mid)$, cần xem vào ngày thứ $mid$ sau khi diễn ra sự kiện, tổng độ hạnh phúc của $(u, v)$ có $\le k$ hay không? 

Ta sẽ sử dụng **Disjoint Set Union** và lưu thêm thông tin là tổng số học sinh thuộc cùng một thành phần liên thông. Lần lượt duyệt từ cuối về từ ngày $d$ đến ngày $0$, thực hiện các sự kiện của từng ngày. Nếu sự kiện tại ngày thứ $i$ là $(*)$, ta sẽ **join** $(u_i, v_i)$, ngược lại với sự kiện $(**)$, ta sẽ giảm số học sinh thuộc thành phần liên thông của $u_i$ đi $x_i$. 

Tại ngày $i$ sẽ bao gồm các truy vấn $(u, v, k, i)$, ta sẽ chỉ cần tìm tổng của $S_u$ và $S_v$ với $S_i$ là tổng số học sinh của thành phần liên thông chứa đỉnh $i$.

**Độ phức tạp:** $O((m + q) \times log^2(m))$.

### Code
```cpp
#include <bits/stdc++.h>
#define int long long
#define ii pair<int, int>
#define st first
#define nd second
#define endl "\n"

using namespace std;
const int MAXN = 2e5 + 5;
int n, m, q, a[MAXN], s;
struct Q{
    int type, u, v;
} query[MAXN];
struct DSU{
    int par[MAXN], sz[MAXN];
    void init(){
        for (int i = 1; i <= n; i++){
            par[i] = i;
            sz[i] = max(0ll, a[i]);
        }
    }
    int find(int u){
        return (u == par[u] ? u : par[u] = find(par[u]));
    }
    void join(int u, int v){
        u = find(u);
        v = find(v);
        if (u == v) return;
        if (sz[u] < sz[v]) swap(u, v);
        par[v] = u;
        sz[u] += sz[v];
    }
} d;
int l[MAXN], r[MAXN], b[MAXN];
vector<int> candidate[MAXN];
bool check[MAXN];
struct Edge{
    int u, v;
} edges[MAXN];
struct dmm{
    int u, v, k;
} t[MAXN];
signed main(){
    ios_base::sync_with_stdio(false);
    cin.tie(0);
    cin >> n >> m >> q >> s;
    for (int i = 1; i <= n; i++) cin >> a[i];
    d.init();
    map<ii, int> x;
    for (int i = 1; i <= m; i++){
        int u, v;
        cin >> u >> v;
        x[{u, v}] = x[{v, u}] = i;
        edges[i] = {u, v};
    }
    fill (check + 1, check + m + 1, true);
    for (int i = 1; i <= q; i++){
        cin >> query[i].type >> query[i].u >> query[i].v;
        if (query[i].type == 1){
            check[x[{query[i].u, query[i].v}]] = 0;
        }
        else{
            a[query[i].u] -= query[i].v;
        }
    }

    for (int i = 1; i <= s; i++){
        cin >> t[i].u >> t[i].v >> t[i].k;
        l[i] = -1;
        r[i] = q + 1;
    }

    for (int loop = 1; loop <= 18; loop++){
        /****RESET****/
        for (int i = 1; i <= n; i++) b[i] = a[i];
        d.init();
        for (int i = 1; i <= m; i++){
            if (check[i]) d.join(edges[i].u, edges[i].v);
        }
        /********/
        for (int i = 1; i <= s; i++){
            if (l[i] + 1 == r[i]) continue;
            int mid = (l[i] + r[i]) >> 1;
            candidate[mid].push_back(i);
        }

        for (int i = q; i >= 0; i--){
            auto [type, u, v] = query[i + 1];
            if (type == 1){
                d.join(u, v);
            }
            else if (type == 2){
                b[i] += v;
                if (b[i] - v >= 0) d.sz[d.find(u)] += v;
                else if (b[i] > 0) d.sz[d.find(u)] += b[i];
            }
            for (auto idx : candidate[i]){
                auto [u, v, k] = t[idx];
                if (d.sz[d.find(u)] + d.sz[d.find(v)] >= k) l[idx] = i;
                else r[idx] = i;
            }
            candidate[i].clear();
        }
    }

    for (int i = 1; i <= s; i++) cout << l[i] << endl;
}
```

## Một số bài tập áp dụng
- [Codeforces 484E](https://codeforces.com/contest/484/problem/E)
- [ICPC 2022 - Vietnam Central Provincial Contest - Median](https://oj.vnoi.info/problem/icpc22_mt_d)
- [Meteors](https://szkopul.edu.pl/problemset/problem/7JrCYZ7LhEK4nBR5zbAXpcmM/site/?key=statement)
- [Travel in Hackerland](https://www.hackerrank.com/contests/may-world-codesprint/challenges/travel-in-hackerland)
- [TopCoder SRM 675](https://community.topcoder.com/stat?c=problem_statement&pm=14088)
- [COCI '20 Contest 6 #5 Index](https://dmoj.ca/problem/coci20c6p5)
- [HackerRank - Selective Additions](https://www.hackerrank.com/contests/hourrank-23/challenges/selective-additions/problem)
- [VNOJ - vosplay](https://oj.vnoi.info/problem/vosplay)

## Một số nguồn tham khảo

- [Robert1003](https://robert1003.github.io/2020/02/05/parallel-binary-search.html)
- [Codeforces](https://codeforces.com/blog/entry/45578)