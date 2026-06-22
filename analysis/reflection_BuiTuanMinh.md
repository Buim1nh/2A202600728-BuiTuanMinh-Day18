# Reflection - Bui Tuan Minh

## Phần 1: Mapping bài giảng

| Lecture Concept | Module | Hàm cụ thể | Observation |
|----------------|--------|-------------|-------------|
| Semantic chunking | M1 | `chunk_semantic()` | "Giúp nhóm các câu có cùng chủ đề dựa trên embedding similarity, không cắt ngang ý nghĩa câu. Với threshold 0.85 tạo chunking ổn định hơn split basic." |
| BM25 + Dense fusion | M2 | `reciprocal_rank_fusion()` | "RRF giải quyết việc kết quả text search và semantic vector không cùng scale, giúp merge kết quả từ hai phương thức hiệu quả." |
| Cross-encoder reranking | M3 | `CrossEncoderReranker.rerank()` | "Mang lại precision cao nhưng tradeoff là latency (chậm hơn so với dot product search thông thường)." |
| RAGAS 4 metrics | M4 | `evaluate_ragas()` | "Đo đạc 4 khía cạnh giúp identify rõ pipeline đang gặp vấn đề do retrieval hay do LLM hallucinate." |
| Contextual embeddings | M5 | `contextual_prepend()` | "Giảm retrieval failure bằng cách thêm title và bối cảnh vào trước chunk để LLM và retriever dễ nhận diện nội dung." |

## Phần 2: Khó khăn & giải quyết

- Lỗi gặp phải: Lỗi khởi động Qdrant do không chạy được Docker Desktop trên máy tính cá nhân ("failed to connect to the docker API at npipe:////./pipe/dockerDesktopLinuxEngine").
- Cách debug: Chuyển đổi mã QdrantClient từ sử dụng Host qua Memory Mode `QdrantClient(location=":memory:")` để bỏ qua sự phụ thuộc vào Container/Docker daemon.
- Kiến thức thiếu: Quên cách sử dụng In-Memory của Qdrant trong Python. Cách bổ sung: Tra cứu document của qdrant_client để tìm ra flag location.

## Phần 3: Action Plan cho project

### Project: AI Knowledge Base cho Hệ thống Học vụ

### Hiện tại
- RAG pipeline hiện tại: Sử dụng LangChain text splitter cơ bản và dense search qua ChromaDB, chưa có reranking và đánh giá.
- Known issues: Đôi khi fetch phải chunk chả liên quan, khiến LLM trả lời sai nội dung hoặc báo không tìm thấy.

### Plan áp dụng
1. [x] Chunking strategy: Semantic Chunking kết hợp với cấu trúc markdown, giúp các quy chế học vụ không bị cắt ghép sai ý.
2. [x] Search: Chuyển qua dùng Hybrid Search (BM25 + BGE Dense) để tận dụng độ chính xác của từ khóa chính tả.
3. [x] Reranking: Có. Sẽ áp dụng bge-reranker-v2-m3 để chốt top 3 kết quả xịn nhất, tránh overload context LLM.
4. [x] Evaluation: Áp dụng RAGAS tự động với tập test QA nội bộ.
5. [x] Enrichment: Áp dụng _enrich_single_call để generate metadata và hyqa.

### Timeline
- Tuần 1: Setup M1 Chunking và cấu hình DB.
- Tuần 2: Thêm M2 Hybrid Search và Reranker, kiểm thử hiệu năng.
- Tuần 3: Đánh giá RAGAS với test QA tự tạo và optimize enrichment.
