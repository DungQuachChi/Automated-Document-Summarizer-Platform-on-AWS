---
title: "Worklog Tuần 9"
date: 2026-05-25
weight: 1
chapter: false
pre: " <b> 1.9. </b> "
---

### Mục tiêu tuần 9:

* Làm chủ hệ thống lưu trữ kết hợp (hybrid storage), điều phối quy trình công việc (workflow orchestration) và tích hợp các công cụ dữ liệu nâng cao.
* Hiểu về cấu trúc phân tích dữ liệu, thiết lập kho dữ liệu (data warehousing) và các dịch vụ AI/ML cốt lõi.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2 | - Kết nối các cấu trúc hỗn hợp sử dụng **AWS Storage Gateway** và **Amazon FSx cho Windows** <br> - Xây dựng các ứng dụng nâng cao với Amazon DynamoDB & điều phối quy trình bằng **AWS Step Functions** | 2026-05-25 | 2026-05-25 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Hoàn thành **Storage Performance Workshop** <br> - Nghiên cứu các kiến thức cốt lõi về Data Lake trên AWS và xây dựng một Data Lake với dữ liệu tùy chỉnh của riêng bạn | 2026-05-26 | 2026-05-26 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Hoàn thành **Data Engineering Immersion Day** <br> - Chạy các truy vấn phân tích Serverless với **Amazon Athena** và trực quan hóa qua **Amazon QuickSight** | 2026-05-27 | 2026-05-27 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Học các mô hình dữ liệu nâng cao: **Advanced PostgreSQL trên AWS** (Phần 1 & Phần 2) <br> - Nghiên cứu các kiến thức cốt lõi về Machine Learning trên AWS | 2026-05-28 | 2026-05-29 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Tham gia **SageMaker Immersion Day** <br> - **Thực hành:** Xây dựng, huấn luyện và triển khai một mô hình Machine Learning toàn diện sử dụng **Amazon SageMaker** | 2026-05-29 | 2026-05-29 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 9:

* **Tích hợp hệ thống lưu trữ kết hợp:** Liên kết hạ tầng cục bộ (on-premises) với các tài nguyên đám mây bằng cách sử dụng **AWS Storage Gateway** và cấu hình các cụm **Amazon FSx cho Windows File Server** để xử lý các tác vụ lưu trữ tệp tin chia sẻ của doanh nghiệp.
* **Điều phối máy trạng thái (State Machine) cho Microservices:** Thiết kế các quy trình công việc backend phức tạp, gồm nhiều bước bằng cách sử dụng **AWS Step Functions**. Xây dựng các máy trạng thái trực quan với cơ chế thử lại (retry) và xử lý ngoại lệ (exception handling) được tích hợp sẵn để điều phối các tác vụ Lambda phân tán.
* **Xây dựng Hồ dữ liệu (Data Lake) doanh nghiệp an toàn:** Xây dựng một data lake hoàn chỉnh từ đầu đến cuối trên Amazon S3. Cấu hình các mô hình phân vùng dữ liệu tối ưu và các pipeline truy vấn, cho phép phân tích quy mô lớn hiệu quả trong khi vẫn giữ chi phí lưu trữ dữ liệu ở mức thấp.
* **Phân phối Dashboard BI Analytics Serverless:** Tham gia ngày hội trải nghiệm Công nghệ dữ liệu (Data Engineering Immersion Day). Thực hiện phân tích dữ liệu đặc ứng (ad-hoc) trên các tập dữ liệu thô bằng cách sử dụng **Amazon Athena** và chuyển đổi các kết quả truy vấn đó thành các bảng quản trị phân tích thông minh doanh nghiệp (BI) có tính tương tác cao trong **Amazon QuickSight**.
* **Tối ưu hóa hiệu năng cấu hình Database nâng cao:** Hoàn thành khóa đào tạo chuyên sâu về PostgreSQL trên AWS. Áp dụng các kỹ thuật tối ưu hóa hiệu năng trên các thực thể Amazon RDS, bao gồm cân bằng tải cho read-replica, cấu hình connection pooling và tinh chỉnh chỉ mục (index tuning) chuyên sâu cho các khối lượng công việc nặng.
* **Triển khai Pipeline vòng đời Machine Learning:** Tham gia chương trình **SageMaker Immersion Day**. Cấu hình các công cụ Jupyter notebooks bên trong **Amazon SageMaker**, chuẩn bị dữ liệu huấn luyện, thực thi các tác vụ huấn luyện mô hình và host một endpoint dự đoán thời gian thực có khả năng tự động mở rộng (auto-scaling).