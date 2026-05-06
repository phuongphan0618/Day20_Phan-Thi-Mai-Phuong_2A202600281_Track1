| Cột | Vấn đề (Problem) | Sẽ làm (Direction) |
| --- | ---------------- | ------------------ |
| **NOW** | Người dùng không tìm được paper phù hợp với query một cách đáng tin cậy (kết quả search chưa đủ relevance + noise cao) | Tối ưu **Query Paper + Rank relevance** để đảm bảo top result thực sự đáng đọc (improve scoring, filtering, rerank nhẹ) |
| **NOW** | Người dùng mất thời gian đọc abstract nhưng không chắc chắn paper có đáng đọc không | Build **Summarize single paper** để cung cấp quick understanding (main idea, limitation, why it matters) |
| **NEXT** | Người dùng khó nhận ra mối tương quan giữa các paper (method giống nhau, đối lập, kế thừa) | Làm **Highlight relationship between papers** (group theo method / similarity / contrast) |
| **LATER** | Hệ thống chưa đủ khả năng tổng hợp insight cấp cao từ nhiều paper một cách đáng tin cậy | Phát triển **Cross Paper Synthesizer** để tạo overview + theme + gap dựa trên structure đã có |

