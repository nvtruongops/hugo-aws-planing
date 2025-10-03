---
title: "01 - Requirements"
weight: 2
---

# Yêu Cầu Dự Án - Smart Cooking App

## Functional Requirements

### 1. User Management (FR-UM)

#### FR-UM-01: User Registration
- **Mô tả**: Người dùng có thể đăng ký tài khoản mới
- **Input**: Email, password, username, full_name
- **Process**:
  - Validation email format
  - Password strength check (min 8 characters)
  - Username uniqueness check
  - Email verification via Cognito
- **Output**: User account created, verification email sent
- **Priority**: **MUST HAVE** (P0)

#### FR-UM-02: User Login
- **Mô tả**: Người dùng có thể đăng nhập vào hệ thống
- **Input**: username, password
- **Process**: Cognito authentication
- **Output**: JWT token, redirect to dashboard
- **Priority**: **MUST HAVE** (P0)

#### FR-UM-03: User Profile Management
- **Mô tả**: Người dùng có thể cập nhật thông tin cá nhân
- **Input**: Full name, date_of_birth, gender, country, avatar
- **Validation**:
  - Date_of_birth: Valid date, user >= 13 years old
  - Gender: Enum (male, female, other)
  - Country: Valid country code
- **Output**: Updated profile
- **AI Usage**: Date_of_birth, gender, country được sử dụng cho AI personalization
- **Priority**: **MUST HAVE** (P0)

#### FR-UM-04: User Preferences
- **Mô tả**: Người dùng có thể thiết lập sở thích món ăn
- **Input**:
  - preferred_cooking_methods: Array ['xào', 'hấp', 'canh', 'chiên', 'kho']
  - favorite_cuisines: Array ['Việt Nam', 'Ý', 'Nhật', 'Thái']
  - allergies: Array ['tôm', 'cua', 'sữa']
  - dietary_restrictions: Array ['chay', 'halal', 'low-carb']
- **Output**: Updated preferences, used for AI suggestions
- **Priority**: **MUST HAVE** (P0)

### 2. Ingredient Management (FR-IM)

#### FR-IM-01: Add Ingredient
- **Mô tả**: Người dùng nhập nguyên liệu có sẵn
- **Input**: Ingredient name (string)
- **Process**:
  - Validate với master_ingredients table
  - Fuzzy search nếu không exact match
  - Auto-correct nếu có similar match
- **Output**:
  - Success: Ingredient added
  - Warning: Auto-corrected to similar ingredient
  - Error: Invalid ingredient, suggestions provided
- **Priority**: **MUST HAVE** (P0)

#### FR-IM-02: List Ingredients
- **Mô tả**: Hiển thị danh sách nguyên liệu của user
- **Output**: Array of ingredients với added_at timestamp
- **Priority**: **MUST HAVE** (P0)

#### FR-IM-03: Remove Ingredient
- **Mô tả**: Xóa nguyên liệu khỏi danh sách
- **Input**: Ingredient ID
- **Output**: Ingredient removed
- **Priority**: **MUST HAVE** (P0)

#### FR-IM-04: Validate Ingredients (Batch)
- **Mô tả**: Validate nhiều nguyên liệu cùng lúc
- **Input**: Array of ingredient names
- **Output**:
  - valid: Array of valid ingredients
  - invalid: Array of invalid ingredients
  - corrected: Array of auto-corrected matches
  - suggestions: Array of similar ingredients
- **Priority**: **MUST HAVE** (P0)

### 3. AI Recipe Suggestion (FR-AI)

#### FR-AI-01: Request Recipe Suggestions
- **Mô tả**: User yêu cầu AI gợi ý món ăn
- **Input**:
  - ingredients: Array of ingredient names (min 2)
  - recipe_count: Number (1-5, default: 1)
  - user_preferences: From user profile & preferences
- **Process**:
  1. Validate ingredients với master_ingredients
  2. Query DB cho approved recipes (match ingredients + categories)
  3. Calculate gap: requested_count - db_count
  4. Generate gap recipes bằng AI (nếu cần)
  5. Ensure category diversity (xào, canh, hấp...)
