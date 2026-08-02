---
title: "GameDay thi đấu Cloud Architect"
date: 2026-06-20
weight: 1
chapter: false
pre: " <b> 4.1 </b> "
---

# GameDay thi đấu Cloud Architect

### Mục tiêu sự kiện

- Cuộc thi đố vui theo đội về kiến thức kiến trúc cloud, từ cơ bản đến các câu hỏi tình huống thực tế nâng cao
- 8 đội thi đấu đối kháng qua nhiều vòng

### Diễn giả

- *(bổ sung sau)*

### Thể lệ

- Mỗi vòng thi đấu đối kháng giữa 2 đội
- Mỗi đội có 2 "quyền năng" dùng một lần trong suốt trận:
  - **50/50**: nếu trả lời đúng, đội nhận được nửa số điểm của câu đó; nếu trả lời sai, không bị trừ điểm
  - **X2**: nhân đôi số điểm của câu đó, bất kể trả lời đúng hay sai
- Đội vô địch: **Ngũ Đại Hiệp**

### Điểm nổi bật

- Các câu hỏi trải rộng độ khó từ định nghĩa dịch vụ cơ bản đến các tình huống xử lý sự cố kiến trúc thực tế
- Nhiều vòng thi có biến động điểm số đáng chú ý ở cuối trận do các đội dùng quyền năng X2 vào những câu hỏi điểm cao
- Câu hỏi mẫu (tình huống thực tế, phong cách AWS Solutions Architect):

  > Một công ty triển khai website thương mại điện tử trên một Auto Scaling group gồm các EC2 instance, đặt sau một Application Load Balancer. Website nhận được nhiều request bất hợp pháp từ nhiều hệ thống với địa chỉ IP thay đổi liên tục, gây ra vấn đề hiệu năng. Giải pháp nào chặn được các request này mà ít ảnh hưởng nhất đến traffic hợp lệ?
  >
  > A. Tạo rule thông thường trong AWS WAF và gắn web ACL vào Application Load Balancer
  > B. Tạo kết nối riêng bằng AWS PrivateLink để chặn các request này
  > C. Tạo rate-based rule trong AWS WAF và gắn web ACL vào Application Load Balancer
  > D. Tạo network ACL tùy chỉnh và gắn vào subnet của Application Load Balancer để chặn

  **Đáp án: C** rate-based rule trong WAF chặn dựa trên tốc độ request theo từng IP thay vì dựa trên danh sách IP cố định, nên vẫn hoạt động hiệu quả kể cả khi các IP gây hại liên tục thay đổi, trong khi traffic hợp lệ dưới ngưỡng vẫn không bị ảnh hưởng.

### Bài học rút ra

- Củng cố lại sự khác biệt giữa WAF regular rule (điều kiện khớp cố định) và rate-based rule (dựa trên hành vi/tốc độ, thích ứng với nguồn thay đổi) đây là điểm hay gây nhầm lẫn trong các câu hỏi tình huống
- Ôn lại lý do vì sao các công cụ ở tầng network (NACL, PrivateLink) không phù hợp để lọc dựa trên *hành vi* traffic thay vì địa chỉ/cổng cố định
- Định dạng thi đố vui với cơ chế rủi ro/phần thưởng (50/50, X2) giúp việc ôn lại kiến thức nền tảng AWS trở nên thú vị hơn so với việc ôn tập thụ động

### Áp dụng vào công việc

- Ghi nhớ sự khác biệt giữa WAF regular rule và rate-based rule cho các công việc bảo mật endpoint công khai của dự án doc-summarizer sau này
- Sử dụng định dạng câu hỏi tình huống này như một phương pháp tự ôn tập cho việc luyện thi chứng chỉ AWS hoặc trước các buổi phỏng vấn kỹ thuật

### Trải nghiệm sự kiện

Tham dự ngày hội thi đố vui Cloud Architect với tư cách là một trong 8 đội thi đấu. Câu hỏi trải dài từ cơ bản đến các tình huống kiến trúc cloud thực tế nâng cao, kèm theo quyền năng 50/50 và X2 tạo thêm yếu tố chiến thuật/cạnh tranh ở mỗi vòng. Đội Ngũ Đại Hiệp đã giành chiến thắng. Thể lệ thi đấu rất cuốn hút, với nhiều pha lội ngược dòng điểm số ở cuối mỗi trận đấu giữa các đội.