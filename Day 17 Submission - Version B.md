# Day 17 Submission - Version B

**Họ và tên:** Phan Thị Mai Phương

**Mã học viên:** 2A202600281

---

## Workshop 1 - MVP Boundary Sheet:

**Idea:**
Xây dựng một hệ thống multi-agent RAG để tìm kiếm, đọc và tóm tắt tài liệu học thuật hoặc văn bản chuyên ngành, hỗ trợ tạo survey toàn diện cho các paper.  

**Rickiest Assumption:**
Người dùng sẽ tin và sẵn sàng sử dụng AI-generated summaries (có kèm nguồn) để thay thế một phần việc đọc paper thủ công trong quá trình làm literature review.

### IN-SCOPE: 

(Tính năng cốt lõi bắt buộc để test giả thuyết)

- Query ra top N paper liên quan nhất với một topic cụ thể
- Tạo tóm tắt “đủ để ra quyết định” cho từng bài (bao gồm: tóm tắt + trích dẫn + metadata)
- Cho phép người dùng chấp nhận/loại bỏ bài báo và mở bài gốc để kiểm chứng

### OUT-OF-SCOPE:

(Tính năng tốt nhưng không cần cho MVP) 

- Cross-paper synthesizer (tổng hợp insight giữa nhiều paper)
- So sánh paper theo dimension (method, dataset, result…)
- Highlight mối quan hệ giữa các paper (support / contradict)
- Xử lý full-text PDF nâng cao (bảng biểu, hình ảnh, công thức toán học…)
- UI nâng cao (graph, knowledge map…)
- Gợi ý / tinh chỉnh truy vấn tìm kiếm (query refinement/suggestion)

### NON-GOALS:

(Ranh giới đỏ - tuyệt đối không làm trong giai đoạn này)

- Collaborative workflow giữa nhiều người dùng (group, shared workspace)
- Phân chia task tự động cho nhiều user (task decomposition + assignment)

---

## Workshop 2 - PRD Skeleton:

**Problem Statement:**

Sinh viên năm cuối/nghiên cứu sinh mới bắt đầu thường cần đọc khoảng 15-30 paper để viết một phần literature review có hệ thống. Tuy nhiên, để chọn được paper phù hợp, phải duyêt qua số lượng lớn hơn (ít nhất 40-50 paper), và mỗi paper tốn thêm thời gian để đọc và trích xuất ý chính.

**Target User:**
- Sinh viên năm cuối làm đồ án/khóa luận
- Nghiên cứu sinh làm research 

**User Stories:**

1. As a sinh viên năm cuối đang lần đầu làm survey paper, I want xem danh sách paper liên quan và đọc tóm tắt của từng paper trong ứng dụng/web, so that tôi có thể quyết định giữ hoặc bỏ một paper trong thời gian ngắn hơn so với việc đọc trực tiếp toàn bộ paper.

2. As a sinh viên đang chọn lọc paper cho literature review, I want mở lại tài liệu gốc từ phần tóm tắt để kiểm tra thông tin liên quan, so that tôi có thể xác nhận nội dung trước khi quyết định giữ lại paper mà không phải tìm lại thủ công trong tài liệu.

**Success Metrics:**
- Primary metrics:
  - Thời gian để user tìm được paper đầu tiên thấy “useful” (time-to-first-useful-paper)
  - % paper được user giữ lại (accept) sau khi xem summary
  - % summary được dùng mà không cần đọc lại toàn bộ paper
- Ngưỡng thành công:
  - User có thể tìm được ít nhất 1–2 paper hữu ích trong <5 phút
  - ≥50% paper được user giữ lại sau khi xem summary

**Dependencies & Constraints:**
- API/Service cần tích hợp: 
  - Data API: Semantic Scholar, arXiv
  - LLM API (cho summarization)
- Timeline constraint:
  - MVP cần hoàn thành trong 2–4 tuần
  - Ưu tiên feature core: retrieval + single-paper summary + citation
- Budget constraint:
  - Hạn chế chi phí API (đặc biệt là LLM inference)
  - Ưu tiên giảm số lần gọi model (batch hoặc cache nếu có thể)
- Legal/Compiance constraint:
  - Chỉ sử dụng dữ liệu public (arXiv, Semantic Scholar)
  - Không lưu trữ hoặc phân phối lại full paper nếu vi phạm license
  - Phải giữ nguyên attribution (citation, source link)


### AI-SPECIFIC Section