- **Output**:
  - suggestions: Array of recipes (DB + AI mix)
  - stats: { requested, from_database, from_ai }
  - warnings: Array of invalid ingredients
- **Constraints**:
  - AI generation timeout: 60s
- **Priority**: **MUST HAVE** (P0)

#### FR-AI-02: AI Personalization
- **Mô tả**: AI sử dụng user context để personalize suggestions
- **AI Input Data**:
  - Age (from birth_year): Khuyến nghị dinh dưỡng phù hợp
  - Gender: Khuyến nghị khẩu phần
  - Country: Ưu tiên món địa phương
  - Preferred cooking methods: Ưu tiên phương pháp nấu
  - Favorite cuisines: Ưu tiên món quốc gia
  - Allergies: **TUYỆT ĐỐI TRÁNH** các nguyên liệu này
- **Privacy Policy**:
  - KHÔNG sử dụng: Email, full name, địa chỉ
  - Mục đích: Cá nhân hóa gợi ý món ăn
  - Không chia sẻ hoặc khai thác thông tin
- **Priority**: **MUST HAVE** (P0)

#### FR-AI-03: Invalid Ingredient Handling
- **Mô tả**: Xử lý nguyên liệu không hợp lệ gracefully
- **Process**:
  1. Log invalid ingredient to CloudWatch
  2. Increment report count in database
  3. Provide suggestions cho user
  4. Nếu report_count >= 5 → Notify admin
  5. Vẫn generate suggestions với valid ingredients
- **Output**:
  - warnings: Array of invalid items
  - suggestions: Alternative ingredients
  - reported: Boolean (đã log để admin review)
- **Priority**: **MUST HAVE** (P0)

#### FR-AI-04: AI Suggestion History
- **Mô tả**: Lưu lịch sử AI suggestions
- **Saved Data**:
  - prompt_text, ingredients_used
  - requested_recipe_count, recipes_from_db, recipes_from_ai
  - invalid_ingredients (for analytics)
  - ai_response, was_from_cache
- **Output**: History list sorted by created_at DESC
- **Priority**: **SHOULD HAVE** (P1)

### 4. Cooking History & Rating (FR-CH)

#### FR-CH-01: Start Cooking
- **Mô tả**: User bắt đầu nấu một món
- **Input**: recipe_id, suggestion_id (optional)
- **Output**: history_id, status = 'cooking'
- **Priority**: **MUST HAVE** (P0)

#### FR-CH-02: Complete Cooking
- **Mô tả**: User hoàn thành nấu món
- **Input**: history_id
- **Output**:
  - Status updated to 'completed'
  - Prompt rating form
- **Priority**: **MUST HAVE** (P0)

#### FR-CH-03: Rate Recipe
- **Mô tả**: User đánh giá món đã nấu
- **Input**:
  - recipe_id, history_id
  - rating: Number (1-5 stars)
  - comment: String (optional)
- **Process**:
  1. Save rating to recipe_ratings table
  2. Update user_cooking_history với personal_rating
  3. Calculate average rating cho recipe
  4. **Auto-approval**: Nếu average_rating >= 4.0
     - Set recipe.is_approved = true
     - Set recipe.is_public = true
     - Notify user: "Công thức đã được thêm vào database!"
- **Output**:
  - success, rating_saved
  - average_rating, rating_count
  - auto_approved: Boolean
  - message: String
- **Priority**: **MUST HAVE** (P0)

#### FR-CH-04: View Cooking History
- **Mô tả**: Xem lịch sử nấu ăn cá nhân
- **Output**: Array of cooking sessions với:
  - recipe details
  - status, cook_date
  - personal_rating, personal_notes
  - is_favorite
- **Sort**: created_at DESC (newest first)
- **Priority**: **MUST HAVE** (P0)

#### FR-CH-05: Favorite Recipes
- **Mô tả**: Đánh dấu món yêu thích
- **Input**: history_id, is_favorite: Boolean
- **Output**: Updated cooking history
- **Filter**: GET /user/cooking-history?favorites=true
- **Priority**: **SHOULD HAVE** (P1)

