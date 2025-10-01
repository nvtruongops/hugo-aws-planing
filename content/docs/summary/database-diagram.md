---
title: "Database Schema Diagram"
weight: 3
---

# Sơ Đồ Cấu Trúc Cơ Sở Dữ Liệu

## ERD (Sơ Đồ Quan Hệ Thực Thể)

```dbml
// Bảng Người Dùng
Table users {
  user_id varchar(36) [pk, note: 'UUID từ Cognito']
  email varchar(255) [unique, not null]
  username varchar(50) [unique, not null]
  full_name varchar(100)
  date_of_birth date
  gender enum('male', 'female', 'other')
  country varchar(50)
  avatar_url varchar(500)
  role enum('user', 'admin') [default: 'user']
  is_active boolean [default: true]
  created_at timestamp [default: `now()`]
  updated_at timestamp [default: `now()`]
  last_login timestamp

  indexes {
    email
    username
    role
  }
}

// Tùy Chọn Người Dùng
Table user_preferences {
  preference_id varchar(36) [pk]
  user_id varchar(36) [ref: > users.user_id, not null]
  dietary_restrictions json [note: 'chay, thuần chay, halal, kosher, v.v.']
  allergies json [note: 'Danh sách dị ứng']
  favorite_cuisines json [note: 'Ý, Việt Nam, Nhật Bản, v.v.']
  cooking_methods json [note: 'hấp, chiên, nướng, súp, v.v.']
  spice_level enum('none', 'mild', 'medium', 'hot', 'very_hot')
  created_at timestamp [default: `now()`]
  updated_at timestamp [default: `now()`]
}

// Cài Đặt Quyền Riêng Tư
Table privacy_settings {
  setting_id varchar(36) [pk]
  user_id varchar(36) [ref: > users.user_id, not null, unique]
  profile_visibility enum('public', 'friends', 'private') [default: 'public', note: 'Ai có thể xem hồ sơ']
  email_visibility enum('public', 'friends', 'private') [default: 'private']
  date_of_birth_visibility enum('public', 'friends', 'private') [default: 'friends']
  gender_visibility enum('public', 'friends', 'private') [default: 'public']
  country_visibility enum('public', 'friends', 'private') [default: 'public']
  recipes_visibility enum('public', 'friends', 'private') [default: 'public']
  ingredients_visibility enum('public', 'friends', 'private') [default: 'friends']
  preferences_visibility enum('public', 'friends', 'private') [default: 'friends']
  created_at timestamp [default: `now()`]
  updated_at timestamp [default: `now()`]

  indexes {
    user_id
  }
}

// Quan Hệ Bạn Bè
Table friendships {
  friendship_id varchar(36) [pk]
  user_id varchar(36) [ref: > users.user_id, not null]
  friend_id varchar(36) [ref: > users.user_id, not null]
  status enum('pending', 'accepted', 'blocked') [default: 'pending']
  requested_at timestamp [default: `now()`]
  responded_at timestamp

  indexes {
    (user_id, friend_id) [unique]
    user_id
    friend_id
    status
  }
}

// Nguyên Liệu Người Dùng (Danh sách đơn giản - không theo dõi số lượng)
Table user_ingredients {
  ingredient_id varchar(36) [pk]
  user_id varchar(36) [ref: > users.user_id, not null]
  ingredient_name varchar(100) [not null]
  category varchar(50) [note: 'thịt, rau, gia vị, sữa, v.v.']
  added_at timestamp [default: `now()`]

  indexes {
    (user_id, ingredient_name) [unique]
  }
}

// Công Thức (Do người dùng tạo & AI tạo)
Table recipes {
  recipe_id varchar(36) [pk]
  user_id varchar(36) [ref: > users.user_id, note: 'Người tạo - null nếu là công thức hệ thống']
  title varchar(200) [not null]
  normalized_title varchar(200) [note: 'Chữ thường, không dấu để tìm kiếm']
  description text
  cuisine_type varchar(50)
  cooking_method varchar(50)
  difficulty enum('easy', 'medium', 'hard')
  prep_time_minutes int
  cook_time_minutes int
  servings int
  calories_per_serving int
  instructions json [note: 'Mảng các đối tượng bước']
  nutrition json [note: 'Protein, carbs, chất béo, v.v.']
  is_public boolean [default: false]
  is_ai_generated boolean [default: false]
  ai_cache_hit_count int [default: 0, note: 'Số lần tái sử dụng từ cache']
  created_at timestamp [default: `now()`]
  updated_at timestamp [default: `now()`]

  indexes {
    user_id
    normalized_title
    cuisine_type
    cooking_method
  }
}

// Bộ Nhớ Cache Công Thức (Công thức AI tạo để tái sử dụng)
Table recipe_cache {
  cache_id varchar(36) [pk]
  recipe_name varchar(200) [not null]
  normalized_name varchar(200) [unique, not null, note: 'Khóa tìm kiếm']
  recipe_data json [note: 'JSON công thức đầy đủ từ AI']
  ingredients_hash varchar(64) [note: 'Hash của danh sách nguyên liệu']
  cuisine_type varchar(50)
  hit_count int [default: 0, note: 'Bộ đếm tái sử dụng']
  last_accessed timestamp
  created_at timestamp [default: `now()`]

  indexes {
    normalized_name
    ingredients_hash
    cuisine_type
  }
}

// Nguyên Liệu Công Thức
Table recipe_ingredients {
  id varchar(36) [pk]
  recipe_id varchar(36) [ref: > recipes.recipe_id, not null]
  ingredient_name varchar(100) [not null]
  quantity varchar(50) [note: '2 chén, 500g, v.v.']
  unit varchar(20)
  is_optional boolean [default: false]

  indexes {
    recipe_id
  }
}

// Lịch Sử Gợi Ý AI
Table ai_suggestions {
  suggestion_id varchar(36) [pk]
  user_id varchar(36) [ref: > users.user_id, not null]
  recipe_id varchar(36) [ref: > recipes.recipe_id]
  cache_id varchar(36) [ref: > recipe_cache.cache_id, note: 'Nếu từ cache']
  prompt_text text [note: 'Đầu vào của người dùng cho AI']
  ingredients_used json [note: 'Danh sách nguyên liệu từ user_ingredients']
  ai_response json [note: 'Phản hồi AI đầy đủ']
  was_from_cache boolean [default: false, note: 'True nếu tái sử dụng từ cache']
  was_accepted boolean [default: false]
  feedback_rating int [note: '1-5 sao']
  feedback_comment text
  created_at timestamp [default: `now()`]

  indexes {
    user_id
    created_at
    cache_id
  }
}

// Yêu Thích
Table favorites {
  favorite_id varchar(36) [pk]
  user_id varchar(36) [ref: > users.user_id, not null]
  recipe_id varchar(36) [ref: > recipes.recipe_id, not null]
  created_at timestamp [default: `now()`]

  indexes {
    (user_id, recipe_id) [unique]
    user_id
  }
}

// Kế Hoạch Bữa Ăn
Table meal_plans {
  plan_id varchar(36) [pk]
  user_id varchar(36) [ref: > users.user_id, not null]
  plan_name varchar(100)
  start_date date [not null]
  end_date date [not null]
  is_active boolean [default: true]
  created_at timestamp [default: `now()`]

  indexes {
    user_id
    start_date
  }
}

// Mục Kế Hoạch Bữa Ăn
Table meal_plan_items {
  item_id varchar(36) [pk]
  plan_id varchar(36) [ref: > meal_plans.plan_id, not null]
  recipe_id varchar(36) [ref: > recipes.recipe_id, not null]
  meal_date date [not null]
  meal_type enum('breakfast', 'lunch', 'dinner', 'snack')
  notes text

  indexes {
    plan_id
    meal_date
  }
}

// Nhật Ký Hoạt Động Người Dùng
Table activity_logs {
  log_id varchar(36) [pk]
  user_id varchar(36) [ref: > users.user_id, not null]
  activity_type enum('login', 'recipe_view', 'recipe_create', 'ai_suggestion', 'ingredient_add')
  activity_data json
  ip_address varchar(45)
  user_agent text
  created_at timestamp [default: `now()`]

  indexes {
    user_id
    created_at
    activity_type
  }
}
```

