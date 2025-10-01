---
title: "Kế Hoạch Triển Khai AI Agent"
weight: 2
---

# KẾ HOẠCH TRIỂN KHAI AI AGENT

Hệ Thống Gợi Ý Công Thức Thông Minh

## 📋 TỔNG QUAN

**Tính năng**: AI Agent gợi ý món ăn thông minh dựa trên:
- Thông tin cá nhân (giới tính, tuổi, quốc gia)
- Sở thích ẩm thực (món canh, món chiên, món hấp…)
- Nguyên liệu nhập đơn giản (chỉ tên, không quản lý số lượng/hạn dùng)

**Phạm vi:**
- ✅ User chỉ nhập danh sách nguyên liệu có sẵn
- ✅ AI tạo công thức phù hợp với user profile

**Thời gian**: 2 tuần (Tuần 3-4 trong deployment plan)

## 🎯 LUỒNG NGƯỜI DÙNG

```
┌─────────────────────────────────────────────────────────┐
│                    ĐẦU VÀO NGƯỜI DÙNG                   │
├─────────────────────────────────────────────────────────┤
│  Người dùng A:                                          │
│  • Nam, 1990 (34 tuổi), Vietnam                        │
│  • Thích ăn: canh, món hấp                             │
│  • Nguyên liệu: Cá rô, bông súng, gừng, hành           │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│              XỬ LÝ AI AGENT                             │
├─────────────────────────────────────────────────────────┤
│  1. Lấy thông tin từ user_preferences                  │
│  2. Lấy nguyên liệu từ user_ingredients                │
│  3. Xây dựng prompt tùy chỉnh:                         │
│     "Tạo món ăn với:                                   │
│      - Nguyên liệu: Cá rô, bông súng                  │
│      - Cho: nam, trung niên                            │
│      - Món: Việt Nam                                   │
│      - Loại: có thể có canh hoặc món hấp"             │
│  4. Gọi Bedrock Claude API                             │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│              PHẢN HỒI AI & KẾT QUẢ CUỐI CÙNG           │
├─────────────────────────────────────────────────────────┤
│  Gợi ý: "Canh cá rô nấu bông súng"                    │
│  • Phù hợp: nam trung niên, món Việt, có canh         │
│  • Nguyên liệu match: 4/4                              │
│  • Độ khó: Dễ                                          │
│  • Thời gian: 30 phút                                  │
│                                                         │
│  📝 Công thức chi tiết (từ AI)                          │
│  🥘 Thông tin dinh dưỡng                               │
│  ⭐ Lưu vào favorites                                   │
└─────────────────────────────────────────────────────────┘
```

## 🏗️ TRIỂN KHAI KỸ THUẬT

### Backend: Node.js 20 Lambda Functions

**Lambda Gợi Ý AI:**
```javascript
// Sử dụng AWS SDK v3 cho Bedrock
const { BedrockRuntimeClient, InvokeModelCommand } = require('@aws-sdk/client-bedrock-runtime');

// Model: Claude 3 Haiku (tiết kiệm chi phí)
// Đầu vào: Sở thích người dùng + nguyên liệu
// Đầu ra: JSON gợi ý công thức
```

**Dịch Vụ Chính:**
- **Amazon Bedrock**: Claude 3 Haiku model
- **DynamoDB**: Lưu trữ dữ liệu người dùng & gợi ý AI
- **Lambda**: Serverless compute (Node.js 20)
- **API Gateway**: REST API endpoints

### Cơ Sở Dữ Liệu: Amazon DynamoDB

**Các Bảng Sử Dụng:**
- `Users` (PK: user_id) - Hồ sơ & vai trò
- `UserData` (PK: user_id, SK: PREFERENCES | INGREDIENTS) - Cài đặt người dùng
- `PrivacySettings` (PK: user_id) - Cấu hình quyền riêng tư
- `Friendships` (PK: user_id, SK: friend_id) - Kết nối xã hội
- `AISuggestions` (PK: user_id, SK: timestamp) - Lịch sử AI

**Triển Khai Quyền Riêng Tư:**
```javascript
// Bộ lọc quyền riêng tư áp dụng trước khi trả về dữ liệu người dùng
if (privacy.ingredients_visibility === 'friends' && !isFriend) {
  delete userProfile.ingredients;
}
```

### API Endpoints

**Luồng Gợi Ý AI:**
```
POST /ai/suggest
Authorization: Bearer <JWT>
Body: {
  "request": "Something light for dinner",
  "dietary_preferences": ["no pork"]
}

Response:
{
  "suggestion_id": "uuid",
  "recipe": {
    "name": "Canh cá rô nấu bông súng",
    "ingredients": [...],
    "instructions": [...],
    "nutrition": {...}
  }
}
```

**Endpoints Quyền Riêng Tư & Xã Hội:**
```
PUT /user/privacy - Cập nhật cài đặt quyền riêng tư
GET /user/profile/{userId} - Lấy hồ sơ (đã lọc quyền riêng tư)
POST /friends/request - Gửi yêu cầu kết bạn
GET /friends - Danh sách bạn bè
```

**Endpoints Quản Trị:**
```
GET /admin/users - Danh sách tất cả người dùng (chỉ admin)
PUT /admin/users/{id}/ban - Cấm người dùng (chỉ admin)
GET /admin/statistics - Thống kê hệ thống (chỉ admin)
```

## 🔒 QUYỀN RIÊNG TƯ & BẢO MẬT

### Các Mức Quyền Riêng Tư
- **Public**: Mọi người có thể xem
- **Friends**: Chỉ bạn bè đã chấp nhận có thể xem
- **Private**: Chỉ người dùng có thể xem

### Thuộc Tính Được Kiểm Soát Quyền Riêng Tư
- Email (mặc định: private)
- Ngày sinh (mặc định: friends)
- Giới tính (mặc định: public)
- Quốc gia (mặc định: public)
- Công thức (mặc định: public)
- Nguyên liệu (mặc định: friends)
- Sở thích (mặc định: friends)

### Kiểm Soát Truy Cập Theo Vai Trò
- **User**: Truy cập thông thường
- **Admin**: Truy cập đầy đủ + công cụ quản trị

## Thời Gian & Mốc Quan Trọng

Xem [Hướng Dẫn Triển Khai Khách Hàng](../customer-deployment-guide) để biết thời gian chi tiết.
