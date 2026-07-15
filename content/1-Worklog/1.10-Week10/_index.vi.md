---
title: "Worklog Tuần 10"
date: 2026-06-01
weight: 2
chapter: false
pre: " <b> 1.10. </b> "
---
{{% notice warning %}}
⚠️ **Lưu ý:** Các thông tin dưới đây chỉ nhằm mục đích tham khảo, vui lòng **không sao chép nguyên văn** cho bài báo cáo của bạn kể cả warning này.
{{% /notice %}}


### Mục tiêu tuần 10:

* Lựa chọn và chốt đề xuất dự án Capstone.
* Xác định phạm vi chức năng, các cấu phần và các điểm loại trừ của kiến trúc serverless.
* Hoàn thành các bước chuẩn bị môi trường cục bộ ban đầu và thiết lập hệ thống bảo vệ chi phí đám mây.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2 | - Nghiên cứu các chủ đề dự án và chọn ý tưởng *Automated Document Summarizer Platform* <br> - Phác thảo các mục tiêu kiến trúc cốt lõi | 2026-06-01 | 2026-06-01 | Hướng dẫn nội bộ FCAJ |
| 3 | - Chi tiết hóa các tầng kiến trúc (User, Edge, Compute, AI, Data, Scheduling, Security) <br> - Tài liệu hóa cụ thể các thành phần nằm trong phạm vi và loại trừ của dự án | 2026-06-02 | 2026-06-02 | Kiến trúc Serverless AWS |
| 4 | - Cung cấp một tài khoản AWS sandbox chuyên dụng và cô lập cho dự án capstone <br> - Cấu hình các rào chắn an toàn chi phí qua **AWS Budgets** với ngưỡng 25 USD | 2026-06-03 | 2026-06-03 | AWS Billing Console |
| 5 | - Cài đặt các công cụ kỹ thuật cốt lõi tại cục bộ: Terraform (v1.5+), Git và công cụ AWS CLI <br> - Xác minh quyền truy cập terminal theo lập trình tới môi trường AWS mới | 2026-06-04 | 2026-06-04 | <https://registry.terraform.io/> |
| 6 | - Hoàn thiện tài liệu đề xuất dự án chính thức <br> - Lập bản đồ vòng đời phát triển dự án gồm 13 giai đoạn và nộp cho các mentor | 2026-06-05 | 2026-06-05 | Tài liệu đề xuất dự án |


### Kết quả đạt được tuần 10:

* **Lựa chọn & Hoàn thiện Ý tưởng Dự án:** Viết bản đề xuất sẵn sàng cho môi trường production cho dự án *Automated Document Summarizer Platform trên AWS*. Lập sơ đồ đường truyền yêu cầu serverless sử dụng API Gateway, Lambda, Cognito và Bedrock.
* **Quản trị chi phí đám mây chủ động:** Thiết lập các rào chắn tài chính nghiêm ngặt bên trong tài khoản sandbox mới thông qua **AWS Budgets**, cấu hình các cảnh báo tự động ở mức trần 25 USD hàng tháng nhằm ngăn chặn tình trạng chi phí tăng vọt ngoài tầm kiểm soát trong các chu kỳ kiểm thử tải.
* **Cấu hình Công cụ DevOps tại cục bộ:** Cài đặt và xác minh thành công bộ công cụ làm việc tại môi trường cục bộ—liên kết Terraform (v1.5+), Git và AWS CLI lại với nhau. Cấu hình các thông tin xác thực quản trị IAM cục bộ bằng cách sử dụng các ngữ cảnh thực thi an toàn.