## Công Nghệ Cơ Sở Dữ Liệu

**Cơ Sở Dữ Liệu Chính: Amazon DynamoDB (NoSQL)**
- Khả năng mở rộng cao
- Độ trễ thấp
- Serverless (không cần quản lý hạ tầng)
- Hiệu quả chi phí cho khối lượng công việc đọc nhiều

**Phương Án Thay Thế: Amazon RDS PostgreSQL**
- Nếu cần các truy vấn quan hệ phức tạp
- Tốt hơn cho phân tích và báo cáo

## Mẫu Truy Cập Dữ Liệu

### Mẫu Truy Cập Chính
1. **Lấy Hồ Sơ Người Dùng**: users.user_id → Dữ liệu người dùng + vai trò
2. **Lấy Tùy Chọn Người Dùng**: user_preferences.user_id → Tùy chọn
3. **Lấy Nguyên Liệu Người Dùng**: user_ingredients.user_id → Danh sách nguyên liệu
4. **Lấy Cài Đặt Quyền Riêng Tư**: privacy_settings.user_id → Cấu hình riêng tư
5. **Kiểm Tra Quan Hệ Bạn Bè**: friendships(user_id, friend_id) → Trạng thái bạn bè
6. **Lấy Danh Sách Bạn Bè**: friendships.user_id → Tất cả bạn bè
7. **Lấy Chi Tiết Công Thức**: recipes.recipe_id → Công thức + Nguyên liệu
8. **Lấy Công Thức Người Dùng**: recipes.user_id → Công thức của người dùng (lọc theo quyền riêng tư)
9. **Lấy Gợi Ý AI**: ai_suggestions.user_id + created_at → Gợi ý gần đây
10. **Lấy Yêu Thích**: favorites.user_id → Công thức yêu thích
11. **Lấy Kế Hoạch Bữa Ăn**: meal_plans.user_id + start_date → Kế hoạch bữa ăn đang hoạt động