### 5. Recipe Management (FR-RM)

#### FR-RM-01: View Recipe Details
- **Mô tả**: Xem chi tiết công thức
- **Input**: recipe_id
- **Output**: Recipe với:
  - Title, description, cuisine_type
  - Cooking method, meal type
  - Ingredients với quantities
  - Instructions (step-by-step)
  - Nutritional info
  - Average rating, rating count
  - is_approved, approval_type
- **Priority**: **MUST HAVE** (P0)

#### FR-RM-02: Search Recipes
- **Mô tả**: Tìm kiếm công thức approved
- **Input**:
  - Query: String (search in title, description)
  - Filters: cuisine_type, cooking_method, meal_type
  - Sort: rating, created_at, popularity
- **Output**: Paginated list of approved recipes
- **Priority**: **SHOULD HAVE** (P1)

#### FR-RM-03: User Created Recipes
- **Mô tả**: User tự tạo recipe (không qua AI)
- **Input**: Recipe data (title, ingredients, instructions...)
- **Process**: Save as is_approved=false, requires rating
- **Output**: Recipe created
- **Priority**: **COULD HAVE** (P2 - Phase 2)

### 6. Social Features (FR-SF) - Phase 2

#### FR-SF-01: Privacy Settings
- **Mô tả**: User thiết lập privacy cho data
- **Settings**:
  - profile_visibility: public/friends/private
  - email_visibility: private (default)
  - date_of_birth_visibility: friends (default)
  - recipes_visibility: public (default)
  - ingredients_visibility: friends (default)
- **Priority**: **SHOULD HAVE** (P1)

#### FR-SF-02: Friend Requests
- **Mô tả**: Gửi/nhận friend requests
- **Actions**:
  - Send request: POST /friends/request
  - Accept: PUT /friends/{id}/accept
  - Reject: PUT /friends/{id}/reject
  - Remove: DELETE /friends/{id}
- **Priority**: **SHOULD HAVE** (P1)

#### FR-SF-03: Posts & Comments
- **Mô tả**: Chia sẻ cooking experience
- **Post**: content, images, recipe_id (optional)
- **Comment**: nested comments support
- **Reactions**: like, love, wow
- **Privacy**: respect user privacy settings
- **Priority**: **SHOULD HAVE** (P1)

#### FR-SF-04: Notifications
- **Mô tả**: Real-time notifications
- **Types**:
  - friend_request, friend_accept
  - comment, like, mention
  - recipe_share, recipe_approved
- **Delivery**: In-app (future: Push)
- **Priority**: **SHOULD HAVE** (P1)

## Non-Functional Requirements

### 1. Performance (NFR-P)

#### NFR-P-01: API Response Time
- **Requirement**: < 500ms for non-AI endpoints
- **Measurement**: CloudWatch metrics, p95 latency
- **Priority**: **MUST HAVE** (P0)

#### NFR-P-02: AI Generation Time
- **Requirement**: < 5 seconds per recipe
- **Fallback**: Show loading indicator, timeout after 60s
- **Priority**: **MUST HAVE** (P0)

#### NFR-P-03: Database Query Performance
- **Requirement**: < 100ms for single-item queries
- **Optimization**: GSI indexes, query optimization
- **Priority**: **MUST HAVE** (P0)

### 2. Scalability (NFR-S)

#### NFR-S-01: Concurrent Users
- **MVP**: Support 100 concurrent users
- **Growth**: Scale to 10,000 concurrent users by Month 12
- **Strategy**: Serverless auto-scaling
- **Priority**: **MUST HAVE** (P0)

#### NFR-S-02: Data Growth
- **Storage**: Plan for 100MB → 10GB in Year 1
- **DynamoDB**: On-demand mode, auto-scaling
- **S3**: Lifecycle policies for old images
- **Priority**: **MUST HAVE** (P0)

### 3. Availability (NFR-A)

