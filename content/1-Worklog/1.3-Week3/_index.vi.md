---
title: "Worklog Tuần 3"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:
* Tìm hiểu các giải pháp sao lưu và lưu trữ trên AWS.
* Thực hành AWS Backup và AWS Storage Gateway.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --------- | ------------ | --------------- | -------------- |
| 2 | - Tìm hiểu AWS Backup - Tạo S3 bucket phục vụ lab và chuẩn bị resource cần backup | 04/05/2026 | 04/05/2026 | [000013.awsstudygroup.com](https://000013.awsstudygroup.com/) |
| 3 | - Tạo Backup Plan - Cấu hình backup rule, lifecycle, resource assignment và thông báo SNS | 05/05/2026 | 05/05/2026 | [000013.awsstudygroup.com](https://000013.awsstudygroup.com/) |
| 4 | - Thực hiện restore dữ liệu từ Backup Vault - Kiểm tra trạng thái và đánh giá kết quả restore | 06/05/2026 | 06/05/2026 | |
| 5 | - Tìm hiểu AWS Storage Gateway - Tạo Storage Gateway instance và kết nối gateway với AWS | 07/05/2026 | 07/05/2026 | [000024.awsstudygroup.com](https://000024.awsstudygroup.com/) |
| 6 | - Tạo File Gateway và mount storage - Tạo static website trên Amazon S3 để kiểm tra lưu trữ | 08/05/2026 | 08/05/2026 | [000057.awsstudygroup.com](https://000057.awsstudygroup.com/) |

### Kết quả đạt được tuần 3:
* Nắm được tổng quan về các phương pháp sao lưu và phục hồi dữ liệu cấp doanh nghiệp bằng dịch vụ AWS Backup.
* Xây dựng chiến lược dự phòng hoàn chỉnh (Backup Plan), bao gồm cấu hình lifecycle, tự động gán tài nguyên và cảnh báo trạng thái qua Amazon SNS.
* Thực hiện thành công quá trình restore dữ liệu thực tế từ Backup Vault, đảm bảo tính toàn vẹn và khả năng phục hồi khi có sự cố.
* Khởi tạo và kết nối thành công AWS Storage Gateway, thử nghiệm triển khai File Gateway và kết nối S3 phục vụ lưu trữ tại local.
