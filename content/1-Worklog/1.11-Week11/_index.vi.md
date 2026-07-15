---
title: "Worklog Tuần 11"
date: 2026-06-08
weight: 2
chapter: false
pre: " <b> 1.11. </b> "
---
{{% notice warning %}}
⚠️ **Lưu ý:** Các thông tin dưới đây chỉ nhằm mục đích tham khảo, vui lòng **không sao chép nguyên văn** cho bài báo cáo của bạn kể cả warning này.
{{% /notice %}}


### Mục tiêu tuần 11:

* Thiết lập kho lưu trữ mã nguồn (codebase repository) và các pipeline quản lý phiên bản.
* Thiết kế kiến trúc module chi tiết của Terraform và cấu hình lưu trữ trạng thái từ xa (remote state).
* Lập sơ đồ các cấu trúc truy cập dữ liệu và thiết kế các cấu trúc mã nguồn khung (boilerplate) cho ứng dụng.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2 | - Khởi tạo một kho lưu trữ GitHub riêng tư (private) để theo dõi mã nguồn <br> - Cấu hình các chiến lược phân nhánh cho các bản cập nhật hạ tầng | 2026-06-08 | 2026-06-08 | Tài liệu hướng dẫn GitHub |
| 3 | - Thiết kế cấu trúc thư mục phân cấp cho cấu hình định hình **Terraform** dạng mô-đun sắp tới <br> - Phác thảo chiến lược theo dõi trạng thái từ xa trong tệp `backend.tf` | 2026-06-09 | 2026-06-09 | Thực hành tốt nhất với Terraform |
| 4 | - Lập sơ đồ thiết kế schema Đơn bảng (Single-Table) cho DynamoDB <br> - Tài liệu hóa cấu trúc partition key và sort key cho các endpoint xem lịch sử | 2026-06-10 | 2026-06-10 | Hướng dẫn Amazon DynamoDB |
| 5 | - Tạo các thư mục khung chứa mã nguồn cho các hàm AWS Lambda <br> - Nghiên cứu các tham số của Bedrock Boto3 SDK và giới hạn xác thực payload | 2026-06-11 | 2026-06-11 | Tài liệu AWS Boto3 SDK |
| 6 | - Thiết lập môi trường ảo Python cục bộ độc lập (`venv`) <br> - Phác thảo các bản đồ cấu hình hạ tầng ban đầu và xác minh các gói phụ thuộc | 2026-06-12 | 2026-06-12 | Môi trường ảo Python |

### Kết quả đạt được tuần 11:

* **Xây dựng Mô hình Quản lý Phiên bản:** Tạo mã nguồn nền tảng bằng cách thiết lập một kho lưu trữ GitHub riêng tư an toàn, cấu hình các ma trận tệp `.gitignore` để bảo vệ các thông tin xác thực cục bộ và tổ chức các đường dẫn mã nguồn được phân tách rõ ràng.
* **Lập sơ đồ Kiến trúc Hạ tầng IaC:** Định hình một cấu trúc thư mục mô-đun sạch sẽ cho Terraform (gồm các thư mục `auth`, `compute`, `api`, `data`). Xây dựng chiến lược triển khai an toàn tận dụng bộ lưu trữ trạng thái từ xa trên S3 và cơ chế khóa trạng thái (state locking) thông qua DynamoDB.
* **Lập kế hoạch Chiến lược Truy cập Dữ liệu:** Hoàn thành các bản phác thảo kiến trúc cho các lớp truy cập cơ sở dữ liệu, lập kế hoạch luồng dữ liệu để hỗ trợ các truy vấn nhanh của người dùng và các tác vụ báo cáo hàng tuần trước khi bắt đầu khởi tạo tài nguyên thực tế.
