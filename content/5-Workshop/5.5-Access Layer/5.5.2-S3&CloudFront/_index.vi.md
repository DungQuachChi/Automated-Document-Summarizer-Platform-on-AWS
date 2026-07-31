---
title : "Frontend Hosting — S3 & CloudFront"
date : 2026-07-14
weight : 2
chapter : false
pre : " 5.5.2. "
---

#### Mục tiêu

Lưu trữ frontend tĩnh trên S3, phân phối thông qua CloudFront, và tự động truyền tên miền (domain) kết quả sang Cognito và API Gateway bằng cách kết nối thông qua Terraform output.

#### Tài nguyên Terraform

modules/frontend:

- aws_s3_bucket ở chế độ private (chặn toàn bộ quyền truy cập công khai) + aws_cloudfront_distribution sử dụng Origin Access Control — CloudFront xác thực với S3 bằng SigV4, nhờ đó bucket luôn ở trạng thái riêng tư hoàn toàn. HTTP được chuyển hướng sang HTTPS, sử dụng PriceClass_100.
- Bucket policy cấp quyền cloudfront.amazonaws.com s3:GetObject, được giới hạn phạm vi thông qua AWS:SourceArn chỉ dành cho đúng distribution này.
- Versioning, CloudFront access logging, và WAF Web ACL đều cố tình được bỏ qua — chi phí không tương xứng với quy mô của dự án này.

#### Kết nối giữa các Module

module.frontend.cloudfront_domain_name được truyền trực tiếp sang cả hai module còn lại:

```hcl
module "auth" { cloudfront_domain_name = module.frontend.cloudfront_domain_name }
module "api"  { cloudfront_domain_name = module.frontend.cloudfront_domain_name }
```

Cả callback_urls/logout_urls của Cognito lẫn CORS origin của API Gateway đều được lấy từ chính một output này. Chỉ cần một lần terraform apply là đã khởi tạo bucket/distribution, đọc lấy domain kết quả, và truyền nó vào cả hai module — không cần thêm bước riêng nào để cập nhật thủ công Cognito hay CORS sau đó.

#### Áp dụng và triển khai file

```bash
terraform apply
terraform output cloudfront_domain_name
```

```bash
# trước tiên hãy đặt API_BASE_URL trong frontend/script.js thành giá trị output api_invoke_url
aws s3 cp frontend/index.html s3://$(terraform output -raw frontend_bucket_name)/
aws s3 cp frontend/script.js  s3://$(terraform output -raw frontend_bucket_name)/
aws cloudfront create-invalidation \
  --distribution-id $(terraform output -raw cloudfront_distribution_id) \
  --paths "/*"
```

**Cách kiểm tra:** mở domain CloudFront — frontend tải lên bình thường, việc đăng nhập chuyển hướng chính xác. Một request truy cập trực tiếp vào URL object của S3 sẽ trả về lỗi **AccessDenied**.

#### Các lỗi thường gặp và cách khắc phục

| Lỗi | Nguyên nhân | Cách khắc phục |
|---|---|---|
| AccessDenied khi tải URL CloudFront | SourceArn trong bucket policy chưa khớp với ARN của distribution | Chạy terraform apply lại — policy được sinh ra dựa trên resource distribution nên sẽ tự động sửa lại đúng |
| script.js cũ vẫn được phục vụ sau khi upload lại | CloudFront cache tại edge | Chạy lại bước invalidation sau mỗi lần cập nhật file frontend |
| Cognito chuyển hướng đến trang trống/trang lỗi sau khi đăng nhập | Chưa chạy terraform apply kể từ khi module frontend được thêm vào | Chạy terraform apply — không cần chỉnh sửa Cognito thủ công |

---

### Tóm tắt phần này

Chúc mừng bạn đã hoàn thành tầng access layer. Trong phần này, bạn đã thiết lập Cognito để cấp phát và xác thực JWT, cấu hình API Gateway để bắt buộc mỗi request phải kèm theo token đó cùng với một API key và usage plan, và lưu trữ frontend tĩnh trên S3 phía sau CloudFront bằng cách sử dụng Origin Access Control để bucket luôn ở trạng thái riêng tư. Các thành phần này tự động kết nối với nhau vì tên miền CloudFront được truyền dưới dạng Terraform output vào cả callback URL của app client Cognito lẫn cấu hình CORS của API Gateway — chỉ một lệnh **terraform apply** duy nhất là giữ cho cả ba đồng bộ với nhau, không cần đối chiếu thủ công qua lại giữa các console. Điều này thể hiện chiến lược phòng thủ theo chiều sâu (defense in depth) ngay tại lớp biên: xác thực (authentication), phân quyền (authorization) và giới hạn tần suất (rate limiting) đều được thực thi trước khi một request có thể chạm tới code ứng dụng, và frontend chỉ có thể được truy cập thông qua CDN, không bao giờ truy cập trực tiếp.