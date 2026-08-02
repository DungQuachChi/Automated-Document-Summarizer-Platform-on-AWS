---
title : "Backend Foundation — DynamoDB & Lambda"
date : 2026-07-14
weight : 1
chapter : false
pre : " 5.4.1. "
---

#### Mục tiêu

Xây dựng tầng lưu trữ và tầng tính toán mà mọi thành phần khác trong workshop này đều kết nối tới, cả hai đều được định nghĩa trong Terraform (terraform/modules/data, terraform/modules/compute).

#### Bảng DynamoDB

terraform/modules/data/main.tf:

```
data "aws_caller_identity" "current" {}

#tfsec:ignore:aws-dynamodb-table-customer-key
resource "aws_dynamodb_table" "summarizer" {
  name         = "${var.project_name}-${var.table_name}"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "user_id"
  range_key    = "timestamp"

  attribute {
    name = "user_id"
    type = "S"
  }

  attribute {
    name = "timestamp"
    type = "S"
  }

  # summary_date is written as YYYY-MM-DD by the summarizer Lambda.
  # The weekly report Lambda uses this GSI to fetch all records in a
  # date range across all users (one Query per day, up to 7 for a week).
  attribute {
    name = "summary_date"
    type = "S"
  }

  server_side_encryption {
    enabled = true
  }

  point_in_time_recovery {
    enabled = true
  }

  global_secondary_index {
    name            = "summary-date-index"
    hash_key        = "summary_date"
    range_key       = "timestamp"
    projection_type = "ALL"
  }
}

#tfsec:ignore:aws-s3-enable-bucket-logging
#tfsec:ignore:aws-s3-enable-versioning
resource "aws_s3_bucket" "reports" {
  bucket = "${var.project_name}-reports-${data.aws_caller_identity.current.account_id}"
}

#tfsec:ignore:aws-s3-encryption-customer-key
resource "aws_s3_bucket_server_side_encryption_configuration" "reports" {
  bucket = aws_s3_bucket.reports.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "reports" {
  bucket                  = aws_s3_bucket.reports.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_lifecycle_configuration" "reports" {
  bucket = aws_s3_bucket.reports.id
  rule {
    id     = "archive-to-glacier"
    status = "Enabled"
    filter {}
    transition {
      days          = 30
      storage_class = "GLACIER"
    }
  }
}

```

- Chế độ tính phí PAY_PER_REQUEST: không tốn chi phí khi idle, không cần lên kế hoạch dung lượng.
- Mã hóa dữ liệu khi lưu trữ, khôi phục theo thời điểm.
- GSI trên summary_date: cần thiết để hàm Lambda báo cáo hàng tuần có thể truy vấn "tất cả các bản tóm tắt trong 7 ngày gần nhất trên toàn bộ người dùng", điều mà partition key user_id của bảng gốc không thể xử lý hiệu quả một mình.

#### IAM Role cho Lambda

Log group được tạo thủ công (aws_cloudwatch_log_group, giữ log trong 7 ngày) trước khi hàm Lambda thực sự bắt đầu thực thi, thay vì sử dụng log group được Lambda tự động tạo mà không có thời hạn hết hạn. Timeout được đặt là 30 giây, vì các lệnh gọi Bedrock cùng với việc thử lại cũng cần thêm thời gian ngoài mức mặc định 3 giây.

Nếu request không cung cấp bất kỳ claim nào từ Cognito, user_id sẽ được gán một giá trị hard-code sẵn, hữu ích cho việc kiểm thử thủ công/cục bộ (local/manual testing) và phải được xóa bỏ trước khi triển khai ứng dụng ở môi trường production.

#### Hàm Lambda

Log group được tạo thủ công (aws_cloudwatch_log_group, giữ log trong 7 ngày) và cần được tạo trước khi hàm được gọi, thay vì phụ thuộc vào log group tự động tạo không có thời hạn hết hạn. Thời gian timeout được đặt là 30 giây vì các lệnh gọi Bedrock cùng với thời gian chờ giữa các lần retry cần nhiều thời gian hơn mức mặc định 3 giây.

Nếu request không chứa bất kỳ claim nào từ Cognito, user_id sẽ được đặt thành một giá trị test cụ thể. Tính năng này được dùng cho việc kiểm thử thủ công/cục bộ và cần được loại bỏ trước khi triển khai lên môi trường production thực tế.

#### Triển khai và kiểm thử

```bash
cd terraform
terraform init
terraform apply
```

```bash
aws lambda invoke \
  --function-name doc-summarizer-summarizer \
  --payload '{"httpMethod":"POST","path":"/summarize","body":"{\"text\":\"Amazon Web Services offers cloud computing services worldwide. Businesses use AWS to reduce infrastructure costs and scale applications globally.\"}","requestContext":{"authorizer":{"claims":{"sub":"test-user-123"}}}}' \
  --cli-binary-format raw-in-base64-out \
  response.json
cat response.json
```

**Cách kiểm tra:** file response.json hiển thị "statusCode": 200 kèm theo các trường summary, timestamp và summary_date. Nếu gặp lỗi 502 tại bước này nghĩa là quyền truy cập model Bedrock chưa được cấp (xem thêm ở Mục 5.4.2).

#### Các lỗi thường gặp và cách khắc phục

| Lỗi | Nguyên nhân | Cách khắc phục |
|---|---|---|
| AccessDeniedException trên DynamoDB | dynamodb_table_arn không được truyền đúng vào module compute | Kiểm tra lại root main.tf để đảm bảo module.data.table_arn được nối vào module.compute |
| ResourceNotFoundException | Biến môi trường DYNAMODB_TABLE của Lambda không khớp với tên bảng thực tế | Kiểm tra output module.data.table_name có khớp với giá trị được truyền vào module.compute không |
| terraform apply thất bại khi tạo Lambda: "no such file" | var.lambda_zip_path không trỏ đến gói triển khai (deployment package) đã được build | Nén (zip) thư mục src/lambda_fn/ trước khi chạy terraform apply |