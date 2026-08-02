---
title: "Đề Xuất: Nền Tảng Tóm Tắt Tài Liệu Serverless Tự Động Trên AWS"
date: 2026-08-02
weight: 2
chapter: false
pre: " <b> 2. </b> "
---


# Đề Xuất: Nền Tảng Tóm Tắt Tài Liệu Serverless Tự Động Trên AWS

## 1. Tổng Quan Dự Án

Dự án này xây dựng một nền tảng tóm tắt tài liệu hoàn toàn serverless, ở mức sẵn sàng sản xuất (production-grade) trên AWS, được phát triển bởi một nhóm gồm hai người. Người dùng đã xác thực gửi một đoạn văn bản thông qua REST API (hoặc một trang web đơn giản) và nhận về bản tóm tắt do AI tạo ra, được hỗ trợ bởi Amazon Bedrock. Mỗi bản tóm tắt được lưu trữ theo từng người dùng, có thể duyệt qua endpoint lịch sử, và được tổng hợp thành báo cáo sử dụng CSV hàng tuần được tạo tự động theo lịch.

Ngoài bản thân ứng dụng, mục đích cốt lõi của dự án là thực hành kỹ thuật cloud hiện đại từ đầu đến cuối: toàn bộ hạ tầng được định nghĩa dưới dạng code bằng Terraform, được triển khai thông qua pipeline CI/CD tự động kèm quét bảo mật, được giám sát bằng dashboard và alarm, và được tăng cường bảo mật theo CIS AWS Foundations Benchmark — tất cả trong giới hạn ngân sách chặt chẽ của sinh viên.

**Phân chia nhóm:** một thành viên phụ trách backend, tích hợp Bedrock, và toàn bộ hạ tầng/CI-CD; người còn lại phụ trách giao diện frontend, logic nghiệp vụ báo cáo hàng tuần, và kiểm thử tải.

## 2. Mục Tiêu

- **Compute serverless:** AWS Lambda (Python) gọi Amazon Bedrock (Amazon Nova Lite) để tóm tắt văn bản. Các lệnh gọi đồng bộ, đầu vào lên đến 5.000 ký tự.
- **Xác thực:** Amazon Cognito với OAuth 2.0 (trang đăng nhập Hosted UI, JWT token) bảo vệ mọi route API.
- **Quản lý API:** Amazon API Gateway (REST) với Cognito authorizer, một API key, và một usage plan áp đặt giới hạn tốc độ và hạn mức hàng tháng.
- **Lưu trữ dữ liệu:** Thiết kế single-table trên Amazon DynamoDB (partition key user_id, sort key timestamp) với một GSI cho các truy vấn báo cáo theo ngày.
- **Báo cáo định kỳ:** Amazon EventBridge cron kích hoạt một Lambda báo cáo hàng tuần, ghi các bản tóm tắt CSV theo từng người dùng vào S3 kèm lifecycle policy chuyển sang Glacier.
- **CI/CD:** AWS CodePipeline + CodeBuild: unit test bằng pytest, quét bảo mật bằng bandit và tfsec, terraform plan, phê duyệt thủ công, sau đó terraform apply.
- **Infrastructure as Code:** Terraform theo module (các module auth, api, compute, data, scheduling, frontend, monitoring, pipeline) với remote state trên S3 và khóa state bằng DynamoDB.
- **Khả năng quan sát:** Dashboard CloudWatch (thời gian thực thi Lambda và độ trễ Bedrock ở p50/p95/p99, lỗi API, mức sử dụng DynamoDB) kèm alarm email qua SNS.
- **Bảo mật & tuân thủ:** IAM theo nguyên tắc least-privilege, CloudTrail đa vùng, AWS Config kèm conformance pack CIS AWS Foundations Benchmark.
- **Demo frontend:** trang HTML/JS tĩnh trên S3 phía sau CloudFront, đăng nhập qua Cognito Hosted UI và gọi API bằng JWT.

## 3. Phát Biểu Vấn Đề

Triển khai các ứng dụng dựa trên AI một cách thủ công rất dễ gãy: việc cấu hình console thủ công gây ra hiện tượng lệch môi trường (environment drift), các dependency không được tài liệu hóa, và các thiết lập không thể tái lập hay rà soát được. Việc gọi các API generative-AI trong production còn đặt ra thêm các vấn đề mà một script đơn giản thường bỏ qua — xác thực và định danh theo từng người dùng, phòng chống lạm dụng và giới hạn tốc độ, lỗi tạm thời của model và giới hạn hạn mức, khả năng quan sát chi phí, và khả năng kiểm toán.

Dự án này giải quyết những vấn đề đó bằng một kiến trúc hoàn toàn serverless, có thể tái lập: mọi tài nguyên đều được quản lý phiên bản bằng Terraform, mọi lần triển khai đều vượt qua các bài kiểm thử tự động và quét bảo mật, mọi request đều được xác thực và giới hạn tốc độ, và luồng gọi AI có tính phòng thủ (retry với exponential backoff, thất bại nhanh khi hết hạn mức). Đối với một nhóm sinh viên, dự án cũng trả lời một câu hỏi thực tế: liệu một backend GenAI ở mức production có thể vận hành với ngân sách vài chục đô-la mỗi tháng hay không?

## 4. Kiến Trúc Giải Pháp

