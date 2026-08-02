---
title : "AI Integration — Bedrock"
date : 2026-07-14
weight : 2
chapter : false
pre : " 5.4.2. "
---

#### Mục tiêu

Yêu cầu quyền truy cập vào Amazon Nova Lite trên Bedrock, hiểu cách Lambda gọi model này qua các region khác nhau, và xử lý tình huống hạn ngạch on-demand của tài khoản bị cạn kiệt.

#### Kết nối trong Terraform: IAM và Model ID

- IAM policy giới hạn phạm vi của bedrock:InvokeModel chỉ đúng hai tài nguyên: ARN của foundation model và ARN của cross-region inference profile, không sử dụng wildcard.
- BEDROCK_MODEL_ID = "us.amazon.nova-lite-v1:0" là biến môi trường của Lambda được Terraform thiết lập, không hardcode trong Python.

#### Yêu cầu về Cross-Region

ap-southeast-1 không nằm trong nhóm inference khu vực AP dành cho Nova Lite. Bedrock client của Lambda được hardcode để gọi tới us-east-1 thay thế:

```python
bedrock_client = boto3.client('bedrock-runtime', region_name='us-east-1')
```

Bản thân Lambda vẫn tiếp tục chạy tại ap-southeast-1; chỉ có lệnh gọi API Bedrock là vượt qua region khác.

#### Thực trạng về Quota

Các tài khoản AWS mới được cấp hạn ngạch on-demand là **0 request/giây** cho một model Bedrock nhất định, ngay cả sau khi quyền truy cập model đã được chấp thuận. Đây là một giới hạn thực sự ở cấp độ tài khoản, không phải lỗi của dự án này. Mọi lệnh gọi **/summarize** đều thất bại với ThrottlingException.

Chúng mình đã mở một AWS Support case để xin tăng hạn ngạch. AWS hiện từ chối, giải thích rằng khả năng truy cập model phụ thuộc vào tuổi tài khoản, lịch sử thanh toán và mức độ sử dụng, rằng điều này được đánh giá lại tự động theo thời gian, và không phải là một giới hạn vĩnh viễn. Nói cách khác, rào cản nằm ở lịch sử thanh toán, không phải ở cấu hình.

```python
is_daily_quota = (
    error_code == 'ThrottlingException'
    and ('daily' in error_msg or 'per day' in error_msg or 'toomanytokens' in error_msg)
)
```

Lỗi throttling tạm thời sẽ được thử lại với backoff; còn lỗi vượt quota hàng ngày sẽ thất bại ngay lập tức thay vì retry, vì việc thử lại chỉ tốn hết 30 giây timeout của Lambda mà không mang lại lợi ích gì.

Lambda đã triển khai **không có đường dẫn mock (mock path)** của riêng nó. Nó luôn gọi Bedrock thật và trả về lỗi 429 thực sự cho người dùng thật khi quota đã cạn:

```json
{"message": "Summarization limit reached for today. Please try again after midnight UTC."}
```

Để giữ cho phần demo ở frontend vẫn dùng được trong khi quyền truy cập còn bị giới hạn, trang tĩnh có một flag tên là `MOCK_SUMMARIZE` trả về một bản tóm tắt giả. Việc tắt cờ này chỉ là thay đổi một flag, không cần sửa code, nên chức năng suy luận thật sẽ hoạt động ngay khi tài khoản đủ điều kiện. Bản thân phần tích hợp đã được kiểm chứng bằng bộ test pytest: nó mock Bedrock và kiểm tra rằng payload của request, luồng retry, và việc phân loại lỗi 429 do vượt quota đều đúng.

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