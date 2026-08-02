---
title : "Operations — Monitoring & Security"
date : 2026-07-14
weight : 2
chapter : false
pre : " 5.6.2. "
---

#### Mục tiêu

Khả năng quan sát hệ thống đang chạy (dashboard, cảnh báo, custom metric) và một mức nền tảng cơ bản về bảo mật/tuân thủ.

#### Custom Metrics

src/lambda_fn/lambda_function.py gửi dữ liệu đến Custom/Bedrock quanh mỗi lệnh gọi model: Latency, SuccessCount, và khi thất bại là BedrockErrors được phân loại theo dimension ErrorType.

#### Cảnh báo (modules/monitoring)

Một SNS topic + đăng ký email, hai cảnh báo:
- **Tỷ lệ** lỗi 5xx của API Gateway (tính bằng metric-math errors/requests*100) > 5% trong 5 phút (là tỷ lệ, không phải số lượng tuyệt đối).
- Bất kỳ điểm dữ liệu nào xuất hiện tại Custom/Bedrock/BedrockErrors (ngưỡng = 0).

```bash
terraform apply   # xác nhận đăng ký email SNS sau đó, nếu không cảnh báo sẽ không được gửi
aws cloudwatch put-metric-data --namespace Custom/Bedrock --metric-name BedrockErrors --value 1
```

#### Dashboard

Một resource aws_cloudwatch_dashboard, sáu widget, số lượt gọi/lỗi/phân vị thời gian thực thi của Lambda, độ trễ/tỷ lệ thành công/lỗi của Bedrock, lưu lượng/lỗi 4xx/lỗi 5xx của API Gateway, dung lượng/throttle của DynamoDB. Các dimension được truyền vào dưới dạng biến module từ root main.tf.

#### CloudTrail & AWS Config (modules/security)

- CloudTrail đa vùng, có xác thực tính toàn vẹn file log, bucket S3 riêng biệt được mã hóa và ở chế độ private, đồng thời cũng được stream sang CloudWatch Logs.
- AWS Config recorder theo dõi tất cả các loại resource được hỗ trợ, bao gồm cả resource toàn cục (IAM), sử dụng conformance pack CIS v1.4 Level 1.

**Mức độ tuân thủ hiện tại: 60%**, với hai lỗ hổng được chấp nhận và ghi nhận công khai thay vì che giấu:
- IAM role của CodeBuild sử dụng AdministratorAccess.
- MFA chưa được bắt buộc trên toàn tài khoản thông qua IAM policy.

Cả hai vấn đề này sẽ được xem xét lại ở Mục 6.2 (Simple Security Hardening). Mục đích của việc chạy Config ở đây là để có một bức tranh chính xác, chứ không phải một bảng điểm hoàn hảo.

#### Các lỗi thường gặp và cách khắc phục

| Lỗi | Nguyên nhân | Cách khắc phục |
|---|---|---|
| Cảnh báo bị kẹt ở trạng thái **Insufficient Data** | Chưa có điểm dữ liệu nào được gửi lên | Tạo lưu lượng thực tế, hoặc gửi thủ công một metric thử nghiệm |
| Email từ SNS không bao giờ được gửi tới | Đăng ký (subscription) chưa được xác nhận | Kiểm tra thư mục spam; đăng ký lại nếu đã hết hạn |
| Conformance pack bị kẹt ở trạng thái **Evaluating** | Đây là điều bình thường ở lần chạy đầu tiên | Chờ 15–30 phút thay vì deploy lại |

---

### Tóm tắt phần này

Chúc mừng bạn đã hoàn thành tầng tự động hóa và vận hành! Trong phần này, bạn đã tạo một Lambda chạy theo lịch để tạo báo cáo sử dụng hàng tuần mà không cần bất kỳ sự can thiệp nào của con người, một pipeline CI/CD sẽ kiểm thử, quét lỗi, và cần con người phê duyệt trước khi thực hiện bất kỳ thay đổi hạ tầng nào, cùng với một nền tảng giám sát và bảo mật gồm dashboard, cảnh báo, CloudTrail, và AWS Config để theo dõi hệ thống sau khi đã triển khai. Tầng này hoạt động hiệu quả vì mọi giai đoạn có thể thay đổi hạ tầng thực tế đều có các