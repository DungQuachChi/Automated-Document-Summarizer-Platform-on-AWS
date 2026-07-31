---
title: "Workshop"
date: 2026-07-14
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Dự án: Infrastructure as Code cho Khối lượng Công việc Serverless AI Bảo mật trên AWS

Bài thực hành này là một dự án DevOps và Cloud Engineering sử dụng ứng dụng tóm tắt tài liệu bằng AI làm ví dụ thực tế để hướng dẫn về kiến trúc serverless bảo mật, Infrastructure as Code, tự động hóa CI/CD và khả năng giám sát/quan sát (observability) trên AWS.

Trong bài thực hành này, bạn sẽ xây dựng một nền tảng tóm tắt tài liệu dạng serverless. Nền tảng tiếp nhận các văn bản dài thông qua một REST API được bảo mật và trả về bản tóm tắt do AI tạo ra nhờ sức mạnh của Amazon Bedrock.

Có bốn thành phần cốt lõi phối hợp hoạt động trong toàn bộ quy trình xử lý yêu cầu (request pipeline): Amazon Cognito, Amazon API Gateway, AWS Lambda và Amazon Bedrock. Mỗi thành phần đảm nhận một nhiệm vụ riêng trong luồng xử lý yêu cầu, từ xác thực người dùng cho đến suy luận AI (inference).

+ **Amazon Cognito**: Quản lý việc đăng ký, đăng nhập của người dùng và tạo JWT token thông qua Hosted UI, giúp API của bạn không cần trực tiếp xử lý hay lưu trữ bất kỳ mật khẩu nào.
+ **Amazon API Gateway**: Đứng trước khối xử lý tính toán (compute) để xác thực từng JWT token, đồng thời áp dụng hạn mức sử dụng (usage plan) thông qua API key.
+ **AWS Lambda**: Thực thi các yêu cầu hợp lệ, gọi Amazon Bedrock để lấy bản tóm tắt từ AI, đồng thời đọc/ghi dữ liệu vào DynamoDB. Được kích hoạt theo nhu cầu (on-demand) và tự động thu gom/giảm quy mô về 0 khi nhàn rỗi.
+ **Amazon Bedrock**: Cung cấp bản tóm tắt văn bản dựa trên mô hình nền tảng Amazon Nova Lite, được gọi thông qua hồ sơ suy luận đa vùng (cross-region inference profile).

Bạn cũng sẽ xây dựng hạ tầng phục vụ quy trình cốt lõi này cho sản phẩm: Terraform giúp tái tạo hạ tầng nhanh chóng, quy trình CI/CD với CodePipeline tích hợp kiểm thử tự động và bước phê duyệt thủ công, hệ thống giám sát CloudWatch đi kèm cảnh báo thực tế, cùng tiêu chuẩn an toàn bảo mật tuân thủ định hướng CIS.

---

#### Nội dung

1. [Tổng quan](1.1-Overview/)
2. [Điều kiện tiên quyết](2-Prerequisites/)
3. [Thiết kế kiến trúc](3-Architecture/)
4. [Backend & AI](5.4-Backend&AI/)
   1. [DynamoDB & Lambda](5.4.1-DynamoDB&Lambda/)
   2. [Tích hợp AI](5.4.2-AI_Integration/)
5. [Tầng truy cập](5.5-Access Layer/)
   1. [Tầng API — API Gateway](5.5.1-APIGateway/)
   2. [S3 & CloudFront](5.5.2-S3&CloudFront/)
6. [Tự động hóa & Vận hành](5.6-Automation&Operations/)
   1. [CI/CD & Tự động hóa](5.6.1-CICD&Automation/)
   2. [Giám sát & Bảo mật](5.6.2-Monitoring&Security/)
7. [Kiểm thử](5.7-Testing/)
8. [Dọn dẹp tài nguyên](5.8-Cleanup/)