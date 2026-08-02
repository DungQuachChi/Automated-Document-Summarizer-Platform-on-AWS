---
title : "Dọn Dẹp"
date : 2026-07-14
weight : 8
chapter : false
pre : " <b> 5.8. </b> "
---


#### Thứ Tự Gỡ Bỏ Hạ Tầng

Terraform thường tự động xử lý thứ tự phụ thuộc, nhưng một vài tài nguyên AWS sẽ chặn việc xóa cho đến khi một điều kiện thủ công được đáp ứng bất kể Terraform cố gắng làm gì. Hai điều quan trọng nhất ở đây:

- **Các bucket S3 phải được làm trống trước khi có thể xóa**, trừ khi tài nguyên bucket đặt force_destroy = true. Nếu bucket còn chứa object, việc xóa sẽ thất bại với lỗi BucketNotEmpty.
- **Các CloudFront distribution phải được tắt trước khi có thể xóa an toàn origin bên dưới (bucket S3)**, và bản thân distribution phải hoàn tất quá trình tắt trước khi có thể bị xóa.

Nếu làm sai thứ tự thì không gây hỏng hóc gì cả, nó chỉ khiến terraform destroy thất bại giữa chừng, và cần chạy lại sau khi xử lý xong vướng mắc.

#### Những Bucket Nào Thực Sự Cần Làm Trống Thủ Công

Chỉ một trong bốn bucket S3 của dự án này đặt force_destroy = true: bucket lưu artifact CI/CD (modules/pipeline), vì các build artifact vốn được thiết kế để có thể bỏ đi. Ba bucket còn lại thì không, và sẽ khiến terraform destroy thất bại nếu bên trong còn dữ liệu:

| Bucket | Module | force_destroy | Cần làm trống thủ công? |
|---|---|---|---|
| File tĩnh frontend | modules/frontend | không đặt | Có |
| Báo cáo hàng tuần | modules/data | không đặt | Có |
| Log CloudTrail + Config | modules/security | không đặt | Có |
| Artifact pipeline | modules/pipeline | true | Không, vẫn xóa sạch dù còn object bên trong |

Không có bucket nào trong bốn bucket này bật S3 versioning, nên không có phiên bản object cũ hay delete marker nào cần lo khi làm trống chúng. Bucket remote state là một bucket riêng biệt, được khởi tạo thủ công, nằm ngoài cấu hình Terraform này hoàn toàn, nên terraform destroy không bao giờ đụng tới nó. Tuy nhiên bạn vẫn đã tạo nó cho dự án này, nên nó sẽ được gỡ bỏ thủ công ở Bước 6 sau khi mọi thứ khác đã bị xóa.

#### Bước 1 — Làm Trống Ba Bucket Không Có Force-Destroy

Thực hiện việc này trước khi chạy terraform destroy, cho bucket frontend, bucket báo cáo, và bucket log bảo mật:

1. Vào console **S3** → chọn bucket → **Empty**.
2. Gõ permanently delete để xác nhận.
3. Lặp lại cho cả ba bucket trong bảng trên. Bucket artifact pipeline có thể bỏ qua (force_destroy sẽ tự động xử lý).

#### Bước 2 — Tắt (Disable) CloudFront Distribution

1. Vào console **CloudFront** → chọn distribution.
2. Nhấn **Disable**.
3. Đợi **Status** chuyển từ **Deploying** sang **Disabled**.

#### Bước 3 — Chạy Terraform Destroy

Sau khi ba bucket đã trống và CloudFront đã tắt, tiến hành gỡ bỏ hạ tầng:

```bash
cd terraform
AWS_PROFILE=YOUR-PROFILE terraform plan -destroy
```

Xem lại kế hoạch — xác nhận mọi tài nguyên được liệt kê đều đúng là cần xóa và không có gì bất thường.

```bash
AWS_PROFILE=YOUR-PROFILE terraform destroy
```

Gõ yes khi được hỏi.

Nếu việc này được chạy qua pipeline CI/CD thay vì chạy cục bộ, cùng một cổng phê duyệt thủ công sẽ áp dụng, hãy xem lại kế hoạch destroy trước khi phê duyệt, giống hệt như khi xem lại kế hoạch apply.

