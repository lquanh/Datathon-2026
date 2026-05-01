# Datathon 2026 Round 1 — E-commerce Forecasting Pipeline

README này hướng dẫn cách chạy notebook:

```text
datathon_2026_ecommerce_pipeline.ipynb
```

Notebook dùng dữ liệu lịch sử 2012–2022 để dự đoán daily `Revenue` và `COGS` cho giai đoạn 2023-01-01 đến 2024-07-01.

---

## 1. Mục tiêu của notebook

Pipeline thực hiện các bước chính:

1. Đọc và giải nén dữ liệu từ file `datathon-2026-round-1.zip`.
2. Load các bảng dữ liệu `.csv`.
3. EDA nhanh cho `Revenue` và `COGS`.
4. Tạo đặc trưng theo ngày từ các bảng phụ như `orders`, `order_items`, `products`, `payments`, `returns`, `reviews`, `inventory`, `web_traffic`, `shipments`, `promotions`.
5. Xây dựng seasonal baseline.
6. Tạo supervised features gồm calendar features, seasonal profile, lag, rolling.
7. Train và cross-validation theo chiều thời gian.
8. Train full history đến 2022-12-31.
9. Forecast recursive cho giai đoạn submission.
10. Lưu file submission, CV report và feature importance.

---

## 2. Yêu cầu môi trường

Khuyến nghị dùng:

```text
Python 3.10 hoặc 3.11
Jupyter Notebook / JupyterLab / Google Colab
```

Các thư viện chính:

```bash
pip install numpy pandas matplotlib scikit-learn lightgbm xgboost catboost
```

Notebook hiện có biến:

```python
INSTALL_EXTRA = True
```

Nên nếu thiếu `lightgbm`, `xgboost`, hoặc `catboost`, notebook sẽ tự cài thêm bằng `pip`.

Nếu môi trường không có internet, hãy cài thủ công trước rồi đổi:

```python
INSTALL_EXTRA = False
```

---

## 3. Cấu trúc file cần có

Cách đơn giản nhất là đặt notebook và file zip cùng một thư mục:

```text
project/
│
├── datathon_2026_ecommerce_pipeline.ipynb
└── datathon-2026-round-1.zip
```

File zip cần chứa các file dữ liệu như:

```text
sales.csv
sample_submission.csv
orders.csv
order_items.csv
products.csv
payments.csv
returns.csv
reviews.csv
shipments.csv
inventory.csv
web_traffic.csv
customers.csv
geography.csv
promotions.csv
```

Notebook sẽ tự tìm file zip ở các vị trí sau:

```text
/content/datathon-2026-round-1.zip
/mnt/data/datathon-2026-round-1.zip
datathon-2026-round-1.zip
./datathon-2026-round-1.zip
```

Nếu chạy trên Google Colab, hãy upload file zip vào `/content`.

---

## 4. Cách chạy trên Google Colab

### Bước 1: Upload notebook

Mở Google Colab, sau đó upload file:

```text
datathon_2026_ecommerce_pipeline.ipynb
```

### Bước 2: Upload dữ liệu

Upload file:

```text
datathon-2026-round-1.zip
```

vào thư mục `/content`.

Có thể upload trực tiếp bằng panel bên trái của Colab hoặc dùng đoạn code:

```python
from google.colab import files
files.upload()
```

Sau khi upload, cần đảm bảo file nằm ở:

```text
/content/datathon-2026-round-1.zip
```

### Bước 3: Chạy toàn bộ notebook

Chọn:

```text
Runtime -> Run all
```

Notebook sẽ tự:

1. Cài thư viện còn thiếu.
2. Giải nén dữ liệu vào `/content/datathon_data`.
3. Train model.
4. Sinh file kết quả.

---

## 5. Cách chạy trên máy cá nhân

### Bước 1: Tạo thư mục project

Ví dụ:

```text
D:/Datathon/project/
```

Đặt 2 file vào cùng thư mục:

