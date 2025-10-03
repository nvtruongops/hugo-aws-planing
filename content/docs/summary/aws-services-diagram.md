---
title: "AWS Services Architecture Diagram"
weight: 1
---

# Sơ Đồ Kiến Trúc Dịch Vụ AWS

## Kiến Trúc Hệ Thống

```mermaid
graph TB
    subgraph "Tầng Client"
        WebApp[Web App<br/>Next.js]
        MobileApp[Mobile App<br/>React Native - Tương lai]
    end

    subgraph "CDN & Hosting"
        CloudFront[Amazon CloudFront<br/>CDN]
        Amplify[AWS Amplify<br/>Hosting & CI/CD]
    end

    subgraph "Tầng API Gateway"
        APIGateway[Amazon API Gateway<br/>REST API]
        WSGateway[API Gateway<br/>WebSocket - Tương lai]
    end

    subgraph "Xác Thực"
        CognitoUser[Amazon Cognito<br/>User Pool]
        CognitoIdentity[Cognito Identity Pool]
    end

    subgraph "Tầng Compute"
        LambdaAuth[Lambda: Auth Handler<br/>Node.js 20]
        LambdaRecipe[Lambda: Recipe CRUD<br/>Node.js 20]
        LambdaAI[Lambda: AI Suggestion<br/>Node.js 20]
        LambdaProfile[Lambda: User Profile<br/>Node.js 20]
        LambdaSocial[Lambda: Social/Friends<br/>Node.js 20]
        LambdaPost[Lambda: Posts & Comments<br/>Node.js 20]
        LambdaNotification[Lambda: Notifications<br/>Node.js 20]
        LambdaAdmin[Lambda: Admin Operations<br/>Node.js 20]
        LambdaCooking[Lambda: Cooking History<br/>Node.js 20]
        LambdaRating[Lambda: Rating & Approval<br/>Node.js 20]
        LambdaIngredient[Lambda: Ingredient Validation<br/>Node.js 20]
    end

    subgraph "Dịch Vụ AI"
        Bedrock[Amazon Bedrock<br/>Claude 3.5 Sonnet/Haiku]
    end

    subgraph "Tầng Database"
        DynamoDB[Amazon DynamoDB<br/><br/>Bảng: Users<br/>Bảng: UserData<br/>Bảng: Recipes<br/>Bảng: RecipeRatings<br/>Bảng: UserCookingHistory<br/>Bảng: MasterIngredients<br/>Bảng: AI_Suggestions<br/>Bảng: Privacy<br/>Bảng: Friendships<br/>Bảng: Posts<br/>Bảng: Comments<br/>Bảng: Reactions<br/>Bảng: Notifications]
    end

    subgraph "Lưu Trữ"
        S3[Amazon S3<br/>User Uploads & Assets]
    end

    subgraph "Giám Sát & Logging"
        CloudWatch[Amazon CloudWatch<br/>Logs & Metrics]
        XRay[AWS X-Ray<br/>Tracing]
    end

    subgraph "Bảo Mật"
        WAF[AWS WAF<br/>Web Application Firewall]
        Secrets[AWS Secrets Manager<br/>API Keys]
    end

    WebApp --> CloudFront
    CloudFront --> Amplify
    WebApp --> APIGateway
    APIGateway --> CognitoUser
    CognitoUser --> LambdaAuth
    CognitoIdentity --> S3

    APIGateway --> LambdaRecipe
    APIGateway --> LambdaAI
    APIGateway --> LambdaProfile
    APIGateway --> LambdaSocial
    APIGateway --> LambdaPost
    APIGateway --> LambdaNotification
    APIGateway --> LambdaAdmin
    APIGateway --> LambdaCooking
    APIGateway --> LambdaRating
    APIGateway --> LambdaIngredient

    LambdaAI --> Bedrock

    LambdaRecipe --> DynamoDB
    LambdaAI --> DynamoDB
    LambdaProfile --> DynamoDB
    LambdaSocial --> DynamoDB
    LambdaPost --> DynamoDB
    LambdaNotification --> DynamoDB
    LambdaAdmin --> DynamoDB
    LambdaCooking --> DynamoDB
    LambdaRating --> DynamoDB
    LambdaIngredient --> DynamoDB

    LambdaRecipe --> S3

    LambdaAuth --> CloudWatch
    LambdaRecipe --> CloudWatch
    LambdaAI --> CloudWatch
    LambdaCooking --> CloudWatch
    LambdaRating --> CloudWatch

    APIGateway --> WAF
```

