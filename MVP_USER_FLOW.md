# USER FLOW - MVP SIMPLIFIED

## 🎯 Core User Journey (Demo Focus)

### Flow 1: Đăng ký & Setup (5 phút)

```
┌─────────────────────────────────────────────┐
│  BƯỚC 1: ĐĂNG KÝ                            │
├─────────────────────────────────────────────┤
│  • Vào trang web                            │
│  • Click "Đăng ký"                          │
│  • Nhập: Email, Password, Tên               │
│  • Xác nhận email (Cognito)                 │
│  • ✅ Tài khoản đã tạo                      │
└────────────────┬────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────┐
│  BƯỚC 2: NHẬP NGUYÊN LIỆU                   │
├─────────────────────────────────────────────┤
│  Màn hình: "Bạn có những nguyên liệu gì?"   │
│                                             │
│  [Input field]: Nhập nguyên liệu            │
│  [+ Thêm]                                   │
│                                             │
│  Ví dụ user nhập:                           │
│  • Thịt gà                                  │
│  • Cà chua                                  │
│  • Hành tây                                 │
│  • Tỏi                                      │
│  • Gừng                                     │
│                                             │
│  [Tiếp tục →]                               │
└────────────────┬────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────┐
│  ✅ Setup hoàn tất!                         │
│  Chuyển sang màn hình chính                 │
└─────────────────────────────────────────────┘
```

---

### Flow 2: AI Gợi Ý Món Ăn (Core Feature)

```
┌─────────────────────────────────────────────┐
│  MÀNG HÌNH CHÍNH                            │
├─────────────────────────────────────────────┤
│  Nguyên liệu của bạn:                       │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│  🍗 Thịt gà         [x]                     │
│  🍅 Cà chua         [x]                     │
│  🧅 Hành tây        [x]                     │
│  🧄 Tỏi             [x]                     │
│  🫚 Gừng            [x]                     │
│  + Thêm nguyên liệu                         │
│                                             │
│  [🤖 Gợi ý món ăn]  ← BUTTON CHÍNH          │
└────────────────┬────────────────────────────┘
                 │ Click
                 ▼
┌─────────────────────────────────────────────┐
│  ⏳ ĐANG TẠO GỢI Ý...                       │
├─────────────────────────────────────────────┤
│  AI đang phân tích nguyên liệu...           │
│  [Loading animation]                        │
│                                             │
│  (Backend: Lambda → Bedrock → 3-5 recipes)  │
└────────────────┬────────────────────────────┘
                 │ 3-5 seconds
                 ▼
┌─────────────────────────────────────────────┐
│  ✨ 5 MÓN ĂN GỢI Ý CHO BẠN                  │
├─────────────────────────────────────────────┤
│  1. GÀ XÀO SẢ ỚT                            │
│     ⭐ Match: 100% (5/5 nguyên liệu)        │
│     ⏱️ 30 phút • 🍽️ 4 người                 │
│     [Xem chi tiết] [❤️ Yêu thích]          │
│                                             │
│  2. CANH GÀ HẦM GỪNG                        │
│     ⭐ Match: 80% (4/5 nguyên liệu)         │
│     ⏱️ 45 phút • 🍽️ 4 người                 │
│     [Xem chi tiết] [❤️ Yêu thích]          │
│                                             │
│  3. GÀ CHIÊN SỐT CÀ CHUA                    │
│     ⭐ Match: 80% (4/5 nguyên liệu)         │
│     ⏱️ 25 phút • 🍽️ 2 người                 │
│     [Xem chi tiết] [❤️ Yêu thích]          │
│                                             │
│  4. CƠM GÀ HÀNH TỎI                         │
│     ⭐ Match: 60% (3/5 nguyên liệu)         │
│     ⏱️ 40 phút • 🍽️ 3 người                 │
│     [Xem chi tiết] [❤️ Yêu thích]          │
│                                             │
│  5. GÀ NƯỚNG MUỐI ỚT                        │
│     ⭐ Match: 60% (3/5 nguyên liệu)         │
│     ⏱️ 35 phút • 🍽️ 2 người                 │
│     [Xem chi tiết] [❤️ Yêu thích]          │
└────────────────┬────────────────────────────┘
                 │ Click "Xem chi tiết"
                 ▼
┌─────────────────────────────────────────────┐
│  GÀ XÀO SẢ ỚT                               │
├─────────────────────────────────────────────┤
│  ⏱️ Thời gian: 30 phút                      │
│  🍽️ Khẩu phần: 4 người                      │
│  ⭐ Match: 100%                              │
│                                             │
│  NGUYÊN LIỆU:                               │
│  • 500g thịt gà (đùi hoặc ức)               │
│  • 2 củ hành tây                            │
│  • 3 quả cà chua                            │
│  • 5 tép tỏi                                │
│  • 1 khúc gừng                              │
│  • 2 cây sả                                 │
│  • Ớt, nước mắm, dầu ăn                     │
│                                             │
│  HƯỚNG DẪN:                                 │
│  1. Sơ chế gà, ướp với tỏi gừng             │
│  2. Xào thơm sả, hành tây                   │
│  3. Cho gà vào xào săn                      │
│  4. Thêm cà chua, nêm nếm                   │
│  5. Đảo đều cho chín, tắt bếp               │
│                                             │
│  [❤️ Yêu thích] [🔙 Quay lại]              │
└─────────────────────────────────────────────┘
```