#### NFR-A-01: System Uptime
- **Requirement**: 99.5% uptime (Monthly)
- **Downtime tolerance**: ~3.6 hours/month
- **Monitoring**: CloudWatch alarms, X-Ray tracing
- **Priority**: **MUST HAVE** (P0)

#### NFR-A-02: Error Rate
- **Requirement**: < 1% error rate
- **Tracking**: CloudWatch errors, Lambda failures
- **Alerting**: SNS notifications on threshold breach
- **Priority**: **MUST HAVE** (P0)

### 4. Security (NFR-SEC)

#### NFR-SEC-01: Authentication
- **Method**: Cognito User Pool, JWT tokens
- **Password Policy**: Min 8 chars, 1 uppercase, 1 number
- **MFA**: Optional (future enhancement)
- **Priority**: **MUST HAVE** (P0)

#### NFR-SEC-02: Authorization
- **Method**: IAM roles, API Gateway authorizers
- **Principle**: Least privilege access
- **Validation**: Request validation at API Gateway
- **Priority**: **MUST HAVE** (P0)

#### NFR-SEC-03: Data Encryption
- **At Rest**: DynamoDB encryption, S3 encryption
- **In Transit**: HTTPS only, TLS 1.2+
- **Keys**: AWS KMS managed keys
- **Priority**: **MUST HAVE** (P0)

#### NFR-SEC-04: API Security
- **WAF**: AWS WAF rules (SQL injection, XSS)
- **Rate Limiting**: 1000 requests/5min per IP
- **CORS**: Whitelist origins only
- **Priority**: **MUST HAVE** (P0)

### 5. Usability (NFR-U)

#### NFR-U-01: Mobile Responsive
- **Requirement**: Work on all screen sizes
- **Testing**: iOS Safari, Android Chrome
- **Priority**: **MUST HAVE** (P0)

#### NFR-U-02: Accessibility
- **Standard**: WCAG 2.1 Level AA (target)
- **Features**: Alt text, keyboard navigation, screen reader support
- **Priority**: **SHOULD HAVE** (P1)

#### NFR-U-03: Internationalization
- **MVP**: Vietnamese + English
- **Future**: Support 5+ languages
- **Priority**: **COULD HAVE** (P2)

### 6. Maintainability (NFR-M)

#### NFR-M-01: Code Quality
- **Standards**: ESLint, Prettier
- **Testing**: Unit tests (>=70% coverage)
- **Documentation**: JSDoc comments
- **Priority**: **SHOULD HAVE** (P1)

#### NFR-M-02: Monitoring & Logging
- **Logging**: CloudWatch Logs, structured JSON
- **Tracing**: AWS X-Ray for distributed tracing
- **Alarms**: Critical errors, high latency, cost spikes
- **Priority**: **MUST HAVE** (P0)

### 7. Cost Efficiency (NFR-C)

#### NFR-C-01: MVP Cost Target
- **Target**: $135-160/month for 1,000 users
- **Monitoring**: AWS Cost Explorer, budget alarms
- **Optimization**: Flexible DB/AI mix, caching
- **Priority**: **MUST HAVE** (P0)

#### NFR-C-02: Cost per User
- **Target**: < $0.20/month (MVP), < $0.03/month (Scale)
- **Strategy**: DB coverage growth 0% → 80%
- **Metrics**: Track DB vs AI usage ratio
- **Priority**: **MUST HAVE** (P0)

## Acceptance Criteria

### MVP Release Criteria
- All P0 requirements implemented
- User can register, login, manage profile
- User can add ingredients, get AI suggestions
- AI suggestions work with flexible DB/AI mix
- User can track cooking history and rate recipes
- Auto-approval system working (>= 4 stars)
- System uptime >= 99%
- API response time < 500ms (p95)
- Monthly cost <= $160
- 0 critical security vulnerabilities

### Phase 2 Release Criteria
- Privacy settings functional
- Friend system working
- Posts & comments working
- Notifications delivered
- Database coverage >= 30%

##    Related Documents

- [00 - Overview](00-overview.md)
- [02 - Constraints](02-constraints.md)
- [13 - Security](13-security.md)
- [23 - Tasks](23-tasks.md)
