---
title : "Automation — Weekly Reports & CI/CD"
date : 2026-07-14
weight : 1
chapter : false
pre : " 5.6.1. "
---

#### Mục tiêu

Hai thành phần tự vận hành sau khi đã triển khai: một Lambda chạy theo lịch tạo báo cáo sử dụng hàng tuần, và một pipeline CI/CD kiểm thử và kiểm soát mọi thay đổi hạ tầng.

### Pipeline báo cáo hàng tuần (EventBridge)

**Bucket chứa báo cáo** (modules/data): private, được mã hóa SSE-S3, có lifecycle rule chuyển object sang Glacier sau 30 ngày kể từ khi tạo.

**Lập lịch** (modules/scheduling):
- IAM role được giới hạn phạm vi đúng ba hành động: log CloudWatch của chính nó, dynamodb:Query trên bảng + cụ thể là GSI summary-date-index, và s3:PutObject vào bucket chứa báo cáo.
- aws_lambda_function: Python 3.12, timeout 60 giây, 256MB bộ nhớ.
- aws_cloudwatch_event_rule (cron(0 8 ? * MON *), ENABLED) + aws_cloudwatch_event_target + aws_lambda_permission. Cả ba đều được tạo cùng nhau trong một lần apply, nên lịch trình không bao giờ bị thiếu kết nối.

**Logic của Lambda báo cáo**: truy vấn summary-date-index cho từng ngày trong 7 ngày gần nhất, tổng hợp theo user_id, ghi ra một dòng CSV cho mỗi người dùng. Các biến môi trường được lấy từ Terraform output, không gõ thủ công. Nếu không có hoạt động nào trong khoảng thời gian đó → trả về mã 200 kèm thông báo "no data" thay vì tạo một file rỗng.

```bash
terraform apply
# Lambda báo cáo được đặt tên theo dạng <project_name>-report, tức doc-summarizer-report với project_name mặc định
aws lambda invoke --function-name doc-summarizer-report --payload '{}' response.json
aws s3 ls s3://$(terraform output -raw reports_bucket_name)/reports/
```

### Pipeline CI/CD (CodePipeline)

```
GitHub push (main) → Source → Test (pytest, bandit, tfsec, terraform fmt, terraform plan) → Approve (human reviews the plan) → Apply (terraform apply)
                             
```

Đây chính là pipeline được định nghĩa bằng Terraform (modules/pipeline). tfplan.binary được chuyển nguyên trạng từ giai đoạn Test sang Apply; không có việc plan lại sau khi đã được phê duyệt, để đảm bảo những gì đã được xem xét chính là những gì sẽ được apply.

**Thiết lập một lần:**
1. Khởi tạo remote state - bucket S3 có bật versioning và lock table DynamoDB.
2. Cần thực hiện xác thực thủ công một lần cho kết nối GitHub: **CodePipeline > Settings > Connections > Update pending connection**.

**IAM:** Role của CodePipeline vốn đã bị giới hạn chặt chẽ. Role hiện tại của CodeBuild đang gắn AdministratorAccess, điều này được đánh dấu là một khoảng cách đã biết cần thu hẹp phạm vi.
```bash
cd terraform && terraform apply
```
Sau đó push một thay đổi nhỏ lên main và theo dõi các giai đoạn Source → Test → Approve → Apply chuyển sang màu xanh trên console CodePipeline.

#### Các lỗi thường gặp và cách khắc phục

| Lỗi | Nguyên nhân | Cách khắc phục |
|---|---|---|
| Rule EventBridge đã được lên lịch nhưng Lambda không bao giờ chạy | aws_lambda_permission bị thiếu hoặc bị xóa bên ngoài Terraform | terraform plan phải không hiển thị drift; chạy apply để khôi phục |
| Báo cáo không bao giờ được chuyển sang Glacier | Object chưa đủ 30 ngày tuổi, hoặc lifecycle rule bị tắt | aws s3api get-bucket-lifecycle-configuration kiểm tra trạng thái Enabled |
| Giai đoạn Source thất bại ngay lập tức | Kết nối CodeStar bị kẹt ở trạng thái **Pending** | Hoàn tất bước xác thực (handshake) với GitHub |
| Giai đoạn Test thất bại với lỗi state lock | Lần chạy trước không giải phóng lock trên DynamoDB | terraform force-unlock LOCK_ID |
| Giai đoạn Apply thất bại giữa chừng với lỗi về quyền | Role của CodeBuild thiếu quyền cho một loại resource mới được thêm vào | Khoảng cách đã biết dưới AdministratorAccess (xem 5.6.2) |