---
layout: post
title:  "[Vietnamese] BFS on Grid"
date:   2021-03-16 08:10:51 +0700
categories: programming cpp cp
---

BFS on Grid (BFS trên lưới, loang trên lưới) là một trong những kiến thức cơ bản của BFS, và hoàn toàn có thể ra trong đề thi OLP 30/4 cho khối 10.

## Đề bài

Cho một lưới kích thước `n x m`. Tìm đường đi **ngắn nhất** từ ô `'S'` đến ô `'D'`. Những ô `'.'` là những ô được đi qua, còn những ô `'#'` thì không.
Từ một ô, chỉ được phép di chuyển sang các ô kề cạnh (trên, dưới, trái, phải, **không được đi chéo**). In ra `-1` nếu không tìm thấy đường đi.<br/>
VD:
```
Input:
#.#S
.#..
#...
D...

Output:
6

Input:
#.#S
.#..
#...
D###

Output:
-1
```

## Phân tích

Việc dùng BFS là ý tưởng hiển nhiên. **BFS luôn cho phép tìm đường đi ngắn nhất giữa 2 điểm bất kì (cạnh không trọng số)**.<br/>
Trước hết, tôi sẽ giới thiệu loại mảng đặc biệt, để hỗ trợ việc di chuyển kề cạnh:
```cpp
const int numDir = 4; // có 4 ô kề cạnh
const int dx[] = {-1,  0,  1,  0};
const int dy[] = { 0,  1,  0, -1};
```
Ý nghĩa của 2 mảng trên để làm gì?<br/>
Giả sử, ta đang ở ô có tọa độ **(x, y)**:

![grid1](/assets/bfs_on_grid/grid1.png)

Di chuyển sang các ô **kề cạnh**, tức là di chuyển sang các ô **(x - 1, y)**, **(x, y + 1)**, **(x + 1, y)**, **(x, y - 1)**:


![grid2](/assets/bfs_on_grid/grid2.png)

Vậy, ta chỉ cần dùng một vòng for đơn giản:
```cpp
int x, y; // tọa độ hiện tại

for (int i = 0; i < numDir; i++) {
    int nx = x + dx[i];
    int ny = y + dy[i];
    // lúc này, (nx, ny) là ô kề với ô (x, y)
    // xử lý với (nx, ny)
}
```

Ta sẽ xây dựng mô hình lời giải như sau:
```cpp
int n, m; // kích thước của lưới
char grid[N][N]; // lưới nhập vào

int stx, sty; // tọa độ ô 'S'
int enx, eny; // tọa độ ô 'D'
queue<pair<int, int>> q;
bool visited[N][N]; // đánh dấu ô đã thăm chưa
int steps[N][N]; // đánh dấu khoảng cách

const int numDir = 4;
const int dx[] = {-1,  0,  1,  0};
const int dy[] = { 0,  1,  0, -1};

bool inside(int x, int y) {
    return (1 <= x && x <= n) && (1 <= y && y <= m);
}

int BFS() {
    q.push(make_pair(stx, sty));
    steps[stx][sty] = 0;

    while (!q.empty()) {
        // tọa độ hiện tại
        int x = q.front().first;
        int y = q.front().second;
        q.pop();

        // tìm được ô đích
        if (x == enx && y == eny) {
            return steps[x][y];
        }

        // xét các ô kề
        for (int i = 0; i < numDir; i++) {
            int nx = x + dx[i];
            int ny = y + dy[i];

            // Lưu ý: (nx, ny) có thể sẽ đi ra khỏi lưới
            // => kiểm tra (nx, ny) có nằm trong lưới không
            if (!inside(nx, ny)) continue;

            // (nx, ny) == '#' => không đi được
            if (grid[nx][ny] == '#') continue;

            // ô (nx, ny) chưa thăm
            if (!visited[nx][ny]) {
                visited[nx][ny] = true;
                steps[nx][ny] = steps[x][y] + 1;
                q.push(make_pair(nx, ny));
            }
        }
    }

    // BFS xong nhưng vẫn chưa tìm được ô đích
    return -1;
}
```

Ngoài ra, nếu các bài toán yêu cầu ta di chuyển sang các ô **kề cạnh** và **kề góc**, ta chỉ cần chỉnh sửa mảng `dx[]` và `dy[]` như sau:
```cpp
const int numDir = 8;
const int dx[]  = {-1, -1,  0,  1,  1,  1,  0, -1};
const int dy[]  = { 0,  1,  1,  1,  0, -1, -1, -1};
```

## Nguồn
* [geeksforgeeks](https://www.geeksforgeeks.org/shortest-distance-two-cells-matrix-grid/)

## Bài tập liên quan
* [VNOI - VMUNCH](https://codeforces.com/group/FLVn1Sc504/contest/274857/problem/W)
* [CSES - Counting Rooms](https://cses.fi/problemset/task/1192)
* [PTIT - BCLKCOUN](https://www.spoj.com/PTIT/problems/BCLKCOUN)
* [CSES - Labyrinth](https://cses.fi/problemset/task/1193)
* [CSES - Monsters](https://cses.fi/problemset/task/1194)
* [Codeforces - 598D](https://codeforces.com/problemset/problem/598/D)
