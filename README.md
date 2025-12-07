![ETL pipeline](https://github.com/HUYLAKE/ETL_polars_project/blob/main/ETL_pipeline.img)
# 🚀 High-Performance ETL Pipeline: Polars & Patito

[![Python 3.10+](https://img.shields.io/badge/Python-3.10%2B-blue)](https://www.python.org/)
[![Polars](https://img.shields.io/badge/Polars-Fast-995222)](https://www.pola.rs/)
[![Patito](https://img.shields.io/badge/Patito-Data%20Validation-fe5296)](https://patito.github.io/)

Project này trình bày một quy trình ETL (Extract, Transform, Load) được tối ưu hóa về hiệu suất và chất lượng dữ liệu. Chúng tôi sử dụng thư viện **Polars** để xử lý DataFrame siêu tốc và **Patito** (dựa trên Pydantic) để áp đặt schema và kiểm tra tính hợp lệ của dữ liệu ngay trong pipeline.

## 🌟 Tính Năng Nổi Bật

* **Tốc độ vượt trội:** Tận dụng hiệu suất của Polars, đặc biệt trên các bộ dữ liệu lớn.
* **Data Quality (DQ) First:** Tích hợp Patito để đảm bảo chất lượng dữ liệu là bước bắt buộc, phân tách dữ liệu sạch và dữ liệu lỗi.
* **Khả năng Truy vết Lỗi:** Dữ liệu lỗi được cô lập (`invalid_data_quarantine.csv`) cùng với lý do lỗi để dễ dàng phân tích và khắc phục.

---

## 🛠️ Cài Đặt (Installation)

Yêu cầu Python 3.10 trở lên.

```bash
pip install polars patito