```text
D:/Datathon/project/datathon_2026_ecommerce_pipeline.ipynb
D:/Datathon/project/datathon-2026-round-1.zip
```

### Bước 2: Tạo môi trường Python

Có thể dùng `conda`:

```bash
conda create -n datathon2026 python=3.10 -y
conda activate datathon2026
```

Hoặc dùng `venv`:

```bash
python -m venv .venv
.venv\Scripts\activate
```

Trên macOS/Linux:

```bash
python -m venv .venv
source .venv/bin/activate
```

### Bước 3: Cài thư viện

```bash
pip install numpy pandas matplotlib scikit-learn lightgbm xgboost catboost jupyter
```

### Bước 4: Mở notebook

```bash
jupyter notebook
```

Sau đó mở file:

```text
datathon_2026_ecommerce_pipeline.ipynb
```

và chọn:

```text
Cell -> Run All
```

Khi chạy local, dữ liệu sẽ được giải nén vào:

```text
./datathon_data
```

---

## 6. Các biến cấu hình quan trọng

Trong cell đầu tiên, có các biến quan trọng:

```python
FAST_MODE = False
TARGETS = ["Revenue", "COGS"]

LAGS_TARGET = [1, 2, 3, 7, 14, 28, 30, 56, 90, 180, 365]
ROLL_WINDOWS_TARGET = [7, 14, 30, 56, 90, 180, 365]

LAGS_EXOG = [1, 7, 14, 30, 90]
ROLL_WINDOWS_EXOG = [7, 30, 90]

MAX_EXOG_LAG_COLS = 70 if FAST_MODE else 140
TOP_MODELS_PER_TARGET = 3
FINAL_BLEND_ML_WEIGHT = 0.90
CLIP_NEGATIVE = True
VALID_YEARS = [2022] if FAST_MODE else [2020, 2021, 2022]
```

Ý nghĩa:

| Biến | Ý nghĩa |
|---|---|
| `FAST_MODE` | Nếu `True`, notebook chạy nhanh hơn nhưng ít fold/model hơn. |
| `TARGETS` | Hai biến cần dự đoán: `Revenue`, `COGS`. |
| `LAGS_TARGET` | Các mốc lag dùng cho target. |
| `ROLL_WINDOWS_TARGET` | Các cửa sổ rolling cho target. |
| `LAGS_EXOG` | Các mốc lag cho biến phụ. |
| `ROLL_WINDOWS_EXOG` | Các cửa sổ rolling cho biến phụ. |
| `MAX_EXOG_LAG_COLS` | Số lượng biến phụ tối đa được chọn. |
| `TOP_MODELS_PER_TARGET` | Số model tốt nhất được blend cho mỗi target. |
| `FINAL_BLEND_ML_WEIGHT` | Trọng số của mô hình ML khi blend với seasonal baseline. |
| `CLIP_NEGATIVE` | Nếu `True`, dự đoán âm sẽ bị đưa về 0. |
| `VALID_YEARS` | Các năm dùng để cross-validation theo thời gian. |

Nếu muốn chạy thử nhanh trước:

```python
FAST_MODE = True
```

Nếu muốn chạy bản đầy đủ để nộp kết quả:

```python
FAST_MODE = False
```

---

## 7. Output sau khi chạy

Sau khi chạy xong, notebook sẽ lưu các file trong thư mục dữ liệu.

Trên Colab:

```text
/content/datathon_data/
```

Trên local:

```text
./datathon_data/
```

Các file output chính:

```text
submission_pipeline.csv
pipeline_report.json
cv_results_detailed.csv
cv_summary.csv
```

Ý nghĩa:

| File | Ý nghĩa |
|---|---|
| `submission_pipeline.csv` | File dự đoán cuối cùng, gồm `Date`, `Revenue`, `COGS`. |
| `pipeline_report.json` | Thông tin cấu hình pipeline và model được chọn. |
| `cv_results_detailed.csv` | Kết quả chi tiết từng fold CV. |
| `cv_summary.csv` | Kết quả trung bình theo từng model và target. |

