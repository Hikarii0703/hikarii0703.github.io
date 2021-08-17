---
layout: post
title:  "[Vietnamese] Disjoint Set Union"
date:   2021-03-22 21:27:00 +0700
categories: programming cpp cp
---

**Disjoint Set Union (DSU)** là một data structure đa dụng, có thể dùng để giải quyết nhiều bài toán điển hình.
DSU thường được dùng để **nhóm các phần tử (union)** và **kiểm tra xem phần tử thuộc nhóm nào (find)**.
Chính vì thế, mà DSU còn có tên gọi khác là **Union-Find**.

## Xây dựng DSU

Mỗi nhóm trong DSU sẽ có một phần tử làm đại diện cho toàn bộ nhóm. Phần tử đó gọi là **leader/parent**.
(**parent** là tên phổ biến hơn, nhưng **leader** lại thể hiện được đúng tính chất hơn. Để mang tính "hội nhập", ta sẽ thống nhất gọi là **parent**).
Vì thế, ta có thể chia các nhóm, dựa trên việc ai là **parent** của mỗi phần tử.<br/>
VD:

![dsu1](/assets/dsu/dsu1.png)

Như trên hình, ta có thể thấy có 3 nhóm:
* Nhóm 1: 1, 2, 3, 4
* Nhóm 2: 5, 6, 7
* Nhóm 3: 8, 9, 10, 11, 12, 13

Giả sử, ta gọi mảng `parent[]` là mảng lưu lại **parent** của một đỉnh.
Dễ thấy, `parent[1] == parent[2]` vì 2 đỉnh này cùng chung một nhóm.
Trong khi đó, `parent[6] != parent[9]` vì 2 đỉnh này khác nhóm.

Vậy, `parent[x]` bằng mấy? Như đã đề cập bên trên, mỗi nhóm đều phải có một **parent** xác định nào đó để phân chia các phần tử
còn lại thành các nhóm.

Vì thế, nếu `parent[x] == p` thì **`x` sẽ là phần tử thuộc nhóm do `p` làm parent**. Nếu `parent[x] == x` thì **chính `x` sẽ là parent trong nhóm của x**.

VD: với hình trên, mảng `parent[]` của chúng ta **có thể** sẽ như sau:
`parent = {0, 1, 1, 1, 1, 5, 5, 5, 8, 8, 8, 8, 8, 8}`<br/>
(lưu ý: phần tử `0` kia là vì ta không có phần tử `0` trong đồ thị, tuy nhiên mảng C++ đánh chỉ số từ `0`).

Trong lúc cài đặt, mảng `parent[]` hoàn toàn có thể khác ví dụ mẫu bên trên, miễn là ta đảm bảo được những tính chất vừa trình bày như trên.<br/>
VD: `parent = {0, 2, 2, 2, 2, 6, 6, 6, 9, 9, 9, 9, 9, 9}`

### Thuật trẻ trâu

Đầu tiên, mọi đỉnh đều sẽ thuộc một nhóm riêng, chưa gộp ai với ai cả. Vì thế, với mỗi đỉnh thứ `i`, `parent[i] = i` (i.e. `parent[1] = 1`, `parent[42]` = 42...).
```cpp
int main() {
    for (int i = 1; i <= n; i++)
        parent[i] = i;

    // hoặc
    iota(a + 1, a + n + 1, 1);
}
```

Ta sẽ có hàm đệ quy `find()` để tìm **parent** như sau: 
```cpp
int find(int x) {
    if (parent[x] == x)
        return x;

    return find(parent[x]);
}
```

Nếu `parent[x] == x` thì lúc này `x` là **leader/parent**, vì thế nên ta không cần tìm nữa.
Ngược lại, ta lại tìm tiếp parent của parent của `x`, tìm nguyên dòng họ cho đến khi tìm thấy **leader/parent**.

Ngoài ra, ta có thể kiểm tra xem hai đỉnh có cùng chung nhóm không:
```cpp
bool sameSet(int a, int b) {
    return find(a) == find(b);
}
```
Đơn giản, hai thằng chung nhóm thì parent của cả hai thằng thực chất là một.

Hàm `join()` để nhóm hai nhóm với nhau như sau:
```cpp
void join(int a, int b) {
    a = find(a);
    b = find(b);
    if (a != b) parent[b] = a;
}
```

Dễ thấy, để gộp hai nhóm với nhau, ta chỉ đơn giản là lôi đầu parent của mỗi nhóm ra, rồi cho một thằng là parent của thằng còn lại.<br/>
Đầu tiên, ta thay `a` và `b` bằng parent của hai đỉnh này (i.e. `a` hiện tại là parent của `a` lúc mới nhập, tương tự với b).
Nếu hai thằng khác nhau, nghĩa là `a` và `b` lúc mới nhập không cùng nhóm, ta gán một thằng là parent thằng kia.

Cách này trẻ trâu ở điểm nào? 

Giả sử, đồ thị của chúng ta sau khi gán một hồi nhìn như này:

![dsu2](/assets/dsu/dsu2.png)

