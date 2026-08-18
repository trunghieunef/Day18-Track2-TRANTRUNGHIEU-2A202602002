# REFLECTION — Day 18 Lab

> **Câu hỏi (≤ 200 từ):** Trong "Top 5 Lakehouse Anti-Patterns", data team của bạn
> dễ vướng nhất vào anti-pattern nào, và vì sao?

## Anti-pattern chọn: Bảo trì (maintenance) chạy như "sau nghĩ" — tập trung ghi mà quên job dọn dẹp đi kèm

### Vì sao
Pipeline của team là kiểu `Kafka → lakehouse` với nhiều append nhỏ liên tục, đúng
hoàn cảnh tạo _small-file problem_ như kịch bản 200 batch của NB6. Cái bẫy team dễ
vướng nhất là **tưởng rằng lịch `OPTIMIZE`/`VACUUM` là đủ**, rồi bỏ lỡ **job đi kèm
mà không dashboard nào chỉ ra**.

Lab đã đo ra điều này cụ thể:
1. **`VACUUM` (delta-rs) chỉ thu hồi file bị *tombstone* trong log.** File do writer
   crash ghi xuống mà chưa từng commit thì *vô hình* với vacuum ở mọi retention —
   phải tự viết phép hiệu tập hợp mới dọn được (Job 4).
2. **`expire_snapshots` (Iceberg) chỉ đụng metadata**: hạ 20 → 3 snapshot nhưng
   **0 file avro bị xoá**, metadata còn phình ra. Nghĩa là Job 3 và Job 4 là **một
   cặp** — chạy expiry mà không quét orphan chính là lý do "đã expire mà hoá đơn
   S3 không giảm".

Thế nên, nếu phải đánh đổi, team tôi dễ bỏ nhất bước **orphan sweep**, vì nó không
lộ bằng con số file-count nào quen thuộc — nhưng lại là thứ khiến bill bộ nhớ
không bao giờ giảm.