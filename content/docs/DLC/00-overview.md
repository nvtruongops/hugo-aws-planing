---
title: "00 - Project Overview"
weight: 1
---

# Tổng Quan Dự Án - Smart Cooking App

## Mục Tiêu Dự Án

### Vision
Xây dựng nền tảng ứng dụng nấu ăn thông minh sử dụng AI để:
- **Cá nhân hóa** gợi ý công thức dựa trên nguyên liệu có sẵn
- **Tối ưu hóa** việc sử dụng nguyên liệu để giảm lãng phí thực phẩm
- **Kết nối** cộng đồng người yêu thích nấu ăn
- **Tự động hóa** quá trình phê duyệt công thức bằng rating system

### Mission
Tạo trải nghiệm nấu ăn thông minh, dễ dàng và thú vị cho người dùng thông qua:
- AI Agent gợi ý món ăn phù hợp với profile và sở thích cá nhân
- Hệ thống tự động approval công thức dựa trên đánh giá cộng đồng
- Nền tảng chia sẻ kinh nghiệm nấu ăn xã hội
- Kiến trúc serverless tiết kiệm chi phí và dễ scale

## Phạm Vi Dự Án

### MVP (Minimum Viable Product) - Phase 1

#### Core Features
1. **Authentication & User Management**
   - Đăng ký/Đăng nhập với Cognito
   - Profile với thông tin cá nhân (năm sinh, giới tính, quốc gia)
   - Quản lý sở thích món ăn (canh, món chiên, món hấp...)

2. **Ingredient Management**
   - Nhập danh sách nguyên liệu đơn giản (chỉ tên)
   - Validation với master ingredients database
   - Fuzzy search & auto-correct

3. **AI Recipe Suggestion Engine** 
   - Flexible mix: Database + AI generated recipes
   - User request 1-5 món, system tự động mix:
     - Query approved recipes từ DB (match ingredients + categories)
     - Generate thiếu số món bằng AI
   - Category-based diversity (xào, canh, hấp, chiên...)
   - Invalid ingredient reporting system

4. **Cooking History & Rating System** 
   - Theo dõi lịch sử nấu ăn cá nhân
   - Rating system (1-5 sao) cho mỗi món đã nấu
   - **Auto-approval**: Recipe >= 4.0 sao → Tự động thêm vào DB
   - Personal notes & favorites

5. **Recipe Database**
   - Lưu trữ approved recipes
   - Tìm kiếm theo ingredients, category, cooking method
   - Public access cho approved recipes

#### Technical MVP
- **Frontend**: Next.js hosted on AWS Amplify
- **Backend**: Lambda Functions (Node.js 20)
- **Database**: DynamoDB single-table design
- **AI**: Amazon Bedrock (Claude 3 Haiku)
- **Auth**: Amazon Cognito
- **Storage**: S3 for images
- **CDN**: CloudFront

### Phase 2 - Social Features

1. **Privacy Settings**
   - Kiểm soát visibility cho profile, ingredients, recipes
   - 3 levels: Public, Friends, Private

2. **Social Network**
   - Friend requests & connections
   - Privacy-aware data filtering

3. **Posts & Comments**
   - Chia sẻ công thức đã nấu
   - Upload ảnh món ăn
   - Comment & reactions
   - Newsfeed

4. **Notifications**
   - Friend requests, comments, likes
   - Recipe approval notifications
   - Real-time updates (future: WebSocket)

## 👥 Stakeholders

### Primary Stakeholders
1. **End Users** (Home Cooks)
   - Age range: 20-50 tuổi
   - Tech-savvy, mobile-first
   - Muốn nấu ăn hiệu quả và sáng tạo

2. **Project Owner** (You)
   - Quản lý dự án
   - Định hướng product
   - Theo dõi metrics & costs

### Secondary Stakeholders
1. **AWS** (Infrastructure Provider)
   - Serverless services
   - Cost optimization
   - Scalability support

2. **AI Provider** (Amazon Bedrock)
   - Claude 3 Haiku/Sonnet models
   - API reliability
   - Cost-effective AI

3. **Community Contributors** (Future)
   - Recipe contributors
   - Content moderators
   - Beta testers

## Success Metrics

### User Metrics
- **User Growth**: 1,000 users trong 3 tháng đầu
- **Engagement**:
  - MAU (Monthly Active Users): >= 60%
  - AI suggestion usage: >= 10 suggestions/user/month
  - Recipe cooking rate: >= 30% (users thực sự nấu món)
  - Rating participation: >= 50% (users rate sau khi nấu)

### Technical Metrics
- **Performance**:
  - API response time: < 500ms (excluding AI generation)
  - AI generation time: < 5s per recipe
  - System uptime: >= 99.5%
- **Quality**:
  - Recipe approval rate: >= 40% (AI recipes đạt 4+ sao)
  - Invalid ingredient rate: < 5%
  - Error rate: < 1%

### Business Metrics
- **Cost Efficiency**:
  - Cost per user: < $0.20/month (MVP phase)
  - DB coverage growth: 0% → 60% trong 6 tháng
  - AI cost reduction: 30-50% through flexible mix
- **Revenue** (Future):
  - Freemium conversion: >= 5%
  - Premium tier: $4.99/month
  - Break-even: Month 3

## Key Differentiators

### 1. Flexible AI/DB Mix Strategy
- **Smart**: Tự động balance giữa DB queries và AI generation
- **Cost-effective**: Tiết kiệm 30-70% AI cost khi DB coverage tăng
- **Scalable**: Auto-build DB từ popular AI recipes

### 2. Community-Driven Approval
- **No manual approval**: Auto-approve dựa trên rating >= 4.0 sao
- **Democratic**: Community quyết định recipe quality
- **Scalable**: Không cần moderator team

### 3. Privacy-First Social
- **User control**: Granular privacy settings
- **Transparent**: Clear data usage policy cho AI
- **Safe**: Friend-only or private sharing options

### 4. Serverless Architecture
- **Zero idle cost**: Pay only for usage
- **Auto-scaling**: Từ 10 users → 10K users
- **Low maintenance**: Managed services

## Product Philosophy

### Design Principles
1. **Simplicity First**: MVP focus on core value - AI suggestions
2. **User Privacy**: Transparent AI data usage, user control
3. **Community Quality**: Let users decide what's good
4. **Cost Awareness**: Build for scale, optimize for cost
5. **Data-Driven**: Metrics-based feature decisions

### Development Principles
1. **Serverless First**: Use managed services
2. **Single Responsibility**: Each Lambda does one thing well
3. **Fail Fast**: Validate early, fail gracefully
4. **Monitor Everything**: CloudWatch, X-Ray, alarms
5. **Security by Default**: IAM, WAF, encryption

## Timeline Overview

| Phase | Duration | Key Deliverables |
|-------|----------|------------------|
| **Phase 0**: Planning & Design | Week 1-2 | Architecture, Database design, Cost analysis |
| **Phase 1**: Core MVP | Week 0-3 | Auth, Ingredients, AI Engine, Rating System |
| **Phase 2**: Social Features | Week 3-8 | Privacy, Friends, Posts, Notifications |


## Expected Outcomes

### Short-term (3 months)
- MVP deployed và stable
- 500-1,000 active users
- 40%+ AI recipes approved
- Database coverage: 20-30%
- Monthly cost: $135-160

##    Related Documents

- [01 - Requirements](01-requirements.md)
- [02 - Constraints](02-constraints.md)
- [10 - Architecture](10-architecture.md)
- [30 - Cost Analysis](30-cost-analysis.md)