---

### Flow 3: Quản Lý Yêu Thích

```
┌─────────────────────────────────────────────┐
│  MENU: YÊU THÍCH ❤️                         │
├─────────────────────────────────────────────┤
│  Món ăn đã lưu (5)                          │
│                                             │
│  1. Gà xào sả ớt                            │
│     Đã lưu: 2 ngày trước                    │
│     [Xem] [Xóa]                             │
│                                             │
│  2. Canh gà hầm gừng                        │
│     Đã lưu: 1 tuần trước                    │
│     [Xem] [Xóa]                             │
│                                             │
│  3. Phở gà                                  │
│     Đã lưu: 2 tuần trước                    │
│     [Xem] [Xóa]                             │
└─────────────────────────────────────────────┘
```

---

## 📊 User Metrics (Demo)

### Phase 1: 100 Users (Tuần 1-2)
- **Signup rate**: 80% complete profile
- **AI suggestions**: 5-10 per user
- **Favorites saved**: 3-5 per user
- **Daily active**: 40-50 users

### Phase 2: 500 Users (Tuần 3-4)
- **Signup rate**: 75%
- **AI suggestions**: 8-12 per user
- **Favorites saved**: 4-6 per user
- **Daily active**: 150-200 users

### Phase 3: 1,000 Users (Tuần 5-6)
- **Signup rate**: 70%
- **AI suggestions**: 10-15 per user
- **Favorites saved**: 5-8 per user
- **Daily active**: 300-400 users

---

## ✅ Demo Script (5 phút)

### Phút 1-2: Đăng ký
1. Mở trang web
2. Đăng ký tài khoản
3. Nhập 5 nguyên liệu

### Phút 3-4: AI Suggestion
1. Click "Gợi ý món ăn"
2. Xem 5 món được đề xuất
3. Click xem chi tiết 1 món
4. Lưu món yêu thích

### Phút 5: Review
1. Xem lại danh sách yêu thích
2. Xem lịch sử gợi ý
3. Kết thúc demo

---

## 🎯 Success Metrics (Báo cáo)

✅ **Technical**:
- AI response time < 5s
- 99% uptime
- Support 1,000 concurrent users

✅ **Business**:
- 70%+ signup completion
- 80%+ users request AI suggestion
- 60%+ users save favorites
- 50%+ users return within 7 days

✅ **Cost**:
- < $80/month for 1,000 users
- < $0.08 per user per month
