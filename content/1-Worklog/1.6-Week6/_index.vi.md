---
title: "Worklog Tuần 6"
date: 2026-05-04
weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---


### Mục tiêu tuần 6:

* Thiết lập mạng lưới đa vùng và các kiến trúc hệ thống có khả năng phục hồi cao.
* Kết nối hạ tầng microservices tiên tiến sử dụng các công cụ container được quản lý và các pipeline CI/CD.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2 | - Tập trung hóa các chính sách bảo vệ dữ liệu sử dụng **AWS Backup** <br> - Học cấu trúc mạng phức tạp với **VPC Peering** và **AWS Transit Gateway** | 2026-05-04 | 2026-05-04 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Triển khai các bộ định tuyến tin nhắn microservice bất đồng bộ với **Amazon SQS** và **SNS** <br> - Thiết lập cấu trúc lưu trữ volume chia sẻ qua **Amazon EBS Multi-Attach** | 2026-05-05 | 2026-05-05 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Phân tích tính sẵn sàng cao cho cơ sở dữ liệu doanh nghiệp (WSFC và SQL Server HA trên AWS 2019/2022) <br> - Xem xét các giải pháp tối ưu chi phí (**Savings Plans / Reserved Instances**) và Trực quan hóa & Phân tích chi phí | 2026-05-06 | 2026-05-06 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Thực hiện phân tích cấu trúc dữ liệu lớn (big-data) qua **AWS Glue** và các truy vấn **Amazon Athena** <br> - Xây dựng nền tảng microservices: Đóng gói container với **Docker** và **Amazon ECS** | 2026-05-07 | 2026-05-08 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Khởi tạo hạ tầng cho ECS bằng CDK <br> - Triển khai các pipeline hoàn chỉnh: **AWS CodePipeline**, Tự động hóa triển khai và các thực hành DevOps | 2026-05-08 | 2026-05-08 | <https://cloudjourney.awsstudygroup.com/> |


### Kết quả đạt được tuần 6:
* **Kiến trúc mạng Transit doanh nghiệp tập trung:** Thiết kế và triển khai kiến trúc mạng hình sao (hub-and-spoke) sử dụng **AWS Transit Gateway**. Thành phần này giúp tập trung hóa việc định tuyến mạng giữa các môi trường VPC riêng biệt, loại bỏ sự phức tạp khi phải quản lý các kết nối **VPC Peering** dạng lưới hỗn loạn.
* **Khử liên kết Microservices bất đồng bộ:** Triển khai các kiến trúc ứng dụng phi tập trung (decoupled) sử dụng **Amazon SQS** (với cấu hình hàng đợi First-In, First-Out - FIFO) và **Amazon SNS**. Xây dựng mô hình phân phối tin nhắn dạng fan-out, trong đó một thông báo sự kiện duy nhất có thể kích hoạt nhiều hệ thống xử lý backend song song cùng một lúc.
* **Xử lý ETL Big Data dạng Serverless:** Xây dựng một pipeline phân tích dữ liệu không máy chủ. Sử dụng **AWS Glue Crawlers** để tự động khám phá các schema dữ liệu trong S3, lập danh mục để có thể truy vấn ngay lập tức bằng ngôn ngữ SQL tiêu chuẩn thông qua **Amazon Athena**.
* **Triển khai hạ tầng Container hóa:** Phát triển các **Dockerfile** đa tầng (multi-stage) để đóng gói mã nguồn ứng dụng vào các container runtime tối ưu và nhẹ nhàng. Đăng tải các image này lên Amazon Elastic Container Registry (ECR) và triển khai chúng bằng **Amazon ECS**.
* **Tự động hóa Pipeline DevOps với CDK:** Sử dụng **AWS CDK** để định nghĩa cấu hình hạ tầng bằng mã nguồn cho một stack microservice ECS. Quản lý việc triển khai bằng **AWS CodePipeline**, tạo ra một pipeline CI/CD tự động hóa hoàn toàn từ khâu kéo cập nhật source code, xây dựng container image cho đến triển khai các bản cập nhật lên môi trường production mà không gây downtime.

