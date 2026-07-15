---
title: "Worklog Tuần 8"
date: 2026-05-18
weight: 1
chapter: false
pre: " <b> 1.8. </b> "
---


### Mục tiêu tuần 8:

* Hoàn thành chuỗi bài học Hệ thống quản lý tài liệu (DMS), cùng với các framework quản lý container.
* Khám phá các công cụ điều phối trên Elastic Beanstalk, Amazon ECS/Fargate, EKS và môi trường doanh nghiệp ROSA.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2 | - **Serverless DMS Series:** Xây dựng tính năng CRUD dạng serverless với Lambda/DynamoDB <br> - Tích hợp dịch vụ lưu trữ/xác thực của **AWS Amplify** và liên kết frontend qua API Gateway | 2026-05-18 | 2026-05-18 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - **Serverless DMS Series (Tiếp tục):** Triển khai sử dụng SAM, cấu hình CloudFront và thêm tính năng tìm kiếm nâng cao <br> - Thiết lập các pipeline DevOps và thực hiện truy vết phân tán bằng **AWS X-Ray** | 2026-05-19 | 2026-05-19 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Hoàn thành **Serverless Web App Workshop** (Ứng dụng Chat & tạo API) <br> - Hoàn thành **Elastic Beanstalk Workshop**: Triển khai ứng dụng Node.js với CDK Pipelines | 2026-05-20 | 2026-05-20 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Triển khai kiến trúc **WordPress trên AWS** có tính sẵn sàng cao trên Amazon EC2 <br> - Tham gia **Amazon ECS Workshop**: Đóng gói container với ECS và **AWS Fargate** | 2026-05-21 | 2026-05-22 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Tham gia **Amazon EKS Workshop**: Bắt đầu với EKS và EKS Blueprints cho CDK <br> - Xem xét quy trình CI/CD cho EKS và tìm hiểu về Red Hat OpenShift Service trên AWS (**ROSA**) | 2026-05-22 | 2026-05-22 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 8:

* **Ứng dụng lưu trữ Serverless Full-Stack:** Xây dựng một Hệ thống quản lý tài liệu (DMS) hoàn chỉnh với tính năng xử lý dữ liệu tự động thông qua **Amazon API Gateway** và các tiến trình Lambda CRUD phía backend. Tích hợp các thư viện **AWS Amplify** ở phía frontend để quản lý lưu trữ tệp tin một cách an toàn.
* **Chẩn đoán ứng dụng đám mây phân tán:** Triển khai tính năng truy vết hiệu năng toàn diện (end-to-end performance tracing) trên các dịch vụ microservices serverless sử dụng **AWS X-Ray**. Bổ sung các điểm đánh dấu vết vào các khối thực thi Lambda để xác định chính xác các điểm nghẽn hiệu năng và theo dõi hành trình ngữ cảnh của các yêu cầu trên nhiều thành phần AWS.
* **Quản lý cụm Container Serverless:** Cấu hình và khởi chạy các microservices web có khả năng mở rộng bên trong **Amazon ECS sử dụng AWS Fargate**. Thiết lập này giúp chạy các container trực tiếp trên hạ tầng serverless, loại bỏ các chi phí vận hành cho việc cung cấp, vá lỗi hoặc mở rộng quy mô các nút cluster EC2 bên dưới.
* **Lắp ráp cụm Kubernetes doanh nghiệp:** Hoàn thành các bài thực hành quản lý container nâng cao thông qua **Amazon EKS Workshop**. Triển khai thành công các control plane Kubernetes được quản lý bằng cách sử dụng **EKS Blueprints cho AWS CDK**, thiết lập các pipeline phân phối GitOps cốt lõi để quản lý các workload ứng dụng.
* **Kiến trúc Monolith doanh nghiệp có tính sẵn sàng cao:** Thiết kế một cấu hình production có khả năng chịu lỗi cao cho ứng dụng WordPress trên AWS. Sử dụng các nút tính toán EC2 tách biệt trên các Vùng sẵn sàng (Availability Zones) khác nhau, kết nối với một cấu trúc lưu trữ chung và một backend cơ sở dữ liệu RDS để ngăn chặn các điểm lỗi đơn lẻ (single points of failure).