#### Bước 4 — Những Thứ Còn Sót Lại Mà Terraform Không Xử Lý

Các log group của Lambda summarizer và report được định nghĩa rõ ràng dưới dạng tài nguyên aws_cloudwatch_log_group trong modules/compute và modules/scheduling, nên terraform destroy sẽ xóa /aws/lambda/doc-summarizer-fn và /aws/lambda/doc-summarizer-report cùng với mọi thứ khác. Không cần thao tác thủ công cho hai cái này. Những thứ thực sự còn sót lại sau khi destroy:

- **Log group của CodeBuild**: các tài nguyên aws_codebuild_project trong modules/pipeline không có tài nguyên aws_cloudwatch_log_group tương ứng, nên CodeBuild tự tạo log group của nó khi chạy lần đầu và Terraform không bao giờ theo dõi hay xóa nó. Hãy kiểm tra **CloudWatch → Log groups** và xóa thủ công mọi nhóm /aws/codebuild/doc-summarizer-* còn sót lại.
- **Tiền tố domain của Cognito**: tiền tố domain bạn đặt trong `cognito_domain_prefix` được giải phóng khi aws_cognito_user_pool_domain bị xóa, nhưng có thể mất vài phút để có thể sử dụng lại. Nếu chạy lại workshop này từ đầu, hãy xác nhận tiền tố đó thực sự đã được giải phóng trước khi tạo lại.
- **Trạng thái CloudTrail multi-region trail**: CloudTrail có thể mất một khoảng thời gian ngắn để hoàn toàn ngừng ghi log sau khi tài nguyên Terraform của nó bị xóa; việc có thêm một hoặc hai sự kiện sau khi dọn dẹp là bình thường.
- **Conformance pack của AWS Config**: gói CIS trong modules/security là tài nguyên chậm bị xóa nhất trong toàn bộ stack này, thường mất mười phút hoặc lâu hơn. Terraform xóa gói này trước recorder và delivery channel nhờ chuỗi phụ thuộc, nhưng lời gọi API thường bị timeout trước đó. Đây là nguyên nhân dễ làm treo quá trình destroy nhất. Nếu gặp tình trạng này, hãy vào **Config → Conformance packs** xác nhận gói đã thực sự bị xóa hay chưa, rồi chạy lại terraform destroy. Nếu gói bị kẹt ở trạng thái DELETE_FAILED, hãy xóa nó từ console rồi chạy lại.

#### Bước 5 — Xác Minh Chi Phí Vận Hành Về $0

1. Mở **Billing and Cost Management** → **Cost Explorer**.
2. Lọc trong 24–48 giờ gần nhất, nhóm theo dịch vụ.
3. Xác nhận không có dịch vụ nào hiển thị chi phí đang hoạt động, liên tục. Một khoản phí nhỏ còn sót lại từ vài giờ trước khi dọn dẹp là bình thường, nhưng không nên có hoạt động mới nào sau khi destroy hoàn tất.
4. Đối chiếu với danh sách tài nguyên:

```bash
aws lambda list-functions --profile YOUR-PROFILE --query 'Functions[?starts_with(FunctionName, `doc-summarizer`)]'
aws dynamodb list-tables --profile YOUR-PROFILE --query 'TableNames[?starts_with(@, `SummarizerTable`)]'
aws s3 ls --profile YOUR-PROFILE | grep doc-summarizer | grep -v tfstate
aws cognito-idp list-user-pools --max-results 20 --profile YOUR-PROFILE --query 'UserPools[?starts_with(Name, `doc-summarizer`)]'
```

Tất cả các lệnh trên phải trả về kết quả rỗng ([] hoặc không có dòng nào khớp).

Phần `grep -v tfstate` trong lệnh kiểm tra bucket là bắt buộc. Bucket remote state cũng được đặt tên với tiền tố `doc-summarizer` và tại thời điểm này nó vẫn đang được sử dụng, nên nếu không lọc bỏ, lệnh sẽ báo còn sót bucket ngay cả khi destroy đã thành công hoàn toàn. Bước 6 sẽ xóa nó.

