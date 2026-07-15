---
title: "Tuần 12 Worklog"
date: 2026-06-15
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---

### Mục tiêu Tuần 12:
* Triển khai tầng tính toán serverless cốt lõi và tích hợp xử lý văn bản AI bằng Amazon Bedrock.
* Cấu hình các cụm định danh (identity pools) để xác thực người dùng và xây dựng mạng định tuyến API an toàn.
* Áp dụng cơ chế giới hạn tốc độ yêu cầu (rate limiting), các tầng xác thực token và cơ chế cache tối ưu hóa chi phí.

### Các công việc thực hiện trong tuần:
| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 2 | - Yêu cầu cấp quyền truy cập mô hình cho Claude 3 Haiku và Titan Text bên trong vùng `us-east-1` <br> - Thiết lập môi trường ảo Python 3.12 sạch tại local và viết các mã script mô phỏng (mock) | 2026-06-15 | 2026-06-15 | Amazon Bedrock Console |
| 3 | - Lập trình logic xử lý cho hàm Lambda để gọi API của Amazon Bedrock <br> - Viết các bộ xử lý phân tích payload cho các tài liệu dài lên đến 10.000 ký tự | 2026-06-16 | 2026-06-16 | Tích hợp AWS Lambda / Bedrock |
| 4 | - Khắc phục các xung đột cấu hình thông tin xác thực Boto3 tại cục bộ <br> - Triển khai thực thể bảng đơn (single-table) DynamoDB và tích hợp các hook ghi dữ liệu từ Lambda | 2026-06-17 | 2026-06-17 | Amazon DynamoDB Console |
| 5 | - Khởi tạo một **Amazon Cognito User Pool** với tính năng bắt buộc xác thực qua email <br> - Tạo một App Client và kiểm thử các luồng đăng ký người dùng thông qua giao diện Hosted UI | 2026-06-18 | 2026-06-18 | Amazon Cognito Console |
| 6 | - Xây dựng cấu trúc API Gateway REST với các tuyến đường dẫn `/summarize` và `/history` <br> - Đính kèm bộ xác thực Cognito Authorizer và gỡ lỗi sai lệch cấu trúc kiểm tra mã JWT payload <br> - Kết nối các proxy hook của Lambda, triển khai gói giới hạn sử dụng (100 req/phút) và bật bộ nhớ đệm cache 1 giờ | 2026-06-19 | 2026-06-20 | Hướng dẫn Nhà phát triển Amazon API Gateway |

### Thành tựu Tuần 12:
* **Cô lập Pipeline Mô hình AI:** Điều hướng thành công các yêu cầu xử lý mô hình Amazon Bedrock cho Claude 3 Haiku và Titan Text. Xây dựng các cấu trúc Python giả lập (mock) mạnh mẽ cục bộ bên trong môi trường Python 3.12 độc lập bằng Boto3 để bảo vệ tiến độ phát triển dự án trong khi chờ phê duyệt quyền truy cập mô hình từ AWS.
* **Kỹ nghệ hóa Payload & Logic Tính toán:** Phát triển thành công hàm xử lý cốt lõi trên AWS Lambda có khả năng xử lý an toàn các văn bản đầu vào lớn lên đến 10.000 ký tự. Xây dựng các bộ biến đổi đầu vào linh hoạt để ánh xạ chính xác các cấu trúc yêu cầu khác nhau theo quy định của hai công cụ runtime LLM Anthropic và Amazon.
* **Khởi tạo Cơ sở dữ liệu Đơn bảng (Single-Table):** Tạo lập cấu trúc đơn bảng DynamoDB cho dự án, tận dụng `user_id` làm Khóa phân vùng (Partition Key - PK) và `timestamp` làm Khóa sắp xếp (Sort Key - SK). Vượt qua các xung đột về thông tin xác thực trên terminal cục bộ và triển khai các hook ghi dữ liệu tự động bên trong luồng thực thi của Lambda để ghi nhật ký các chỉ số giao dịch một cách mượt mà.
* **Quản lý Định danh chuẩn OAuth 2.0:** Cấu hình các đường dẫn đăng ký và làm quen (onboarding) an toàn cho người dùng bằng cách khởi chạy một Amazon Cognito User Pool bắt buộc xác thực qua email. Triển khai một Cognito App Client chuyên dụng hỗ trợ các luồng cấp mã code (authorization code grant flows) và xác thực các quy trình đăng nhập của người dùng sử dụng giao diện Cognito Hosted UI.
* **Hạ tầng Mạng Định tuyến API An toàn:** Xây dựng một thực thể Amazon API Gateway REST chuẩn production bao gồm các tuyến tài nguyên `/summarize` và `/history`. Bảo mật các endpoint bằng bộ xác thực Cognito Authorizer và xử lý thành công lỗi từ chối xác thực token bằng cách truy vết cấu trúc payload của header yêu cầu (sửa đổi yêu cầu để truyền chính xác Cognito ID Token thay vì Access Token).
* **Kiểm soát Lưu lượng & Tối ưu hóa Chi phí:** Cấu hình các kế hoạch sử dụng (usage plans) của API Gateway để thực thi giới hạn tốc độ nghiêm ngặt ở mức 100 yêu cầu mỗi phút trên mỗi người dùng, bảo vệ các hàm backend khỏi các hành vi lạm dụng hoặc quá tải lưu lượng. Giảm thiểu chi phí gọi mô hình Amazon Bedrock bằng cách kích hoạt tầng bộ nhớ đệm cache 1 giờ trên API Gateway nhằm chặn các yêu cầu tóm tắt văn bản trùng lặp giống hệt nhau.