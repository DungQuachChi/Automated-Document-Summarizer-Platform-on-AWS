---
title: "Worklog Tuần 13"
date: 2026-06-22
weight: 13
chapter: false
pre: " <b> 1.13. </b> "
---

### Mục tiêu Tuần 13:
* Mở rộng khả năng xử lý tệp tài liệu thực tế (PDF/DOCX) sử dụng Amazon S3 Event Triggers và AWS Lambda Layers.
* Triển khai giải pháp giám sát hệ thống, phân tích hiệu năng và truy vết phân tán toàn diện bằng **AWS X-Ray** và **CloudWatch Alarms**.
* Phát triển giao diện người dùng Web (Frontend) và triển khai tĩnh qua **Amazon S3** & **Amazon CloudFront CDN**.

### Các công việc thực hiện trong tuần:
| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| Thứ 2 | - Tạo S3 Bucket lưu trữ tài liệu đầu vào (documents-upload-bucket) <br> Đóng gói một **AWS Lambda Layer** chứa các thư viện trích xuất văn bản (pypdf / python-docx) | 2026-06-22 | 2026-06-22 | AWS Lambda Layers Guide |
| Thứ 3 | Lập trình luồng xử lý bất đồng bộ S3 Event Notification kích hoạt Lambda trích xuất văn bản <br> Chuyển đổi các định dạng tệp đính kèm được tải lên thành chuỗi văn bản thô trước khi chuyển tới Amazon Bedrock | 2026-06-23 | 2026-06-23 | Amazon S3 Event Notifications |
| Thứ 4 | Bật **AWS X-Ray Tracing** trên API Gateway, Lambda và DynamoDB <br> Cấu hình **CloudWatch Alarms** để giám sát ngưỡng lỗi HTTP 5xx và độ trễ của mô hình AI | 2026-06-24 | 2026-06-24 | AWS X-Ray Developer Guide |
| Thứ 5 | Xây dựng giao diện Web UI đáp ứng (HTML5/TailwindCSS/JS ES6) hỗ trợ Tải tệp lên, Xác thực Cognito và hiển thị kết quả tóm tắt <br> Cấu hình Chia sẻ tài nguyên đa nguồn (CORS) trên API Gateway | 2026-06-25 | 2026-06-25 | Amazon API Gateway CORS |
| Thứ 6 | Triển khai tài nguyên Frontend lên S3 Static Website Hosting tích hợp với **Amazon CloudFront** <br> Thực hiện kiểm thử toàn bộ quy trình End-to-End (E2E) (Đăng ký -> Đăng nhập -> Tải tệp -> Tóm tắt AI -> Truy vấn lịch sử) | 2026-06-26 | 2026-06-26 | Amazon CloudFront Docs |

### Thành tựu chính:
* **Lambda Layer & Trích xuất tài liệu nâng cao:** Đóng gói thành công một Lambda Layer tùy chỉnh chứa pypdf và python-docx để mở rộng khả năng tóm tắt từ văn bản thô sang các tệp PDF/DOCX được tải lên.
* **Luồng xử lý tệp tự động bất đồng bộ:** Cấu hình Amazon S3 Event Notifications tự động kích hoạt các hàm Lambda xử lý ngay khi tệp được tải lên, trích xuất văn bản thô một cách an toàn trước khi đẩy vào quy trình LLM trên Bedrock.
* **Khả năng quan sát hệ thống phân tán (Observability):** Kích hoạt **AWS X-Ray** trên toàn bộ luồng yêu cầu (API Gateway -> Lambda -> Bedrock -> DynamoDB) để tạo Sơ đồ dịch vụ (Service Maps) và truy vết độ trễ. Cấu hình CloudWatch Alarms để phát cảnh báo nếu tỷ lệ lỗi backend vượt quá 5%.
* **Ứng dụng Web Frontend & Bảo mật xác thực:** Xây dựng giao diện Web UI đáp ứng cho phép người dùng xác thực qua Cognito Hosted UI, lấy ID Token an toàn, tải tệp trực tiếp và xem lịch sử các bản tóm tắt đã lưu.
* **Phân phối nội dung toàn cầu qua CloudFront:** Phân phối các tài nguyên Frontend sử dụng **Amazon CloudFront** kết hợp với S3 Bucket Origin, cấu hình các quy tắc CORS trên API Gateway để đảm bảo giao tiếp an toàn giữa các domain khác nhau.
* **Kiểm thử tích hợp End-to-End (E2E):** Hoàn thành kiểm thử toàn diện từ khâu người dùng đăng ký, tải tệp PDF, nhận kết quả tóm tắt từ Claude 3 Haiku dưới 3 giây và truy vấn lại dữ liệu lịch sử từ DynamoDB.