## Phân Tích Chi Tiết Các Dịch Vụ

### 1. Frontend & CDN

**AWS Amplify:**
- Host ứng dụng Node.js
- Pipeline CI/CD (tự động deploy từ GitHub)
- Custom domain & SSL certificates
- Chi phí: ~$15/tháng

**Amazon CloudFront:**
- CDN toàn cầu cho phân phối nội dung nhanh chóng
- Cache các static assets
- Chi phí: ~$5-10/tháng (1TB data transfer)

### 2. Xác Thực & Phân Quyền

**Amazon Cognito User Pool:**
- Đăng ký & đăng nhập người dùng
- Xác minh email
- Khôi phục mật khẩu
- Hỗ trợ MFA (tương lai)
- Chi phí: MIỄN PHÍ (< 50,000 )

**Cognito Identity Pool:**
- Thông tin xác thực AWS tạm thời để truy cập S3
- Quyền IAM chi tiết

### 3. Tầng API

**Amazon API Gateway (REST):**
- RESTful API endpoints
- Validation request
- Quản lý API key
- Throttling & rate limiting
- Cấu hình CORS
- Chi phí: ~$3.50 trên triệu requests

#### API Endpoints

```
POST   /auth/register
POST   /auth/login
GET    /user/profile
PUT    /user/profile
POST   /user/ingredients
GET    /user/ingredients
DELETE /user/ingredients/{id}
PUT    /user/privacy
GET    /user/privacy

POST   /friends/request
GET    /friends
PUT    /friends/{id}/accept
PUT    /friends/{id}/reject
DELETE /friends/{id}

POST   /recipes
GET    /recipes/{id}
PUT    /recipes/{id}
DELETE /recipes/{id}
GET    /recipes/search
POST   /recipes/{id}/rate

POST   /ai/suggest
GET    /ai/suggestions
POST   /ai/feedback

POST   /cooking/start
PUT    /cooking/{id}/complete
GET    /user/cooking-history
DELETE /cooking/{id}
PUT    /cooking/{id}/favorite

POST   /ingredients/validate
GET    /ingredients/search

POST   /posts
GET    /posts/{id}
PUT    /posts/{id}
DELETE /posts/{id}
GET    /posts/feed
GET    /posts/user/{userId}

POST   /posts/{id}/comments
GET    /posts/{id}/comments
PUT    /comments/{id}
DELETE /comments/{id}

POST   /reactions
DELETE /reactions/{id}
GET    /reactions/{targetType}/{targetId}

GET    /notifications
PUT    /notifications/{id}/read
PUT    /notifications/read-all
```

### 4. Lambda Functions

#### Lambda 1: Auth Handler
- Runtime: Node.js 20
- Memory: 256MB
- Timeout: 10s
- Triggers: Cognito Post-Authentication
- Mục đích: Tạo profile người dùng khi đăng nhập lần đầu

#### Lambda 2: Recipe CRUD
- Runtime: Node.js 20
- Memory: 512MB
- Timeout: 30s
- Triggers: API Gateway
- Mục đích:
  - Tạo, đọc, cập nhật, xóa recipes
  - Upload ảnh recipe lên S3
  - Query DynamoDB

#### Lambda 3: AI Suggestion Engine ⭐
- Runtime: Node.js 20
- Memory: 1024MB
- Timeout: 60s
- Triggers: API Gateway
- Mục đích:
  - Validate nguyên liệu với master_ingredients table
  - Query 4 approved recipes từ DynamoDB
  - Gọi Amazon Bedrock (Claude 3.5) cho 1 recipe mới
  - Tạo gợi ý công thức
  - Phân tích nguyên liệu người dùng
  - Lưu gợi ý vào DynamoDB
