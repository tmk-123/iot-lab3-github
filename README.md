# Bài thực hành số 3 — Trích chọn đặc trưng và học từ dữ liệu IoT

**Học phần:** IoT và Ứng dụng (INT14149)  
**Sinh viên thực hiện:** Trần Mạnh Khiêm — **Mã sinh viên:** B23DCCN437  
**Lớp và nhóm thực hành:** Nhóm 04, tổ 2  

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/tmk-123/iot-lab3-pipeline/blob/main/Bai3_Sinh_vien.ipynb)

---

## 1. Giới thiệu bài toán
Nối tiếp hệ thống thu thập telemetry ở Bài 1 và Bài 2 (ESP32 Wokwi -> ThingsBoard / InfluxDB Cloud):
* Sử dụng dữ liệu chuỗi thời gian 3 kênh cảm biến vật lý: `temperature` (°C), `humidity` (%RH) từ DHT22 và `distance_cm` (cm) từ HC-SR04 với chu kỳ lấy mẫu chuẩn 5 giây.
* Phân chia cửa sổ trượt 60 giây ($N = 12$ mẫu), trích xuất 21 đặc trưng miền thời gian (mean, std, min, max, range, delta, slope).
* Lựa chọn $k = 6$ đặc trưng tốt nhất bằng `SelectKBest(f_regression, k=6)` chỉ fit trên tập Train.
* Huấn luyện mô hình hồi quy `Ridge(alpha=1.0)` với `Pipeline` chống rò rỉ dữ liệu để dự báo nhiệt độ trung bình của 60 giây kế tiếp.

---

## 2. Cấu trúc thư mục

| Tên tệp / thư mục | Mô tả nội dung |
| :--- | :--- |
| `Bai3_Sinh_vien.ipynb` | Notebook Colab chính thức gồm 10 ô code, vượt qua các kiểm tra assert tự động |
| `du_lieu_mau_B2.csv` | File dữ liệu nguồn 720 dòng (60 phút, nhịp 5 giây, đủ 3 kênh cảm biến) |
| `ket_qua_B3/` | Thư mục chứa các tệp kết quả đầu ra |
| ├── `tom_tat.json` | Tệp tổng kết tham số, số lượng mẫu và kết quả đánh giá mô hình |
| ├── `bang_so_sanh.csv` | Bảng số liệu MAE và RMSE của 4 cấu hình |
| ├── `chuoi_cam_bien.png` | Hình 1: Ba chuỗi cảm biến theo thời gian và vạch phân chia Train (70%) - Test (30%) |
| ├── `du_bao_test.png` | Hình 3: So sánh đường thực tế với các mô hình dự báo trên tập Test |
| ├── `dac_trung_train.csv` | Bảng 81 cặp train với 21 cột đặc trưng và nhãn y |
| └── `dac_trung_test.csv` | Bảng 33 cặp test với 21 cột đặc trưng và nhãn y |

---

## 3. Hướng dẫn chạy trên Google Colab

1. Bấm vào nút **Open in Colab** ở đầu trang hoặc truy cập https://colab.research.google.com -> *File -> Open notebook -> GitHub* -> dán link repo: `https://github.com/tmk-123/iot-lab3-pipeline`.
2. Tải tệp `du_lieu_mau_B2.csv` (hoặc `du_lieu_B2.csv`) lên thanh Files bên trái của Colab.
3. Chọn menu **Runtime -> Run all** để chạy toàn bộ 10 ô code.

---

## 4. Kết quả đánh giá trên tập Test

| Cấu hình mô hình | MAE (°C) | RMSE (°C) | Đánh giá |
| :--- | :---: | :---: | :--- |
| **Mean_train** | 2.6040 | 2.8742 | Mốc đối chứng thô, sai số lớn |
| **Persistence** | 0.3205 | 0.4656 | Mốc tham chiếu thực tế mạnh nhờ quán tính nhiệt |
| **Ridge_21** | 0.1388 | 0.1823 | Tốt nhất, tận dụng đầy đủ 21 đặc trưng và độ dốc slope |
| **Ridge_6** | 0.3346 | 0.4500 | Vượt mốc Persistence về chỉ số RMSE (0.4500 < 0.4656) |
