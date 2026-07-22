+++
title = "Mathematics for Computer Science & Software Engineering"
date = "2026-07-20T07:00:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Toán học không phải là môn học bạn phải "vượt qua" để trở thành lập trình viên.
> Toán học là **ngôn ngữ mô tả và giải quyết vấn đề** trong Computer Science.

## Tài liệu này dành cho ai?

Software Engineer, Backend Engineer, Frontend Engineer, Fullstack Engineer, AI Engineer, Data Engineer, Senior Developer và Solution Architect — những người muốn hiểu **bản chất toán học** đằng sau thuật toán, cấu trúc dữ liệu và hệ thống phần mềm, thay vì chỉ học công thức hoặc học thuộc lời giải.

Bạn **không cần giỏi toán** để đọc tài liệu này. Mọi chương đều đi từ trực giác và First Principles, chứng minh vừa đủ để hiểu bản chất, và luôn gắn với bài toán lập trình hoặc hệ thống thực tế.

## Triết lý trình bày

Mỗi chủ đề đi theo dòng tư duy:

```
Problem
  ↓
Tại sao bài toán này xuất hiện?
  ↓
Tại sao trực giác thông thường thất bại?
  ↓
Toán học giải quyết như thế nào?
  ↓
Thuật toán được xây dựng ra sao?
  ↓
Trade-off
  ↓
Ứng dụng thực tế → Production
  ↓
Sai lầm phổ biến
```

Mỗi chương tuân theo template 10 phần: **Problem Statement → Trực giác → First Principles → Mathematical Model → Thuật toán → Trade-off → Production Applications → Interview → Anti-pattern → Best Practices.**

Ví dụ code viết bằng **Go**.

## Mục lục

### Level 1 – Mathematical Thinking

| # | Chương | Nội dung chính |
|---|--------|----------------|
| 01 | [Logic](/series/math-for-engineers/level-1-mathematical-thinking/01-logic/) | Mệnh đề, suy luận, De Morgan, điều kiện — nền tảng của mọi câu lệnh `if` và query optimizer |
| 02 | [Set Theory, Functions & Relations](/series/math-for-engineers/level-1-mathematical-thinking/02-set-theory-functions-relations/) | Tập hợp, ánh xạ, quan hệ — nền tảng của SQL, type system, hash map |
| 03 | [Proof Techniques](/series/math-for-engineers/level-1-mathematical-thinking/03-proof-techniques/) | Induction, Contradiction, Invariant — vì sao thuật toán và vòng lặp của bạn đúng |
| 04 | [Mathematical Modeling](/series/math-for-engineers/level-1-mathematical-thinking/04-mathematical-modeling/) | Biến bài toán thực tế thành mô hình toán — kỹ năng quan trọng nhất của Solution Architect |

### Level 2 – Discrete Mathematics

| # | Chương | Nội dung chính |
|---|--------|----------------|
| 05 | [Counting & Combinatorics](/series/math-for-engineers/level-2-discrete-mathematics/05-counting-combinatorics/) | Permutation, Combination, Pigeonhole, Inclusion-Exclusion — đếm state space, ước lượng collision |
| 06 | [Recurrence Relations](/series/math-for-engineers/level-2-discrete-mathematics/06-recurrence/) | Quan hệ truy hồi, Master Theorem — ngôn ngữ của đệ quy và chi phí thuật toán |
| 07 | [Trees](/series/math-for-engineers/level-2-discrete-mathematics/07-trees/) | Cây, B-Tree, BST — vì sao database index có dạng cây |
| 08 | [Graph Theory](/series/math-for-engineers/level-2-discrete-mathematics/08-graphs/) | BFS, DFS, Shortest Path, MST, Topological Sort, Network Flow |
| 09 | [Boolean Algebra](/series/math-for-engineers/level-2-discrete-mathematics/09-boolean-algebra/) | Đại số Boole — từ mạch logic đến bitmap index và query rewriting |

### Level 3 – Algorithm Mathematics

