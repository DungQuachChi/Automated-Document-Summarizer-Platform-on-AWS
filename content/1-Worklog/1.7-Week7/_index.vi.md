---
title: "Worklog Tuần 7"
date: 2026-05-11
weight: 1
chapter: false
pre: " <b> 1.7. </b> "
---


### Mục tiêu tuần 7:

* Làm chủ các kiến trúc hiện đại hóa ứng dụng thông qua các workshop thực hành về serverless.
* Hoàn thành chuỗi phát triển ứng dụng DevAx và Serverless Book Store.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2 | - **Serverless - DevAx Series:** Học các chiến lược chuyển đổi từ Monolith sang Microservices <br> - Xây dựng các microservices, thiết lập CI/CD để phát hành ứng dụng và quản lý tái cấu trúc dữ liệu | 2026-05-11 | 2026-05-11 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - **Serverless - DevAx Series (Tiếp tục):** Triển khai Kiến trúc hướng sự kiện (Event-Driven) <br> - Cấu hình xác thực cho ứng dụng Single Page Application & tích hợp các dịch vụ AWS AI | 2026-05-12 | 2026-05-12 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - **Serverless - Book Store Series:** Thiết lập hệ thống backend serverless với Lambda, S3 và DynamoDB <br> - Phát triển ứng dụng frontend cho các Serverless APIs và tự động hóa triển khai bằng **AWS SAM** | 2026-05-13 | 2026-05-13 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - **Serverless - Book Store Series (Tiếp tục):** Triển khai đăng nhập người dùng qua **Amazon Cognito** <br> - Cấu hình Custom Domains, chứng chỉ SSL và Xử lý sự kiện qua SQS/SNS | 2026-05-14 | 2026-05-15 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - **Serverless - Book Store Series (Tiếp tục):** Xây dựng các GraphQL APIs với **AWS AppSync** <br> - Hoàn thiện cấu hình CI/CD và giám sát hệ thống cho các ứng dụng serverless | 2026-05-15 | 2026-05-15 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 7:

* **Hiện đại hóa và tái cấu trúc Monolith:** Phân tích các mã nguồn ứng dụng web đa tầng kiểu cũ để bẻ nhỏ các thiết lập dạng nguyên khối (monolithic) thành các microservices độc lập, triển khai các mẫu cấu trúc nhằm cô lập thành công các cơ sở dữ liệu riêng biệt.
* **Xây dựng Serverless Application Framework:** Sử dụng **AWS Serverless Application Model (AWS SAM)** để xây dựng và đóng gói một cấu trúc ứng dụng hoàn chỉnh. Viết các định nghĩa tệp `template.yaml` có cấu trúc rõ ràng để triển khai các lớp AWS Lambda layers, các S3 asset buckets và các thực thể lưu trữ DynamoDB bằng các lệnh gọi thực thi CloudFormation hợp nhất.
* **Quản lý định danh & Bảo mật truy cập:** Bảo mật giao diện frontend của ứng dụng Single Page Application (SPA) bằng cách triển khai **Amazon Cognito User Pools**. Bước này bổ sung các quy trình đăng ký và xác thực dựa trên token an toàn, hoàn thiện với tính năng định tuyến domain tùy chỉnh và các bước xác thực lớp mã hóa SSL/TLS được quản lý.
* **Tổng hợp lớp API GraphQL được quản lý:** Triển khai các endpoint dữ liệu thời gian thực được quản lý bằng **AWS AppSync**. Xây dựng các bộ xử lý GraphQL resolvers hiệu quả giúp đọc và ghi dữ liệu một cách an toàn vào các hệ thống backend Amazon DynamoDB, loại bỏ nhu cầu quản lý các bộ điều khiển endpoint REST truyền thống.
* **Xử lý luồng sự kiện hướng sự kiện:** Kết nối logic xử lý bất đồng bộ giữa các thành phần ứng dụng bằng cách sử dụng các hàng đợi SQS và SNS phi tập trung. Thiết lập này giúp ghi lại các sự kiện giao dịch trong thời gian thực, quản lý an toàn các đợt lưu lượng truy cập tăng vọt đột biến của ứng dụng.