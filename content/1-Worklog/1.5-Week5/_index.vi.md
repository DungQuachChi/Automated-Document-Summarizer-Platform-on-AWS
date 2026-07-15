---
title: "Worklog Tuần 5"
date: 2026-04-27
weight: 1
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:

* Bảo mật tài nguyên đám mây trên các tầng mạng, định danh, dữ liệu và môi trường hệ thống.
* Đi sâu vào quản trị tuân thủ (compliance), hệ thống tường lửa tập trung và tự động phát hiện mối đe dọa.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2 | - Nghiên cứu Liên kết định danh qua **AWS Single Sign-On (AWS IAM Identity Center)** <br> - Làm chủ **IAM Permission Boundaries** và các chính sách nâng cao kèm điều kiện (Conditions) | 2026-04-27 | 2026-04-27 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Đánh giá trạng thái bảo mật cốt lõi qua **AWS Security Hub** <br> - Khóa chặt chu vi mạng sử dụng **Truy cập riêng tư tới S3 qua VPC Endpoints** | 2026-04-28 | 2026-04-28 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Bảo vệ các endpoint công khai bằng **AWS WAF (Web Application Firewall)** <br> - Thực thi cơ chế mã hóa dữ liệu qua **AWS KMS (Key Management Service)** | 2026-04-29 | 2026-04-29 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Khám phá các tài sản dữ liệu nhạy cảm qua **Amazon Macie** <br> - Lưu trữ các mã khóa ứng dụng an toàn qua **AWS Secrets Manager** <br> - Tập trung hóa các chính sách tường lửa qua **AWS Firewall Manager** | 2026-04-30 | 2026-05-01 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Triển khai theo dõi mối đe dọa liên tục qua **AWS GuardDuty** <br> - Tự động hóa pipeline tạo ảnh hệ điều hành chuẩn (golden image) bằng **EC2 Image Builder** <br> - Xác thực người dùng cuối qua **Amazon Cognito** & đánh giá **S3 Security Best Practices** | 2026-05-01 | 2026-05-01 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 5:

* **Quản lý định danh & Cô lập đặc quyền doanh nghiệp:** Cấu hình các cấu trúc kiểm soát truy cập của người dùng bằng **AWS IAM Identity Center (Single Sign-On)**. Thực thi **IAM Permission Boundaries** trên các định danh sub-admin để ngăn chặn tình trạng leo thang đặc quyền, đảm bảo người dùng không thể tự cấp các quyền truy cập trái phép cho chính mình.
* **Củng cố chu vi mạng & Truy cập mạng riêng tư:** Khởi tạo một **Gateway VPC Endpoint cho Amazon S3**, cho phép lưu lượng truy cập từ các thực thể EC2 riêng tư nội bộ giao tiếp với các nút lưu trữ dữ liệu thông qua mạng xương sống nội bộ của AWS, hoàn toàn bỏ qua mạng Internet công cộng.
* **Bảo vệ lớp ứng dụng tại vùng biên:** Triển khai các quy tắc **AWS WAF** trực tiếp phía trước các bộ Application Load Balancers. Xây dựng các ruleset kiểm tra tùy chỉnh để lọc bỏ các mối đe dọa web phổ biến như SQL Injection (SQLi) và Cross-Site Scripting (XSS), giữ cho các endpoint công khai luôn an toàn.
* **Quản trị bảo mật mã hóa dữ liệu:** Cấu hình các khóa mã hóa do khách hàng quản lý tự động bằng **AWS KMS**. Tích hợp **Amazon Macie** để quét các kho dữ liệu S3 bằng học máy (machine learning), tự động phát hiện và cảnh báo về các Thông tin nhận dạng cá nhân (PII) bị rò rỉ.
* **Tự động xoay vòng bí mật & Phát hiện mối đe dọa:** Cấu hình **AWS Secrets Manager** để tự động xoay vòng mật khẩu cơ sở dữ liệu mà không gây ra thời gian gián đoạn cho ứng dụng. Kích hoạt **AWS GuardDuty** để liên tục phân tích CloudTrail và VPC logs nhằm phát hiện các hoạt động đáng ngờ hoặc các thực thể bị xâm nhập.
* **Tự động hóa Pipeline Golden Image:** Sử dụng **EC2 Image Builder** để xây dựng một pipeline vá lỗi hệ điều hành tự động. Thiết lập này tự động hóa việc tạo, kiểm thử và triển khai các Amazon Machine Images (AMIs) bảo mật, đã được đóng gói an toàn trên toàn bộ hạ tầng AWS.