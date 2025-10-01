# Together – AWS Project Planning

Dự án **Together** là hệ thống gợi ý công thức nấu ăn thông minh, được triển khai trên hạ tầng **AWS serverless** với AI Agent.  
Ứng dụng kết hợp **gợi ý công thức, quản lý nguyên liệu, tính năng mạng xã hội** và tối ưu chi phí vận hành trên AWS.

---

## Nội dung tài liệu
- [Tổng quan dự án](#-tổng-quan-dự-án)
- [Kiến trúc dịch vụ AWS](#-kiến-trúc-dịch-vụ-aws)
- [Sơ đồ cơ sở dữ liệu](#-sơ-đồ-cơ-sở-dữ-liệu)
- [Triển khai AI Agent](#-triển-khai-ai-agent)
- [Phân tích chi phí](#-phân-tích-chi-phí)
- [Roadmap](#-roadmap)

---

## Tổng quan dự án
- **Mục tiêu**: Xây dựng ứng dụng nấu ăn với AI gợi ý công thức thông minh.  
- **Chức năng chính**:
  - Gợi ý món ăn dựa trên nguyên liệu + hồ sơ cá nhân:contentReference[oaicite:0]{index=0}
  - Quản lý công thức, nguyên liệu
  - Tính năng xã hội: bạn bè, chia sẻ, lưu món yêu thích
  - Cài đặt quyền riêng tư chi tiết
- **Công nghệ**: Hugo (docs), Next.js, AWS Lambda, DynamoDB, Bedrock:contentReference[oaicite:1]{index=1}

---

## Kiến trúc dịch vụ AWS
Hệ thống được xây dựng hoàn toàn serverless trên AWS:contentReference[oaicite:2]{index=2}:

- **Frontend**: Next.js + AWS Amplify Hosting  
- **CDN**: Amazon CloudFront  
- **Auth**: Amazon Cognito (User & Identity Pool)  
- **API**: API Gateway (REST + WebSocket tương lai)  
- **Compute**: AWS Lambda (Node.js 20)  
- **AI**: Amazon Bedrock (Claude 3.5 Sonnet/Haiku)  
- **Database**: DynamoDB (Users, Recipes, AI Suggestions, Privacy, Friendships)  
- **Storage**: Amazon S3 (ảnh công thức, avatar, assets)  
- **Security**: WAF, Secrets Manager, IAM least-privilege  
- **Monitoring**: CloudWatch, X-Ray  

---

## Sơ đồ cơ sở dữ liệu
Thiết kế DB kết hợp **DynamoDB NoSQL** (chính) và khả năng thay thế RDS PostgreSQL (nếu cần).:contentReference[oaicite:3]{index=3}

- **Users** – tài khoản, roles  
- **User Preferences / Ingredients** – sở thích, nguyên liệu  
- **Recipes** – công thức (user tạo + AI tạo)  
- **AI Suggestions** – lịch sử gợi ý AI  
- **Friendships** – kết nối xã hội  
- **Privacy Settings** – quyền riêng tư chi tiết  
- **Favorites / Meal Plans / Activity Logs** – nâng cao  

---

## Triển khai AI Agent
AI Agent gợi ý công thức dựa trên profile + nguyên liệu:contentReference[oaicite:4]{index=4}:

- Input: Thông tin cá nhân + nguyên liệu + sở thích  
- AI Flow:  
  1. Lấy user profile từ DynamoDB  
  2. Tạo prompt cho Bedrock (Claude 3 Haiku/Sonnet)  
  3. Nhận kết quả JSON → công thức + nutrition  
  4. Lưu vào DynamoDB (`AISuggestions`)  
- Endpoint:  
  ```http
  POST /ai/suggest
  Body: { "request": "...", "dietary_preferences": [...] }