### Mẫu Thứ Cấp
- Tìm kiếm công thức theo món ăn/phương pháp (với lọc quyền riêng tư)
- Lấy công thức công khai phổ biến
- Quản trị: Lấy tất cả người dùng với bộ lọc vai trò
- Quản trị: Lấy thống kê người dùng
- Truy vấn phân tích (nhật ký hoạt động)

## Thiết Kế Bảng DynamoDB (Mẫu Bảng Đơn)

### Bảng Chính: `smart-cooking-data`

#### Người Dùng
- **PK**: `USER#<user_id>`
- **SK**: `PROFILE`
- **Thuộc tính**: email, username, full_name, role, is_active, v.v.
- **GSI1PK**: `ROLE#<role>` (cho truy vấn quản trị)

#### Tùy Chọn Người Dùng
- **PK**: `USER#<user_id>`
- **SK**: `PREFERENCES`
- **Thuộc tính**: dietary_restrictions, allergies, favorite_cuisines, v.v.

#### Cài Đặt Quyền Riêng Tư
- **PK**: `USER#<user_id>`
- **SK**: `PRIVACY`
- **Thuộc tính**: profile_visibility, email_visibility, v.v.

#### Nguyên Liệu Người Dùng
- **PK**: `USER#<user_id>`
- **SK**: `INGREDIENT#<ingredient_name>`
- **Thuộc tính**: category, added_at

#### Quan Hệ Bạn Bè
- **PK**: `USER#<user_id>`
- **SK**: `FRIEND#<friend_id>`
- **Thuộc tính**: status, requested_at, responded_at
- **GSI1PK**: `USER#<friend_id>` (để tra cứu ngược)
- **GSI1SK**: `FRIEND#<user_id>`

#### Công Thức
- **PK**: `RECIPE#<recipe_id>`
- **SK**: `METADATA`
- **Thuộc tính**: title, description, cuisine_type, difficulty, v.v.
- **GSI1PK**: `USER#<user_id>` (công thức của người dùng)
- **GSI1SK**: `RECIPE#<created_at>`

#### Nguyên Liệu Công Thức
- **PK**: `RECIPE#<recipe_id>`
- **SK**: `INGREDIENT#<number>`
- **Thuộc tính**: ingredient_name, quantity, unit

#### Gợi Ý AI
- **PK**: `USER#<user_id>`
- **SK**: `SUGGESTION#<timestamp>`
- **Thuộc tính**: recipe_id, prompt_text, ai_response, was_accepted

#### Yêu Thích
- **PK**: `USER#<user_id>`
- **SK**: `FAVORITE#<recipe_id>`
- **Thuộc tính**: created_at

### Chỉ Mục Thứ Cấp Toàn Cục (GSI)

**GSI1**: Truy vấn dựa trên người dùng
- **PK**: GSI1PK (ví dụ: `ROLE#admin`, `USER#<user_id>`)
- **SK**: GSI1SK (ví dụ: timestamp, `RECIPE#<created_at>`)
- **Trường hợp sử dụng**:
  - Lấy tất cả quản trị viên
  - Lấy công thức của người dùng sắp xếp theo ngày
  - Lấy quan hệ bạn bè (tra cứu ngược)

**GSI2**: Tìm kiếm & khám phá công thức
- **PK**: GSI2PK (ví dụ: `CUISINE#<type>`, `METHOD#<type>`)
- **SK**: GSI2SK (ví dụ: `RECIPE#<created_at>`)
- **Trường hợp sử dụng**:
  - Tìm kiếm công thức theo món ăn
  - Tìm kiếm công thức theo phương pháp nấu
  - Lấy công thức phổ biến

## Ước Tính Dung Lượng Lưu Trữ & Chi Phí

### Giả Định
- 1.000 người dùng hoạt động
- Trung bình 20 nguyên liệu mỗi người dùng
- Trung bình 5 công thức đã lưu mỗi người dùng
- Trung bình 10 gợi ý AI mỗi người dùng mỗi tháng

### Ước Tính Dung Lượng
- Người dùng: 1.000 × 1KB = 1MB
- Nguyên liệu người dùng: 1.000 × 20 × 0.5KB = 10MB
- Công thức: 5.000 × 5KB = 25MB
- Gợi ý AI: 10.000 × 2KB = 20MB
- **Tổng cộng: ~60MB**

### Chi Phí DynamoDB (Hàng Tháng)
- Lưu trữ: 60MB × $0.25/GB = ~$0.02
- Khả năng đọc: ~$5-10
- Khả năng ghi: ~$5-10
- **Tổng cộng: ~$10-20/tháng**
