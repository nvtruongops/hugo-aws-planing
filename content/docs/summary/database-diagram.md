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
  preferred_cooking_methods json [note: 'xào, hấp, canh, kho, chiên - ưu tiên khi AI suggest']
  preferred_recipe_count int [default: 1, note: '1-5 món muốn nấu mỗi lần, FREE: 1, PREMIUM: 5']
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
  ingredient_name varchar(256) [not null]
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
  cooking_method varchar(50) [note: 'xào, hấp, luộc, chiên, nướng, kho, rim, trộn, canh, lẩu']
  meal_type varchar(50) [note: 'món chính, món phụ, canh, khai vị, tráng miệng']
  prep_time_minutes int
  cook_time_minutes int
  servings int
  calories_per_serving int
  instructions json [note: 'Mảng các đối tượng bước']
  is_public boolean [default: false]
  is_ai_generated boolean [default: false]
  is_approved boolean [default: false, note: 'Auto-approved nếu rating >= 4 sao']
  approval_type enum('manual', 'auto_rating', 'auto_popular', 'system') [note: 'Cách thức approval']
  average_rating decimal(2,1) [default: 0.0, note: 'Rating trung bình']
  rating_count int [default: 0, note: 'Số lượng đánh giá']
  ai_cache_hit_count int [default: 0, note: 'Số lần tái sử dụng từ cache']
  created_at timestamp [default: `now()`]
  updated_at timestamp [default: `now()`]
  approved_at timestamp [note: 'Thời điểm được approve']

  indexes {
    user_id
    normalized_title
    cuisine_type
    cooking_method
    meal_type
    is_approved
    average_rating
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

// Lịch Sử Gợi Ý AI (Enhanced)
Table ai_suggestions {
  suggestion_id varchar(36) [pk]
  user_id varchar(36) [ref: > users.user_id, not null]
  recipe_id varchar(36) [ref: > recipes.recipe_id]
  cache_id varchar(36) [ref: > recipe_cache.cache_id, note: 'Nếu từ cache']
  prompt_text text [note: 'Đầu vào của người dùng cho AI']
  ingredients_used json [note: 'Danh sách nguyên liệu từ user_ingredients']
  requested_recipe_count int [default: 1, note: '1-5 món user yêu cầu']
  recipes_from_db int [default: 0, note: 'Số món từ database']
  recipes_from_ai int [default: 0, note: 'Số món AI tạo mới']
  invalid_ingredients json [note: 'Nguyên liệu không hợp lệ - logged']
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

// Lịch Sử Nấu Ăn Cá Nhân (Thay thế favorites - mở rộng hơn)
Table user_cooking_history {
  history_id varchar(36) [pk]
  user_id varchar(36) [ref: > users.user_id, not null]
  recipe_id varchar(36) [ref: > recipes.recipe_id, not null]
  suggestion_id varchar(36) [ref: > ai_suggestions.suggestion_id, note: 'Nếu từ AI suggestion']
  status enum('planned', 'cooking', 'completed', 'failed') [default: 'planned']
  personal_rating int [note: '1-5 sao - đánh giá cá nhân']
  personal_notes text [note: 'Ghi chú cá nhân']
  is_favorite boolean [default: false, note: 'Đánh dấu yêu thích']
  cook_date timestamp [note: 'Ngày nấu thực tế']
  created_at timestamp [default: `now()`]
  updated_at timestamp [default: `now()`]

  indexes {
    user_id
    recipe_id
    status
    is_favorite
    cook_date
    (user_id, recipe_id) [note: 'Cho phép multiple entries - lịch sử nấu nhiều lần']
  }
}

// Đánh Giá Công Thức (Cho auto-approval system)
Table recipe_ratings {
  rating_id varchar(36) [pk]
  recipe_id varchar(36) [ref: > recipes.recipe_id, not null]
  user_id varchar(36) [ref: > users.user_id, not null]
  history_id varchar(36) [ref: > user_cooking_history.history_id, note: 'Link đến lịch sử nấu']
  rating int [not null, note: '1-5 sao']
  comment text
  is_verified_cook boolean [default: false, note: 'User đã thực sự nấu món này']
  created_at timestamp [default: `now()`]
  updated_at timestamp [default: `now()`]

  indexes {
    recipe_id
    user_id
    rating
    (recipe_id, user_id) [unique, note: 'Mỗi user chỉ rate 1 lần mỗi recipe']
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

// Master Ingredients (Danh sách nguyên liệu hợp lệ cho validation)
Table master_ingredients {
  ingredient_id varchar(36) [pk]
  name varchar(256) [unique, not null]
  normalized_name varchar(256) [unique, not null, note: 'Không dấu, chữ thường']
  category varchar(50) [note: 'thịt, rau, gia vị, sữa, v.v.']
  aliases json [note: 'Các tên gọi khác: ["thịt bò", "bò", "beef"]']
  is_active boolean [default: true]
  created_at timestamp [default: `now()`]
  updated_at timestamp [default: `now()`]

  indexes {
    normalized_name
    category
    is_active
  }
}

// Nhật Ký Hoạt Động Người Dùng (Enhanced với invalid ingredient tracking)
Table activity_logs {
  log_id varchar(36) [pk]
  user_id varchar(36) [ref: > users.user_id, not null]
  activity_type enum('login', 'recipe_view', 'recipe_create', 'ai_suggestion', 'ingredient_add', 'recipe_cook', 'recipe_rate', 'invalid_ingredient')
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

// Invalid Ingredients Reports (Optional - Post MVP)
Table invalid_ingredients_reports {
  report_id varchar(36) [pk]
  ingredient_name varchar(256) [not null]
  normalized_name varchar(256) [not null, note: 'Chữ thường, không dấu']
  user_id varchar(36) [ref: > users.user_id, not null]
  user_report_count int [default: 1, note: 'Số lần user này report']
  total_reports int [default: 1, note: 'Tổng số reports từ tất cả users']
  status enum('pending', 'reviewed', 'added_to_master', 'spam') [default: 'pending']
  admin_notes text
  created_at timestamp [default: `now()`]
  updated_at timestamp [default: `now()`]

  indexes {
    (normalized_name, user_id) [unique]
    total_reports
    status
  }
}

// Bài Đăng (Posts)
Table posts {
  post_id varchar(36) [pk]
  user_id varchar(36) [ref: > users.user_id, not null]
  recipe_id varchar(36) [ref: > recipes.recipe_id, note: 'Optional - nếu chia sẻ công thức']
  content text [note: 'Nội dung bài đăng']
  images json [note: 'Mảng URLs ảnh']
  is_public boolean [default: true]
  likes_count int [default: 0]
  comments_count int [default: 0]
  created_at timestamp [default: `now()`]
  updated_at timestamp [default: `now()`]

  indexes {
    user_id
    recipe_id
    created_at
    is_public
  }
}

// Bình Luận (Comments)
Table comments {
  comment_id varchar(36) [pk]
  post_id varchar(36) [ref: > posts.post_id, not null]
  user_id varchar(36) [ref: > users.user_id, not null]
  parent_comment_id varchar(36) [ref: > comments.comment_id, note: 'Cho reply/thread']
  content text [not null]
  created_at timestamp [default: `now()`]
  updated_at timestamp [default: `now()`]

  indexes {
    post_id
    user_id
    parent_comment_id
    created_at
  }
}

// Lượt Thích/Reactions
Table reactions {
  reaction_id varchar(36) [pk]
  user_id varchar(36) [ref: > users.user_id, not null]
  target_type enum('post', 'recipe', 'comment') [not null]
  target_id varchar(36) [not null, note: 'ID của post/recipe/comment']
  reaction_type enum('like', 'love', 'wow', 'sad', 'angry') [default: 'like']
  created_at timestamp [default: `now()`]

  indexes {
    (user_id, target_type, target_id) [unique]
    target_type
    target_id
    user_id
  }
}

// Thông Báo (Notifications)
Table notifications {
  notification_id varchar(36) [pk]
  user_id varchar(36) [ref: > users.user_id, not null]
  actor_id varchar(36) [ref: > users.user_id, note: 'Người thực hiện hành động']
  type enum('friend_request', 'friend_accept', 'comment', 'like', 'mention', 'recipe_share') [not null]
  target_type enum('post', 'recipe', 'comment', 'friend_request') [not null]
  target_id varchar(36) [not null]
  content text [note: 'Nội dung thông báo']
  is_read boolean [default: false]
  created_at timestamp [default: `now()`]

  indexes {
    user_id
    is_read
    created_at
    type
  }
}

// Người Theo Dõi (Followers) - Nếu muốn mô hình follow ngoài friendship
Table user_followers {
  follow_id varchar(36) [pk]
  follower_id varchar(36) [ref: > users.user_id, not null, note: 'Người theo dõi']
  following_id varchar(36) [ref: > users.user_id, not null, note: 'Người được theo dõi']
  created_at timestamp [default: `now()`]

  indexes {
    (follower_id, following_id) [unique]
    follower_id
    following_id
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

### Mẫu Truy Cập Chính (Enhanced)
1. **Lấy Hồ Sơ Người Dùng**: users.user_id → Dữ liệu người dùng + vai trò
2. **Lấy Tùy Chọn Người Dùng**: user_preferences.user_id → Tùy chọn + preferred_recipe_count
3. **Lấy Nguyên Liệu Người Dùng**: user_ingredients.user_id → Danh sách nguyên liệu
4. **Validate Nguyên Liệu**: master_ingredients.normalized_name → Kiểm tra hợp lệ
5. **Lấy Cài Đặt Quyền Riêng Tư**: privacy_settings.user_id → Cấu hình riêng tư
6. **Kiểm Tra Quan Hệ Bạn Bè**: friendships(user_id, friend_id) → Trạng thái bạn bè
7. **Lấy Danh Sách Bạn Bè**: friendships.user_id → Tất cả bạn bè
8. **Lấy Chi Tiết Công Thức**: recipes.recipe_id → Công thức + Nguyên liệu
9. **Lấy Công Thức Đã Duyệt**: recipes.is_approved=true → Công thức công khai
10. **Query Recipes by Category**: recipes.cooking_method + meal_type → Diverse suggestions ⭐
11. **Lấy Gợi Ý AI**: ai_suggestions.user_id + created_at → Gợi ý gần đây
12. **Lấy Lịch Sử Nấu Ăn**: user_cooking_history.user_id → Lịch sử cá nhân
13. **Lấy Món Yêu Thích**: user_cooking_history.user_id + is_favorite=true → Món yêu thích
14. **Lấy Đánh Giá Công Thức**: recipe_ratings.recipe_id → Tất cả rating
15. **Tính Toán Auto-Approval**: recipe_ratings.recipe_id + rating>=4 → Kiểm tra điều kiện
16. **Log Invalid Ingredients**: activity_logs.activity_type='invalid_ingredient' → CloudWatch ⭐
17. **Admin: Top Invalid Reports**: invalid_ingredients_reports.total_reports → Admin dashboard ⭐
18. **Lấy Bài Đăng Người Dùng**: posts.user_id + created_at → Danh sách bài đăng
19. **Lấy Newsfeed**: posts(bạn bè) + created_at → Bài đăng của bạn bè
20. **Lấy Bình Luận**: comments.post_id + created_at → Bình luận của bài đăng
21. **Lấy Reactions**: reactions(target_type, target_id) → Lượt thích/reactions
22. **Lấy Thông Báo**: notifications.user_id + is_read → Thông báo chưa đọc
23. **Lấy Followers**: user_followers.following_id → Danh sách người theo dõi

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

#### Lịch Sử Nấu Ăn
- **PK**: `USER#<user_id>`
- **SK**: `COOKING#<timestamp>#<history_id>`
- **Thuộc tính**: recipe_id, status, personal_rating, is_favorite, cook_date
- **GSI1PK**: `USER#<user_id>#FAVORITE` (nếu is_favorite=true)
- **GSI1SK**: `COOKING#<cook_date>`

#### Đánh Giá Công Thức
- **PK**: `RECIPE#<recipe_id>`
- **SK**: `RATING#<user_id>`
- **Thuộc tính**: rating, comment, is_verified_cook, history_id
- **GSI1PK**: `USER#<user_id>` (ratings của user)
- **GSI1SK**: `RATING#<created_at>`

#### Master Ingredients
- **PK**: `INGREDIENT#<ingredient_id>`
- **SK**: `METADATA`
- **Thuộc tính**: name, normalized_name, category, aliases
- **GSI1PK**: `CATEGORY#<category>`
- **GSI1SK**: `INGREDIENT#<name>`

#### Bài Đăng (Posts)
- **PK**: `POST#<post_id>`
- **SK**: `METADATA`
- **Thuộc tính**: user_id, recipe_id, content, images, is_public, likes_count, comments_count
- **GSI1PK**: `USER#<user_id>` (bài đăng của người dùng)
- **GSI1SK**: `POST#<created_at>`

#### Bình Luận (Comments)
- **PK**: `POST#<post_id>`
- **SK**: `COMMENT#<timestamp>#<comment_id>`
- **Thuộc tính**: user_id, parent_comment_id, content
- **GSI1PK**: `USER#<user_id>` (bình luận của người dùng)
- **GSI1SK**: `COMMENT#<timestamp>`

#### Reactions (Lượt thích)
- **PK**: `<target_type>#<target_id>` (ví dụ: POST#123, RECIPE#456)
- **SK**: `REACTION#<user_id>`
- **Thuộc tính**: reaction_type, created_at
- **GSI1PK**: `USER#<user_id>` (reactions của người dùng)
- **GSI1SK**: `REACTION#<created_at>`

#### Thông Báo (Notifications)
- **PK**: `USER#<user_id>`
- **SK**: `NOTIFICATION#<timestamp>#<notification_id>`
- **Thuộc tính**: actor_id, type, target_type, target_id, content, is_read
- **GSI1PK**: `USER#<user_id>#UNREAD` (nếu is_read=false)
- **GSI1SK**: `NOTIFICATION#<timestamp>`

#### Followers
- **PK**: `USER#<following_id>`
- **SK**: `FOLLOWER#<follower_id>`
- **Thuộc tính**: created_at
- **GSI1PK**: `USER#<follower_id>` (tra cứu ngược - ai người này đang follow)
- **GSI1SK**: `FOLLOWING#<following_id>`

### Chỉ Mục Thứ Cấp Toàn Cục (GSI)

**GSI1**: Truy vấn dựa trên người dùng
- **PK**: GSI1PK (ví dụ: `ROLE#admin`, `USER#<user_id>`)
- **SK**: GSI1SK (ví dụ: timestamp, `RECIPE#<created_at>`)
- **Trường hợp sử dụng**:
  - Lấy tất cả quản trị viên
  - Lấy công thức của người dùng sắp xếp theo ngày
  - Lấy quan hệ bạn bè (tra cứu ngược)
  - Lấy bài đăng của người dùng
  - Lấy bình luận của người dùng
  - Lấy reactions của người dùng
  - Lấy thông báo chưa đọc
  - Lấy danh sách following

**GSI2**: Tìm kiếm & khám phá công thức (Enhanced)
- **PK**: GSI2PK (ví dụ: `CUISINE#<type>`, `METHOD#<cooking_method>`, `MEALTYPE#<meal_type>`) ⭐
- **SK**: GSI2SK (ví dụ: `RECIPE#<created_at>`)
- **Trường hợp sử dụng**:
  - Tìm kiếm công thức theo món ăn
  - Tìm kiếm công thức theo phương pháp nấu (xào, hấp, canh...) ⭐
  - Tìm kiếm theo loại món (món chính, món phụ...) ⭐
  - Lấy công thức phổ biến
  - Diverse recipe suggestions ⭐

**GSI3**: Social Feed & Discovery
- **PK**: GSI3PK (ví dụ: `FEED#<user_id>`, `PUBLIC`)
- **SK**: GSI3SK (ví dụ: `POST#<created_at>`)
- **Trường hợp sử dụng**:
  - Lấy newsfeed (bài đăng của bạn bè)
  - Lấy bài đăng công khai (explore feed)
  - Lấy bài đăng trending

## Ước Tính Dung Lượng Lưu Trữ & Chi Phí

### Giả Định
- 1.000 người dùng hoạt động
- Trung bình 20 nguyên liệu mỗi người dùng
- Trung bình 5 công thức đã lưu mỗi người dùng
- Trung bình 10 gợi ý AI mỗi người dùng mỗi tháng
- Trung bình 10 bài đăng mỗi người dùng
- Trung bình 30 bình luận mỗi người dùng
- Trung bình 100 reactions mỗi người dùng
- Trung bình 50 thông báo mỗi người dùng
- Trung bình 20 followers mỗi người dùng

### Ước Tính Dung Lượng (Enhanced)
- Người dùng (with preferences): 1.000 × 1.2KB = 1.2MB
- Nguyên liệu người dùng: 1.000 × 20 × 0.5KB = 10MB
- Master ingredients: 5.000 × 0.8KB = 4MB
- Công thức (with categories): 5.000 × 5.5KB = 27.5MB ⭐
- Lịch sử nấu ăn: 1.000 × 50 × 0.8KB = 40MB
- Đánh giá công thức: 10.000 × 0.5KB = 5MB
- Gợi ý AI (enhanced): 10.000 × 2.5KB = 25MB ⭐
- Invalid reports (optional): 500 × 0.5KB = 0.25MB ⭐
- Bài đăng: 10.000 × 3KB = 30MB
- Bình luận: 30.000 × 0.5KB = 15MB
- Reactions: 100.000 × 0.2KB = 20MB
- Thông báo: 50.000 × 0.5KB = 25MB
- Followers: 20.000 × 0.3KB = 6MB
- **Tổng cộng: ~209MB**

### Chi Phí DynamoDB (Hàng Tháng) - Enhanced
- Lưu trữ: 209MB × $0.25/GB = ~$0.05
- Khả năng đọc: ~$15-20 (tăng do category queries) ⭐
- Khả năng ghi: ~$15-20 (tăng do invalid logging, enhanced suggestions) ⭐
- GSI (4 indexes): ~$10-15 (tăng do GSI2 category index) ⭐
- **Tổng cộng: ~$40-55/tháng** (vs $32-48 trước đó)
