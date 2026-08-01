---
title: "AgentForge Deepdive - Ngày 1"
date: 2026-08-01
weight: 1
chapter: false
pre: " <b> 4.x </b> "
---

# AgentForge - Xây dựng hệ thống Agentic sẵn sàng Production với Amazon Bedrock AgentCore (Ngày 1)

### Mục tiêu sự kiện

- Lý thuyết: Giới thiệu Amazon Bedrock AgentCore mức L300 (Runtime, Gateway, Identity)
- Thực hành: Xây dựng & Triển khai AI Agent trên Amazon Bedrock AgentCore bằng Vibe Coding với Kiro
  - Triển khai một agent cơ bản trên AgentCore
  - Kết nối với các công cụ bên ngoài và knowledge base
  - Thêm giao diện web với xác thực Cognito

### Diễn giả

- *(bổ sung sau)*

### Điểm nổi bật

#### MCP và A2A giải quyết vấn đề gì

- **A2A (Agent2Agent)**: giao thức giao tiếp giữa các agent với nhau
- **MCP (Model Context Protocol)**: giao thức chuẩn để agent truy cập công cụ/dữ liệu (Slack, GitLab, S3, Jira, API nội bộ) mà không cần viết tích hợp riêng cho từng công cụ

#### Strands Agents SDK

- SDK mã nguồn mở để xây dựng agent với lượng code tối thiểu
- Vòng lặp cốt lõi: prompt → agent → gọi model (nhận reasoning/lựa chọn tool) → thực thi tool → trả kết quả → phản hồi cuối cùng
- Hỗ trợ tool có sẵn + MCP, tích hợp với các dịch vụ AWS, hỗ trợ nhiều nhà cung cấp model tùy chỉnh

#### Amazon Bedrock AgentCore — tổng quan nền tảng

- Ba trụ cột: triển khai agent nhanh, kết nối với mọi thứ, tối ưu liên tục
- Nguyên tắc nền tảng: bảo mật ở quy mô lớn, sẵn sàng cho doanh nghiệp, kiểm soát tất định (deterministic)

#### AgentCore Runtime

- Môi trường runtime serverless bảo mật, chuyên dùng để triển khai/mở rộng agent và tool (ví dụ: MCP server), không phụ thuộc framework/giao thức/model
- Đóng gói dưới dạng Docker image (tối đa 2GB, qua ECR) hoặc file zip (tối đa 250MB nén / 750MB chưa nén, qua S3)
- Endpoint và phiên bản: có thể tạo/cập nhật phiên bản agent độc lập với endpoint nào (ví dụ DEFAULT, PROD) đang trỏ tới nó
- Cô lập phiên (session) thực sự: mỗi session chạy trong một microVM riêng (Firecracker) — tách biệt compute, memory, filesystem cho từng session
- Hỗ trợ tác vụ chạy nền bất đồng bộ/dài hạn và streaming âm thanh + văn bản hai chiều (ví dụ: Nova Sonic 2, Google Live API, OpenAI Realtime API)

#### AgentCore Identity

- Xử lý xác thực chiều vào (user → app → agent) và chiều ra (agent → tool/vault)
- Bốn thành phần: Workload Identities, Credential Providers, Token Vault, Broker Logic
- Client secret không bao giờ rời khỏi vault và không bao giờ tiếp cận được code của agent hay LLM

#### AgentCore Gateway

- Điểm truy cập duy nhất kết nối nhiều agent với nhiều API/tool/tài nguyên phía sau
- Tính năng sẵn có: hỗ trợ MCP, tạo/tìm kiếm tool, phân quyền, kiểm soát truy cập chi tiết, kết nối riêng tư (private connectivity), lọc tool
- Được hỗ trợ bởi AgentCore Identity (xác thực) và CloudWatch (observability)
- Hỗ trợ kết nối inbound riêng tư an toàn (PrivateLink) cho client nằm trên VPC khác hoặc mạng nội bộ công ty

#### Thực hành (vibe coding với Kiro)

- Kiro: IDE tích hợp AI, sinh code từ mô tả bằng ngôn ngữ tự nhiên (vibe coding) — không cần viết code thủ công
- Bài lab xây dựng agent Returns & Refunds theo từng giai đoạn: agent cơ bản → bộ nhớ liên tục (persistent memory) → Gateway/Lambda/xác thực Cognito → triển khai Runtime → observability → evaluations → policies
- Tiến độ hôm nay: hoàn thành Lab 1 (thiết lập/tính năng Kiro) và một phần Lab 2 (khoảng đến phần xây dựng agent + bộ nhớ); chưa đến các phần Gateway, giao diện web/Cognito, observability, evaluations, và policies

### Bài học rút ra

- AgentCore phân tách rõ ràng các mối quan tâm: Runtime (thực thi/mở rộng quy mô), Gateway (truy cập tool/API), Identity (xác thực) — mỗi phần có thể cấu hình độc lập
- Cô lập session bằng microVM là một đảm bảo kiến trúc thực sự, không chỉ là lời quảng cáo — quan trọng khi xử lý trạng thái riêng theo từng user
- Vibe coding (Kiro) chuyển trọng tâm từ việc viết code hạ tầng sang việc mô tả ý định một cách chính xác; các tài nguyên AWS bên dưới (Lambda, Cognito, IAM) vẫn được tạo ra và vẫn cần được hiểu rõ để debug và kiểm soát chi phí
- Các mẫu xử lý tác vụ bất đồng bộ/chạy nền quan trọng với bất kỳ agent nào gọi tool chậm hoặc chạy quy trình nhiều bước

### Áp dụng vào công việc

- Xem xét liệu các lệnh gọi Lambda/Bedrock trực tiếp hiện tại của [[doc-summarizer]] có nên dùng AgentCore Gateway hay không, nếu dự án cần gọi nhiều tool/API bên ngoài qua một lớp quản lý duy nhất trong tương lai
- So sánh cách xử lý xác thực hiện tại của dự án với mô hình AgentCore Identity (token workload ngắn hạn, secret không chạm vào code ứng dụng)
- Xem lại cách xử lý session/state trong các hàm Lambda của dự án so với mô hình cô lập session của Runtime
- Tiếp tục bài lab trong buổi sau để hoàn thành các phần Gateway, giao diện web Cognito, và observability

### Trải nghiệm sự kiện

Tham dự AgentForge Deepdive Ngày 1, bao gồm phần lý thuyết về Amazon Bedrock AgentCore (Runtime, Gateway, Identity) và bài thực hành xây dựng agent Returns & Refunds bằng vibe coding với Kiro. Đã hoàn thành Lab 1 và một phần Lab 2; các phần lab còn lại (Gateway, giao diện web Cognito, observability, evaluations, policies) vẫn còn phải làm.