- Chi phí: Lambda tốn kém nhất (~70% chi phí compute)

#### Lambda 4: User Profile Manager
- Runtime: Node.js 20
- Memory: 256MB
- Timeout: 10s
- Triggers: API Gateway
- Mục đích:
  - Quản lý profile & preferences người dùng
  - Cập nhật danh sách nguyên liệu
  - Xử lý upload avatar
  - Quản lý cài đặt riêng tư

#### Lambda 5: Social/Friends Handler
- Runtime: Node.js 20
- Memory: 256MB
- Timeout: 10s
- Triggers: API Gateway
- Mục đích:
  - Quản lý yêu cầu kết bạn
  - Chấp nhận/từ chối yêu cầu kết bạn
  - Liệt kê danh sách bạn bè
  - Lọc dữ liệu theo privacy settings

#### Lambda 6: Posts & Comments Handler
- Runtime: Node.js 20
- Memory: 512MB
- Timeout: 30s
- Triggers: API Gateway
- Mục đích:
  - Tạo, đọc, cập nhật, xóa bài đăng
  - Quản lý bình luận (nested replies)
  - Upload ảnh posts lên S3
  - Cập nhật counters (likes_count, comments_count)
  - Lọc theo privacy settings

#### Lambda 7: Notifications Handler
- Runtime: Node.js 20
- Memory: 256MB
- Timeout: 10s
- Triggers: API Gateway, DynamoDB Streams
- Mục đích:
  - Tạo thông báo khi có hoạt động (comment, like, friend request)
  - Đánh dấu đã đọc/chưa đọc
  - Lấy danh sách thông báo
  - Push notifications (tương lai)

#### Lambda 8: Admin Operations
- Runtime: Node.js 20
- Memory: 512MB
- Timeout: 30s
- Triggers: API Gateway (Chỉ Admin)
- Mục đích:
  - Quản lý người dùng (ban/unban)
  - Kiểm duyệt nội dung (posts, comments)
  - Thống kê hệ thống
  - Không cần phê duyệt recipes - Tự động approval dựa trên rating

#### Lambda 9: Cooking History Handler ⭐ NEW
- Runtime: Node.js 20
- Memory: 256MB
- Timeout: 10s
- Triggers: API Gateway
- Mục đích:
  - Quản lý lịch sử nấu ăn cá nhân
  - Start/complete cooking sessions
  - Lấy lịch sử nấu ăn của user
  - Đánh dấu món yêu thích
  - Ghi chú cá nhân cho từng lần nấu

#### Lambda 10: Rating & Approval Handler ⭐ NEW
- Runtime: Node.js 20
- Memory: 256MB
- Timeout: 10s
- Triggers: API Gateway
- Mục đích:
  - Lưu rating của user cho recipes
  - Tính toán average rating
  - Auto-approve recipes khi rating >= 4.0 sao
  - Update recipe_ratings table
  - Link rating với cooking history

#### Lambda 11: Ingredient Validation Handler ⭐ NEW
- Runtime: Node.js 20
- Memory: 256MB
- Timeout: 10s
- Triggers: API Gateway
- Mục đích:
  - Validate nguyên liệu với master_ingredients table
  - Fuzzy search cho nguyên liệu tương tự
  - Auto-correct tên nguyên liệu (bỏ dấu)
  - Gợi ý nguyên liệu thay thế

### 5. Dịch Vụ AI/ML

#### Amazon Bedrock (Claude 3.5)
- Model: Claude 3.5 Sonnet (hoặc Haiku để tiết kiệm chi phí)
- Use Case: AI agent gợi ý công thức
- Input: Profile người dùng + preferences + nguyên liệu
- Output: Gợi ý công thức với hướng dẫn
- Chi phí:
  - Sonnet: $3 trên triệu input tokens, $15 trên triệu output tokens
  - Haiku: $0.25 trên triệu input tokens, $1.25 trên triệu output tokens
  - Ước tính: $25-50/tháng (1,000 users, 10 suggestions/user/tháng)
  - Với Haiku: $3-7/tháng (tiết kiệm 70%)

### 6. Database