**Luồng request:** người dùng đăng nhập qua Cognito Hosted UI và được chuyển hướng trở lại kèm JWT. Frontend gọi POST /summarize hoặc GET /history đến API Gateway kèm token; Cognito authorizer xác thực token và usage plan áp dụng giới hạn tốc độ trước khi Lambda chạy. Lambda tóm tắt gọi Amazon Nova Lite trên Bedrock, lưu văn bản đầu vào và bản tóm tắt vào DynamoDB theo khóa danh tính người dùng, và trả về bản tóm tắt. Hàng tuần, EventBridge kích hoạt một Lambda thứ hai tổng hợp hoạt động của tuần vừa qua từ DynamoDB thành một báo cáo CSV trong S3.

**Các lớp kiến trúc:**

| Lớp | Dịch vụ |
|---|---|
| User / Edge | HTML/JS tĩnh trên S3 + CloudFront (HTTPS) |
| API | API Gateway REST API, Cognito authorizer, usage plan + API key, CORS |
| Compute | Hai hàm Lambda (Python): bộ tóm tắt đồng bộ, bộ báo cáo định kỳ hàng tuần |
| AI | Amazon Bedrock — Amazon Nova Lite |
| Data | DynamoDB (single table + GSI), bucket S3 báo cáo (AES-256, lifecycle Glacier) |
| Scheduling | Rule cron hàng tuần của EventBridge |
| DevOps | CodePipeline + CodeBuild (pytest, bandit, tfsec, plan/approve/apply), remote state Terraform (S3 + khóa DynamoDB) |
| Bảo mật & Quan sát | CloudTrail (đa vùng), AWS Config + conformance pack CIS, dashboard + alarm CloudWatch, SNS |

**Region chính:** ap-southeast-1 (Singapore).

## 5. Lộ Trình

Giai đoạn thực tập 12 tuần, tháng 4 – tháng 7 năm 2026:

| Tuần | Giai đoạn |
|---|---|
| 1–5 | Học nền tảng AWS: networking, IAM, compute, storage, DNS, container |
| 6 | Khởi động dự án: đề xuất, thiết kế kiến trúc, lựa chọn model, phân chia công việc |
| 7 | Thiết lập môi trường: AWS CLI, Terraform, repository dùng chung, nghiên cứu dịch vụ |
| 8 | Backend cốt lõi: tích hợp Bedrock, parsing phản hồi, lưu trữ DynamoDB, retry, unit test |
| 9 | Lớp xác thực & API: Cognito User Pool + Hosted UI, các route API Gateway, authorizer, usage plan, CORS |
| 10 | Infrastructure as Code: Terraform theo module, remote state kèm khóa |
| 11 | Pipeline CI/CD + khả năng quan sát: CodePipeline, tự động hóa buildspec, metric tùy chỉnh, dashboard, alarm |
| 12 | Tăng cường bảo mật & bàn giao: CIS benchmark, kiểm thử tải, triển khai frontend trên CloudFront, dọn dẹp, tài liệu |

## 6. Ngân Sách

Giới hạn cứng: $50/tháng cho toàn bộ các dịch vụ AWS. Các quyết định thiết kế đều xuất phát từ giới hạn này: dùng DynamoDB on-demand thay vì provisioned capacity, không dùng NAT gateway hay VPC endpoint, dùng mức bộ nhớ Lambda nhỏ nhất khả thi, và bỏ qua hoàn toàn tính năng cache phản hồi của API Gateway (chỉ riêng cụm cache nhỏ nhất đã tốn khoảng $14–19/tháng).

Ước tính chi phí hàng tháng ở mức traffic dự kiến (vài nghìn request):

| Mục | Chi phí ước tính/tháng |
|---|---|
| Lambda (cả hai hàm) | ~$0 (free tier) |
| DynamoDB on-demand + PITR | < $1 |
| API Gateway REST | < $1 |
| Bedrock — token Nova Lite | ~$1 |
| S3 + CloudFront (trang tĩnh + báo cáo) | < $1 |
| Phút CodePipeline + CodeBuild | ~$2 |
| CloudTrail (trail đầu tiên) + recorder & rule AWS Config | ~$3–5 |
| Alarm/metric CloudWatch, SNS | ~$1 |
| **Tổng** | **≈ $8–12 / tháng — nằm khá xa dưới giới hạn $50** |

## 7. Rủi Ro

| Rủi ro | Tác động | Xác suất | Biện pháp giảm thiểu |
|---|---|---|---|
| Quyền truy cập model Bedrock / giới hạn hạn mức on-demand trên một tài khoản AWS mới | Cao | Trung bình | Yêu cầu tăng hạn mức sớm qua AWS Support; giữ một luồng summarize giả lập (mock) để việc phát triển, kiểm thử, và demo không bao giờ bị chặn bởi hạn mức model |
| Vượt ngân sách trên khoản tiền dành cho sinh viên | Trung bình | Trung bình | Ưu tiên mặc định free-tier, billing on-demand, cảnh báo ngân sách, tránh tài nguyên chạy liên tục; kiểm tra mỗi thay đổi so với giới hạn $50 |
| Hai người làm hỏng hạ tầng dùng chung (xung đột state, cấu hình Cognito/CORS) | Trung bình | Trung bình | Remote state Terraform kèm khóa DynamoDB, bảo vệ nhánh (branch protection) kèm rà soát PR, phối hợp trước khi thay đổi tài nguyên dùng chung |
| Cấu hình bảo mật sai (IAM quá rộng, bucket public) | Cao | Thấp | Cổng kiểm tra tfsec + bandit trong pipeline, IAM theo least-privilege, conformance pack CIS, kiểm toán CloudTrail |
| Khả năng sử dụng model/region (model được chọn không được phục vụ ở region chính) | Trung bình | Thấp | Dùng cross-region inference profile; giữ model ID có thể cấu hình qua biến môi trường |