`find(5)` sẽ đi hết nguyên đoạn đường ngược về đỉnh `1`. Tương tự, `find(10)` sẽ đi hết nguyên đoạn đường ngược về đỉnh `6`.
Độ phức tạp của cách cài này sẽ là O(n).

Hai hàm `sameSet()` và `join()` đều phụ thuộc vào hàm `find()`. Vì thế, độ phức tạp của hai hàm này cũng là O(n) luôn.

Ta hoàn toàn có thể tối ưu cách cài đặt của chúng ta xuống độ phức tạp thấp hơn đáng kể.

### Path compression

Giả sử ta có hình sau:

![dsu3](/assets/dsu/dsu3.png)

Lúc này, `parent[3] = 2, parent[2] = 1`. Việc gọi `find(3)` sẽ phải đi qua đỉnh `2`, rồi mới qua đỉnh `1`.
Trong khi đó, tại sao ta không nối thẳng `3` vào `1` luôn cho khỏe, để đỡ phải qua `2`?

Vậy, ta chỉ cần gán thẳng `parent[3] = find(parent[2])` (hay `parent[3] = 1`).

Chỉ thêm một chút như sau, tốc độ thuật toán cải thiện cực kì rõ rệt:
```cpp
int find(int x) {
    if (parent[x] == x)
        return x;

    return parent[x] = find(parent[x]);
}
```

Lúc này, độ phức tạp của `find()` chỉ còn O(log n), và khiến cả độ phức tạp của `join()` cũng trở thành O(log n) luôn.

Tuy nhiên, ta còn một phương pháp tối ưu nữa, làm cho thuật toán của chúng ta nhanh hơn nữa.

### Union by size/rank

Ta sẽ chỉ đề cập đến **Union by size**. **Union by rank** thực chất cũng có cùng tư tưởng, chỉ có khác biệt rất nhỏ ở cách cài.

Ta sẽ cần thêm mảng `size[]`. `size[x]` sẽ cho biết có bao nhiêu phần tử là con của `x` (bao gồm chính nó).<br/>
VD:

![dsu4](/assets/dsu/dsu4.png)

Dễ dàng thấy được `size[1] = 6`, `size[3] = 3`. Thế thì, mảng `size[]` này để làm gì?

Nếu gộp 2 nhóm này với nhau, ta sẽ có hai trường hợp:
* Gộp nhóm lớn vào nhóm nhỏ: độ cao của cây sẽ là 5.

    ![dsu5](/assets/dsu/dsu5.png)

* Gộp nhóm nhỏ vào nhóm lớn: độ cao của cây sẽ là 4.

    ![dsu6](/assets/dsu/dsu6.png)

Ta nhận xét rằng: gộp nhóm nhỏ vào nhóm lớn sẽ có lợi thế hơn so với gộp nhóm lớn vào nhóm nhỏ (độ cao 4 < 5).
Vậy, trong hai parent, **ta sẽ gán parent có size nhỏ hơn vào size lớn hơn**.
Đồng thời, ta nhận thấy **size của parent được gộp vào sẽ được tăng thêm một lượng bằng size của parent kia**.

VD: lúc này, `parent[7] = 1` và `size[1] += size[7]`.

Trước khi cài đặt lại hàm `join()`, nên nhớ là mọi phần tử trong `size[]` đều có giá trị ban đầu là 1.
```cpp
int main() {
    for (int i = 1; i <= n; i++)
        size[i] = 1;

    // hoặc
    fill(size + 1, size + n + 1, 1);

    // hoặc
    fill_n(size + 1, n, 1);
}
```

Ta cài đặt lại hàm `join()` như sau:

```cpp
void join(int a, int b) {
    a = find(a);
    b = find(b);

    if (a == b) return;

    if (size[a] < size[b]) swap(a, b);

    // lúc này size[a] >= size[b]
    size[a] += size[b];
    parent[b] = a;
}
```

Phương pháp cải tiến này cũng sẽ giảm độ phức tạp của hàm `join()` còn O(log n).

### Kết luận
Mỗi phương pháp cải tiến sẽ hạ độ phức tạp xuống còn O(log n).

