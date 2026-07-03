---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# CÁCH XÂY DỰNG CHIẾN LƯỢC AWS KMS TIẾT KIỆM CHI PHÍ CHO ỨNG DỤNG MULTI-TENANT

Trong quá trình học và thực tập, mình nhận ra rằng khi xây dựng ứng dụng SaaS phục vụ nhiều khách hàng (multi-tenant), việc quản lý mã hóa dữ liệu chỉ là một phần của bài toán. Thách thức thực sự xuất hiện khi cần cân bằng giữa bảo mật mạnh, chi phí hợp lý và khả năng quản lý dễ dàng khi số lượng tenant tăng lên. Amazon KMS cung cấp công cụ mã hóa mạnh mẽ, nhưng nếu tạo key riêng cho từng tenant và từng service thì chi phí sẽ tăng nhanh, việc quản lý trở nên phức tạp.

Để giải quyết vấn đề đó, AWS đề xuất cách tiếp cận tập trung hóa việc quản lý key. Giải pháp này xây dựng một lớp quản lý khóa trung tâm hoạt động bên trên AWS KMS, kết hợp cùng IAM role, alias, AWS STS và các service khác. Kiến trúc này giúp giảm chi phí, dễ quản lý, vẫn đảm bảo isolation giữa các tenant, đồng thời tận dụng tối đa khả năng mã hóa của KMS.

![KMS Multi-tenant Architecture](../../images/kms-architecture.png)

## DƯỚI ĐÂY LÀ NHỮNG ĐIỂM NỔI BẬT CỦA GIẢI PHÁP:

*   **Tập trung hóa quản lý key ở một account riêng**
    Tất cả customer managed KMS key của các tenant đều được đặt ở Account A (Centralized key management service). Các service phục vụ khách hàng ở Account B chỉ được cấp quyền tạm thời khi cần dùng. Cách này tránh tình trạng key phân tán lung tung, giúp kiểm soát toàn bộ vòng đời của key (tạo, xoay, xóa) ở một chỗ.
*   **Sử dụng alias và IAM role để chia sẻ key an toàn**
    Mỗi tenant có một alias dạng `alias/customer-<tenant-id>`. Service B (thường là Lambda) nhận JWT chứa tenant ID, sau đó assume ServiceBRole → assume ServiceARole ở Account A để truy cập key qua alias. Nhờ AWS STS, quyền chỉ tồn tại tạm thời và được kiểm soát chặt chẽ.
*   **Mã hóa dữ liệu client-side và lưu trữ an toàn**
    Dữ liệu nhạy cảm (mật khẩu, API key, thông tin giấy phép…) được mã hóa bằng Boto3 hoặc AWS Encryption SDK trước khi lưu vào DynamoDB hoặc S3. Mỗi tenant dùng key riêng nên dữ liệu được isolation hoàn toàn, ngay cả khi lưu chung một database.
*   **Dễ mở rộng khi hệ thống lớn**
    Khi số tenant tăng từ hàng chục lên hàng nghìn hoặc hơn, kiến trúc vẫn hoạt động tốt. Chỉ cần tạo key mới và alias mới ở Account A là xong. Không cần thay đổi code ở các service facing customer. Chi phí vẫn kiểm soát được vì mỗi key chỉ tốn khoảng 1-3 USD/tháng.
*   **Tuân thủ best practice security và vận hành**
    Giải pháp sử dụng IAM policy với điều kiện (condition) để giới hạn quyền chỉ trên alias/customer-, kết hợp CloudTrail để audit. Điều này giúp giảm rủi ro và dễ tuân thủ các tiêu chuẩn bảo mật.

## CÁCH HOẠT ĐỘNG TRONG THỰC TẾ

Giả sử khách hàng gửi thông tin nhạy cảm (mật khẩu, API key, giấy phép...) qua Service B.
*   Service B nhận JWT chứa tenant ID.
*   Lambda (hoặc service khác) kiểm tra JWT.
*   Assume Service B Role → Assume Service A Role ở Account A.
*   Dùng alias (ví dụ: `alias/customer-<tenant-id>`) để truy cập key đúng của tenant đó.
*   Mã hóa dữ liệu bằng Boto3 hoặc AWS Encryption SDK.
*   Lưu dữ liệu đã mã hóa vào DynamoDB.

Mọi thứ diễn ra mượt mà và dữ liệu luôn được bảo vệ bằng key riêng của từng tenant.

## VAI TRÒ CỦA CÁC DỊCH VỤ AWS

Kiến trúc được xây dựng dựa trên sự phối hợp rõ ràng của nhiều dịch vụ:
- **AWS KMS**: Cung cấp customer managed key cho từng tenant.
- **IAM & AWS STS**: Quản lý quyền assume role giữa các account.
- **AWS Lambda**: Xử lý request, kiểm tra JWT và thực hiện mã hóa.
- **Amazon DynamoDB / S3**: Nơi lưu dữ liệu đã mã hóa.
- **Alias**: Giúp code sạch sẽ, dễ maintain mà không cần hardcode key ID.

Mỗi thành phần đảm nhận một nhiệm vụ riêng biệt. Nhờ đó hệ thống dễ quản lý, dễ debug và giảm phụ thuộc vào một service duy nhất.

## GIÁ TRỊ MANG LẠI CHO DOANH NGHIỆP

Việc áp dụng chiến lược KMS tập trung mang lại nhiều lợi ích thực tế:
- Giảm đáng kể chi phí quản lý key.
- Dễ dàng vận hành và audit khi key tập trung một chỗ.
- Đảm bảo isolation và bảo mật cao giữa các tenant.
- Hỗ trợ mở rộng nhanh khi business phát triển.
- Giảm tải cho đội ngũ DevOps, không còn lo “key explosion”.
- Linh hoạt áp dụng cho nhiều dịch vụ (DynamoDB, S3, EBS…).

Đây là giải pháp phù hợp cho các ứng dụng SaaS, nền tảng doanh nghiệp, hệ thống xử lý dữ liệu nhạy cảm hoặc bất kỳ dự án multi-tenant nào trên AWS.

## KẾT LUẬN

AWS KMS là dịch vụ mã hóa mạnh mẽ, tuy nhiên khi dùng cho multi-tenant ở quy mô lớn thì cần cách tiếp cận thông minh hơn để kiểm soát chi phí và vận hành. Bằng cách xây dựng dịch vụ quản lý khóa trung tâm, kết hợp IAM role, alias và quy trình assume role qua JWT, chúng ta có thể giải quyết tốt bài toán này.

Giải pháp này thể hiện cách tiếp cận phổ biến trong kiến trúc AWS hiện đại: mỗi dịch vụ tập trung làm tốt một việc, sau đó kết nối với nhau để tạo thành hệ thống linh hoạt, an toàn và tiết kiệm.

Mình là thực tập sinh nên chưa được làm task thực tế về KMS multi-tenant, nhưng sau khi đọc bài gốc và diagram, mình thấy đây là pattern rất đáng học. Nó giúp mình hiểu rõ hơn về việc thiết kế hệ thống cloud cân bằng giữa security, chi phí và scalability.

**Nguồn tham khảo:** [Simplify multi-tenant encryption with a cost-conscious AWS KMS key strategy](https://aws.amazon.com/vi/blogs/architecture/simplify-multi-tenant-encryption-with-a-cost-conscious-aws-kms-key-strategy/)
