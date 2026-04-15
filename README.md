[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=23574061&assignment_repo_type=AssignmentRepo)
# Day 10 Lab: Data Pipeline & Data Observability

**Student ID:** 2A202600211  
**Name:** Nguyen Thi Thanh Huyen  
**Email:** nguyennguyen2004555@gmail.com

---

## Mô Tả

Bài lab này xây dựng một **ETL Pipeline tự động** để xử lý và làm sạch dữ liệu JSON. Pipeline gồm 4 bước chính:
1. **Extract:** Đọc dữ liệu từ file JSON (`raw_data.json`)
2. **Validate:** Kiểm tra chất lượng, loại bỏ records không hợp lệ (price <= 0, category rỗng)
3. **Transform:** Chuẩn hóa dữ liệu (tính discounted_price = price * 0.9, chuyển category sang Title Case, thêm timestamp `processed_at`)
4. **Load:** Lưu kết quả ra file CSV (`processed_data.csv`)

Bài lab cũng minh chứng tầm quan trọng của **Data Observability** (theo dõi dữ liệu) bằng cách so sánh hiệu suất của AI agent khi xử lý dữ liệu sạch vs dữ liệu rác (garbage data).

---

## Cách Chạy (How to Run)

### Prerequisites
```bash
pip install pandas pytest
```

### Chạy ETL Pipeline
```bash
python solution.py
```

**Output:**
- File `processed_data.csv` được tạo với 3 records hợp lệ (2 records lỗi bị loại)
- Logs hiển thị số records processed và dropped

### Chạy Agent Simulation (Stress Test)
```bash
# Tạo dữ liệu rác
python generate_garbage.py

# Chạy agent với clean data và garbage data
python agent_simulation.py
```

### Chạy Tests
```bash
pytest tests/test_autograder.py -v
```

---

## Cấu Trúc Tệp

```
├── solution.py                 # ETL Pipeline (Extract, Validate, Transform, Load)
├── processed_data.csv          # Output của pipeline (clean data - 3 records)
├── garbage_data.csv            # Dữ liệu "rác" cho stress test
├── experiment_report.md        # Báo cáo kết quả stress test
├── agent_simulation.py         # AI agent simulation
├── generate_garbage.py         # Tạo dữ liệu rác
├── raw_data.json               # Dữ liệu đầu vào (5 records)
├── README.md                   # File này
└── tests/
    └── test_autograder.py      # Tests tự động (chấm điểm)
```

---

## Kết Quả

### Input Data (`raw_data.json`)
- **Total records:** 5
- **Valid records:** 3 (Laptop, Chair, Monitor)
- **Invalid records:** 2 (1 record có price = -10, 1 record có category rỗng)

### Output Data (`processed_data.csv`)
- **Total records:** 3
- **Columns:** id, product, price, category, discounted_price, processed_at
- **Data Quality:** All records have price > 0 and non-empty category

**Sample Output:**
| id | product | price | category   | discounted_price | processed_at |
|---|---------|-------|------------|---|---|
| 1  | Laptop  | 1200  | Electronics | 1080.0 | 2026-04-15T14:36:19.927680 |
| 2  | Chair   | 45    | Furniture  | 40.5   | 2026-04-15T14:36:19.927680 |
| 5  | Monitor | 300   | Electronics | 270.0  | 2026-04-15T14:36:19.927680 |

### Stress Test Results (Agent Simulation)
- **Clean Data:** Agent correctly identifies "Laptop at $1200" as best electronic product ✅ (100% accuracy)
- **Garbage Data:** Agent incorrectly identifies "Nuclear Reactor at $999999" due to outliers ❌ (0% accuracy)

**Key Finding:** Data quality is MORE IMPORTANT than prompt quality for AI agent performance.

---

## Kỹ Thuật Sử Dụng

- **Language:** Python 3.x
- **Libraries:** `pandas`, `json`, `datetime`, `csv`
- **Testing:** `pytest`
- **Data Format:** JSON input → CSV output

---

## Tổng Quan ETL Pipeline

### Extract Phase
- Đọc file JSON bằng `json.load()`
- Xử lý lỗi nếu file không tồn tại

### Validate Phase
- **Rule 1:** Loại bỏ records có `price <= 0`
- **Rule 2:** Loại bỏ records có `category` rỗng hoặc None
- **Output:** Danh sách records hợp lệ + số lượng records bị loại

### Transform Phase
- Tạo DataFrame từ clean records
- Tính `discounted_price = price * 0.9` (giảm 10%)
- Chuyển `category` sang Title Case (e.g., "electronics" → "Electronics")
- Thêm cột `processed_at` với timestamp ISO 8601

### Load Phase
- Lưu DataFrame ra file CSV
- Format: `df.to_csv(output_path, index=False)`

---

## Chấm Điểm (Rubric - 100 points)

| Test | Yêu Cầu | Điểm |
|-----|---------|------|
| test_script_runs | Script chạy không lỗi & tạo CSV | 20 |
| test_validation | Loại bỏ price <= 0 & empty category | 10 |
| test_transformation | discounted_price = price*0.9, Title Case | 10 |
| test_logging | In ra processed/dropped counts | 10 |
| test_timestamp | processed_at column trong CSV | 10 |
| test_report_exists | experiment_report.md + bảng | 10 |
| test_report_analysis | Phân tích >= 50 từ | 10 |
| test_structure | Files bắt buộc tồn tại | 10 |
| test_readme | README >= 200 chars + instructions | 10 |
| **TOTAL** | | **100** |

---

## Ghi Chú & Lưu Ý

1. **Data Quality Matters:** Clean data từ ETL pipeline cho phép AI agent hoạt động hiệu quả. Garbage data (với outliers, duplicates, nulls) khiến agent thất bại.

2. **Validation Rules:** ETL pipeline loại bỏ ~40% dữ liệu không hợp lệ (2/5 records từ raw data).

3. **Observability:** Logging hiển thị:
   - Số records được xử lý (processed)
   - Số records bị loại (dropped)
   - Timestamp khi xử lý (processed_at)

4. **Testing:** Chạy `pytest` để kiểm tra tất cả 9 test cases.

---

## Thực hiện bởi
**Nguyen Thi Thanh Huyen** (2A202600211)  
**Ngày:** April 15, 2026

