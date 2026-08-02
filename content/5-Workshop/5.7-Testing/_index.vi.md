---
title : "Testing & Measurement"
date : 2026-07-14
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

#### Log (Logs)

**Log của Lambda.** Mỗi lần gọi doc-summarizer-fn và doc-summarizer-report-fn đều tự động ghi log vào CloudWatch Logs.

1. Console **CloudWatch** → **Log groups** → /aws/lambda/doc-summarizer-fn.
2. Nhấn vào log stream gần nhất để xem output của từng lần gọi, bao gồm cả các câu lệnh print() trong code Lambda.

**Log thực thi của API Gateway.** Mặc định bị tắt phải bật thủ công cho từng stage.

1. Console **API Gateway** → doc-summarizer-api → **Stages** → v1 → tab **Logs and tracing**.
2. Bật **CloudWatch Logs**, mức log **INFO** hoặc **ERROR**, và **Log full requests/responses data** để phục vụ debug, nhớ tắt lại sau khi dùng xong, vì tính năng này có thể ghi log cả những nội dung nhạy cảm trong request body.
3. Yêu cầu phải có một IAM role với quyền CloudWatch Logs được gắn ở cấp tài khoản cho API Gateway (**API Gateway** → **Settings** → **CloudWatch log role ARN**) đây là một thiếu sót thường gặp nói chung, tuy nhiên module API Gateway của dự án này đã cung cấp sẵn role đó thông qua Terraform.

**Log của CodeBuild.** Mỗi giai đoạn (stage) trong pipeline có một project CodeBuild tự động stream output build lên CloudWatch Logs.

1. Console **CodePipeline** → doc-summarizer-pipeline → nhấn vào một stage → link **Details** ở action tương ứng.
2. Thao tác này sẽ mở trang build của CodeBuild, hiển thị toàn bộ output của các lệnh đã chạy.

**Cách đọc một stage pipeline bị thất bại:** cuộn xuống dòng đầu tiên chứa từ FAILED hoặc exit code khác 0. Log của CodeBuild được sắp xếp theo trình tự thời gian, và nguyên nhân thất bại thực sự thường nằm gần cuối chứ không phải ở đầu, vì các bước sau vẫn sẽ cố gắng chạy cleanup ngay cả khi đã có lỗi xảy ra trước đó trong cùng giai đoạn.

#### Số liệu 

**Metric có sẵn:**

| Metric | Nguồn | Cho biết điều gì |
|---|---|---|
| Duration, Errors, Throttles, Invocations | Lambda | Thời gian thực thi, tỷ lệ thất bại, giới hạn concurrency bị chạm tới |
| 4XXError, 5XXError, Count, Latency | API Gateway | Tỷ lệ lỗi từ phía client so với server, tổng lưu lượng truy cập |
| ConsumedReadCapacityUnits, ConsumedWriteCapacityUnits | DynamoDB | Mức sử dụng on-demand thực tế hữu ích để theo dõi chi phí |

**Custom metric**: namespace Custom/Bedrock, được code Lambda gửi lên một cách tường minh:

| Metric | Ý nghĩa |
|---|---|
| Latency | Số mili-giây để lệnh gọi invoke_model của Bedrock hoàn tất |
| SuccessCount | Tăng lên sau mỗi lần gọi Bedrock thành công |
| BedrockErrors | Tăng lên khi có bất kỳ lỗi Bedrock nào, được phân loại theo dimension ErrorType |

Dimension ErrorType chính là yếu tố giúp phân biệt được "đã chạm hạn ngạch hàng ngày" với "throttling thông thường" hay "model timeout" trên dashboard, thay vì gộp tất cả các lỗi Bedrock vào chung một con số lỗi không phân loại.

#### Cảnh báo 

Bước xác minh: gửi thủ công một điểm dữ liệu thử nghiệm tới Custom/Bedrock / BedrockErrors, chờ hết khoảng thời gian đánh giá (evaluation period) của cảnh báo, rồi xác nhận cả trạng thái CloudWatch alarm chuyển sang **In alarm** lẫn email từ SNS đã được gửi tới. Bước này đáng để chạy lại tại đây như một phần của đợt kiểm thử tổng thể, chứ không chỉ chạy một lần khi thiết lập ban đầu **cấu hình cảnh báo có thể âm thầm bị hỏng mà không có dấu hiệu rõ ràng nào cho đến đúng thời điểm cần dùng đến nó**.

#### Kế hoạch kiểm thử thủ công

