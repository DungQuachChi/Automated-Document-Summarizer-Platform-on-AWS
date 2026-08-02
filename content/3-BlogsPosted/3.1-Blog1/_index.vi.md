---
title: "Tìm hiểu CloudWatch Logs Enhanced Automatic Dashboard – Trợ thủ giám sát dung lượng và tối ưu chi phí Log trên AWS"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Tìm hiểu CloudWatch Logs Enhanced Automatic Dashboard – Trợ thủ giám sát dung lượng và tối ưu chi phí Log trên AWS

Xin chào các anh chị và các bạn, trong lúc đọc blog và tìm hiểu các dịch vụ AWS, mình có thấy bài blog về dịch vụ Amazon CloudWatch Logs là một trong những dịch vụ quen thuộc nhất để ghi log và giám sát ứng dụng. Bài viết làm mình nhớ về lúc trước khi mình mới bắt đầu tìm hiểu và sử dụng các dịch vụ của AWS, mình nghĩ việc theo dõi dung lượng log chỉ đơn giản là nhìn vào tổng hóa đơn hàng tháng. Tuy nhiên, khi hệ thống mở rộng, việc xác định xem log group nào đang tốn tiền nhiều nhất hoặc dịch vụ nào đang gặp lỗi gửi log lại không hề dễ dàng.

Trước đây, nếu muốn theo dõi chi tiết, chúng ta thường phải tự tạo Custom Dashboard (vốn sẽ tốn thêm chi phí). Nhưng vừa qua, AWS đã ra mắt CloudWatch Logs Enhanced Automatic Dashboard — một dashboard dựng sẵn (out-of-the-box) hoàn toàn miễn phí, giúp trực quan hóa toàn bộ tình trạng sử dụng CloudWatch Logs trong tài khoản.

## Một ví dụ đơn giản về cách trải nghiệm Dashboard

Để xem thử dashboard này hoạt động ra sao, mình đã thao tác theo các bước đơn giản sau:

- **Bước 1:** Đăng nhập vào AWS Management Console và tìm dịch vụ CloudWatch.
- **Bước 2:** Tại menu bên trái, tìm đến mục Dashboards chọn tab Automatic dashboards.
- **Bước 3:** Tìm và chọn CloudWatch Logs.
- **Bước 4:** Chọn đúng Region và điều chỉnh Time range cần phân tích ở góc trên bên phải.
- **Bước 5:** Tương tác với biểu đồ bằng cách nhấp vào tên một Log Group trong phần legend để tập trung xem duy nhất Log Group đó.

## Các thông tin hữu ích thu được từ Dashboard

Dashboard mới này chia dữ liệu thành 8 khu vực cực kỳ trực quan, giúp mình bao quát được gần như mọi khía cạnh của Log:

- **Log ingestion by Account & Log Group:** Cho biết tổng GB log đã nạp vào tài khoản và phân rã theo dạng Pie chart cho từng Log Group. Rất dễ để phát hiện ra nguồn sử dụng dung lượng lớn nhất.
- **Service Usage:** Theo dõi tần suất gọi các API như PutLogEvents, StartQuery và hiển thị các trường hợp bị lỗi hoặc bị nghẽn API.
- **Embedded Metric Format (EMF):** Giám sát các lỗi parse hoặc validation khi ứng dụng trích xuất metric trực tiếp từ log dạng JSON.
- **Subscription Filters:** Theo dõi số lượng sự kiện log được đẩy sang các dịch vụ khác như Lambda, Firehose và nhận biết sớm các lỗi gián đoạn luồng dữ liệu.
- **Log Anomaly Detection:** Thống kê các phát hiện bất thường trong log dựa trên Machine Learning của AWS.
- **Log Data Protection:** Theo dõi số lượng log chứa thông tin nhạy cảm như thẻ tín dụng, API key, PII bị phát hiện và masking.
- **Log Transformers:** Theo dõi số lượng sự kiện và dung lượng log được tự động biến đổi hoặc chuẩn hóa dữ liệu ngay khi nạp vào.

## Một vài điểm mình thấy hữu ích

Sau khi một quãng thời gian sử dụng mình rút ra được một số ưu điểm như là:

- **Hoàn toàn miễn phí:** Sử dụng tính năng này không tốn thêm chi phí, trong khi việc tự dựng Custom Dashboard sẽ bị tính phí tạo dashboard hàng tháng.
- **Giúp phát hiện sớm vấn đề:** Nhìn thấy ngay được các lỗi nghẽn API hoặc lỗi gửi log stream.
- **Hỗ trợ tối ưu chi phí:** Dựa vào bảng xếp hạng Log Group ngốn dung lượng nhất, team có thể chủ động chuyển các Log Group ít quan trọng sang lớp Infrequent Access log class hoặc điều chỉnh lại độ chi tiết của log.

## Một số điểm cần lưu ý

Bên cạnh những ưu điểm trên, có một số điểm bạn nên lưu ý khi sử dụng:

- **Tính năng theo từng Region:** Dashboard chỉ hiển thị dữ liệu của Region và Account hiện tại. Nếu ứng dụng chạy Multi-region, bạn cần chuyển đổi Region trên console để xem từng khu vực.
- **Chỉ mang tính chất quan sát (Observability):** Dashboard giúp bạn biết được chuyện gì đang diễn ra, còn việc tối ưu chi phí thì vẫn phải chủ động cấu hình.
- **Tự tạo Custom Dashboard vẫn tốn phí:** Nếu bạn muốn tự thiết kế lại giao diện dashboard theo ý muốn cá nhân thay vì dùng bản dựng sẵn này, AWS sẽ tính phí theo bảng giá CloudWatch Dashboard tiêu chuẩn.

## Khi nào nên sử dụng?

Theo mình, bạn nên check dashboard xem thường xuyên trong các trường hợp:

- Hóa đơn CloudWatch Logs đột ngột tăng cao mà không rõ nguyên nhân.
- Đang phát triển/triển khai ứng dụng mới và muốn kiểm tra xem log có đang bị gửi thừa hoặc bị lỗi Throttling hay không.
- Muốn theo dõi hiệu quả của việc bảo mật dữ liệu nhạy cảm (Data Protection) hoặc lọc dữ liệu trong hệ thống lớn.
- Cần xuất báo cáo nhanh về tình trạng hạ tầng Observability cho team.

## Kết luận

Chiếc Enhanced Automatic Dashboard này là một nâng cấp rất đáng giá nhưng lại hoàn toàn miễn phí từ AWS. Dù bạn là thực tập sinh hay lập trình viên đã có kinh nghiệm, việc tận dụng dashboard này sẽ giúp bạn hiểu rõ hơn về luồng dữ liệu log của ứng dụng và có tư duy tối ưu chi phí ngay từ sớm.

Nếu anh/chị hoặc các bạn đã thử tính năng này trên CloudWatch hoặc có thêm mẹo nào để quản lý chi phí Log hiệu quả, rất mong nhận được chia sẻ bên dưới để cùng học hỏi thêm!

## Tài liệu tham khảo

- AWS Blog – Analyze logs usage with Amazon CloudWatch enhanced automatic dashboard
  https://aws.amazon.com/blogs/mt/analyze-logs-usage-with-amazon-cloudwatch-enhanced-automatic-dashboard/
- AWS Documentation – Analyzing, optimizing, and reducing CloudWatch costs
  https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Cost-Optimization.html
- Amazon CloudWatch Pricing
  https://aws.amazon.com/cloudwatch/pricing/