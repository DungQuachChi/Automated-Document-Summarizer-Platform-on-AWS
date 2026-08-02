---
title: "Tìm hiểu Multi-Region Event-Driven Failover Architecture với Amazon EventBridge và Route 53 – Kiến trúc dự phòng đa vùng cho ứng dụng dựa trên sự kiện"
date: 2026-08-01
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# Tìm hiểu Multi-Region Event-Driven Failover Architecture với Amazon EventBridge và Route 53 – Kiến trúc dự phòng đa vùng cho ứng dụng dựa trên sự kiện

Xin chào anh chị và các bạn, trong lúc mình tìm hiểu về thiết kế ứng dụng trên AWS, mình thấy việc xây dựng một hệ thống xử lý theo sự kiện ở phạm vi Single Region với các dịch vụ như API Gateway, EventBridge, SQS và Lambda đã khá phổ biến. Tuy nhiên, khi bài toán đặt ra yêu cầu High Availability và Disaster Recovery — tức là nếu nguyên một Region của AWS gặp sự cố thì ứng dụng vẫn phải hoạt động trơn tru — việc thiết kế sẽ phức tạp hơn nhiều.

Mình có đang đọc một bài viết rất hay trên AWS Compute Blog hướng dẫn cách kết hợp Amazon Route 53, Amazon EventBridge, API Gateway và DynamoDB Global Tables để tạo ra một kiến trúc Active-Passive Multi-Region Failover tự động hoàn toàn cho hệ thống Event-Driven.

## Cách kiến trúc này hoạt động

Mô hình này hoạt động dựa trên cơ chế Active-Passive giữa 2 Region:

- **Bước 1: Điều hướng traffic bằng Route 53**
  Client/Ứng dụng gửi request chứa event tới một Custom Domain chung qua Route 53. Route 53 sẽ liên tục chạy Health Check để kiểm tra trạng thái hoạt động của endpoint API Gateway ở Primary Region.

- **Bước 2: Xử lý Event ở Primary Region (Trạng thái bình thường)**
  Event gửi đến API Gateway → API Gateway chuyển tiếp trực tiếp (Service Integration) vào Amazon EventBridge Bus → EventBridge đẩy event vào SQS Queue → AWS Lambda đọc message từ SQS và ghi kết quả vào DynamoDB Global Table.

- **Bước 3: Tự động đồng bộ dữ liệu liên Region**
  DynamoDB Global Table tự động sao chép dữ liệu được ghi từ Primary Region sang Secondary Region gần như theo thời gian thực.

- **Bước 4: Tự động chuyển vùng (Failover) khi sự cố xảy ra**
  Nếu Primary Region gặp sự cố, Route 53 Health Check sẽ phát hiện thất bại và tự động chuyển hướng toàn bộ traffic sang API Gateway ở Secondary Region. Hệ thống ở Secondary Region tiếp tục tiếp nhận và xử lý event.

## Một vài điểm mình thấy cực kỳ hữu ích

Sau khi đọc bài viết, mình thấy mô hình này có nhiều ưu điểm đáng để học hỏi:

- **Tự động chuyển vùng không cần con người can thiệp (Zero Manual Intervention):** Nhờ cơ chế Route 53 Failover Routing dựa trên Health Check, việc chuyển đổi luồng dữ liệu khi sự cố xảy ra diễn ra hoàn toàn tự động.
- **Không tốn Lambda trung gian ở tầng tiếp nhận:** Sử dụng trực tiếp tính năng AWS Service Integration của API Gateway để gửi event thẳng vào EventBridge, giúp giảm bớt glue code và giảm latency.
- **Đảm bảo dữ liệu nhất quán với DynamoDB Global Tables:** Nhờ khả năng replicate đa vùng tự động của DynamoDB, Secondary Region luôn có sẵn dữ liệu mới nhất để tiếp tục phục vụ ứng dụng.
- **Kiến trúc Loose Coupling (Ghép nối lỏng lẻo):** Việc phân tách rõ ràng giữa khâu tiếp nhận (API Gateway/EventBridge), khâu hàng chờ (SQS) và khâu xử lý (Lambda) giúp hệ thống chống chịu tải tốt và không bị mất event trong quá trình switch-over giữa 2 vùng.

## Một số điểm cần lưu ý

- **Độ trễ chuyển vùng (Failover propagation time):** Health check của Route 53 cần một khoảng thời gian nhất định để xác nhận Primary Region thực sự gặp sự cố trước khi đổi hướng DNS.
- **Cấu hình SSL Certificate ở cả 2 Region:** Bạn cần tạo sẵn chứng chỉ SSL/TLS bằng AWS Certificate Manager (ACM) ở cả 2 Region cho cùng một Custom Domain Name.
- **Chi phí triển khai Multi-Region:** Việc nhân đôi tài nguyên ở 2 vùng sẽ làm tăng tổng hóa đơn AWS so với mô hình Single-Region.
- **Thứ tự dọn dẹp tài nguyên (Cleanup):** Khi thực hành xóa stack CloudFormation, cần xóa Secondary Stack trước rồi mới xóa Primary Stack, vì Primary Stack là nơi khởi tạo và nắm giữ DynamoDB Global Table.

## Sử dụng kiến trúc một cách phù hợp

- Các hệ thống thanh toán, xử lý đơn hàng, giao dịch tài chính hoặc hệ thống IoT nơi mà event loss hoặc ngắt kết nối có thể gây ra hậu quả lớn.
- Các ứng dụng có cam kết chất lượng dịch vụ (SLA) rất cao.
- Các hệ thống cần đáp ứng chiến lược Disaster Recovery cũng như hỗ trợ di chuyển traffic chủ động trong các đợt bảo trì hệ thống định kỳ (Planned Maintenance) mà người dùng không hề hay biết.

## Kết luận

Kiến trúc Multi Region Event Driven Failover kết hợp giữa EventBridge, Route 53 và DynamoDB Global Tables là một bài mẫu rất hay về tư duy xây dựng Fault Tolerant System trên AWS. Quy trình thiết lập ban đầu tốn nhiều công sức hơn, nhưng kết quả mang lại là một hạ tầng bền bỉ.

Nếu anh chị và các bạn đã từng triển khai kiến trúc Multi-Region cho ứng dụng Serverless rồi, có sai sót gì trong bài của mình hoặc là muốn bổ sung kiến thức thêm rất mong mình được nhận chia sẻ và góp ý của mọi người ở bên dưới ạ.

## Tài liệu tham khảo

- AWS Compute Blog – Multi-Region event-driven failover architecture with Amazon EventBridge and Route 53
  https://aws.amazon.com/blogs/compute/multi-region-event-driven-failover-architecture-with-amazon-eventbridge-and-route-53/
- Amazon EventBridge Documentation
  https://docs.aws.amazon.com/eventbridge/
- Amazon Route 53 Health Checks and DNS Failover
  https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-failover.html