Sử dụng curl để gọi vào API đã triển khai (https://YOUR-API-ID.execute-api.ap-southeast-1.amazonaws.com/v1):

**1. Request hợp lệ có xác thực:**
```bash
curl -X POST https://YOUR-API-ID.execute-api.ap-southeast-1.amazonaws.com/v1/summarize \
  -H "Content-Type: application/json" \
  -H "Authorization: YOUR-ID-TOKEN" \
  -H "x-api-key: YOUR-API-KEY" \
  -d '{"text": "Amazon Web Services provides a broad set of global cloud services including compute, storage, databases, and AI tools."}'
```
Kết quả mong đợi: 200 kèm trường summary, hoặc 429 nếu hạn ngạch Bedrock hàng ngày hiện đã cạn.

**2. Thiếu token:**
```bash
curl -X POST https://YOUR-API-ID.execute-api.ap-southeast-1.amazonaws.com/v1/summarize \
  -H "x-api-key: YOUR-API-KEY" \
  -d '{"text": "test"}'
```
Kết quả mong đợi: 401 Unauthorized xác nhận Cognito authorizer đang được thực thi.

**3. Thiếu API key:**
```bash
curl -X POST https://YOUR-API-ID.execute-api.ap-southeast-1.amazonaws.com/v1/summarize \
  -H "Authorization: YOUR-ID-TOKEN" \
  -d '{"text": "test"}'
```
Kết quả mong đợi: 403 Forbidden xác nhận yêu cầu API key của usage plan đang được thực thi.

**4. GET /history:**
```bash
curl https://YOUR-API-ID.execute-api.ap-southeast-1.amazonaws.com/v1/history \
  -H "Authorization: YOUR-ID-TOKEN" \
  -H "x-api-key: YOUR-API-KEY"
```
Kết quả mong đợi: 200 kèm một mảng history chứa các mục trước đó của người dùng này.

**5. Ranh giới kiểm tra tính hợp lệ của input:**
```bash
# Quá ngắn (dưới 20 ký tự)
curl -X POST .../summarize -H "..." -d '{"text": "short"}'
# Kết quả mong đợi: 400

# Quá dài (trên 5000 ký tự)
curl -X POST .../summarize -H "..." -d '{"text": "'"$(python3 -c 'print("a"*5001)')"'"}'
# Kết quả mong đợi: 400
```

#### Kiểm thử tự động

Thư mục tests/ chứa các test pytest chạy trực tiếp trên các hàm xử lý của Lambda, sử dụng moto để giả lập DynamoDB thay vì gọi tới tài khoản AWS thật.

1. Cài đặt các dependency:
   ```bash
   pip install -r requirements-test.txt --break-system-packages
   ```
2. Chạy bộ test:
   ```bash
   pytest tests/ -v
   ```
3. Phạm vi kiểm thử điển hình: ranh giới kiểm tra tính hợp lệ của input, trường summary_date được tính toán và ghi đúng trên mỗi lần PutItem, hàm handle_history trả về kết quả được sắp xếp mới nhất trước, và luồng xử lý DailyQuotaExceededError trả về mã 429 thay vì để lỗi lan truyền thành mã 500.

moto chặn các lệnh gọi boto3 và mô phỏng DynamoDB trong bộ nhớ, nhờ đó các test này chạy được trong CI mà không cần AWS credentials thật hay đụng đến SummarizerTable thực tế.

#### Kiểm thử tải

locustfile.py có thể chạy ở hai chế độ khác nhau, và điều quan trọng là phải biết chế độ nào đã tạo ra một kết quả cụ thể trước khi đưa ra kết luận từ đó:

- **MOCK_MODE=true** (mặc định) bỏ qua bước xác thực, thay vào đó dùng một Bearer token giả cố định, và các request được gửi tới bất kỳ đâu mà --host trỏ tới. Chế độ này kiểm tra hợp đồng request/response và khả năng xử lý concurrency mà không phụ thuộc vào việc Cognito hay Bedrock có sẵn sàng hay không.
- **MOCK_MODE=false** script gọi trực tiếp cognito-idp:InitiateAuth với USER_PASSWORD_AUTH thông qua boto3 để lấy JWT thật, và mỗi request đều mang theo header x-api-key thật. Đây là chế độ duy nhất thực sự kiểm thử Bedrock.

**Kết quả ở chế độ mock 50 người dùng đồng thời:**

| Metric | Giá trị |
|---|---|
| Tổng số request | 11.363 |
| Thất bại | 0 (0.00%) |
| Thời gian phản hồi trung bình | 4 ms |
| p95 | 5 ms |
| p99 | 7 ms |
| Thông lượng ổn định | ~24 req/giây |

| Endpoint | Số request | TB | Trung vị | p95 | p99 | Max |
|---|---|---|---|---|---|---|
| GET /health | 2.279 | 3 ms | 4 ms | 5 ms | 6 ms | 25 ms |
| GET /history | 2.282 | 3 ms | 4 ms | 5 ms | 7 ms | 24 ms |
| POST /summarize | 6.802 | 3 ms | 4 ms | 6 ms | 7 ms | 23 ms |

Số lượng người dùng tăng dần từ 0 lên 50 trong khoảng ~50 giây đầu; sau khi ổn định, thông lượng giữ ở mức ~24 req/giây với 0 thất bại trên cả ba endpoint trong suốt phần còn lại của lần chạy, thời gian phản hồi không hề suy giảm dù mức concurrency tăng gấp 5 lần so với baseline 10 người dùng trước đó (4ms TB / 6ms p95 / 7ms p99). Điều này cho thấy nút thắt cổ chai được mô tả bên dưới là đặc thù của các lệnh gọi Bedrock thật, chứ không phải do việc xử lý request, luồng xác thực, hay cách truy cập DynamoDB.

**Chạy kiểm thử:**
```bash
pip install boto3 locust --break-system-packages
locust -f locustfile.py --headless -u 50 -r 1 --run-time 60s --host <target-host>
```
Đặt `--host` là `http://localhost:8000` để test với mock server, hoặc dùng URL invoke thật của API Gateway để kiểm thử luồng xác thực mock-mode với endpoint thật.

**Thiết lập cho API thật** (MOCK_MODE=false), để kiểm thử luồng xử lý thật có gọi Bedrock:

1. Tạo một user Cognito dùng cho load testing:
   ```bash
   AWS_PROFILE=YOUR-PROFILE aws cognito-idp admin-create-user \
     --user-pool-id YOUR-USER-POOL-ID \
     --username loadtest@example.com
   AWS_PROFILE=YOUR-PROFILE aws cognito-idp admin-set-user-password \
     --user-pool-id YOUR-USER-POOL-ID \
     --username loadtest@example.com \
     --password YourStrongPass123! \
     --permanent
   ```
2. Thiết lập .env.test:
   ```
   MOCK_MODE=false
   COGNITO_REGION=ap-southeast-1
   CLIENT_ID=<value from: terraform output -raw cognito_app_client_id>
   TEST_USERNAME=loadtest@example.com
   TEST_PASSWORD=YourStrongPass123!
   API_KEY=<value from: terraform output -raw api_key_value>
   ```
3. Chạy Locust nhắm vào endpoint thật:
   ```bash
   locust -f locustfile.py --host https://YOUR-API-ID.execute-api.ap-southeast-1.amazonaws.com/v1
   ```
4. Mở giao diện web của Locust (mặc định `http://localhost:8089`), thiết lập số lượng người dùng và tốc độ tăng dần (ramp-up rate), rồi bắt đầu kiểm thử.


**Kết quả load testing của dự án:** một lần chạy với 5 người dùng đồng thời, tăng dần với tốc độ 1 người/giây ở chế độ real-API, cho thấy GET /history hoạt động đúng, trong khi mọi request POST /summarize đều thất bại với mã 502/504 gần như đúng ở giây thứ 29 chính là mức giới hạn timeout tích hợp cứng của API Gateway. Nguyên nhân gốc rễ: logic retry của Lambda đã liên tục thử lại các lỗi throttling từ Bedrock kèm cơ chế backoff theo cấp số nhân, tiêu tốn toàn bộ 30 giây timeout của Lambda trước khi kịp trả về bất kỳ phản hồi nào. Kết quả ở chế độ mock nêu trên đã loại trừ khả năng nguyên nhân đến từ luồng request/xác thực, 50 người dùng đồng thời không gây ra thất bại nào ở chế độ mock, vì vậy lỗi 502/504 là đặc thù của các lệnh gọi Bedrock thật, không phải vấn đề về concurrency hay luồng xử lý code.

![overview](/images/5-Workshop/5.5-Testing/Locust_testingtesting.jpeg)


#### Các lỗi thường gặp và cách khắc phục

| Lỗi | Nguyên nhân | Cách khắc phục |
|---|---|---|
| pytest thất bại với NoRegionError hoặc cố gắng gọi AWS thật | Thiếu decorator moto mock trên một test, hoặc resource DynamoDB được tạo bên ngoài phạm vi được mock | Kiểm tra hàm test đã được bọc bằng moto mock trước khi gọi boto3.resource('dynamodb') chưa |
| Log thực thi của API Gateway vẫn trống dù đã bật cho từng stage | Thiếu IAM role CloudWatch Logs ở cấp tài khoản cho API Gateway | Kiểm tra role còn được gắn tại **API Gateway → Settings → CloudWatch log role ARN** không |
| Locust InitiateAuth thất bại với NotAuthorizedException | Mật khẩu user test chưa được đặt ở chế độ permanent, hoặc USER_PASSWORD_AUTH chưa được bật trên app client | Kiểm tra đã dùng admin-set-user-password --permanent chưa, và cấu hình auth flow đã được áp dụng chưa |
| POST /summarize liên tục timeout ở khoảng giây thứ 29 khi có tải | Logic retry của Bedrock tiêu tốn hết toàn bộ timeout của Lambda khi gặp lỗi quota | Kiểm tra đường xử lý fast-fail cho lỗi quota hàng ngày đã được triển khai chưa, hay vẫn là phiên bản cũ không có nó |
| Một lần chạy locust báo mode: MOCK trong log khởi động | Biến môi trường MOCK_MODE đang là true | Đây là kết quả bình thường cho baseline chạy ở chế độ mock; nếu mục đích là lấy kết quả từ API thật, hãy đặt MOCK_MODE=false và xác nhận .env.test đã được load trước khi chạy lại |