---
title: "Thuật toán Dijkstra: tìm đường đi ngắn nhất"
date: 2026-05-27 00:00:00 +0700
categories: [Algorithms, Graph]
tags: [dijkstra, graph, shortest-path, python]
---

Thuật toán Dijkstra dùng để tìm đường đi ngắn nhất từ một đỉnh bắt đầu đến các đỉnh còn lại trong đồ thị có trọng số không âm. Đây là một trong những thuật toán nên biết khi học về đồ thị, vì nó xuất hiện trong nhiều bài toán như tìm đường, định tuyến mạng, bản đồ, game, và hệ thống gợi ý.

## Ý tưởng chính

Ta lưu khoảng cách ngắn nhất tạm thời từ đỉnh bắt đầu đến mỗi đỉnh. Ban đầu, khoảng cách đến đỉnh bắt đầu là `0`, còn các đỉnh khác là vô cực.

Mỗi lần, ta chọn đỉnh đang có khoảng cách tạm thời nhỏ nhất, rồi thử cập nhật khoảng cách đến các đỉnh kề của nó. Nếu đi qua đỉnh hiện tại giúp đường đến đỉnh kề ngắn hơn, ta thay đổi giá trị khoảng cách đó.

Quá trình lặp lại cho đến khi không còn đỉnh nào cần xử lý.

## Ví dụ

Giả sử có đồ thị:

```text
A --4-- B
|      /|
2    1  5
|  /    |
C --8-- D
```

Nếu bắt đầu từ `A`:

- Đến `C`: ngắn nhất là `A -> C`, tổng chi phí `2`
- Đến `B`: ngắn nhất là `A -> C -> B`, tổng chi phí `3`
- Đến `D`: ngắn nhất là `A -> C -> B -> D`, tổng chi phí `8`

## Code Python

```python
import heapq


def dijkstra(graph, start):
    distances = {node: float("inf") for node in graph}
    distances[start] = 0

    priority_queue = [(0, start)]

    while priority_queue:
        current_distance, current_node = heapq.heappop(priority_queue)

        if current_distance > distances[current_node]:
            continue

        for neighbor, weight in graph[current_node]:
            new_distance = current_distance + weight

            if new_distance < distances[neighbor]:
                distances[neighbor] = new_distance
                heapq.heappush(priority_queue, (new_distance, neighbor))

    return distances


graph = {
    "A": [("B", 4), ("C", 2)],
    "B": [("A", 4), ("C", 1), ("D", 5)],
    "C": [("A", 2), ("B", 1), ("D", 8)],
    "D": [("B", 5), ("C", 8)]
}

print(dijkstra(graph, "A"))
```

Kết quả:

```text
{'A': 0, 'B': 3, 'C': 2, 'D': 8}
```

## Độ phức tạp

Nếu dùng hàng đợi ưu tiên như trong ví dụ trên, độ phức tạp thường là:

```text
O((V + E) log V)
```

Trong đó:

- `V` là số đỉnh
- `E` là số cạnh

## Lưu ý

Dijkstra chỉ hoạt động đúng khi tất cả trọng số cạnh không âm. Nếu đồ thị có cạnh âm, nên dùng thuật toán Bellman-Ford.

Nói ngắn gọn: Dijkstra rất mạnh khi bạn cần tìm đường ngắn nhất trong một đồ thị có chi phí không âm, và bạn muốn cách làm nhanh, gọn, dễ cài đặt.
