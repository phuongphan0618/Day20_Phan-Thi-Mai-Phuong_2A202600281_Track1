# Dependencies Map
## Feature 1 — Query Paper + Rank Relevance

### Tier 1 — Critical Dependencies

| Dependency                               | Bạn đang ký sinh trên     | Nếu nó chết                              |
| ---------------------------------------- | ------------------------- | ---------------------------------------- |
| Paper API (Semantic Scholar / Arxiv API) | Data source chính         | Không có paper để search                 |
| Embedding / LLM API (OpenAI / Gemini)    | Search ngữ nghĩa / rerank | Rank mất semantic, gần như keyword-based |
| Vector DB (MongoDB)                      | Truy xuất embedding       | Không query semantic được                |

### Tier 2 — Important Dependencies

| Dependency      | Risk              |
| --------------- | ----------------- |
| Rate limit API  | query chậm / fail |
| Network latency | UX delay, user bỏ |

**Mitigation**:

* Cache kết quả query phổ biến
* Giảm tần suất gọi API (debounce / batching)
* Batch request khi có thể

### Tier 3 - Watch Dependencies
| Dependency                              | Risk         |
| --------------------------------------- | ------------ |
| Metadata quality (paper thiếu abstract) | rank sai nhẹ |

**Mitigation**:
* Filter paper thiếu abstract
* Lọc hoặc giảm trọng số paper thiếu thông tin quan trọng (abstract, venue)


### Plan B (phải deploy được trong 24h)
#### 1. Khi Paper API không khả dụng

**Fallback: Cached / lightweight data source**

* Dùng cache local: top N papers theo domain/query phổ biến
* Dùng ArXiv RSS hoặc dump JSON đơn giản

**Trade-off:** Tuy không còn real-time, coverage hạn chế, nhưng user vẫn có kết quả để tìm hiểu thêm

#### 2. Khi Embedding / LLM API không khả dụng

**Fallback: Keyword-based retrieval**

* Chuyển sang tìm kiếm dựa trên từ khóa
* Kết hợp rule-based scoring đơn giản

**Trade-off:** Tuy mất semantic understanding, nhưng vẫn đảm bảo relevance ở mức cơ bản

#### 3. Khi retrieval layer gặp sự cố

**Fallback: Small candidate set + simple ranking**

* Giới hạn tập paper nhỏ từ cache hoặc filter
* Áp dụng scoring đơn giản để chọn top kết quả

**Trade-off:** Tuy không scale tốt, độ chính xác giảm, nhưng vẫn đủ cho use case nhỏ, thay vì hệ thống đứng hình

## Feature 2 - Summarize Single Paper

### Tier 1 — Critical Dependencies

| Dependency                          | Vai trò                  | Nếu gặp sự cố                         |
| ----------------------------------- | ------------------------ | ------------------------------------- |
| LLM API (OpenAI / Gemini)           | Tạo nội dung tóm tắt     | Không thể sinh summary                |
| Input text (abstract / PDF parsing) | Cung cấp dữ liệu đầu vào | Summary thiếu, sai hoặc không tồn tại |

### Tier 2 — Important Dependencies

| Dependency    | Rủi ro                                     |
| ------------- | ------------------------------------------ |
| Prompt design | Tóm tắt lệch trọng tâm, thiếu ý chính      |
| Token limit   | Mất thông tin quan trọng khi input quá dài |

**Mitigation**:

* Rút gọn nội dung đầu vào một cách có kiểm soát (ưu tiên phần quan trọng)
* Chuẩn hóa format đầu ra để đảm bảo consistency giữa các lần chạy

### Tier 3 — Watch Dependencies

| Dependency          | Rủi ro                                  |
| ------------------- | --------------------------------------- |
| Model quality drift | Output không ổn định giữa các thời điểm |

**Mitigation**:

* Cố định phiên bản model sử dụng
* Kiểm tra định kỳ chất lượng output qua các sample cố định


### Plan B (phải deploy được trong 24h)
#### 1. Khi LLM API không khả dụng

**Fallback: Extractive summary (không dùng AI sinh)**

Thay vì sinh nội dung mới, hệ thống sẽ:

* Trích xuất trực tiếp 2–3 câu quan trọng nhất từ abstract
* Hiển thị dưới dạng - Key sentences from abstract

**Trade-off:** Tuy không còn "tóm tắt thông minh", nhưng vẫn hỗ trợ được user hiểu nhanh paper gồm những yếu tố cốt lõi nào như method, lý do cần đọc kỹ/skip paper


#### 2. Khi không có đủ dữ liệu đầu vào (abstract thiếu / parse PDF lỗi)

**Fallback: Metadata-based preview**

Hiển thị các thông tin có sẵn:
* Title
* Venue / source
* Citation count
* Keywords / method signals

Kèm thông báo rõ ràng:

> **“Abstract not available — showing metadata instead.”**

**Trade-off:** Tuy không có nội dung để hiểu sâu, nhưng vẫn giúp được user đánh giá độ uy tín (venue/citation), đoán nhanh hướng nghiên cứu.

---

# Critical Path

**CRITICAL PATH (quyết định launch)**
```
Data Acquisition
   → Data Processing & Filtering
   → Query & Ranking System
   → Summarization Engine
   → API Integration
   → Launch
```

**NON-CRITICAL (có buffer, làm song song)**
```
UI/UX Design          → (song song với Data Processing)
Marketing Site        → (song song với Ranking/Summarization)
Documentation         → (song song với API Integration)
Analytics Setup       → (trước Launch, có thể trễ nhẹ)
```