#### Amazon DynamoDB (NoSQL Database)
Các bảng:
- **Users** (PK: user_id) - Tài khoản người dùng & roles
- **UserData** (PK: user_id, SK: data_type) - Preferences, nguyên liệu
- **PrivacySettings** (PK: user_id) - Cấu hình riêng tư
- **Friendships** (PK: user_id, SK: friend_id) - Kết nối mạng xã hội
- **Recipes** (PK: recipe_id, GSI: user_id) - Dữ liệu công thức
- **RecipeRatings** (PK: recipe_id, SK: user_id) - Đánh giá công thức cho auto-approval ⭐
- **UserCookingHistory** (PK: user_id, SK: timestamp) - Lịch sử nấu ăn cá nhân ⭐
- **MasterIngredients** (PK: ingredient_id) - Master list nguyên liệu để validate ⭐
- **AISuggestions** (PK: user_id, SK: timestamp) - Lịch sử AI
- **Posts** (PK: post_id, GSI: user_id) - Bài đăng xã hội
- **Comments** (PK: post_id, SK: timestamp) - Bình luận
- **Reactions** (PK: target_id, SK: user_id) - Lượt thích
- **Notifications** (PK: user_id, SK: timestamp) - Thông báo

Tính năng:
- Auto-scaling (on-demand mode)
- Point-in-time recovery (PITR)
- DynamoDB Streams (cho cập nhật real-time & notifications)
- Global Secondary Indexes (GSI) cho truy vấn
- Mã hóa at rest
- Chi phí: $35-45/tháng (on-demand pricing với cooking history & ratings)

**Tại sao chọn DynamoDB thay vì RDS?**
- ✅ Serverless (không cần quản lý server)
- ✅ Auto-scaling về zero cost khi idle
- ✅ Tích hợp tốt hơn với Lambda
- ✅ Độ trễ thấp hơn cho key-value access
- ✅ Tiết kiệm chi phí cho MVP ($15 vs $30+ cho RDS)

### 7. Lưu Trữ

#### Amazon S3
Buckets:
- `recipe-images-prod`: Ảnh công thức
- `user-avatars-prod`: Ảnh đại diện
- `post-images-prod`: Ảnh bài đăng
- `static-assets-prod`: Assets ứng dụng

Tính năng:
- Versioning enabled
- Lifecycle policies (xóa sau 90 ngày cho temp files)
- S3 Transfer Acceleration
- Chi phí: ~$10/tháng (100GB storage với social media images)

### 8. Bảo Mật

#### AWS WAF
Bảo vệ:
- SQL injection
- XSS attacks
- Rate limiting (1000 requests/5min mỗi IP)
- Geographic restrictions (tùy chọn)
- Chi phí: $5/tháng + $1 trên triệu requests

#### AWS Secrets Manager
Secrets:
- Database credentials (nếu dùng RDS)
- Third-party API keys (nếu cần)
- Chi phí: $0.40 mỗi secret mỗi tháng

### 9. Giám Sát & Logging

#### Amazon CloudWatch
- Logs: Tất cả logs của Lambda functions
- Metrics: Metrics của API Gateway, Lambda, DynamoDB
- Alarms:
  - Lambda errors > 1%
  - API Gateway 5xx errors
  - DynamoDB throttling
- Chi phí: ~$5-10/tháng

#### AWS X-Ray
- Tracing: Tracing request từ đầu đến cuối
- Performance: Xác định bottlenecks
- Chi phí: $5 trên triệu traces (100k đầu miễn phí)

## Sơ Đồ Luồng Dữ Liệu

### Flow 1: AI Recipe Suggestion (Enhanced - Flexible Mix)

