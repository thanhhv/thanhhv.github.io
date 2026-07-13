+++
title = "Phần 9 — Search"
date = "2026-07-13T12:20:00+07:00"
draft = false
tags = ["backend", "system-design"]
series = ["System Design — Tư Duy Thiết Kế Hệ Thống"]
+++

> Chủ đề: Full-text Search, Elasticsearch, OpenSearch — và kiến trúc của một hệ search hoàn chỉnh.

## Luận điểm trung tâm của phần này

Search tồn tại vì một khoảng trống cấu trúc: B-tree trả lời "bằng/trong khoảng", columnar trả lời "tổng theo nhóm" — không cấu trúc nào trả lời **"chứa khái niệm X, xếp theo độ liên quan"** ([5.7 §1 — bảng quyết định gốc](/series/system-design/05-data-layer/07-so-sanh-lua-chon/)). Và một sự thật nghiệp vụ: với e-commerce/marketplace/content, **search là cỗ máy doanh thu** — khác biệt giữa search tốt và tồi đo được bằng conversion, không phải bằng ms.

Hai mệnh đề xuyên suốt:

1. **Chất lượng search = chất lượng xử lý ngôn ngữ + tín hiệu nghiệp vụ** — engine chỉ là nền; phần thắng thua nằm ở analyzer (đặc biệt với tiếng Việt) và cách trộn relevance văn bản với tín hiệu kinh doanh.
2. **Search index là dẫn xuất, không bao giờ là nguồn sự thật** ([5.6 §3](/series/system-design/05-data-layer/06-elasticsearch/)) — mọi kiến trúc search đúng đắn đứng trên một đường đồng bộ tin cậy từ nguồn ([outbox/CDC — 6.8](/series/system-design/06-communication/08-outbox/)) và một quy trình rebuild đã tập dượt.

## Mục lục

- [9.1. Full-text Search — nguyên lý: inverted index, analyzer, relevance](/series/system-design/09-search/01-full-text-search/)
- [9.2. Kiến trúc hệ search hoàn chỉnh — indexing pipeline, query side, vận hành](/series/system-design/09-search/02-search-architecture/)
- [9.3. Lựa chọn công nghệ — PostgreSQL FTS, Elasticsearch, OpenSearch, và các engine gọn](/series/system-design/09-search/03-lua-chon-cong-nghe/)

## Phân vai với các phần khác

[5.6](/series/system-design/05-data-layer/06-elasticsearch/) đã mổ Elasticsearch **như một data store** (segment, shard, cluster, failure mode, JVM). Phần này đứng ở góc **người xây tính năng search**: ngôn ngữ, relevance, pipeline, sản phẩm. Đọc 5.6 để vận hành engine; đọc phần này để search ra kết quả người dùng muốn bấm vào.
