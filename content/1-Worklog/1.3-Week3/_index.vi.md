---
title: "Worklog Tuần 3"
date: 2026-04-13
weight: 1
chapter: false
pre: " <b> 1.3. </b> "
---


### Mục tiêu tuần 3:

* Kết nối, làm quen với các thành viên trong First Cloud AI Journey.
* Hiểu dịch vụ AWS cơ bản, cách dùng console & CLI.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2 | - Học kiến thức Cơ sở dữ liệu NoSQL với **Amazon DynamoDB** <br> - Học về Bộ nhớ đệm trong bộ nhớ (In-Memory Caching) sử dụng **Amazon ElastiCache** | 2026-04-13 | 2026-04-13 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Tham gia **Networking on AWS Workshop** <br> - Nghiên cứu hệ thống Windows Workloads trên AWS và **AWS Managed Microsoft AD** | 2026-04-14 | 2026-04-14 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Đánh giá các kiến trúc để **Xây dựng ứng dụng Web có tính sẵn sàng cao** <br> - Nghiên cứu tự động hóa Serverless với **AWS Lambda** | 2026-04-15 | 2026-04-15 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Tham dự **CloudWatch Advanced Workshop** <br> - Cấu hình giám sát nâng cao tích hợp giữa **CloudWatch và Grafana** | 2026-04-16 | 2026-04-17 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Tổ chức tài nguyên bằng **Tags và Resource Groups** <br> - Triển khai kiểm soát truy cập bằng **IAM và Resource Tags** <br> - Thực hành quản lý hệ thống với **AWS Systems Manager (SSM)** & **Session Manager** | 2026-04-17 | 2026-04-17 | <https://cloudjourney.awsstudygroup.com/> |


### Kết quả đạt được tuần 3:

* Cấu hình Kho dữ liệu NoSQL & Caching: Khởi tạo các bảng Amazon DynamoDB, cấu hình Partition Keys và Sort Keys để tối ưu hóa truy vấn dữ liệu. Liên kết một cụm Amazon ElastiCache (vận hành bởi Redis) để giảm tải cho các tác vụ đọc chuyên sâu, giảm thiểu độ trễ truy vấn phía backend.

* Triển khai định danh Windows Doanh nghiệp: Khám phá các mô hình kiến trúc đám mây doanh nghiệp bằng cách triển khai AWS Managed Microsoft AD bên trong một mảng subnet riêng tư nhằm tạo điều kiện xác thực đối tượng tập trung và quản lý an toàn cho các workload Windows.

* Tự động hóa Logic hướng sự kiện Serverless: Lập trình và triển khai các quy trình vận hành serverless bằng cách sử dụng AWS Lambda. Thiết lập logic thực thi tự động được kích hoạt bởi các tệp tin tải lên S3, phân tích các sự kiện mà không phải trả chi phí cho thời gian máy chủ nhàn rỗi.

* Liên kết dữ liệu Dashboard nâng cao: Hoàn thành khóa học nâng cao về CloudWatch. Kết nối các nền tảng trực quan hóa dữ liệu bên ngoài như Grafana trực tiếp với các chỉ số của Amazon CloudWatch thông qua các IAM access roles, tạo ra các bảng quản trị hiệu năng tập trung.

* Quản trị truy cập dựa trên thuộc tính (ABAC): Thực thi các tiêu chuẩn tag trên toàn tổ chức thông qua AWS Resource Groups. Viết các chính sách IAM nâng cao có chức năng đánh giá các tag môi trường (`Env: Production`), từ đó giới hạn quyền chỉnh sửa dựa trên các tiêu chí khóa tài nguyên.

* Quản lý hệ thống an toàn không cần Bastion Host: Tích hợp AWS Systems Manager (SSM) trên các cụm máy chủ đang chạy. Tận dụng Session Manager để thực thi các lệnh terminal trên các máy chủ EC2 thuộc subnet riêng tư, loại bỏ hoàn toàn nhu cầu mở cổng SSH hoặc định tuyến qua các máy chủ trung chuyển (bastion hosts).

