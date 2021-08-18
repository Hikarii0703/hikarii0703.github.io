---
layout: post
title:  "[Vietnamese] Sparse Table"
date:   2021-08-17 19:50:00 +0700
categories: programming cpp cp
---

**Sparse Table** là một data structure rất hay, có một số ứng dụng liên quan nhưng thường bị các bạn bỏ qua, vì những data structure khác như **Segment Tree** có thể thực hiện được những operator tương tự trong độ phức tạp tệ hơn nhưng chấp nhận được. Tuy nhiên, có một số bài toán có giới hạn thời gian rất chặt, nên việc sử dụng **Sparse Table** đôi khi là cần thiết.

**Sparse Table** thường được build trong O(n log n) và mỗi query được thực hiện trong O(1) (hoặc O(log n) tùy loại query, nhưng những query thực hiện trong O(1) mới là điểm sáng cần quan tâm). Hạn chế duy nhất của data structure này là không thể update. Tức là, nếu có phần tử phải thay đổi giá trị, ta chỉ còn cách tính lại toàn bộ bảng.

## Xây dựng Sparse Table

### Bài toán

Giả sử bài toán cần giải được phát biểu như sau:

> "Cho một mảng A có n phần tử (n <= 10^5), và q truy vấn có dạng (L, R) (q <= 10^5). Với mỗi truy vấn, tìm phần tử nhỏ nhất trong mảng A từ L đến R."

Một trong những cách trâu bò là duyệt thẳng từ `A[L]` đến `A[R]`. Dễ thấy, độ phức tạp của mỗi truy vấn là O(n). Ta hoàn toàn có thể làm tốt hơn.

### Nhận xét

Mọi số nguyên dương đều có thể được viết thành tổng của các lũy thừa giảm dần của 2. VD: 13 = 8 + 4 + 1 = 2^3 + 2^2 + 2^0. Với một số nguyên dương `x` bất kỳ, ta sẽ cần tối đa `ceil(log2(x))` số hạng.

Tương tự, ta có thể chia một đoạn thành nhiều đoạn có độ dài là lũy thừa giảm dần của 2. VD: `[2; 14]` (độ dài 13) có thể được chia ra thành `[2; 9]`, `[10; 13]`, `[14; 14]` (tương ứng với độ dài 8, 4, 1). Ta cũng sẽ có thể chia tối đa `ceil(log2(độ dài của đoạn))` thành nhiều đoạn nhỏ.

Ý tưởng của Sparse Table là tính trước những đoạn có độ dài là lũy thừa của 2, rồi chỉ cần gộp lại những đoạn này là được.

### Xây dựng

Ta sẽ gọi mảng `st[i][j]` là giá trị nhỏ nhất trong đoạn bắt đầu từ phần tử `i`, có độ dài là `2^j`. Như vậy, giá trị của `st[i][j]` sẽ như sau:

![sp1](/assets/sparse_table/sp1.jpg)

Ta cần khai báo mảng `st[maxN][maxK]`. `maxN` là độ dài lớn nhất mà mảng `A` có thể đạt được, còn `maxK` phải vừa đủ sao cho `maxN < 2^maxK`. Thường thì `maxK = 25` là khá ổn, vì `2^25 ~ 3 * 10^7`, trong khi thực tế gần như không có bài toán nào có `n` tới `3 * 10^7` cả.

Đầu tiên, dễ dàng thấy được `2^0 = 1`, nên `st[i][0] = a[i]`.

```cpp
for (int i = 0; i < n; ++i) {
    st[i][0] = a[i];
}
```

Tiếp tục, ta sẽ ghép 2 đoạn con lại với nhau. Giả sử, ta muốn tạo `st[i][j + 1]`, thì ta cần ghép 2 đoạn con sau:
- Đoạn con bắt đầu từ `i`, có `2^j` phần tử, kết thúc tại `i + 2^j - 1`
- Đoạn con bắt đầu từ `i + 2^j`, có `2^j` phần tử, kết thúc tại `i + 2^(j + 1) - 1`

![sp3](/assets/sparse_table/sp3.jpg)

Vậy, ta có `st[i][j + 1] = min(st[i][j], st[i + 2^j][j])`.

Thay `j` bằng `j - 1`, ta có `st[i][j] = min(st[i][j - 1], st[i + (1 << j)][j - 1])`.

```cpp
for (int j = 1; j < maxK; ++j)
    for (int i = 0; i + (1 << j) <= n; ++i)
        st[i][j] = min(st[i][j - 1], st[i + (1 << (j - 1))][j - 1]);
```

Thực hiện dựng bảng như trên có độ phức tạp O(n log n).

### Trả lời truy vấn

Sau khi đã dựng được Sparse Table, ta tiếp tục trả lời truy vấn.

