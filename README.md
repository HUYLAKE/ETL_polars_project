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

```bash
pip install polars patito
Trong tình huống giả định, chúng ta sẽ vào vai 1 data engineer junior và được giao 1 task là xử lý 1 file `data_error.csv`
- Yêu cấu của phòng ban là lọc ra các bảng record lỗi và các record sạch thành 2 file csv riêng biệt( `clean_data.csv` và `filtered_data.csv`) để có thể xử lý vì mỗi record có thể chứa insight quan trọng
- Trong task này ta sẽ dùng Polars làm thao tác chính trong các bước ETL
- Dùng patito để tạo model validate bằng dựa theo các business logic nhằm loại bỏ được những record lỗi đến từ nhập liệu sai

Đầu tiên
```bash
pip install polars patito

Import các thư viện cần thiết cho task
```bash
import patito as pt
import polars as pl 
from datetime import date
from typing import Literal 

Tiếp theo tạo model patito để validate 
```bash
class Order(pt.Model):
    order_id: str    # order_id là dạng string và sẽ báo lỗi nếu không nhập hoặc chỉ nhập số
    customer_id: int # customer_id phải là số 
    order_date: date # order_date phải theo định dạng (YYYY-mm-dd)
    quantity: int = pt.Field(ge=1) # quantity phải là số lớn nguyên dương
    price: float = pt.Field(gt=0) # price là số thập phân và lớn hơn 0
    total: float = pt.Field(gt=0)# tổng phải số thập phân và lớn hơn 0
    status: Literal["pending", "processing", "completed", "cancelled"] # status phải nằm trong các trạng thái như ["pending", "processing", "completed", "cancelled"]

Đọc file bằng Polars
```bash
df = pl.read_csv('data_error.csv')
df

Bắt đầu xác mimh dữ liệu và chia các dữ liệu sạch và hỏng ra các dataframe riêng 
```bash
try: 
    Order.validate(df)
    print('Xác thực thành công')
except pt.DataFrameValidationError as e:
    print(e)
error_rows = []
valid_rows = []

for row in df.to_dicts():
    try:
        validated = Order.model_validate(row)
        valid_rows.append(validated.model_dump())
    except Exception as e:
        error_rows.append({**row, "error": str(e)})

valid_df = pl.DataFrame(valid_rows)
valid_df

error_df =pl.DataFrame(error_rows)
error_df

Thực hiện transform trên dataframe chứa dữ liệu sạch `valid_df` lưu vào biến `gold_df`
```bash
gold_df = valid_df.with_columns(
    (pl.col("price") * pl.col("quantity")).alias("revenue")
)
gold_df

Ghi các dataframe ra 2 file csv(clean_data.csv chứa dữ liệu sạch, filtered_data.csv chứa dữ liệu hỏng)
```bash
error_df.write_csv('filterd_data.csv')
gold_df.write_csv('clean_data.csv')




