---
title : "Auth & API — Cognito & API Gateway"
date : 2026-07-14
weight : 1
chapter : false
pre : " 5.5.1. "
---

#### Mục tiêu

Thiết lập Cognito để cấp phát và xác thực JWT, cùng với API Gateway để bắt buộc mỗi request phải kèm theo token đó cộng với một API key và usage plan. Cả hai đều được khai báo trong Terraform, không cần cấu hình thủ công trên console sau khi deploy.

#### Cognito (modules/auth)

- Đối với aws_cognito_user_pool, việc đăng ký xác định người dùng qua email cùng với việc áp dụng chính sách mật khẩu mạnh, và advanced_security_mode được đặt là "ENFORCED", nhờ đó kích hoạt cơ chế xác thực thích ứng dựa trên rủi ro của Cognito dưới dạng phát hiện hoạt động sử dụng thông tin xác thực bị xâm phạm mà không cần thêm bất kỳ đoạn code nào trong ứng dụng. Ngoài ra, tính năng tự đăng ký được bật mặc định.
- Đối với aws_cognito_user_pool_client, đây là một public client cho phép explicit_auth_flows với ALLOW_USER_PASSWORD_AUTH và ALLOW_REFRESH_TOKEN_AUTH ngay từ đầu. Đường dẫn Hosted UI (cũng như đường dẫn xác thực trực tiếp dùng cho load-testing) đều sử dụng chung một client và không cần phải chuyển đổi client về sau.
- Đối với aws_cognito_user_pool_domain, tiền tố domain của Hosted UI phải được cấu hình thông qua biến.

#### API Gateway (modules/api)

- Bắt buộc phải dùng AWS API Gateway Rest API (loại REST, không phải HTTP) để có thể sử dụng được API key và usage plan. AWS API Gateway Authorizer (COGNITO_USER_POOLS) được kết nối tới cùng user pool ARN nêu trên.
- Các endpoint /summarize (POST) và /history (GET) đều là AWS_PROXY trỏ tới cùng một Lambda (Mục 5.4.1). Cả hai method đều được thiết lập authorization là "COGNITO_USER_POOLS" và tham số api_key_required trong code được đặt là true.
- Việc triển khai CORS đòi hỏi phải có một method OPTIONS tường minh cùng với một MOCK integration. Header Access-Control-Allow-Origin phải được cung cấp chính xác, được giới hạn chỉ trỏ tới domain của CloudFront.
- Sử dụng resource aws_api_gateway_deployment, chúng ta tạo một trigger cho từng ID của resource/method/integration/authorizer.
- Cả role logging ở cấp tài khoản (account-level CloudWatch logging role) lẫn role logging ở cấp stage đều phải được tạo tại đây.
- aws_api_gateway_usage_plan tạo ra một môi trường với giới hạn tần suất, mức burst khả dụng, và hạn ngạch hàng tháng (monthly quota).

#### Áp dụng (Applying)

```bash
terraform apply
terraform output cognito_user_pool_id
terraform output cognito_app_client_id
terraform output api_invoke_url
terraform output -raw api_key_value
```

#### Cách hoạt động của xác thực JWT

Trình duyệt → xác thực qua Hosted UI → Cognito kiểm tra (đánh giá rủi ro) → gửi trả về authorization code → ứng dụng nhận code và đổi lấy JWT thông qua /oauth2/token → với mỗi lệnh gọi API, JWT được gửi kèm trong header HTTP Authorization → Cognito authorizer của API Gateway xác minh JWT trước khi gọi tới hàm Lambda. Claim sub trong JWT được chuyển đổi thành user_id trong DynamoDB; do đó, mỗi người dùng có lịch sử riêng của mình.

Khi cần truy cập theo kiểu script/load-test, InitiateAuth/USER_PASSWORD_AUTH đã được bật sẵn trong cùng một client nhờ explicit_auth_flows, không cần thiết lập riêng cho việc này.

#### Các lỗi thường gặp và cách khắc phục

| Lỗi | Nguyên nhân | Cách khắc phục |
|---|---|---|
| invalid_request khi đăng nhập qua Hosted UI | Redirect URL không khớp với callback_urls | Kiểm tra khớp chính xác không có lỗi gõ nhầm/đường dẫn thừa |
| grant_type=password trả về lỗi 400 | Token endpoint của Hosted UI chỉ hỗ trợ authorization_code/refresh_token | Sử dụng InitiateAuth/USER_PASSWORD_AUTH thay thế |
| {"message": "Unauthorized"} / "Forbidden" từ API | JWT bị thiếu/hết hạn, hoặc x-api-key bị thiếu/không hợp lệ | Xác thực lại thông qua Hosted UI; kiểm tra x-api-key có khớp với terraform output -raw api_key_value không |
| Các thay đổi về Route/CORS dường như không được áp dụng | Nhầm lẫn với việc redeploy thủ công trên console | Chạy terraform plan nếu aws_api_gateway_deployment hiển thị pending replacement, chạy terraform apply |