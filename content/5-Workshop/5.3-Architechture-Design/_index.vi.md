---
title : "Thiết kế cấu trúc"
date : 2026-07-14
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### Luồng dữ liệu tổng quan (End-to-End Dataflow)

![overview](/images/5-Workshop/5.3-Architechture-Design/project_arch.jpeg)

#### Luồng CI/CD Pipeline

![overview](/images/5-Workshop/5.3-Architechture-Design/CI_CD_Pipeline.png)

#### Luồng báo cáo hàng tuần (Weekly Report Path)

![overview](/images/5-Workshop/5.3-Architechture-Design/Weekly_Report_Path.png)

#### Bảng dịch vụ

| Dịch vụ | Vai trò | Lý do lựa chọn |
|---|---|---|
| Cognito | Xác thực người dùng, cấp phát JWT | Dịch vụ định danh được quản lý (không cần viết code xác thực riêng, có sẵn Hosted UI) |
| API Gateway | Xác thực request, định tuyến, giới hạn tần suất (rate limiting) | Cổng vào được quản lý (xác thực JWT và usage plan mà không cần middleware tự viết) |
| Lambda | Thực thi logic nghiệp vụ | Trả phí theo lượt gọi, tự động scale về 0, không cần quản lý server |
| Bedrock | Tóm tắt văn bản bằng AI | Truy cập foundation model được quản lý sẵn, không cần tự host model/quản lý GPU |
| DynamoDB | Lưu trữ dữ liệu | NoSQL serverless, API dựa trên HTTP phù hợp với mô hình gọi hàm không trạng thái (stateless) của Lambda, tính phí theo nhu cầu sử dụng |
| S3 | Lưu trữ frontend tĩnh, lưu báo cáo | Lưu trữ object bền vững, chi phí gần như bằng 0 cho các tài nguyên tĩnh nhỏ |
| CloudFront | CDN cho frontend | Chấm dứt HTTPS, cache tại edge, bắt buộc phải có với site S3 công khai |
| EventBridge | Kích hoạt theo lịch cho báo cáo hàng tuần | Cron được quản lý sẵn, không cần server để duy trì lịch chạy |
| CloudWatch + SNS | Số liệu, dashboard, cảnh báo | Tích hợp sẵn với tất cả các dịch vụ AWS khác được dùng trong dự án |
| CloudTrail + Config | Ghi log kiểm toán, quy tắc tuân thủ | Bắt buộc để đáp ứng conformance pack theo CIS Foundations Benchmark |
| CodePipeline / CodeBuild | Tự động hóa CI/CD | Tích hợp gốc của AWS với CodeStar/GitHub, không cần dịch vụ CI riêng biệt |
| Terraform | Hạ tầng dưới dạng mã (Infrastructure as Code) | Khai báo (declarative), quy trình plan-trước-khi-apply, kỹ năng có thể dùng trên nhiều nền tảng cloud |

#### Nền tảng bảo mật và IAM

**Nguyên tắc đặc quyền tối thiểu (least privilege) — vai trò thực thi của Lambda.** IAM role của hàm Lambda chỉ cấp đúng những hành động mà nó cần:

```hcl
resource "aws_iam_role_policy" "lambda_permissions" {
  policy = jsonencode({
    Statement = [
      {
        Effect   = "Allow"
        Action   = ["dynamodb:PutItem", "dynamodb:Query"]
        Resource = aws_dynamodb_table.summarizer.arn
      },
      {
        Effect   = "Allow"
        Action   = ["bedrock:InvokeModel"]
        Resource = "arn:aws:bedrock:us-east-1::foundation-model/amazon.nova-lite-v1:0"
      }
    ]
  })
}
```

Không có wildcard `dynamodb:`, không có wildcard `bedrock:`, không có quyền truy cập vào bất kỳ bảng hay model nào khác. Nếu thông tin xác thực của role này bị rò rỉ, phạm vi ảnh hưởng chỉ giới hạn ở việc ghi/đọc một bảng DynamoDB và gọi một model Bedrock duy nhất.

