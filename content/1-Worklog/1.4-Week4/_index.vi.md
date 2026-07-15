---
title: "Worklog Tuần 4"
date: 2026-04-20
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:

* Làm chủ kiến thức về Kiến trúc hạ tầng dưới dạng mã (IaC).
* Khám phá tối ưu hóa hệ thống, cơ chế dịch chuyển workload và tự động hóa lưu trữ.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2 | - Học các kiến thức cốt lõi về **AWS CloudFormation** <br> - Đi sâu vào khái niệm cơ bản và nâng cao của **AWS CDK** | 2026-04-20 | 2026-04-20 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Hoàn thành **Infrastructure as Code Workshop Series** <br> - Cấu hình môi trường phát triển cục bộ sử dụng **AWS Toolkit cho VS Code** | 2026-04-21 | 2026-04-21 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Nghiên cứu dịch chuyển lên AWS: **Di chuyển máy ảo qua AWS VM Import/Export** <br> - Nghiên cứu **AWS DMS (Database Migration Service)** và **SCT (Schema Conversion Tool)** | 2026-04-22 | 2026-04-22 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Cấu hình **Khắc phục sự cố với AWS Elastic Disaster Recovery (DRS)** <br> - Thực hiện chuẩn hóa kích cỡ tài nguyên thông qua **EC2 Resource Optimization** <br> - Thiết lập giám sát mạng qua **VPC Flow Logs** | 2026-04-23 | 2026-04-24 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Tìm hiểu về ủy quyền Billing Console và quản lý **Service Quotas** <br> - Tự động hóa quản lý snapshot với **Amazon EBS Data Lifecycle Manager** <br> - Triển khai **Phát hiện bất thường cho EBS Backups** | 2026-04-24 | 2026-04-24 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 4:

* **Xây dựng hạ tầng bằng mã nguồn (IaC):** Phát triển và thực thi các template hạ tầng có thể tái sử dụng qua **AWS CloudFormation** (định dạng YAML) và các cấu trúc mã nguồn lập trình bằng **AWS Cloud Development Kit (CDK)**. Thực hiện thành công các tác vụ tổng hợp (`cdk synth`) và triển khai (`cdk deploy`) trực tiếp từ môi trường cục bộ được tinh chỉnh qua **AWS Toolkit cho VS Code**.
* **Thiết kế chiến lược dịch chuyển doanh nghiệp:** Mô hình hóa các pipeline dịch chuyển phức tạp bằng cách sử dụng **AWS Schema Conversion Tool (SCT)** để chuyển đổi các engine schema cơ sở dữ liệu, đi kèm với các blueprint của **AWS Database Migration Service (DMS)** để cấu hình các tác vụ sao chép dữ liệu liên tục từ on-premises lên đám mây.
* **Hạ tầng khắc phục sự cố liên tục (Disaster Recovery):** Cấu hình tính năng sao chép dữ liệu liên tục ở cấp độ khối (block-level) mà không gây gián đoạn bằng **AWS Elastic Disaster Recovery (DRS)**, cho phép hệ thống sẵn sàng tự động chuyển vùng lỗi (failover) lên đám mây với các chỉ số mục tiêu thời gian phục hồi và điểm phục hồi (RTO/RPO) ở mức thấp.
* **Phân tích và tối ưu hóa lưu lượng mạng:** Kích hoạt **VPC Flow Logs** trên các giao diện mạng mục tiêu, truyền dữ liệu luồng trực tiếp vào CloudWatch Logs. Sử dụng CloudWatch Logs Insights để phân tích các mô hình payload mạng và phát hiện các gói tin bị rớt (dropped packets) hoặc các yêu cầu kết nối trái phép.
* **Quản lý tự động vòng đời bảo vệ dữ liệu:** Tự động hóa các chế độ sao lưu ổ đĩa khối bằng **Amazon EBS Data Lifecycle Manager (DLM)**. Cấu hình các chính sách lưu giữ bản sao lưu và xếp chồng lớp **Phát hiện bất thường cho EBS Backups** để gắn cờ các kích thước volume bất thường hoặc các snapshot bị thiếu, giúp bảo vệ hệ thống trước mã độc tống tiền (ransomware).