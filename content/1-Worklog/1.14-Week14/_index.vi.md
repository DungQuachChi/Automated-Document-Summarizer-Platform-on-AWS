---
title: "Nhật Ký Công Việc Tuần 14"
date: 2026-06-29
weight: 14
chapter: false
pre: " <b> 1.14. </b> "
---

### Mục Tiêu Tuần 14:
* Chuyển đổi kiến trúc cloud đã được kiểm thử thành **Terraform (Infrastructure as Code)**.
* Xây dựng **CI/CD Pipeline** triển khai tự động bằng GitHub Actions.
* Thực hiện kiểm thử tải, tiến hành rà soát bảo mật toàn diện, và hoàn thiện toàn bộ tài liệu dự án Capstone.

### Công Việc Hàng Ngày:
| Ngày | Công việc | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| Thứ 2 | Viết các module Terraform cho auth (Cognito), data (S3, DynamoDB), và compute (Lambda Functions, IAM Roles cho Bedrock) <br> Triển khai quản lý remote state bằng S3 backend và khóa state bằng DynamoDB | 2026-06-29 | 2026-06-29 | Terraform AWS Provider Docs |
| Thứ 3 | Hoàn thiện module Terraform cho api (API Gateway, Routes, Usage Plans, CloudFront OAC) <br> Thực thi terraform apply để dựng toàn bộ hạ tầng cloud từ đầu | 2026-06-30 | 2026-06-30 | Terraform Best Practices |
| Thứ 4 | Xây dựng **GitHub Actions Workflow** để tự động hóa linting, chạy Lambda Unit Tests, và kích hoạt tự động triển khai khi push vào main | 2026-07-01 | 2026-07-01 | GitHub Actions Docs |
| Thứ 5 | Thực hiện Kiểm Thử Tải bằng **k6** <br> Xác thực các chính sách Rate Limiting (100 req/phút) và kiểm tra hiệu năng Cache của API Gateway | 2026-07-02 | 2026-07-02 | k6 Load Testing Guide |
| Thứ 6 | Rà soát các IAM Role theo Nguyên Tắc Đặc Quyền Tối Thiểu (Principle of Least Privilege) <br> Hoàn thiện sơ đồ kiến trúc, đặc tả API, báo cáo phân tích chi phí, và slide bảo vệ Capstone | 2026-07-03 | 2026-07-03 | Project Capstone Docs |

### Thành Tựu Chính:
* **Infrastructure as Code (IaC) với Terraform:** Chuyển đổi thành công toàn bộ kiến trúc cloud (Cognito, DynamoDB, S3, Lambda, API Gateway, CloudFront) thành các module Terraform có cấu trúc, cho phép dựng hạ tầng chỉ với một lệnh duy nhất.
* **Tích Hợp CI/CD Pipeline Tự Động:** Cấu hình một Pipeline GitHub Actions thực hiện kiểm tra chất lượng code (linting), chạy unit test cho các hàm xử lý văn bản, và tự động hóa terraform plan/apply khi merge code vào main.
* **Kiểm Thử Tải & Đánh Giá Hiệu Năng Hệ Thống:** Mô phỏng tải người dùng đồng thời bằng k6, xác nhận rằng Rate Limiting của API chặn hiệu quả traffic lạm dụng và việc caching trong 1 giờ giảm hơn 40% chi phí gọi model Bedrock.
* **Tăng Cường Bảo Mật & Rà Soát Hạ Tầng:** Hạn chế quyền truy cập công khai vào bucket S3 lưu trữ bằng cách triển khai CloudFront Origin Access Control (OAC). Tinh chỉnh các IAM Policy của Lambda để tuân thủ nghiêm ngặt Nguyên Tắc Đặc Quyền Tối Thiểu.
* **Tài Liệu Capstone & Sẵn Sàng Bảo Vệ:** Hoàn thành toàn bộ các sản phẩm bàn giao kỹ thuật, bao gồm sơ đồ Kiến Trúc tổng thể, đặc tả OpenAPI/Swagger, báo cáo chi phí vận hành (< $5/tháng ở mức sử dụng tiêu chuẩn), và slide thuyết trình cho hội đồng bảo vệ Capstone.