Trong trường hợp độ dài không phải lũy thừa của 2, ta cũng sẽ tìm `j` lớn nhất có thể sao cho `2^j < R - L + 1`. Sau đó, ta trả về giá trị nhỏ nhất của 2 đoạn sau:
- Đoạn bắt đầu từ `L`, có `2^j` phần tử
- Đoạn kết thúc tại `R`, có `2^j` phần tử, bắt đầu tại `R - 2^j + 1`

Dễ dàng thấy được, 2 đoạn này sẽ chồng lên nhau và không để hở phần tử nào cả, như hình minh họa dưới đây:

![sp2](/assets/sparse_table/sp2.jpg)

Vậy, ta chỉ cần tính:

```cpp
int j = log2(R - L + 1);
int minimum = min(st[L][j], st[R - (1 << j) + 1][j]);
```

Trong trường hợp độ dài là lũy thừa của 2, đoạn xanh và đoạn đỏ trên hình lúc này sẽ bằng nhau, nên tính đúng đắn của đoạn code trên vẫn đúng.

Ta nhận thấy, mỗi giá trị trong mảng `st` đã được tính từ trước đó, nên độ phức tạp là O(1). Tuy vậy, ta vẫn cần phải tính toán `log2` một cách chính xác và nhanh chóng. Ta có thể tính mảng `log[]` như sau:

```cpp
int log[maxN + 1];
log[1] = 0;
for (int i = 2; i <= maxN; ++i)
    log[i] = log[i / 2] + 1;
```

và `log[x]` sẽ là `log2(x)`.

Còn một cách nữa là dùng hàm `__builtin_popcount` của GCC, có thể dùng `31 - __builtin_popcount(x)` hoặc `63 - __builtin_popcountll(x)` (tùy thuộc vào `x` là `int` hay `long long`). Với cách này thì ta không cần dựng thêm mảng nào, nhưng chỉ hoạt động với GCC. Với những compiler khác thì ta có thể tìm những lệnh tương đương, hoặc sử dụng lookup table.

Đoạn được đánh dấu màu cam là đoạn chồng lặp, được tính 2 lần. Lý do đoạn này được đánh dấu sẽ được giải thích sau đây.

### Đoạn trùng lặp

Ta nhận xét rằng, `min(a, a) = a`, nghĩa là min của một số với chính nó cũng là chính nó. Vậy nên, đoạn màu cam vừa rồi được tính 2 lần sẽ không gây ảnh hưởng tới kết quả cuối cùng.

Vậy, những hàm `f(x, y)` sao cho `f(a, a) = a` (i.e. đưa một số và chính nó vào hàm thì kết quả là chính nó), đều có thể ứng dụng được Sparse Table như trên với độ phức tạp `O(f(x, y))` (i.e. nếu `f` là phép `max` hay `min` thì `O(1)`, phép `gcd` hay `lcm` thì `O(log(min(x, y)))`...).

Những hàm khác, như hàm cộng hay nhân, vẫn có thể ứng dụng được Sparse Table, nhưng lúc này truy vấn không còn độ phức tạp `O(1)`, mà là `O(log n)`. Ý tưởng vẫn là chia nhỏ đoạn của truy vấn thành các đoạn nhỏ kề nhau có độ dài là lũy thừa 2 (xem lại giới thiệu ban đầu):

```cpp
long long sum = 0;
for (int j = maxK - 1; j >= 0; --j) {
    if ((1 << j) <= R - L + 1) {
        sum += st[L][j];
        L += (1 << j);
    }
}
```

Một số data structure tương tự có thể áp dụng với mọi hàm hợp và có thể trả lời query trong `O(1)` bao gồm **Disjoint Sparse Table** và **Sqrt Tree**.

## Ứng dụng

Trong trường hợp thời gian khá chặt và không yêu cầu update, Sparse Table thường được lựa chọn. Ngoài ra, khá nhiều implementation của LCA cũng được cài bằng Sparse Table.

## Bên lề

- Hàm `min` là hàm được ứng dụng rộng rãi nhất bằng Sparse Table, nên một số người còn gọi data structure này là **RMQ** (Range Minimum Query).

- Có cách để cài đặt Sparse Table trong `O(1)` (và thường được dùng để tính LCA trong `O(1)`). Tuy nhiên, vì trình độ của tác giả còn non và trick không quá well-known, nên tác giả sẽ để bạn đọc tự nghiên cứu.

- Ban đầu, tác giả định lựa chọn Segment Tree để viết bài. Tuy nhiên, vì VNOI Wiki đang được rework lại và Segment Tree được số lượng vote rất cao nên tác giả hủy ý tưởng trên. Tình cờ, tác giả nhận thấy chưa có blog tiếng Việt nào viết riêng về Sparse Table, nên topic này đã được lựa chọn để viết.