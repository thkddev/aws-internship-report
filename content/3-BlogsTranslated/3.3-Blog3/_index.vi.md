---
title: "Blog 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# Gửi báo cáo chi phí delta hàng ngày trong môi trường multi-account bằng AWS Lambda và Cost Explorer

Trong môi trường AWS multi-account, việc theo dõi chi phí luôn là một bài toán khá khó, đặc biệt khi đội ngũ finance hoặc quản lý không có quyền truy cập trực tiếp vào AWS Billing Console.

Trong quá trình tìm hiểu về cost management và automation trên AWS, mình đọc được một bài blog chia sẻ cách tự động gửi báo cáo chi phí delta hằng ngày trong môi trường multi-account. Mình thấy giải pháp này khá thực tế vì vừa giúp tăng khả năng “cost visibility”, vừa giảm đáng kể các thao tác thủ công trong quá trình quản lý chi phí cloud.

![Delta Cost Report Architecture](../../images/cost-report-architecture.png)

## Vấn đề thường gặp

AWS Organizations cho phép quản lý và hợp nhất billing của nhiều account trong cùng một tổ chức. Tuy nhiên, báo cáo chi phí thường chỉ khả dụng cho những người có quyền truy cập billing ở management account.
Điều này dẫn đến một số vấn đề:
- Đội ngũ finance hoặc quản lý không có quyền truy cập AWS.
- Việc chia sẻ báo cáo chi phí thường phải làm thủ công.
- Khó theo dõi sự thay đổi chi phí giữa các account.
- Thiếu khả năng “cost visibility” trong tổ chức.
- Quy trình quản lý chi phí trở nên phức tạp hơn khi có nhiều account như Production, Non-Production hoặc Sandbox.

## Giải pháp chính

Giải pháp sử dụng AWS Lambda để tự động tạo và gửi báo cáo “delta cost” hằng ngày qua email.
“Delta cost” ở đây là phần chênh lệch chi phí giữa ngày hiện tại và ngày trước đó, giúp nhanh chóng phát hiện các biến động bất thường về chi phí cloud.
Hệ thống tận dụng các native service của AWS như:
- AWS Lambda
- AWS Cost Explorer API
- Amazon EventBridge
- Amazon Simple Email Service (SES)

Điểm mình thấy hay là toàn bộ kiến trúc đều theo hướng serverless và managed service, nên gần như không cần quản lý hạ tầng vận hành.

## Kiến trúc tổng quan

Flow hoạt động:
1. Amazon EventBridge kích hoạt Lambda theo lịch cron mỗi ngày.
2. Lambda gọi AWS Cost Explorer API để lấy dữ liệu chi phí từ các account trong tổ chức.
3. Lambda tính toán phần trăm thay đổi chi phí (delta) so với ngày trước đó.
4. Dữ liệu được format thành báo cáo HTML.
5. Amazon SES gửi email đến danh sách người nhận.

## Lợi ích nổi bật

- Tự động hóa hoàn toàn quy trình gửi báo cáo chi phí.
- Giúp leadership và finance team dễ tiếp cận thông tin hơn.
- Hoạt động hiệu quả trong môi trường multi-account.
- Hỗ trợ tăng khả năng cost visibility trong tổ chức.
- Dễ triển khai bằng CloudFormation template.
- Có thể mở rộng để lưu lịch sử báo cáo lên S3 phục vụ phân tích dài hạn.

Ngoài ra, giải pháp này cũng giúp technical team và finance team phối hợp hiệu quả hơn trong quá trình tối ưu chi phí cloud theo hướng FinOps.

## Các bước triển khai chính

- Tải CloudFormation template từ bài blog gốc.
- Cấu hình account IDs và email recipients.
- Verify email trong Amazon SES.
- Deploy stack.
- Cấp Billing permission cho IAM Role.
- Cấu hình EventBridge rule với cron expression.
Sau khi hoàn tất, hệ thống sẽ tự động gửi báo cáo chi phí mỗi ngày qua email.

## Nhận định cá nhân

Mình thấy giải pháp này khá thực tế vì nó giải quyết đúng bài toán mà nhiều team dùng AWS thường gặp: chi phí cloud tăng nhưng không phải lúc nào cũng theo dõi kịp.

Điểm hay là kiến trúc khá gọn, chủ yếu dùng các native service của AWS như Lambda, EventBridge và SES nên triển khai không quá phức tạp. Với mình thì đây cũng là một ví dụ khá rõ cho cách AWS dùng automation để giảm các công việc thủ công trong vận hành hệ thống.

Ngoài việc gửi báo cáo chi phí, mình nghĩ mô hình này còn có thể mở rộng thêm như lưu lịch sử lên S3, tạo dashboard theo dõi hoặc kết hợp cảnh báo khi chi phí tăng bất thường.

**Nguồn tham khảo:** [Email delta cost usage report in a multi-account organization using AWS Lambda](https://aws.amazon.com/vi/blogs/architecture/email-delta-cost-usage-report-in-a-multi-account-organization-using-aws-lambda/)