**IAM role so với user và hardcode key.** Lambda sử dụng một IAM **role** — được hàm Lambda sử dụng trong quá trình thực thi thông qua thông tin xác thực tạm thời được cấp tại thời điểm hàm được gọi; không có IAM user nào với access key tồn tại lâu dài được dùng, ngoại trừ trường hợp truy cập CLI của con người (personal profile). Trong suốt dự án, không có giá trị AWS credentials nào bị hardcode vào code ứng dụng hay commit lên git.

**S3 bucket không nên công khai truy cập.** Bucket chứa frontend không được phép truy cập công khai. CloudFront truy cập bucket thông qua cơ chế **Origin Access Control (OAC)**. Do đó, cách duy nhất để truy cập nội dung là qua giao thức HTTPS bảo mật thông qua CloudFront — truy cập trực tiếp bằng URL S3 bị vô hiệu hóa.

**Đánh đổi bảo mật của Cognito.** Các tính năng bảo mật nâng cao được bật (phát hiện thông tin xác thực bị xâm phạm, đăng nhập dựa trên đánh giá rủi ro). Hosted UI được sử dụng thay vì tự xây dựng form đăng nhập, đánh đổi một phần khả năng tùy chỉnh giao diện để lấy một luồng xác thực được duy trì và vá lỗi bảo mật thường xuyên.

**Cách xử lý secret hiện tại so với hướng nâng cấp.** Hiện tại, API key mà frontend sử dụng được lấy thông qua terraform output và đặt thủ công vào cấu hình môi trường. Hướng nâng cấp trong tương lai sẽ chuyển việc này sang **AWS Secrets Manager** với cơ chế xoay vòng (rotation) tự động, loại bỏ hoàn toàn bước thủ công.

**CIS AWS Foundations Benchmark v1.4 Level 1 — mức độ tuân thủ hiện tại: 60%.** Có hai lỗ hổng đã biết được chấp nhận như những đánh đổi có chủ đích trong bối cảnh một dự án học tập có ngân sách hạn chế:

- IAM role của CodeBuild hiện đang cấp AdministratorAccess thay vì tuân theo policy được giới hạn phạm vi, do bộ quyền cần thiết cho Terraform phụ thuộc vào tài nguyên đang được thay đổi, trong khi việc thiết lập quyền hạn chế chính xác tuyệt đối có mức ưu tiên thấp hơn so với ngân sách dự án (50 đô-la/tháng) và thời gian thực hiện.
- Xác thực đa yếu tố (MFA) đã được thiết lập cho một user quản trị cá nhân (xem lại phần Prerequisites), nhưng chưa được áp dụng bắt buộc ở cấp độ IAM policy cho toàn bộ các tài khoản.
Cả hai vấn đề đều được ghi nhận rõ ràng tại đây, kèm kế hoạch khắc phục ở Mục 6.2 (Simple Security Hardening).

#### Khả năng mở rộng (Scalability)

Mọi lựa chọn về compute trong kiến trúc này đều đã có khả năng tự động scale về 0 và scale trở lại: Lambda scale theo từng request, DynamoDB tính phí theo nhu cầu sử dụng scale theo lưu lượng, API Gateway và CloudFront xử lý các đợt tăng đột biến lưu lượng một cách tự nhiên. Không điều gì trong số này cần can thiệp thủ công ở mức lưu lượng mà workshop này hướng tới.
Đối với môi trường production ở quy mô thực tế, sẽ cần đưa vào các thay đổi sau — được đề cập trong tài liệu này như "hiện chưa cần thiết, nhưng đây là những gì sẽ thay đổi" chứ không phải là hạng mục cần thực hiện trong workshop này:
- **Provisioned concurrency** cho Lambda để loại bỏ độ trễ cold-start trong các giai đoạn tải cao kéo dài.
- **API Gateway response caching** hiện đang bị bỏ qua vì lý do chi phí (giải thích ở Mục 6.1), việc bổ sung tính năng này sẽ giúp giảm số lượt gọi Bedrock đối với các input bị lặp lại khi lưu lượng truy cập cao.
- **Triển khai đa vùng (Multi-region deployment)** để hỗ trợ vấn đề độ trễ và khôi phục sau thảm họa (disaster recovery) — workshop này không yêu cầu điều đó do chỉ vận hành trong một vùng duy nhất (ap-southeast-1), với các lệnh gọi Bedrock chỉ được chuyển hướng sang vùng khác nhằm đảm bảo tính khả dụng của model.