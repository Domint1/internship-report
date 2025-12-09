---
title: "Bản đề xuất"
date: "2025-12-08"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

[Bản doc của proposal](https://docs.google.com/document/d/1-WtW93MMZPBzKaP8AzaQ2Zy3GQCm-i-ZW2sAS80ZsUg/edit?usp=sharing)

# Rafilm: Nền Tảng Ghi Lịch Sử Xem & Gợi Ý Phim Tích Hợp Trí Tuệ Nhân Tạo

## Giải pháp Serverless trên AWS cho Khám phá Phim Thông minh

### 1. Tóm tắt

Rafilm là một nền tảng ghi nhật ký và gợi ý phim lấy cảm hứng từ Letterboxd, giúp người dùng thông thường theo dõi phim đã xem, chia sẻ nhận xét và khám phá phim mới thông qua các gợi ý được hỗ trợ bởi AI. Được xây dựng trong khuôn khổ thực tập AWS First Cloud Journey (FCJ), Rafilm tích hợp Amazon Personalize và Bedrock để cung cấp gợi ý phim cá nhân hóa và đề xuất hội thoại thông qua giao diện chatbot.

Nền tảng chạy hoàn toàn trên kiến trúc Serverless của AWS, gồm frontend Next.js được host bằng Amplify, backend serverless dựa trên Lambda kết nối qua API Gateway, và DynamoDB để lưu trữ dữ liệu người dùng và phim có khả năng mở rộng. TMDb cung cấp dữ liệu phim bên ngoài, trong khi Amazon Cognito quản lý xác thực người dùng. Rafilm hướng tới việc chứng minh một kiến trúc có khả năng mở rộng, thông minh và tiết kiệm chi phí, hỗ trợ truy cập đa người dùng và trải nghiệm tương tác.

### 2. Vấn đề

#### Vấn đề là gì?

Mặc dù các nền tảng phim hiện tại như Letterboxd và IMDb cung cấp tính năng ghi nhật ký và mạng xã hội mạnh mẽ, họ thiếu các hệ thống **gợi ý cá nhân hóa** và trải nghiệm **khám phá tương tác**. Người dùng thường phải dựa vào nguồn bên ngoài hoặc danh sách xu hướng chung để tìm phim, dẫn đến các gợi ý không phù hợp hoặc lặp lại.

#### Giải pháp

Rafilm tích hợp một **đường ống gợi ý tùy chỉnh** được hỗ trợ bởi **Amazon Personalize**, kết hợp với **chatbot LLM trên Bedrock** để hiểu sở thích người dùng và tạo ra các gợi ý phim theo dạng hội thoại. Người dùng có thể ghi nhật ký phim, viết nhận xét và nhận các gợi ý được tuyển chọn — tất cả trong một trải nghiệm liền mạch. Khác với Letterboxd, Rafilm tập trung vào cá nhân hóa dựa trên dữ liệu và tương tác trợ giúp AI thay vì chỉ là mạng xã hội.

#### Lợi ích và lợi tức đầu tư

Bằng cách tận dụng dịch vụ Serverless của AWS, Rafilm đạt chi phí bảo trì gần bằng không, mở rộng theo mức sử dụng và cá nhân hóa theo thời gian thực. Với chương trình FCJ, dự án vừa là **mục tiêu kỹ thuật trình diễn** vừa là **tài liệu học tập** cho việc tích hợp dịch vụ AI trong kiến trúc serverless. Chi phí dự kiến dưới $1/tháng trong giai đoạn thử nghiệm, với nhiều dịch vụ đủ điều kiện sử dụng Free Tier của AWS.

### 3. Kiến trúc giải pháp

Rafilm sử dụng kiến trúc serverless mô-đun trên AWS nhằm đạt khả năng mở rộng, tích hợp và tối ưu chi phí.

![architecture-diagram](/images/2-Proposal/solution-architect-rafilm.jpg)

#### Dịch vụ AWS sử dụng

- **AWS Amplify**: Host frontend Next.js cho việc duyệt phim, ghi nhật ký và tương tác chatbot.
- **Amazon Cognito**: Quản lý đăng ký, đăng nhập và phiên người dùng.
- **Amazon API Gateway**: Định tuyến yêu cầu từ client đến các hàm Lambda backend.
- **AWS Lambda**: Thực thi logic serverless (ví dụ: CRUD cho nhận xét, lấy dữ liệu TMDb, kích hoạt gợi ý).
- **Amazon DynamoDB**: Lưu trữ nhật ký người dùng, tương tác phim và sở thích.
- **Amazon Personalize**: Huấn luyện và phục vụ các mô hình gợi ý cá nhân hóa.
- **Amazon Bedrock**: Cung cấp chức năng chatbot hội thoại cho việc gợi ý theo dạng hội thoại.
- **Amazon S3**: Lưu trữ tài sản tĩnh và sao lưu nhật ký cùng đầu ra mô hình.

#### Thiết kế thành phần

- **Frontend (Next.js)**: Giao diện thân thiện cho khám phá phim, ghi nhật ký và gợi ý theo dạng chat.
- **Backend (Lambda + API Gateway)**: Lớp logic không trạng thái xử lý thao tác người dùng, truy vấn phim và lấy gợi ý.
- **Lớp dữ liệu (DynamoDB + S3)**: Lưu trữ tương tác người dùng có cấu trúc và metadata phim để huấn luyện mô hình.
- **Lớp AI (Personalize + Bedrock)**: Personalize phân tích lịch sử tương tác; Bedrock chatbot cung cấp truy cập ngôn ngữ tự nhiên tới kết quả cá nhân hóa.
- **Xác thực (Cognito)**: Quản lý quyền truy cập đa người dùng một cách an toàn.

#### Tổng quan Kiến trúc

1. Người dùng đăng nhập qua Cognito và tương tác với giao diện Next.js.
2. Các hành động như ghi nhật ký hoặc đánh giá kích hoạt luồng API Gateway → Lambda → DynamoDB.
3. Chatbot trên Bedrock truy xuất kết quả từ Personalize để tạo đề xuất phim theo dạng hội thoại.
4. Amplify host frontend để triển khai liền mạch và dễ mở rộng.

### 4. Triển khai kỹ thuật

#### Các giai đoạn triển khai

1. **Thiết kế Kiến trúc (Tháng 1):** Nghiên cứu mẫu tích hợp serverless và AI trên AWS; hoàn thiện sơ đồ kiến trúc.
2. **Tích hợp Prototype (Tháng 2):** Triển khai hosting Amplify, cấu hình Cognito và API backend dựa trên Lambda.
3. **Hệ thống gợi ý (Tháng 3):** Kết nối Personalize và Bedrock để có luồng gợi ý end-to-end và phản hồi chatbot.
4. **Kiểm thử & Triển khai:** Thực hiện kiểm thử chức năng, tối ưu chi phí và triển khai phiên bản sẵn sàng trên Amplify.

#### Yêu cầu kỹ thuật

- **Frontend:** Next.js + React host qua AWS Amplify, sử dụng TMDb API cho dữ liệu phim.
- **Backend:** AWS Lambda (runtime Node.js) kết nối qua API Gateway.
- **Cơ sở dữ liệu:** Amazon DynamoDB cho dữ liệu người dùng và nhận xét có khả năng mở rộng.
- **Thành phần AI:** Amazon Personalize (gợi ý user-item) và Bedrock (đối thoại chatbot).
- **Xác thực:** Amazon Cognito cho truy cập đa người dùng an toàn.
- **Tự động hóa:** AWS SDK & CloudFormation để cấp phát tài nguyên; AWS SAM cho workflow triển khai.

### 5. Lộ trình & Cột mốc

| Giai đoạn  | Thời lượng             | Kết quả chính                                                  |
| ---------- | ---------------------- | -------------------------------------------------------------- |
| Tháng 1    | Nghiên cứu & Kiến trúc | Thiết kế kiến trúc giải pháp                                   |
| Tháng 2    | Phát triển lõi         | Hosting Amplify, cấu hình Cognito, API Lambda, schema DynamoDB |
| Tháng 3    | Tích hợp AI & Kiểm thử | Huấn luyện Personalize, chatbot Bedrock, triển khai hệ thống   |
| Sau ra mắt | Cải tiến liên tục      | Tối ưu chi phí, tính năng mới, cải thiện UX                    |

### 6. Ước tính ngân sách

**Chi phí ước tính hàng tháng:** $40.09 USD ($481.08 cho 12 tháng)

Ước tính chi phí của dự án này được dự kiến trong link sau: [https://calculator.aws/#/estimate?id=dab7fb57dabfb76041cdba98ac2bac7ba9630046](https://calculator.aws/#/estimate?id=dab7fb57dabfb76041cdba98ac2bac7ba9630046)

Chi phí ước tính hàng tháng là **$40.09 USD** được tính toán dựa trên các giả định sử dụng cụ thể sau đây từ AWS Pricing Calculator:

- **Mức sử dụng tiêu chuẩn:** Giả định **10.000 yêu cầu mỗi tháng** cho cả **AWS Lambda** và **Amazon API Gateway**.
- **Cơ sở người dùng:** Giả định tối đa **10.000 Người dùng Hoạt động Hàng tháng (MAU)** cho **Amazon Cognito**.
- **Mức sử dụng AI/ML (Yếu tố chi phí chính):**
    - **Amazon Bedrock** được ước tính hoạt động liên tục (24 giờ mỗi ngày) với trung bình **1 yêu cầu mỗi phút** và **100 token đầu vào/đầu ra cho mỗi yêu cầu**.
    - **Amazon Personalize** bao gồm **1 GB dữ liệu được nhập** và **15 giờ huấn luyện mỗi tháng**.
- **Chi phí bảo mật:** Giả định sử dụng **1 AWS WAF Web ACL** với 4 Rules và 3 Managed Rule Groups.
- **Lưu trữ dữ liệu:** Giả định **0.5 GB** dữ liệu lưu trữ trong **Amazon DynamoDB**.


### 7. Đánh giá rủi ro

| Rủi ro                          | Xác suất   | Ảnh hưởng  | Biện pháp giảm thiểu                               |
| ------------------------------- | ---------- | ---------- | -------------------------------------------------- |
| Giới hạn tần suất API từ TMDb   | Trung bình | Trung bình | Cache kết quả qua Lambda                           |
| Chi phí huấn luyện mô hình tăng | Thấp       | Trung bình | Dùng tập dữ liệu giới hạn cho thử nghiệm           |
| Độ trễ của chatbot              | Trung bình | Thấp       | Tối ưu loại mô hình Bedrock và kích thước phản hồi |
| Xác thực hoặc hết hạn token     | Trung bình | Thấp       | Dùng JWT thời gian sống ngắn và refresh token      |

### 8. Kết quả mong đợi

#### Cải tiến kỹ thuật

- Minh chứng cho việc **tích hợp serverless với AI/ML và LLM** trong ứng dụng thực tế.
- Thiết lập một kiến trúc AWS tái sử dụng cho các ứng dụng dựa trên gợi ý.

#### Giá trị dài hạn

- Cung cấp nền tảng để mở rộng thành **mạng khám phá phim xã hội** trong tương lai.
- Là **dự án minh chứng cho thực tập AWS FCJ**, nhấn mạnh về khả năng mở rộng, cá nhân hóa và AI hội thoại.