| # | Chương | Nội dung chính |
|---|--------|----------------|
| 10 | [Complexity Analysis](/series/math-for-engineers/level-3-algorithm-mathematics/10-complexity-analysis/) | Big-O, Big-Theta, Big-Omega, Amortized Analysis, benchmark thực tế |
| 11 | [Probability](/series/math-for-engineers/level-3-algorithm-mathematics/11-probability/) | Conditional Probability, Bayes, Expected Value — Cache, Load Balancer, Retry, Randomized Algorithms |
| 12 | [Hashing](/series/math-for-engineers/level-3-algorithm-mathematics/12-hashing/) | Hash function, collision, birthday paradox — Redis Hash Table, partitioning |
| 13 | [Divide and Conquer](/series/math-for-engineers/level-3-algorithm-mathematics/13-divide-and-conquer/) | Chia để trị — Merge Sort, MapReduce, tại sao "chia đôi" mạnh đến vậy |
| 14 | [Dynamic Programming](/series/math-for-engineers/level-3-algorithm-mathematics/14-dynamic-programming/) | Optimal substructure, overlapping subproblems — từ Fibonacci đến diff algorithm |
| 15 | [Greedy Algorithms](/series/math-for-engineers/level-3-algorithm-mathematics/15-greedy/) | Exchange argument, matroid intuition — khi nào tham lam là đúng |
| 16 | [Sorting, Searching & Heap](/series/math-for-engineers/level-3-algorithm-mathematics/16-sorting-searching-heap/) | Lower bound Ω(n log n), Binary Search, Heap — vì sao không thể sort nhanh hơn |

### Level 4 – Advanced Mathematics

| # | Chương | Nội dung chính |
|---|--------|----------------|
| 17 | [Linear Algebra](/series/math-for-engineers/level-4-advanced-mathematics/17-linear-algebra/) | Vector, Matrix, Dot Product, Eigenvalue, PCA — nền tảng của AI, Recommendation, Search |
| 18 | [Statistics](/series/math-for-engineers/level-4-advanced-mathematics/18-statistics/) | Percentile, Variance, Confidence Interval, Hypothesis Testing — Monitoring, A/B Testing, Benchmark |
| 19 | [Number Theory](/series/math-for-engineers/level-4-advanced-mathematics/19-number-theory/) | Prime, Modular Arithmetic, GCD, Fast Exponentiation — Cryptography, Hashing, Blockchain |
| 20 | [Information Theory](/series/math-for-engineers/level-4-advanced-mathematics/20-information-theory/) | Entropy, Huffman, Arithmetic Coding — ZIP, Kafka Compression, Storage Engine |
| 21 | [Optimization](/series/math-for-engineers/level-4-advanced-mathematics/21-optimization/) | Branch and Bound, Linear Programming, Gradient Descent — scheduling, resource allocation |
| 22 | [Computational Geometry](/series/math-for-engineers/level-4-advanced-mathematics/22-computational-geometry/) | Distance, Convex Hull, KD-Tree — Maps, GIS, Game, Robotics |

### Level 5 – Production

| # | Chương | Nội dung chính |
|---|--------|----------------|
| 23 | [Probabilistic Data Structures](/series/math-for-engineers/level-5-production/23-probabilistic-data-structures/) | Bloom Filter, HyperLogLog, Skip List — đánh đổi độ chính xác lấy bộ nhớ |
| 24 | [Advanced Data Structures](/series/math-for-engineers/level-5-production/24-advanced-data-structures/) | Trie, Segment Tree, Fenwick Tree, Union-Find, LRU |
| 25 | [Distributed Systems Mathematics](/series/math-for-engineers/level-5-production/25-distributed-systems-math/) | Consistent Hashing, Quorum, CAP, thời gian logic, consensus |
| 26 | [Cryptography Mathematics](/series/math-for-engineers/level-5-production/26-cryptography/) | RSA, ECC, Diffie-Hellman, Hash Functions, Merkle Tree |
| 27 | [Production Case Studies](/series/math-for-engineers/level-5-production/27-production-case-studies/) | PostgreSQL Optimizer, Redis, Kafka, PageRank, Elasticsearch, Git Diff, Kubernetes, Google Maps, Netflix, Blockchain |

## Cách đọc

- **Mới bắt đầu / chuẩn bị phỏng vấn:** đọc tuần tự Level 1 → 3, sau đó chọn chương Level 4–5 theo nhu cầu.
- **Đã có kinh nghiệm, muốn hiểu hệ thống:** đọc chương 10 (Complexity) trước để nắm chuẩn phong cách, rồi nhảy thẳng vào Level 5, quay lại chương nền tảng khi cần.
- **AI/Data Engineer:** ưu tiên 11, 17, 18, 20, 21.
- **Backend/Infra:** ưu tiên 10, 12, 23, 24, 25, 27.

## Một nguyên tắc xuyên suốt

Mỗi khái niệm toán học chỉ thực sự có ý nghĩa khi bạn hiểu: **vấn đề nó được tạo ra để giải quyết, giới hạn của nó, cách nó được hiện thực hóa thành thuật toán, và cách các hệ thống thực tế dùng nó trong production.** Nếu một chương không trả lời được câu hỏi "nếu không có khái niệm này thì điều gì xảy ra?", chương đó đã thất bại.
