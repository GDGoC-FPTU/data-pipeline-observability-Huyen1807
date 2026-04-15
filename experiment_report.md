# Báo Cáo Thí Nghiệm Stress Test - Data Quality Impact on AI Agent

**Student ID:** 2A202600211  
**Name:** Nguyen Thi Thanh Huyen  
**Date:** April 15, 2026

---

## 1. Kết Quả Thí Nghiệm

Chạy `agent_simulation.py` với 2 bộ dữ liệu và ghi lại kết quả:

| Scenario | Data Source | Agent Response | Accuracy | Status |
|----------|-------------|---|---|---|
| Clean Data | `processed_data.csv` (3 records, validated) | "Agent: Based on my data, the best choice is **Laptop at $1200**." | 100% ✅ | SUCCESS |
| Garbage Data | `garbage_data.csv` (5 records, with issues) | "Agent: Based on my data, the best choice is **Nuclear Reactor at $999999**." | 0% ❌ | FAILED |

---

## 2. Phan Tich & Nhan Xet

### Tại sao Agent Trả Lời Sai Khi Dùng Garbage Data?

**Kết quả:**
- **Clean Data:** Agent chọn "Laptop at $1200" ✅ (đúng)
- **Garbage Data:** Agent chọn "Nuclear Reactor at $999999" ❌ (sai)

**Phân Tích Chi Tiết:**

1. **Outlier Problem - Giá Cực Cao:** Nuclear Reactor có giá $999,999 là một outlier hoàn toàn không hợp lý trong thực tế. Agent sử dụng logic `max(price)` để tìm sản phẩm "tốt nhất", nên nó chọn giá cao nhất mà không kiểm tra tính hợp lý. ETL pipeline sạch đã loại bỏ những sản phẩm không hợp lệ, nhưng garbage data không có validation nên outlier vẫn tồn tại.

2. **Duplicate ID Problem:** Garbage data có 2 bản ghi với ID=1 (Laptop và Banana). Điều này gây ra inconsistency, khiến database logic hoặc agent có thể confused. Validation phase trong ETL nên kiểm tra unique constraints để phát hiện và loại bỏ duplicates.

3. **Type Mismatch Problem:** Bản ghi "Broken Chair" có price="ten dollars" (string) thay vì số. Khi agent cố gắng so sánh hoặc tính toán, nó có thể gặp lỗi type coercion hoặc lỗi parsing. ETL pipeline nên validate data types trong Extract phase và coerce chúng đúng cách.

4. **Null Values Problem:** Ghost Item có ID=None và Category=None, điều này vi phạm data integrity. Validation phase nên remove bất kỳ bản ghi nào có NULL trong các cột khóa (key columns). Null values có thể gây ra NaN errors khi agent xử lý dữ liệu.

5. **Invalid Price (Zero):** Ghost Item có price=0, đây là giá không hợp lệ. ETL validation rule `price > 0` sẽ loại bỏ bản ghi này, nhưng garbage data không có quy tắc validation nên nó vẫn xuất hiện.

**Kết Luận:** Data Quality là yếu tố CRITICAL cho hiệu suất Agent. Dữ liệu sạch (từ ETL Pipeline) cho phép agent tập trung vào business logic mà không phải xử lý edge cases. Dữ liệu rác (garbage data) chứa outliers, duplicates, type mismatches, và nulls, khiến agent không thể hoạt động đúng cách.

---

## 3. Kết Luận

**Quality Data > Quality Prompt?** 

**Đồng ý 100%.** Bài lab này chứng minh rằng **chất lượng dữ liệu (Data Quality) quan trọng hơn chất lượng prompt (Prompt Quality)**. 

- Với cùng 1 agent logic và cùng 1 query, kết quả hoàn toàn khác nhau tùy vào chất lượng dữ liệu đầu vào.
- Agent đơn giản đã hoạt động tốt (100% accuracy) với clean data, nhưng thất bại hoàn toàn (0% accuracy) với garbage data.
- ETL Pipeline với validation & transformation rules là chìa khóa để đảm bảo data quality, cho phép AI agent hoạt động hiệu quả.

**Gợi ý cải thiện:** Để xử lý garbage data tốt hơn, ta cần thêm outlier detection (e.g., remove prices > 10x median), deduplication, type validation, và null value handling vào ETL pipeline. AI agents nên được design với defensive logic để handle edge cases, nhưng foundation là phải có dữ liệu sạch từ ETL.
