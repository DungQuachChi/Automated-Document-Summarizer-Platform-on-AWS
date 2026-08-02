---
title: "Tự Đánh Giá"
date: 2026-08-02
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

# Tự Đánh Giá

Trong thời gian thực tập tại **First Cloud Journey (FCJ)** từ **tháng 4 năm 2026** đến **tháng 7 năm 2026**, tôi đã có cơ hội chuyển từ kiến thức trên lớp sang việc xây dựng và vận hành một hệ thống Cloud thực tế. Sau năm tuần học AWS có cấu trúc, tôi dành bảy tuần để xây dựng đồ án tốt nghiệp — một nền tảng tóm tắt tài liệu serverless dựa trên AI — nơi tôi phụ trách backend, tích hợp Amazon Bedrock, và toàn bộ hạ tầng: Terraform, pipeline CI/CD, giám sát, và tăng cường bảo mật.

Để nhìn nhận một cách khách quan về thời gian thực tập của mình, tôi xin tự đánh giá dựa trên các tiêu chí sau:

| STT | Tiêu chí | Đánh giá | Nhận xét |
|---|---|---|---|
| 1 | Kiến thức | Trung bình | Tôi đến với dự án này khi còn thiếu kiến thức về AWS. Tôi đã từng làm một số dự án cơ bản trước khi tham gia chương trình FCAJ, và trong đó bạn cùng nhóm đã hướng dẫn tôi rất nhiều, giúp tôi đi qua nhiều kiến thức mới. Đến cuối dự án, tôi đã có thể theo kịp và vận hành được các phần của một stack serverless đầy đủ (Lambda, API Gateway, Cognito, DynamoDB, Bedrock, Terraform). |
| 2 | Khả năng học hỏi | Khá | Bắt đầu từ việc thiếu kiến thức, tôi hiểu rằng mình cần tiếp thu càng nhiều càng tốt ngay khi bắt đầu dự án, nên vài tuần đầu diễn ra khá chậm trong lúc tôi làm quen với các khái niệm cơ bản như IAM, Lambda, và API Gateway. Tôi có thể theo tài liệu và hướng dẫn để hoàn thành công việc, nhưng thường phải xem lại cùng một chủ đề nhiều lần trước khi thực sự nắm được, và vẫn phải dựa vào đồng đội hoặc các hướng dẫn cho bất kỳ điều gì vượt ra ngoài kiến thức cơ bản. |
| 3 | Tính chủ động | Tốt | Trong quá trình làm lab và dự án có rất nhiều thử thách — các dịch vụ tôi chưa từng đụng tới, các lỗi cấu hình không rõ nguyên nhân, và những lỗ hổng trong hiểu biết của chính mình mà tôi tự nhận ra. Thay vì chờ được giúp đỡ mỗi lần gặp vấn đề, tôi cố gắng tự tìm hiểu vấn đề trước, đọc kỹ tài liệu chính thức, và thử tự sửa trước khi hỏi đồng đội hoặc mentor. |
| 4 | Kỷ luật | Trung bình | Ban đầu tôi giữ được kỷ luật vì biết mình cần theo kịp đồng đội. Tuy nhiên, đôi khi tôi bị phân tâm hoặc bị cuốn vào việc khác trong tuần, dẫn đến việc viết hầu hết các mục worklog dồn lại vào một lần vào cuối tuần thay vì ghi lại theo thời gian thực. Đây là điều tôi muốn khắc phục trong các dự án tương lai bằng cách ghi lại tiến độ ngay khi làm, dù chỉ ngắn gọn, thay vì tái hiện lại sau đó. |
| 5 | Giao tiếp | Trung bình | Trong vài tuần đầu làm việc cùng nhau, đôi lúc có sự hiểu lầm, một phần vì tôi có xu hướng hỏi xin giúp đỡ quá nhanh thay vì cố gắng tự giải quyết vấn đề trước. Đây là điều tôi muốn cải thiện — tự giải quyết vấn đề của mình trước khi tìm đến đồng đội, thay vì để giao tiếp trở thành cách thay thế cho việc tự giải quyết vấn đề. |
| 6 | Làm việc nhóm | Tốt | Việc phân chia công việc giữa hai người (backend/hạ tầng và frontend/báo cáo/kiểm thử tải) có ranh giới trách nhiệm rõ ràng, và chúng tôi tích hợp công việc thông qua PR trên nhánh main được bảo vệ. Việc bàn giao, chẳng hạn như kết nối frontend đã hoàn thiện với API thật, diễn ra suôn sẻ. |
| 7 | Giải quyết vấn đề | Trung bình | Gặp lỗi redirect_mismatch khi kiểm thử chức năng đăng ký, và phải giải quyết theo từng bước thay vì xử lý xong ngay một lần. Ban đầu tôi vào nhầm trang cài đặt Cognito và phải tìm đúng tab "Login pages" trong mục App clients để tìm ra cài đặt callback URL. Sau khi cập nhật các allowed callback và sign-out URL cho khớp với port cục bộ của mình, lỗi vẫn tiếp diễn — hóa ra trình duyệt đang gửi http://0.0.0.0:8080 thay vì http://localhost:8080 do cách tôi khởi động server cục bộ, mà Cognito coi là hai origin khác nhau. Tôi đã khắc phục bằng cách hardcode redirect URI trong script.js và đảm bảo truy cập ứng dụng qua localhost thay vì 0.0.0.0. Việc này mất nhiều vòng để thu hẹp nguyên nhân thực sự thay vì làm đúng ngay từ đầu, đây là điều tôi muốn cải thiện tốc độ trong tương lai. |
| 8 | Đóng góp cho dự án | Trung bình | Đảm nhận và hoàn thành lớp frontend, hạ tầng kiểm thử, và tài liệu: xây dựng toàn bộ UI tĩnh (HTML/CSS/JS) với luồng xác thực OAuth 2.0 Cognito, giao diện dark theme. Kiểm thử bằng script kiểm thử tải Locust có lấy Cognito token, viết README dự án, báo cáo workshop, và phụ trách triển khai frontend lên S3/CloudFront. |

### Cần Cải Thiện

* Tăng cường kỷ luật bằng cách ghi lại tiến độ theo thời gian thực thay vì dồn lại viết worklog vào cuối tuần, và tự áp dụng cho bản thân những quy tắc và tiêu chuẩn tương tự như những gì mình mong đợi trong môi trường công ty hoặc tổ chức
* Cải thiện tư duy giải quyết vấn đề bằng cách thu hẹp nguyên nhân gốc rễ một cách có hệ thống hơn, thay vì thử nhiều cách sửa trước khi xác định được vấn đề thực sự
* Nâng cao kỹ năng giao tiếp bằng cách tự giải quyết vấn đề trước khi hỏi xin giúp đỡ, để việc hợp tác không trở thành cách thay thế cho việc tự giải quyết vấn đề — và bằng cách tự tin hơn khi trình bày công việc kỹ thuật một cách rõ ràng, súc tích với người khác