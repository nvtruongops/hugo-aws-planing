# COST ANALYSIS - MVP SIMPLIFIED

## 💰 Chi Phí Hàng Tháng (MVP - Báo cáo)

### Chi Tiết Chi Phí

| Dịch Vụ | 100 users | 500 users | 1,000 users | Ghi Chú |
|---------|-----------|-----------|-------------|---------|
| **AWS Amplify** | $12 | $15 | $15 | Hosting Next.js |
| **CloudFront** | $3 | $5 | $8 | CDN |
| **API Gateway** | $2 | $4 | $6 | ~50K requests/tháng |
| **Lambda (4 functions)** | $5 | $8 | $12 | Đơn giản hơn |
| **Bedrock (Haiku)** | $3 | $8 | $15 | AI suggestions |
| **DynamoDB** | $5 | $8 | $12 | 1 table, minimal |
| **Cognito** | FREE | FREE | FREE | < 50,000 MAU |
| **CloudWatch** | $2 | $3 | $5 | Logs cơ bản |
| **TOTAL** | **$32** | **$51** | **$73** | |

---

## 📊 So Sánh: Full Version vs MVP

| Metric | Full Version | MVP | Difference |
|--------|--------------|-----|------------|
| **Lambda Functions** | 11 | 4 | -64% |
| **DynamoDB Tables** | 13 | 1 | -92% |
| **API Endpoints** | 40+ | 10 | -75% |
| **Cost (1K users)** | $133 | $73 | **-45%** |
| **Development Time** | 8 tuần | 2 tuần | **-75%** |
| **Features** | 100% | 30% | Core only |

---

## 💡 Chi Phí Theo Quy Mô (MVP)

### Phase 1: 100 Users (Beta Test)
- **Chi phí/tháng**: $32
- **Chi phí/user**: $0.32
- **Timeline**: Tuần 1-2
- **Focus**: Validate concept

### Phase 2: 500 Users (Soft Launch)
- **Chi phí/tháng**: $51
- **Chi phí/user**: $0.10
- **Timeline**: Tuần 3-4
- **Focus**: Gather feedback

### Phase 3: 1,000 Users (Demo/Báo cáo)
- **Chi phí/tháng**: $73
- **Chi phí/user**: $0.073
- **Timeline**: Tuần 5-6
- **Focus**: Final demo/presentation

---

## 📈 Chi Phí Projection (6 tuần)

| Tuần | Users | Chi Phí | Tích Lũy | Note |
|------|-------|---------|----------|------|
| 1 | 50 | $20 | $20 | Setup & testing |
| 2 | 100 | $32 | $52 | Beta launch |
| 3 | 250 | $40 | $92 | User growth |
| 4 | 500 | $51 | $143 | Soft launch |
| 5 | 750 | $62 | $205 | Pre-demo |
| 6 | 1,000 | $73 | $278 | **Final demo** |

**Total 6 tuần**: ~$280 (thay vì $600+ với full version)

---

## 🎯 Tối Ưu Chi Phí MVP

### 1. AI/ML Costs (~20% tổng chi phí)
- ✅ Dùng **Claude 3 Haiku** (rẻ nhất)
- ✅ Giới hạn 10 suggestions/user/tháng (free tier)
- ✅ Cache prompt để giảm token usage
- **Tiết kiệm**: $10-15/tháng

### 2. Database Costs (~15% tổng chi phí)
- ✅ Chỉ 1 DynamoDB table
- ✅ On-demand pricing
- ✅ Không dùng GSI (query đơn giản)
- **Tiết kiệm**: $20-30/tháng

### 3. Lambda Costs (~15% tổng chi phí)
- ✅ Chỉ 4 functions
- ✅ Memory tối thiểu (256MB cho hầu hết)
- ✅ Timeout ngắn (10s thay vì 60s)
- **Tiết kiệm**: $10-15/tháng

### 4. Loại Bỏ Hoàn Toàn
- ❌ S3 (không upload ảnh) → Save $10
- ❌ WAF (không cần cho MVP) → Save $6
- ❌ X-Ray (không cần tracing) → Save $5
- ❌ Secrets Manager (dùng env vars) → Save $2
- **Tiết kiệm**: $23/tháng

---

## 💰 Budget Recommendations

### Cho 6 Tuần Demo/Báo Cáo

**Mức 1: Minimal ($200)**
- 500 users max
- 5 suggestions/user
- Basic monitoring
- ✅ Đủ cho demo cơ bản

**Mức 2: Recommended ($300) ✅**
- 1,000 users
- 10 suggestions/user
- Full monitoring
- ✅ **Lựa chọn tốt nhất cho báo cáo**

**Mức 3: Buffer ($400)**
- 1,500 users
- 15 suggestions/user
- Dự phòng spike traffic
- ✅ An toàn nếu viral

---

## 📊 ROI Analysis (Nếu cần)

### Không cần monetize cho báo cáo
Nhưng nếu muốn test business model:

#### Freemium Model
- **Free tier**: 10 suggestions/tháng
- **Premium**: $2.99/tháng (unlimited)
- **Conversion**: 3-5%

**Break-even tại 1,000 users**:
- Cost: $73/tháng
- Premium users: 50 (5%)
- Revenue: 50 × $2.99 = $149
- **Profit: $76** ✅

#### Ad Model (Alternative)
- **CPM**: $1-2 per 1000 impressions
- **Views/user**: 50/tháng
- **Revenue**: 1,000 users × 50 views × $0.002 = $100/tháng
- **Profit: $27** ✅

---

## ✅ Khuyến Nghị Cuối

### Cho Báo Cáo/Demo (6 tuần)

1. **Budget**: $300 (an toàn)
2. **Target**: 1,000 users
3. **Timeline**: 2 tuần dev + 4 tuần user acquisition
4. **Features**: Core only (ingredients + AI suggestion)
5. **Billing alarm**: Set tại $80, $120, $160

### Backup Plan
- Nếu vượt budget $300 → Tắt AI suggestions tạm thời
- Nếu < 500 users → Chỉ tốn ~$150 (OK)
- Nếu > 1,500 users → Chi phí ~$110 (vẫn OK với $300 budget)

---

## 🎯 Success Metrics

✅ **Technical**:
- Deploy thành công < 2 tuần
- Uptime > 99%
- AI response < 5s

✅ **Cost**:
- Total 6 tuần < $300
- Per-user cost < $0.30

✅ **Demo**:
- 1,000+ users registered
- 5,000+ AI suggestions generated
- 3,000+ recipes saved
- Ready for presentation ✅
