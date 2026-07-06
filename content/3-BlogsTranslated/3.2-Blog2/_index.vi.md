---
title: "Blog 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# Amazon Bedrock Baseline Architecture trong AWS Landing Zone: Nền tảng triển khai Generative AI cho doanh nghiệp

Xây dựng một ứng dụng AI thì không quá khó. Nhưng khi doanh nghiệp có hàng chục ứng dụng AI hoạt động cùng lúc, bài toán lúc này không còn là kết nối được với mô hình AI nữa, mà là quản lý toàn bộ hệ thống AI sao cho bảo mật, ổn định và dễ vận hành.

Thông qua **Amazon Bedrock Baseline Architecture trong AWS Landing Zone**, AWS đề xuất một kiến trúc tham chiếu giúp doanh nghiệp xây dựng nền tảng Generative AI theo hướng tập trung, đảm bảo khả năng quản trị, bảo mật và mở rộng lâu dài.

![Amazon Bedrock Architecture](../../../images/bedrock-architecture.png)

## Vì sao cần một kiến trúc chuẩn cho Generative AI?

Ở giai đoạn thử nghiệm, mỗi nhóm phát triển có thể tự xây dựng ứng dụng AI và kết nối trực tiếp đến Amazon Bedrock.
Tuy nhiên khi số lượng ứng dụng tăng lên, doanh nghiệp sẽ gặp nhiều thách thức:
- Khó quản lý quyền truy cập.
- Khó theo dõi chi phí sử dụng AI.
- Khó kiểm soát dữ liệu và lưu lượng truy cập.
- Khó áp dụng các chính sách bảo mật thống nhất.
- Khó thực hiện kiểm toán và tuân thủ.

## Kiến trúc đề xuất

- **Service Network Account:** Quản lý kết nối mạng, kiểm soát truy cập AI, quản lý chính sách bảo mật, điều phối lưu lượng giữa các tài khoản và tập trung toàn bộ traffic AI tại một đầu mối.
- **Generative AI Account:** Triển khai Amazon Bedrock, Bedrock Knowledge Bases, Bedrock Guardrails, Foundation Models và các dịch vụ AI liên quan; giúp quản trị tập trung, theo dõi chi phí và chuẩn hóa vận hành.
- **Workload Accounts:** Triển khai chatbot khách hàng, trợ lý nội bộ, xử lý tài liệu, ứng dụng RAG và các ứng dụng nghiệp vụ; truy cập AI thông qua lớp kiểm soát trung tâm thay vì kết nối trực tiếp đến Bedrock.

## Những điểm nổi bật của kiến trúc

- Tách riêng AI Platform Account và Workload Account.
- Toàn bộ lưu lượng AI được kiểm soát tập trung thông qua VPC Lattice.
- Amazon Bedrock được triển khai như một dịch vụ dùng chung cho toàn tổ chức.
- Kết hợp Bedrock Guardrails để tăng cường kiểm soát nội dung và bảo mật.
- Phù hợp với mô hình multi-account và AWS Landing Zone.

## Kết luận

Amazon Bedrock Baseline Architecture không đơn thuần là một kiến trúc kỹ thuật mà là một mô hình quản trị Generative AI ở cấp doanh nghiệp.

Thông qua việc tập trung quản lý tài nguyên AI, kiểm soát lưu lượng truy cập, tăng cường bảo mật và chuẩn hóa quy trình vận hành, AWS giúp doanh nghiệp giải quyết ba bài toán lớn nhất khi triển khai AI:
1. Bảo mật.
2. Quản trị.
3. Khả năng mở rộng.

## Kiến nghị

- **Đối với doanh nghiệp đang thử nghiệm AI:** Bắt đầu nhỏ, xây dựng governance sớm, chuẩn hóa phân quyền.
- **Đối với doanh nghiệp đang mở rộng AI:** Tách riêng AI Account, quản lý tập trung tài nguyên AI, theo dõi chi phí sử dụng.
- **Đối với doanh nghiệp quy mô lớn:** Xây dựng AI Platform dùng chung, triển khai Landing Zone, quản trị tập trung, thiết lập AI Governance và AI Security.

**Nguồn tham khảo:** [Amazon Bedrock baseline architecture in an AWS Landing Zone](https://aws.amazon.com/blogs/architecture/amazon-bedrock-baseline-architecture-in-an-aws-landing-zone/)