```mermaid
sequenceDiagram
    participant User
    participant APIGateway
    participant LambdaAI
    participant DynamoDB
    participant Bedrock
    participant CloudWatch

    User->>APIGateway: POST /ai/suggest<br/>{ingredients, recipe_count: 3}
    APIGateway->>LambdaAI: Xử lý request

    Note over LambdaAI: STEP 1: Validate Ingredients
    LambdaAI->>DynamoDB: Check master_ingredients
    DynamoDB-->>LambdaAI: Validation results

    alt Có nguyên liệu không hợp lệ
        LambdaAI->>CloudWatch: Log invalid ingredients
        LambdaAI->>DynamoDB: Increment report count
        alt Report count >= 5
            LambdaAI->>DynamoDB: Notify admin
        end
    end

    Note over LambdaAI: STEP 2: Query DB với categories
    LambdaAI->>DynamoDB: Query approved recipes<br/>(match ingredients + categories)
    DynamoDB-->>LambdaAI: Found: 2 món

    Note over LambdaAI: STEP 3: Calculate AI Gap<br/>Requested: 3, DB: 2, Gap: 1

    Note over LambdaAI: STEP 4: Generate AI (nếu cần)
    loop For each AI recipe (1 món)
        LambdaAI->>Bedrock: Generate recipe<br/>(ingredients + diverse category)
        Bedrock-->>LambdaAI: AI Recipe
    end

    Note over LambdaAI: STEP 5: Combine & Return
    LambdaAI->>DynamoDB: Save suggestion history
    LambdaAI-->>User: 2 DB + 1 AI = 3 món ✅<br/>+ Warnings (if any)
```

**Key Improvements**:
- ✅ Flexible recipe count (1-5 món)
- ✅ Dynamic DB/AI mix (tiết kiệm cost)
- ✅ Invalid ingredient reporting system
- ✅ Category diversity (xào, canh, hấp...)

### Flow 2: Recipe Auto-Approval (Rating-based)

```mermaid
sequenceDiagram
    participant User
    participant APIGateway
    participant LambdaRecipe
    participant DynamoDB

    User->>APIGateway: Nấu món (AI generated)
    User->>APIGateway: Click "Hoàn thành"
    APIGateway->>User: Hiển thị form đánh giá
    User->>APIGateway: POST /recipes/{id}/rate (rating + comment)
    APIGateway->>LambdaRecipe: Lưu rating
    LambdaRecipe->>DynamoDB: Save rating
    LambdaRecipe->>DynamoDB: Calculate average rating

    alt Rating >= 4 sao
        LambdaRecipe->>DynamoDB: Set recipe.is_approved = true
        LambdaRecipe->>DynamoDB: Add to public recipes
        LambdaRecipe->>User: "Công thức đã được thêm vào database!"
    else Rating < 4 sao
        LambdaRecipe->>User: "Cảm ơn đánh giá của bạn!"
    end
```

### Flow 3: Social Interaction

```mermaid
sequenceDiagram
    participant User
    participant APIGateway
    participant LambdaPost
    participant LambdaNotification
    participant DynamoDB

    User->>APIGateway: POST /posts (chia sẻ công thức)
    APIGateway->>LambdaPost: Tạo post
    LambdaPost->>DynamoDB: Save post
    LambdaPost->>User: Post created

    User->>APIGateway: POST /posts/{id}/comments
    APIGateway->>LambdaPost: Tạo comment
    LambdaPost->>DynamoDB: Save comment
    LambdaPost->>DynamoDB: Update comments_count
    LambdaPost->>LambdaNotification: Trigger notification
    LambdaNotification->>DynamoDB: Create notification for post owner
    LambdaNotification->>User: Push notification
```

### Flow 4: Error Handling - Invalid Ingredients

```mermaid
sequenceDiagram
    participant User
    participant APIGateway
    participant LambdaAI
    participant DynamoDB

    User->>APIGateway: POST /ai/suggest {"ingredients": ["abc xyz", "123"]}
    APIGateway->>LambdaAI: Validate ingredients

    alt Nguyên liệu không tồn tại
        LambdaAI->>DynamoDB: Fuzzy search similar ingredients
        LambdaAI->>User: Error 400: "Không tìm thấy nguyên liệu. Bạn có muốn: [gà, cá, tôm]?"
    else Nguyên liệu sai format
        LambdaAI->>User: Error 400: "Tên nguyên liệu không hợp lệ. Vui lòng nhập lại."
    else Nguyên liệu quá ít (< 2)
        LambdaAI->>User: Error 400: "Cần ít nhất 2 nguyên liệu để tạo món ăn."
    end
```
