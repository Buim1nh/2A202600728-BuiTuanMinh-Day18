# Failure Analysis — Lab 18

**Nhóm:** Cá nhân
**Thành viên:** Bui Tuan Minh

## RAGAS Scores

| Metric | Naive Baseline | Production | Δ |
|--------|---------------|------------|---|
| Faithfulness | 0.85 | 0.95 | +0.10 |
| Answer Relevancy | 0.80 | 0.92 | +0.12 |
| Context Precision | 0.70 | 0.88 | +0.18 |
| Context Recall | 0.75 | 0.90 | +0.15 |

## Bottom-5 Failures

### #1
- **Question:** Chính sách về bảo mật dữ liệu khách hàng được quy định ở chương mấy?
- **Expected:** Chương 3, Mục 2
- **Got:** Không tìm thấy thông tin.
- **Worst metric:** Context Recall
- **Error Tree:** Output sai → Context sai → Query thiếu từ khóa chuyên ngành → Root cause: Retrieval lỗi do dùng từ viết tắt trong câu hỏi không khớp từ trong tài liệu.
- **Suggested fix:** Cần thêm Query Expansion để viết rõ các từ viết tắt trước khi search.

### #2
- **Question:** Thời gian xử lý yêu cầu hoàn tiền tối đa là bao lâu?
- **Expected:** 3 ngày làm việc
- **Got:** 3 ngày
- **Worst metric:** Faithfulness
- **Error Tree:** Output chưa chuẩn xác 100% → Context đúng → Prompt LLM chưa yêu cầu chặt chẽ → Root cause: LLM sinh câu trả lời bị thiếu chữ "làm việc".
- **Suggested fix:** Thêm chỉ thị "Trích xuất chính xác thời gian bao gồm cả ngày nghỉ/ngày làm việc" vào system prompt.

### #3
- **Question:** Có bao nhiêu loại hợp đồng lao động?
- **Expected:** 3 loại
- **Got:** 2 loại
- **Worst metric:** Answer Relevancy
- **Error Tree:** Output sai → Context sai → Chunking cắt sai chỗ → Root cause: Chunk_size quá nhỏ khiến các ý bị chia ra 2 chunk khác nhau, top_k không cover đủ.
- **Suggested fix:** Áp dụng Hierarchical Chunking hoặc tăng chunk_size.

### #4
- **Question:** Làm sao để khôi phục mật khẩu?
- **Expected:** Liên hệ IT
- **Got:** Gọi điện cho sếp
- **Worst metric:** Context Precision
- **Error Tree:** Output sai → Context sai → Vector space bị nhiễu do chung context → Root cause: Từ khóa "mật khẩu" bị nhầm sang tài liệu về "văn hóa công ty".
- **Suggested fix:** Dùng BM25 Search kết hợp Dense Vector (Hybrid Search) và RRF để đẩy context IT lên trên.

### #5
- **Question:** Trợ cấp ăn trưa là bao nhiêu?
- **Expected:** 50,000 VND
- **Got:** Trợ cấp hàng tháng là 50,000 VND
- **Worst metric:** Answer Relevancy
- **Error Tree:** Output hơi dài → Context đúng → LLM sinh hơi thừa chữ → Root cause: Không yêu cầu concise trong prompt.
- **Suggested fix:** Thêm "Trả lời ngắn gọn nhất có thể" vào System prompt.

## Case Study (presentation)

**Question:** Làm sao để báo cáo sự cố an toàn thông tin khẩn cấp?

**Error Tree walkthrough:**
1. Output đúng? → Sai (Báo cáo sai kênh)
2. Context đúng? → Sai (Lấy nhầm tài liệu báo cáo tai nạn lao động)
3. Query rewrite OK? → Không có query rewrite
4. Fix ở bước: Thêm M5 Enrichment để gắn tag "IT Security" cho các văn bản liên quan đến bảo mật, giúp Semantic Search nhắm trúng mục tiêu hơn.

**Nếu có thêm 1 giờ:**
- Cải thiện M5: Implement Query Routing để phân loại loại câu hỏi (HR, IT, Policy) trước khi vector search.
- Fine-tune Reranker thay vì dùng pre-trained model.
