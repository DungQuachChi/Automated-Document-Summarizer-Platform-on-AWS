---
title: "Tìm hiểu xây dựng ứng dụng B2C bảo mật với Fine-Grained Access Control dùng Amazon Cognito và Amazon Verified Permissions"
date: 2026-08-02
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# Tìm hiểu xây dựng ứng dụng B2C bảo mật với Fine-Grained Access Control dùng Amazon Cognito và Amazon Verified Permissions

Xin chào anh chị và các bạn, lúc mình làm các dự án cá nhân, mình thường hay gặp phải là Authentication và Authorization. Mình hay sử dụng Amazon Cognito để quản lý việc đăng ký, đăng nhập và cấp Token. Tuy nhiên, khi ứng dụng phát triển, bài toán phân quyền sẽ không dừng lại ở mức "User" hay "Admin" mà đòi hỏi phân quyền tinh xảo hơn theo ngữ cảnh và thuộc tính.

Nếu hardcode logic phân quyền rối rắm này vào trong code backend, ứng dụng sẽ rất khó bảo trì và dễ xuất hiện lỗ hổng bảo mật. Bài viết trên AWS Security Blog đã giới thiệu một giải pháp hoàn hảo: Kết hợp Amazon Cognito (chuyên lo AuthN) và Amazon Verified Permissions (chuyên lo AuthZ với ngôn ngữ Cedar).

## Luồng hoạt động của hệ thống (Quy trình 5 bước)

Thay vì backend tự kiểm tra quyền hạn, nhiệm vụ phân quyền được ủy thác hoàn toàn cho Amazon Verified Permissions (AVP) theo mô hình sau:

- **Bước 1 (AuthN):** Người dùng đăng nhập vào ứng dụng qua Amazon Cognito User Pool. Sau khi xác thực thành công, Cognito trả về các JWT Tokens.
- **Bước 2 (Request):** Client gửi yêu cầu kèm theo Token đến Backend/API Gateway để thực hiện một hành động.
- **Bước 3 (Delegate AuthZ):** Backend không tự kiểm tra điều kiện mà gọi API IsAuthorizedWithToken của Amazon Verified Permissions, truyền vào:
  - **Principal:** Identity Token từ Cognito.
  - **Action:** Hành động muốn thực hiện.
  - **Resource:** Tài nguyên được tác động.
  - **Context:** Các thuộc tính môi trường bổ sung.
- **Bước 4 (Policy Evaluation):** AVP kiểm tra yêu cầu dựa trên các policy được viết bằng ngôn ngữ Cedar và đưa ra kết quả.
- **Bước 5 (Decision):** AVP trả về ALLOW hoặc DENY. Backend dựa vào đó để thực thi logic tiếp theo hoặc trả về lỗi 403 Forbidden.

## Một vài điểm mình thấy cực kỳ hữu ích

- **Tách biệt hoàn toàn (Decoupled Authorization):** Logic phân quyền không còn nằm rải rác trong các hàm của Backend. Security có thể quản lý và thay đổi policies trên Cloud mà không cần phải sửa code hay deploy lại ứng dụng.
- **Sức mạnh của ngôn ngữ Cedar:** Cedar là ngôn ngữ viết Policy-as-Code do AWS phát triển. Cú pháp của Cedar rất dễ đọc, dễ kiểm thử và được tối ưu hóa về mặt toán học để đưa ra quyết định phân quyền chỉ trong vài millisecond.
- **Tích hợp sẵn với Amazon Cognito:** AVP có khả năng hiểu được cấu trúc JWT Token của Cognito, tự động trích xuất các claims và user groups.
- **Hỗ trợ ABAC và RBAC linh hoạt:** Dễ dàng định nghĩa các quy tắc phân quyền phức tạp dựa trên thuộc tính của người dùng hoặc thuộc tính của tài nguyên.

## Một số điểm cần lưu ý

- **Cần thời gian làm quen với Cedar:** Dù Cedar rất trực quan, team phát triển vẫn cần đầu tư thời gian để tìm hiểu cú pháp, cách thiết kế Schema cũng như tư duy viết policy chuẩn xác.
- **AVP chỉ đưa ra quyết định (Decision Engine):** AVP chỉ trả về ALLOW hoặc DENY. Việc thực thi hành động vẫn do code ứng dụng đảm nhận.
- **Tối ưu chi phí API Call:** Vì mỗi thao tác cần phân quyền đều gọi API sang AVP, cần phải tính toán chi phí gọi API IsAuthorized khi hệ thống có hàng triệu lượt truy cập mỗi ngày.

## Cognito và Verified Permissions sẽ rất phù hợp cho

- Ứng dụng B2C lớn có quy định phân quyền phức tạp và thay đổi thường xuyên (như ứng dụng Ngân hàng, Bảo hiểm, Y tế, Thương mại điện tử).
- Các hệ thống SaaS Multi-tenant, nơi mỗi tổ chức/khách hàng cần có các bộ quy tắc truy cập dữ liệu riêng biệt.
- Các doanh nghiệp cần tuân thủ chứng chỉ bảo mật nghiêm ngặt, đòi hỏi khả năng kiểm toán tập trung xem "ai có quyền làm gì trên tài nguyên nào".

## Kết luận

Việc kết hợp Amazon Cognito và Amazon Verified Permissions giúp cho toàn bộ bài toán phân quyền phức tạp cho AVP xử lý bằng ngôn ngữ Cedar, giúp tập trung vào việc phát triển các tính năng cốt lõi cho sản phẩm.

Nếu anh chị và các bạn có các mẹo nào hay trong việc phân quyền, có sai sót gì trong bài của mình hoặc là muốn bổ sung kiến thức thêm rất mong mình được nhận chia sẻ và góp ý của mọi người ở bên dưới ạ.

## Tài liệu tham khảo

- AWS Security Blog – Building secure B2C applications with fine-grained access control using Amazon Cognito and Amazon Verified Permissions
  https://aws.amazon.com/blogs/security/building-secure-b2c-applications-with-fine-grained-access-control-using-amazon-cognito-and-amazon-verified-permissions/
- Amazon Verified Permissions Documentation
  https://docs.aws.amazon.com/verifiedpermissions/
- Cedar Policy Language Official Site
  https://www.cedarpolicy.com/