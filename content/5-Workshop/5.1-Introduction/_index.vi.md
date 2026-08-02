---
title: "Giới thiệu"
date: 2026-07-14
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### Kiến trúc Serverless

- **Kiến trúc Serverless** chỉ thực thi mã nguồn khi có yêu cầu (request) gửi đến, sau đó sẽ tắt ngay lập tức. Không có máy chủ nào cần bạn phải vá lỗi, mở rộng quy mô hay duy trì hoạt động 24/7.
- Bài thực hành này sử dụng bốn dịch vụ AWS cốt lõi để xây dựng một quy trình xử lý yêu cầu (request pipeline) dạng serverless: **Cognito** xác thực người dùng, **API Gateway** kiểm tra tính hợp lệ của yêu cầu, **Lambda** thực thi logic nghiệp vụ và **Bedrock** tạo bản tóm tắt bằng AI.

#### Tổng quan bài thực hành

Trong bài thực hành này, bạn sẽ xây dựng một nền tảng tóm tắt tài liệu hoàn chỉnh từ đầu đến cuối (end-to-end).

- **Tính năng**: Phía ứng dụng (client) gửi văn bản dài (ghi chú bài giảng, tài liệu đọc, bài báo) thông qua một API bảo mật và nhận lại bản tóm tắt do AI tạo ra nhờ sức mạnh của Amazon Bedrock.
- **Nội dung trọng tâm**: Các thực hành DevOps cần thiết để xây dựng, bảo mật và vận hành tính năng đó: Infrastructure as Code với Terraform, quy trình CI/CD với bước phê duyệt thủ công, hệ thống giám sát CloudWatch đi kèm cảnh báo kích hoạt thực tế, cùng tiêu chuẩn an toàn bảo mật tuân thủ định hướng CIS.

Trong quá trình xây dựng hệ thống, tài khoản AWS được sử dụng đã chạm mức **hạn mức on-demand bằng 0 (zero on-demand quota)** đối với các mô hình Bedrock, một giới hạn mà AWS áp dụng cho các tài khoản mới. Một AWS Support case xin tăng hạn mức đã bị từ chối ở thời điểm hiện tại, vì khả năng truy cập phụ thuộc vào tuổi tài khoản và mức độ sử dụng, và được đánh giá lại tự động theo thời gian. Trong khi chờ, cờ tính năng (feature flag) `MOCK_SUMMARIZE` ở phía frontend trả về một bản tóm tắt giả để phần demo vẫn dùng được; bản thân Lambda vẫn luôn gọi Bedrock thật, và chỉ cần tắt cờ này (không cần sửa code) là chức năng suy luận thật sẽ hoạt động ngay khi tài khoản đủ điều kiện (xem mục 5.4.2).

![overview](/images/5-Workshop/5.1-Workshop-overview/diagram1.png)

#### Mục tiêu / Kết quả mong muốn

Sau khi hoàn thành bài thực hành này, các thành phần sau sẽ được triển khai và xác minh:

- Một REST API hoạt động tốt và đã được xác thực (các endpoint `/summarize` và `/history` được bảo vệ bởi Cognito)
- Giao diện tĩnh (static frontend) được triển khai qua S3 + CloudFront
- Một dashboard CloudWatch hiển thị lưu lượng truy cập thực tế
- Ít nhất một cảnh báo SNS được kích hoạt và gửi tới email
- Quy trình CI/CD tự động hoàn toàn với bước phê duyệt thủ công trước khi bất kỳ thay đổi hạ tầng nào được áp dụng
- Dọn dẹp tài nguyên triệt để, đã được xác minh để không phát sinh chi phí duy trì

#### Tiêu chí thành công

- Luồng xác thực hoạt động hoàn chỉnh end-to-end (Cognito Hosted UI → JWT → gọi API hợp lệ)
- Dashboard CloudWatch hiển thị lưu lượng yêu cầu thực tế
- Ít nhất một cảnh báo CloudWatch được kích hoạt và gửi email qua SNS
- Chi phí hàng tháng duy trì dưới $50 cho tất cả các dịch vụ
- Tất cả tài nguyên đã được xóa bỏ hoàn toàn mà không để lại chi phí phát sinh ẩn (được xác minh trong Cost Explorer)