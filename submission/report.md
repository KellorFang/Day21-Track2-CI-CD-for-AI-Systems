# Báo Cáo Lab Day 21 — CI/CD cho AI Systems

**Họ tên:** Phạm Đỗ Ngọc Minh  
**Repo:** https://github.com/KellorFang/Day21-Track2-CI-CD-for-AI-Systems

---

## 1. Bộ siêu tham số đã chọn và lý do

Mô hình sử dụng: **RandomForestClassifier** (scikit-learn).

Qua thực nghiệm Bước 1 với nhiều lần chạy trên MLflow (8 lần chạy, ghi nhận `accuracy` và `f1_score`), bộ siêu tham số cuối cùng được chọn là:

| Tham số | Giá trị | Lý do |
|---|---|---|
| `n_estimators` | 300 | Tăng số cây giúp ổn định kết quả hơn so với 100, không bị overfitting |
| `max_depth` | null (không giới hạn) | Cho phép cây phát triển đầy đủ, phù hợp với tập dữ liệu Wine Quality |
| `min_samples_split` | 2 | Giá trị mặc định, cho phép phân tách ở mức chi tiết nhất |

Bộ tham số này đạt **accuracy ≥ 0.70** trên tập `eval.csv` (500 mẫu held-out), vượt ngưỡng chất lượng của pipeline CI/CD.

---

## 2. Khó khăn gặp phải và cách giải quyết

**Khó khăn 1: SSH key injection trong GitHub Actions**  
Secret `VM_SSH_KEY` chứa newline bị mất khi truyền qua biến môi trường, làm key bị lỗi format. Giải pháp: dùng Python để đọc `os.environ["SSH_KEY"]` và ghi ra file trực tiếp, đảm bảo newline được giữ nguyên.

**Khó khăn 2: Deploy job thất bại do race condition**  
Sau khi `systemctl restart mlops-serve`, service cần thời gian tải model từ GCS (~5 giây), nhưng lệnh `sleep 5 && curl /health` không đủ thời gian chờ trong một số lần chạy. Giải pháp: thay `sleep 5` bằng vòng lặp retry 10 lần × 3 giây (tối đa 30 giây), đảm bảo health check chỉ thực hiện khi service thực sự sẵn sàng.

**Khó khăn 3: `mlflow` không tìm thấy trong terminal**  
Nguyên nhân: MLflow được cài trong virtualenv `.venv` nhưng terminal không kích hoạt venv. Giải pháp: luôn chạy `source .venv/bin/activate` trước khi dùng lệnh MLflow.
