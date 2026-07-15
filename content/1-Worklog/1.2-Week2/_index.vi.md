---
title: "Worklog Tuần 2"
date: 2026-04-06
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---


### Mục tiêu tuần 2:

* Khám phá môi trường phát triển đám mây, các dịch vụ lưu trữ và cơ sở dữ liệu.
* Hiểu về kiến trúc có tính sẵn sàng cao (High Availability), mạng phân phối nội dung (CDN) và kiến trúc tự động mở rộng (Scaling).

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2 | - Học về phát triển ứng dụng trên đám mây với **AWS Cloud9** <br> - Nghiên cứu nguyên lý lưu trữ website tĩnh trên **Amazon S3** | 2026-04-06 | 2026-04-06 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Học kiến thức Cơ sở dữ liệu cốt lõi với **Amazon RDS** <br> - Hiểu về nhóm subnet cho cơ sở dữ liệu (Database subnet groups) và triển khai Multi-AZ | 2026-04-07 | 2026-04-07 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Học về **Amazon Lightsail** để đơn giản hóa tài nguyên tính toán <br> - Đi sâu vào triển khai ứng dụng container với **Amazon Lightsail Containers** | 2026-04-08 | 2026-04-08 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Học cách mở rộng hệ thống bằng **EC2 Auto Scaling** và **Elastic Load Balancing** <br> - Thiết lập giám sát hệ thống cơ bản sử dụng **Amazon CloudWatch** | 2026-04-09 | 2026-04-10 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Học cách quản lý DNS hỗn hợp (Hybrid DNS) với **Amazon Route 53** <br> - Thiết lập phân phối nội dung với **Amazon CloudFront** & **Lambda@Edge** | 2026-04-10 | 2026-04-10 | <https://cloudjourney.awsstudygroup.com/> |


### Kết quả đạt được tuần 2:

* Triển khai môi trường phát triển đám mây: Cấu hình thành công môi trường IDE đám mây AWS Cloud9, tạo ra một terminal được xác thực sẵn giúp hợp lý hóa việc chỉnh sửa mã nguồn và quản lý tài nguyên trực tiếp từ giao diện trình duyệt.

* Lưu trữ & phân phối Website tĩnh qua S3: Cấu hình một Amazon S3 bucket để phân phối một website tĩnh. Chỉnh sửa S3 Bucket Policies để cho phép quyền đọc công khai (public read), đồng thời tích hợp cổng gốc này với Amazon CloudFront nhằm phân phối tài nguyên web đến các vị trí biên toàn cầu (Edge Locations), giúp giảm thiểu độ trễ.

* Quản trị cơ sở dữ liệu quan hệ: Khởi tạo một thực thể cơ sở dữ liệu Amazon RDS bên trong các nhóm subnet biệt lập dành riêng cho database riêng tư. Kích hoạt cấu hình triển khai Multi-AZ để thiết lập cơ chế sao chép dự phòng và tự động chuyển vùng khi xảy ra sự cố (automated failover).

* Triển khai Container đơn giản hóa: Sử dụng Amazon Lightsail Containers để nhanh chóng triển khai các image web gọn nhẹ, loại bỏ các gánh nặng vận hành khi phải cấu hình máy chủ ảo bên dưới hoặc các bộ điều phối phức tạp.

* Kiến trúc Auto-Scaling & Load Balancing: Xây dựng một tầng tính toán có tính sẵn sàng cao bằng cách sử dụng Elastic Load Balancing (ALB) và EC2 Auto Scaling. Cấu hình các chính sách mở rộng động (dynamic scaling) dựa trên các chỉ số của Amazon CloudWatch để tự động tăng cường hoặc hủy bỏ các thực thể EC2 theo giới hạn sử dụng CPU.

* Triển khai logic xử lý tại vùng biên (Edge): Cấu hình các bản ghi định tuyến DNS tùy chỉnh bên trong Amazon Route 53 và đưa logic xử lý của Lambda@Edge vào nhằm chỉnh sửa các request headers ngay lập tức tại các bộ nhớ đệm biên khu vực, giúp cải thiện khả năng định tuyến bảo mật vùng biên.

