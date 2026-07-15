---
title: "Worklog Tuần 1"
date: 2026-03-30
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---


### Mục tiêu tuần 1:

* Kết nối, làm quen với các thành viên thuộc chương trình First Cloud Journey (FCAJ).
* Hiểu các dịch vụ AWS cơ bản, cách sử dụng AWS Management Console & AWS CLI.
* Nắm vững các khái niệm cốt lõi về IAM, VPC và EC2.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2 | - Làm quen với các thành viên FCAJ <br> - Đọc và ghi chú các quy định, nội quy của chương trình thực tập | 2026-03-30 | 2026-03-30 | |
| 3 | - Tìm hiểu về AWS và khám phá các dịch vụ AWS nền tảng <br> - Học cách tạo và quản lý chi phí với **AWS Budgets** và sử dụng **AWS Support** | 2026-03-31 | 2026-03-31 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Tạo tài khoản AWS Free Tier <br> - Tìm hiểu về AWS Console & AWS CLI <br> - **Thực hành:** Tạo tài khoản AWS, cài đặt và cấu hình AWS CLI | 2026-04-01 | 2026-04-01 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Học về Quản lý truy cập với **AWS IAM** <br> - Học kiến thức Mạng cốt lõi với **Amazon VPC** <br> - Học kiến thức Tính toán cốt lõi với **Amazon EC2** và **Instance Profiling với IAM Roles** | 2026-04-02 | 2026-04-03 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - **Thực hành:** <br>&emsp; + Khởi chạy một thực thể EC2 bên trong một VPC tùy chỉnh (Custom VPC) <br>&emsp; + Đính kèm một IAM Role vào thực thể EC2 <br>&emsp; + Kiểm tra các lệnh CLI cơ bản bên trong thực thể | 2026-04-03 | 2026-04-03 | <https://cloudjourney.awsstudygroup.com/> |


### Kết quả đạt được tuần 1:

* Thiết lập bảo mật tài khoản & ngân sách AWS: Khởi tạo thành công tài khoản AWS Free Tier, cấu hình xác thực đa yếu tố (MFA) cho tài khoản Root, và thiết lập cảnh báo hạn mức chi phí bằng AWS Budgets để giám sát chặt chẽ mức tiêu thụ tài nguyên.

* Thành thạo AWS CLI: Cấu hình thành công terminal cục bộ với công cụ AWS CLI bằng Access Keys an toàn. Làm chủ các cấu trúc lệnh cốt lõi (`aws ec2 describe-instances`, `aws iam list-users`) để kiểm tra và điều khiển tài nguyên đám mây thông qua dòng lệnh mà không cần dùng giao diện web.

* Triển khai hạ tầng mạng VPC: Thiết kế và cấu hình một mạng ảo riêng biệt Virtual Private Cloud (VPC) cơ bản, hoàn thiện với các Public Subnet, Internet Gateway (IGW), và chỉnh sửa Route Tables để định tuyến chính xác lưu lượng truy cập Internet hướng ngoại.

* Quản lý máy chủ tính toán an toàn: Cấu hình thành công một máy chủ ảo Amazon EC2 Linux sử dụng Amazon Machine Images (AMIs) tùy chỉnh. Quản lý quyền truy cập mạng qua Security Groups bằng cách thiết lập các inbound rules để giới hạn cổng truy cập (Cổng 22 cho SSH, Cổng 80 cho HTTP).

* Thực thi IAM Instance Profile: Áp dụng các tiêu chuẩn bảo mật tốt nhất bằng cách tạo một AWS IAM Role với các quyền đọc (read-only) cụ thể trên Amazon S3 và đính kèm vào thực thể EC2 thông qua một instance profile. Kiểm chứng thành công các ứng dụng chạy bên trong EC2 có thể truy cập an toàn vào S3 buckets mà không cần mã hóa cứng (hardcode) thông tin xác thực AWS dạng văn bản thô bên trong mã nguồn mã máy chủ.