Kết hợp cả hai phương pháp, độ phức tạp của DSU sẽ chỉ còn O(α(n)) (trong đó α(n) là hàm [Inverse Ackermann](https://en.wikipedia.org/wiki/Ackermann_function#Inverse),
là một hàm có tốc độ gia tăng cực kì chậm, n < 10^600 thì α(n) < 4, rất nhỏ, cực kì gần với O(1)).

### Extra: Một số cách cài khác
* DSU có thể được cài đặt để nối các đỉnh giá trị âm(!) với nhau.

* Có một số người gán lại parent như sau:
```cpp
void join(int u, int v) {
    u = find(u);
    v = find(v);
    if (rand() & 1) parent[u] = v;
    else parent[v] = u;
}
```
Cách này cũng uhhhh được, cơ mà nó hơi ngu, độ phức tạp của nó cũng khá là sát nhưng không tốt bằng **Path Compression**.

* Ngoài ra, ta có thể cài đặt để gộp chung mảng `parent[]` và `size[]` lại thành một mảng duy nhất để tiết kiệm bộ nhớ
    * Ta sẽ gọi mảng gộp chung này là `parent_or_size[]` (nếu thấy dài quá thì cứ gọi đại là `parent[]` luôn cũng được)
    * Mọi phần tử của `parent_or_size[]` có giá trị bằng **-1**
    * Nếu `parent_or_size[x] < 0`, thì **x là parent, và -parent\_or\_size[x] tương đương size[x]**
    * Ngược lại, nếu `parent_or_size[x] > 0` thì **parent\_or\_size[x] là parent của x**
    * Ta có cách cài như sau:

    ```cpp
    int find(int x) {
        if (parent_or_size[x] < 0) return x;
        return parent_or_size[x] = find(parent_or_size[x]);
    }

    void join(int u, int v) {
        u = find(u), v = find(v);
        if (u == v) return;

        if (-parent_or_size[u] < -parent_or_size[v]) {
            swap(u, v);
        }
        parent_or_size[u] += parent_or_size[v];
        parent_or_size[v] = u;
    }
    ```

## Ứng dụng

### Kiểm tra tính liên thông của đồ thị vô hướng
Cái này thì quá dễ hiểu luôn. Nếu hai đỉnh liên thông với nhau trong đồ thị vô hướng, thì hai đỉnh này sẽ nằm chung một nhóm.<br/>
Ta còn có thể ứng dụng tính chất này để cài đặt **Minimum Spanning Tree (Cây khung nhỏ nhất)** một cách dễ dàng bằng **Kruskal's Algorithm**.

### Offline Range Minimum Query (RMQ) trong trung bình O(α(n)) mỗi truy vấn / Arpa's trick
Range Minimum Query là bài toán tìm phần tử nhỏ nhất trong đoạn [L, R]. Có một số cách giải, như ứng dụng Sparse Table, Segment Tree...,
và DSU cũng có thể được sử dụng để tìm RMQ offline (nhập hêt truy vấn, xử lý, rồi mới đưa ra đáp án một lượt).

[Arpa](https://codeforces.com/profile/Arpa) là người đã độc lập tìm ra thuật toán này, và biến thuật này phổ biến, mặc dù trước đó
thuật này đã được tìm ra trước đó rồi.

### Tìm LCA/Tổ tiên chung gần nhất trong O(n) / Tarjan's offline LCA algorithm
Thuật toán này được phát triển bởi [Tarjan](https://en.wikipedia.org/wiki/Robert_Tarjan) (ông còn có một thuật toán khác mang tên ông **(Tarjan's Algorithm)**
để tìm **SCC/Thành phần Liên thông Mạnh**, **Khớp (Articulation Points, Cut Vertex...) và Cầu (Bridge, Cut Edge...)** cực kỳ nổi tiếng).

Thuật toán này có một nhược điểm là yêu cầu biết trước mọi truy vấn trước khi xử lý. Về ưu điểm, thuật toán này có độ phức tạp O(n), cài đặt đơn giản và bộ nhớ không quá nhiều
(so sánh với Binary Lifting, Farach-Colton...)

Ngoài ra, còn rất nhiều ứng dụng khác của DSU, nhiều khi còn chẳng liên quan đến DSU hay liên quan đến đồ thị.

## Bên lề

### Cách gọi tên các hàm
Tôi thích dùng `root()` và `unite()`. "root" là danh từ và "unite" là động từ, trong khi `root()` thuộc kiểu `int` còn `unite()` thuộc kiểu `void`,
nên `root(x)` và `unite(u, v)` nghe tràn đầy sự hợp lý. Một số người như bạn tôi, ***dlbm1302*** thích dùng `getPar()` và `join()`. [jiangly](https://codeforces.com/profile/jiangly)
(Legendary Grandmaster, hiện đang top 1 Trung Quốc trên Codeforces, người tôi cực kì mến mộ) dùng `leader[]` thay vì `parent[]`.

Vậy nên, có muôn vàn cách đặt tên hàm, nên nếu đọc code người khác thì phải để ý một chút.

### Suýt nữa là AC 4 bài Free Contest
Tại kì [Testing Round 6](https://www.facebook.com/kc97blf/photos/3183003871925426) của Free Contest, mình rất akay vì bài METEORITE. Mình mới học DSU ngay chiều hôm đó,
vừa học vừa dụi mắt, thấm mãi không nổi. Y như rằng, buổi tối ra đúng bài cần DSU .\_. Tôi vẫn tiếc, vì suýt nữa mình đã có lần đầu tiên AC 4 bài Free Contest trong đời.
(Mà sau này tôi còn một kì Free Contest nữa đắng hơn, 248/250, nhưng có người 250/250 nên không được vinh danh ;-;).

## Nguồn
* [e-maxx / cp-algorithms](https://cp-algorithms.com/data_structures/disjoint_set_union.html)

## Bài tập liên quan
* [VNOI - IOIBIN](https://codeforces.com/group/FLVn1Sc504/contest/274496/problem/F)
* [CSES - Road Construction](https://cses.fi/problemset/task/1676)
* [Blog Codeforces](https://codeforces.com/blog/entry/22747)
* [sẽ cập nhật tiếp]