File quan trọng nhất để nộp là:

```text
submission_pipeline.csv
```

---

## 8. Cách tải output trên Google Colab

Sau khi notebook chạy xong, dùng:

```python
from google.colab import files

files.download("/content/datathon_data/submission_pipeline.csv")
files.download("/content/datathon_data/cv_summary.csv")
files.download("/content/datathon_data/cv_results_detailed.csv")
files.download("/content/datathon_data/pipeline_report.json")
```

---

## 9. Kiểm tra kết quả submission

File submission đúng nên có dạng:

```text
Date,Revenue,COGS
2023-01-01,...
2023-01-02,...
...
2024-07-01,...
```

Có thể kiểm tra nhanh bằng:

```python
import pandas as pd

sub = pd.read_csv("/content/datathon_data/submission_pipeline.csv")
print(sub.shape)
print(sub.head())
print(sub.tail())
print(sub.isna().sum())
```

Nếu chạy local:

```python
sub = pd.read_csv("./datathon_data/submission_pipeline.csv")
```

---

## 10. Lưu ý về data leakage

Notebook có một số cơ chế hạn chế leakage:

1. Cross-validation theo năm, ví dụ train trước năm validation rồi validate trên năm sau.
2. Các target lag đều dùng `shift(1)` trở lên.
3. Khi tạo feature cho validation, seasonal profile chỉ lấy dữ liệu trước ngày validation.
4. Forecast future dùng recursive forecasting, tức dự đoán từng ngày rồi dùng giá trị đã dự đoán làm lag cho ngày tiếp theo.
5. Không dùng trực tiếp `Revenue` hoặc `COGS` của tương lai.

Vì đây là bài toán time series, không nên dùng random split.

---

## 11. Một số lỗi thường gặp

### Lỗi không tìm thấy file zip

Thông báo thường gặp:

```text
FileNotFoundError: Không tìm thấy datathon-2026-round-1.zip
```

Cách sửa:

- Đảm bảo file tên đúng là:

```text
datathon-2026-round-1.zip
```

- Đặt file zip cùng thư mục với notebook nếu chạy local.
- Nếu chạy Colab, upload file zip vào `/content`.

---

### Lỗi thiếu thư viện

Cài lại các thư viện:

```bash
pip install numpy pandas matplotlib scikit-learn lightgbm xgboost catboost
```

Nếu dùng Colab, có thể restart runtime rồi chạy lại từ đầu.

---

### Lỗi XGBoost `gpu_hist`

Nếu gặp lỗi kiểu:

```text
Invalid Input: 'gpu_hist'
```

hãy đổi tham số của `XGBRegressor` thành:

```python
tree_method="hist"
```

Trong notebook hiện tại, tham số này đã được đặt là:

```python
tree_method="hist"
```

---

### Chạy quá lâu

Cách chạy nhanh hơn:

```python
FAST_MODE = True
```

Khi cần chạy bản cuối để lấy điểm tốt hơn, đổi lại:

```python
FAST_MODE = False
```

---

## 12. Quy trình chạy khuyến nghị

Nên chạy theo thứ tự:

1. Chạy với `FAST_MODE = True` để kiểm tra notebook không lỗi.
2. Kiểm tra output có đủ `Date`, `Revenue`, `COGS`.
3. Chạy lại với `FAST_MODE = False`.
4. Lấy file `submission_pipeline.csv` để nộp.
5. Dùng `cv_summary.csv` và `cv_results_detailed.csv` để viết báo cáo kỹ thuật.

---

## 13. Tóm tắt nhanh

```bash
# 1. Cài thư viện
pip install numpy pandas matplotlib scikit-learn lightgbm xgboost catboost jupyter

# 2. Đặt notebook và datathon-2026-round-1.zip cùng thư mục

# 3. Mở notebook
jupyter notebook

# 4. Run All

# 5. Lấy file output
./datathon_data/submission_pipeline.csv
```