**Model Selection:**

- Model dự kiến sử dụng: 
  - Primary: DeepSeek (cho summarization cơ bản, tối ưu chi phí)
- Tại sao lại chọn model này (không phải model khác):
  - Chi phí thấp, phù hợp để iterate nhanh trong giai đoạn MVP
  - Đủ khả năng xử lý summarization ở mức cơ bản
- Trade-off chấp nhận được:
  - Summary chưa hoàn toàn chính xác 100%
  - Có thể mất một số chi tiết nhỏ trong paper
  - Chất lượng không đồng đều giữa các paper (tùy độ khó)
- Trade-offs không chấp nhận được:
  - Hallucination không có nguồn (không trace được về paper)
  - Tóm tắt sai ý chính hoặc hiểu sai nội dung kỹ thuật
  - Không cung cấp được source hoặc citation rõ ràng

**Data Requirements:**

- Nguồn dữ liệu cần thiết: arXiv, Semantic Scholar
- Ai sở hữu data này?
- Cần update data theo chu kỳ nào?
  - Không cần tự maintain database riêng trong MVP
  - Dữ liệu được lấy trực tiếp từ API theo thời gian thực (query-time retrieval)
  - Việc cập nhật paper mới phụ thuộc vào nguồn (arXiv, Semantic Scholar)
- Rủi ro về data quality: 
  - Không phải paper nào cũng có full-text → chỉ có abstract
  - PDF parsing có thể lỗi (format khác nhau) -> detect parsing quality, nếu thấp quay về abstract
  - Metadata có thể thiếu hoặc không đồng nhất giữa các nguồn
  - Một số paper có chất lượng thấp nhưng vẫn được retrieve

**Fallback UX:**

Khi AI không chắc chắn hoặc có khả năng sai:
- Hiển thị rõ source (link + đoạn trích nếu có)
- Cho phép user:
  - Bỏ qua paper (reject)
  - Giữ lại paper (accept)
- Ghi nhận feedback để cải thiện ranking trong các lần query sau

---

## Workshop 3 - Hypothesis Table & PMF Scorecard:

### Hypothesis Table

**Tính năng:**
Query ra paper liên quan + tóm tắt từng paper kèm source (citation/highlight)

**Hypothesis:**

"Chúng tôi tin rằng việc cung cấp summary kèm nguồn rõ ràng
sẽ giúp sinh viên năm cuối đang làm literature review
giảm thời gian chọn và hiểu paper phù hợp.
Chúng tôi sẽ biết mình đúng khi thấy
≥50% paper được user giữ lại (accept)
trong vòng 1 session sử dụng."


**Rickiest Assumption phía sau hypothesis này:**
Người dùng sẽ tin và sử dụng AI-generated summary (kèm source) để quyết định giữ hoặc bỏ một paper, thay vì phải đọc lại toàn bộ.

**Cách test assumption này với chi phí thấp nhất:**
- Cho user dùng thử với:
  - 1 query → list paper → summary + source
- Quan sát:
  - user có accept paper chỉ dựa trên summary không
  - user có mở lại full paper không
- Hỏi trực tiếp:
  - “Bạn có cần đọc lại paper này không trước khi giữ lại?”

### PMF Scorecard:

**Aha Moment (mô tả hành vi cụ thể):**
User tìm được ít nhất 1 paper mà họ quyết định giữ lại dựa trên summary + source, và không cần thay đổi quyết định đó sau khi kiểm tra thêm.

**Actionable Metric đo Aha Moment:**
(Không dùng: Số download, lượt view, sign-ups)
- % paper được accept và không bị mở lại hoặc reject sau đó trong một khoảng thời gian.
- Thời gian từ lúc query → accept stable paper đầu tiên.

**Phương pháp đo PMF sẽ dùng:**

- [ ] Sean Ellis Test - ngưỡng: >40% "Very disappointed"
- [ ] Retention Curve - ngưỡng: D30 > __%
- [x] Aha Moment Tracking - ngưỡng: 40-50% user đạt trong session đầu
- [ ] Khác: 

**Thời điểm đo PMF lần đầu:**

Sau 4–6 tuần kể từ khi có 20–50 user sử dụng thực tế

**Vanity Metrics tôi sẽ KHÔNG dùng để tự lừa:**
- Số lượt đăng ký (sign-ups)
- Số lượt query tổng
- Thời gian user ở lại app (không gắn với outcome)




