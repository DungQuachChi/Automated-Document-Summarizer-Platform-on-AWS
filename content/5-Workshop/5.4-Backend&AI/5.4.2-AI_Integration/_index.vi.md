---
title : "AI Integration — Bedrock"
date : 2026-07-14
weight : 2
chapter : false
pre : " 5.4.2. "
---

#### Mục tiêu

Yêu cầu quyền truy cập vào Amazon Nova Lite trên Bedrock, hiểu cách Lambda gọi model này qua các region khác nhau, và xử lý tình huống hạn ngạch (quota) on-demand của tài khoản bị cạn kiệt.

#### Kết nối trong Terraform: IAM và Model ID

- IAM policy giới hạn phạm vi của bedrock:InvokeModel chỉ đúng hai tài nguyên: ARN của foundation model và ARN của cross-region inference profile — không sử dụng wildcard.
- BEDROCK_MODEL_ID = "us.amazon.nova-lite-v1:0" là biến môi trường của Lambda được Terraform thiết lập, không hardcode trong Python.

#### Yêu cầu về Cross-Region

ap-southeast-1 không nằm trong nhóm inference (inference pool) khu vực AP dành cho Nova Lite. Bedrock client của Lambda được hardcode để gọi tới us-east-1 thay thế:

```python
bedrock_client = boto3.client('bedrock-runtime', region_name='us-east-1')
```

Bản thân Lambda vẫn tiếp tục chạy tại ap-southeast-1; chỉ có lệnh gọi API Bedrock là vượt qua region khác.

#### Thực trạng về Quota

Các tài khoản AWS mới được cấp hạn ngạch on-demand là **0 request/giây** cho một model Bedrock nhất định, ngay cả sau khi quyền truy cập model đã được chấp thuận — đây là một giới hạn thực sự ở cấp độ tài khoản, không phải lỗi của dự án này. Mọi lệnh gọi **/summarize** đều thất bại với ThrottlingException cho đến khi một AWS Support case được mở để tăng hạn ngạch.

```python
is_daily_quota = (
    error_code == 'ThrottlingException'
    and ('daily' in error_msg or 'per day' in error_msg or 'toomanytokens' in error_msg)
)
```

Lỗi throttling tạm thời sẽ được thử lại (retry) với backoff; còn lỗi vượt quota hàng ngày sẽ thất bại ngay lập tức thay vì retry, vì việc thử lại chỉ tốn hết 30 giây timeout của Lambda mà không mang lại lợi ích gì.

Lambda đã triển khai **không có đường dẫn mock (mock path)** — nó luôn gọi Bedrock thật, và trả về lỗi 429 thực sự cho người dùng thật nếu quota đã cạn:

```json
{"message": "Summarization limit reached for today. Please try again after midnight UTC."}
```

Kiểm tra CloudWatch → Metrics → Custom/Bedrock → BedrockErrors, dimension ErrorType = DailyQuotaExceeded, để xác nhận Lambda đã phân loại lỗi đúng cách.

#### Các lỗi thường gặp và cách khắc phục

| Lỗi | Nguyên nhân | Cách khắc phục |
|---|---|---|
| AccessDeniedException khi gọi Bedrock | Chưa được cấp quyền truy cập model, hoặc ARN của IAM không khớp với model/region | Cấp quyền tại **Bedrock → Model access**; kiểm tra ARN của IAM có khớp với BEDROCK_MODEL_ID không |
| ValidationException: model identifier is invalid | Sai định dạng model ID | Kiểm tra lại BEDROCK_MODEL_ID phải chính xác là us.amazon.nova-lite-v1:0 |
| Lỗi 429 ở mọi request, kể cả ngay sau khi vừa yêu cầu quyền truy cập | Quota on-demand mặc định là 0 trên một số tài khoản mới | Mở một AWS Support case dưới mục Service Quotas |

---

### Tóm tắt phần này

Chúc mừng bạn đã hoàn thành tầng backend và tầng AI! Trong phần này, bạn đã thiết kế một bảng DynamoDB với GSI để thực hiện các truy vấn dựa trên ngày tháng, một IAM role thực thi cho Lambda với quyền hạn tối thiểu cần thiết, và kết nối dịch vụ Lambda để gọi Amazon Bedrock thực hiện tóm tắt văn bản qua các region khác nhau. Thành công của quy trình này đến từ việc mỗi quyền IAM được cấp đều khớp chính xác với một luồng xử lý code; không sử dụng wildcard cho action, không sử dụng wildcard cho resource ngoài phạm vi thực sự cần thiết cho cross-region inference. Hơn nữa, tình trạng cạn quota được xử lý như một trường hợp bình thường; nhờ đó chúng ta gặp lỗi 429 có thể đoán trước và xảy ra nhanh chóng thay vì một lỗi bất ngờ.