#### Bước 6 — Gỡ Bỏ Remote State Backend

Bucket state và bảng khóa DynamoDB được tạo thủ công trước lần terraform apply đầu tiên, nên không có gì trong cấu hình xóa chúng cả. Hãy thực hiện bước này **cuối cùng**, chỉ sau khi Bước 5 đã xác nhận mọi thứ khác đã bị xóa. Một khi bucket state bị xóa, terraform sẽ không còn đọc được file state nữa, nên đây là thao tác một chiều.

Bucket state có bật versioning. Lệnh `aws s3 rm --recursive` chỉ xóa các object hiện tại nhưng để lại toàn bộ phiên bản cũ, và bucket vẫn sẽ từ chối bị xóa. Làm trống bucket từ **console S3** là cách đáng tin cậy hơn, vì thao tác **Empty** của console xóa cả các phiên bản cũ lẫn delete marker.

1. Vào console **S3** → chọn bucket state → **Empty** → gõ `permanently delete` → sau đó **Delete** chính bucket đó.
2. Vào console **DynamoDB** → **Tables** → chọn bảng khóa → **Delete**.

```bash
aws s3 ls --profile YOUR-PROFILE | grep doc-summarizer
aws dynamodb list-tables --profile YOUR-PROFILE
```

Cả hai lệnh giờ đây phải trả về kết quả rỗng. Đến đây thì mọi tài nguyên mà dự án này tạo ra đều đã bị xóa.

#### Danh Sách Kiểm Tra Cuối Cùng

- Bucket frontend, báo cáo, và log bảo mật đã được làm trống (bucket artifact pipeline không cần)
- CloudFront distribution đã tắt và đã bị xóa
- terraform destroy hoàn tất không lỗi
- Không còn log group CloudWatch nào dưới /aws/codebuild/doc-summarizer-*
- Tiền tố domain Cognito đã được giải phóng
- Conformance pack của AWS Config đã bị xóa hoàn toàn, không kẹt ở DELETE_FAILED
- Cost Explorer không hiển thị hoạt động mới sau khi dọn dẹp
- Các lệnh CLI list trả về rỗng cho Lambda, DynamoDB, S3, và Cognito
- Bucket remote state đã được làm trống và xóa, bảng khóa DynamoDB đã được xóa (Bước 6)

#### Các Lỗi Destroy Thường Gặp Và Cách Khắc Phục

| Lỗi | Nguyên nhân | Cách khắc phục |
|---|---|---|
| BucketNotEmpty | Một trong ba bucket không có force-destroy (frontend, reports, security logs) vẫn còn object | Lặp lại Bước 1 cho bucket cụ thể được nêu tên trong lỗi |
| DependencyViolation trên origin của CloudFront distribution | Distribution chưa được tắt hoàn toàn trước khi chạy destroy | Lặp lại Bước 2, đợi trạng thái **Disabled** trước khi thử lại terraform destroy |
| Domain của Cognito user pool không xóa được | Domain vẫn còn gắn với cấu hình Hosted UI của app client | Gỡ liên kết domain khỏi app client trước, hoặc xóa theo thứ tự: app client → domain → user pool |
| terraform destroy bị treo hoặc timeout ở aws_config_conformance_pack | Việc xóa conformance pack rất chậm và thường vượt quá thời gian chờ của API | Đợi, vào **Config → Conformance packs** kiểm tra xem gói đã thực sự bị xóa chưa, rồi chạy lại terraform destroy. Xóa từ console nếu nó kẹt ở DELETE_FAILED |
| Bucket state không xóa được, báo BucketNotEmpty dù đã chạy aws s3 rm --recursive | Bucket state có bật versioning, các phiên bản cũ và delete marker vẫn còn | Làm trống bucket từ console S3, vì cách này xóa cả phiên bản cũ lẫn delete marker |
| terraform destroy chỉ hoàn tất một phần rồi thất bại | Một tài nguyên chặn việc xóa (bất kỳ trường hợp nào ở trên), khiến các tài nguyên phụ thuộc chưa bị xóa | Khắc phục vướng mắc cụ thể, sau đó chạy lại terraform destroy |