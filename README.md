![ETL pipeline](https://github.com/HUYLAKE/ETL_polars_project/blob/main/ETL_pipeline.img)
# 🚀 High-Performance ETL Pipeline: Polars & Patito

[![Python 3.10+](https://img.shields.io/badge/Python-3.10%2B-blue)](https://www.python.org/)
[![Polars](https://img.shields.io/badge/Polars-Fast-995222)](https://www.pola.rs/)
[![Patito](https://img.shields.io/badge/Patito-Data%20Validation-fe5296)](https://patito.github.io/)

Project này trình bày một quy trình ETL (Extract, Transform, Load) được tối ưu hóa về hiệu suất và chất lượng dữ liệu sử dụng thư viện **Polars** để xử lý DataFrame siêu tốc và **Patito** (dựa trên Pydantic) để áp đặt schema và kiểm tra tính hợp lệ của dữ liệu ngay trong pipeline.

## 🌟 Tính Năng Nổi Bật

* **Tốc độ vượt trội:** Tận dụng hiệu suất của Polars, đặc biệt trên các bộ dữ liệu lớn.
* **Data Quality (DQ) First:** Tích hợp Patito để đảm bảo chất lượng dữ liệu là bước bắt buộc, phân tách dữ liệu sạch và dữ liệu lỗi.
* **Khả năng Truy vết Lỗi:** Dữ liệu lỗi được cô lập (`filtered_data.csv`) cùng với lý do lỗi để dễ dàng phân tích và khắc phục.

---

## 🛠️ Cài Đặt (Installation)

Yêu cầu Python 3.10 trở lên.


Trong tình huống giả định, chúng ta sẽ vào vai 1 data engineer junior và được giao 1 task là xử lý 1 file `data_error.csv`
- Yêu cấu của phòng ban là lọc ra các bảng record lỗi và các record sạch thành 2 file csv riêng biệt( `clean_data.csv` và `filtered_data.csv`) để có thể xử lý vì mỗi record có thể chứa insight quan trọng
- Trong task này ta sẽ dùng Polars làm thao tác chính trong các bước ETL
- Dùng patito để tạo model validate bằng dựa theo các business logic nhằm loại bỏ được những record lỗi đến từ nhập liệu sai

💻 Quy Trình Code Chi Tiết
1. Imports
Nhập các thư viện và module cần thiết cho ETL và validation.

```Python

import patito as pt
import polars as pl
from datetime import date
from typing import Literal
```
2. Định nghĩa Validation Model (Patito)
Tạo mô hình Order bằng Patito để xác định kiểu dữ liệu và các ràng buộc chi tiết cho từng cột (ví dụ: quantity phải là số nguyên lớn hơn hoặc bằng 1).
```python
class Order(pt.Model):
    order_id: str
    customer_id: int
    order_date: date
    quantity: int = pt.Field(ge=1)
    price: float = pt.Field(gt=0)
    total: float = pt.Field(gt=0)
    status: Literal["pending", "processing", "completed", "cancelled"]
```
3. Đọc Dữ Liệu Nguồn (Extract)
Đọc file CSV đầu vào (giả định có chứa lỗi) vào Polars DataFrame.

```python
df = pl.read_csv('data_error.csv')
df
```
4. Xác thực và Phân tách DataFrames (Validate)
Thực hiện validation từng hàng dữ liệu. Dữ liệu sạch được thu thập vào valid_df, dữ liệu lỗi được đưa vào error_df cùng với thông tin lỗi.
```pythonpython
try:
    # Validation nhanh cho toàn bộ DataFrame
    Order.validate(df)
    print('Xác thực thành công')
except pt.DataFrameValidationError as e:
    print(e)
    
error_rows = []
valid_rows = []

# Lặp qua từng hàng để xác thực chi tiết và phân tách
for row in df.to_dicts():
    try:
        validated = Order.model_validate(row)
        valid_rows.append(validated.model_dump())
    except Exception as e:
        # Ghi lại hàng bị lỗi cùng thông báo lỗi chi tiết
        error_rows.append({**row, "error": str(e)})

# Chuyển kết quả trở lại Polars DataFrames
valid_df = pl.DataFrame(valid_rows)
valid_df

error_df =pl.DataFrame(error_rows)
error_df
```
5. Biến đổi Dữ liệu Sạch (Transform)
Thực hiện phép tính toán cột doanh thu (revenue) bằng cách nhân price với quantity. Phép biến đổi này chỉ áp dụng trên dữ liệu đã được xác thực (valid_df).

```Python
gold_df = valid_df.with_columns(
    (pl.col("price") * pl.col("quantity")).alias("revenue")
)
gold_df
```
6. Ghi Dữ Liệu (Load)
Lưu trữ kết quả đã biến đổi và dữ liệu lỗi ra các file CSV riêng biệt.
```python
# Lưu trữ dữ liệu lỗi (Quarantine)
error_df.write_csv('filterd_data.csv')

# Lưu trữ dữ liệu sạch đã được biến đổi (Gold Layer)
gold_df.write_csv('clean_data.csv')
```




