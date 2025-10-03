# DATABASE SCHEMA - MVP SIMPLIFIED

## 🎯 Chỉ 4 Bảng Cần Thiết

### 1. users (Tài khoản)
```sql
Table users {
  user_id varchar(36) [pk]
  email varchar(255) [unique]
  username varchar(50)
  full_name varchar(100)
  created_at timestamp
}
```

### 2. user_ingredients (Nguyên liệu)
```sql
Table user_ingredients {
  id varchar(36) [pk]
  user_id varchar(36) [ref: > users.user_id]
  ingredient_name varchar(256)
  added_at timestamp
}
```

### 3. recipes (Công thức - AI generated)
```sql
Table recipes {
  recipe_id varchar(36) [pk]
  user_id varchar(36) [ref: > users.user_id]
  title varchar(200)
  description text
  ingredients json
  instructions json
  is_ai_generated boolean [default: true]
  is_favorite boolean [default: false]
  created_at timestamp
}
```

### 4. ai_suggestions (Lịch sử AI)
```sql
Table ai_suggestions {
  suggestion_id varchar(36) [pk]
  user_id varchar(36) [ref: > users.user_id]
  prompt_ingredients json
  ai_response json
  created_at timestamp
}
```

---

## 💾 DynamoDB Design (Single Table)

**Table Name**: `smart-cooking-mvp`

### Access Patterns

#### Users
- **PK**: `USER#<user_id>`
- **SK**: `PROFILE`

#### User Ingredients
- **PK**: `USER#<user_id>`
- **SK**: `INGREDIENT#<ingredient_name>`

#### Recipes (Favorites)
- **PK**: `USER#<user_id>`
- **SK**: `RECIPE#<timestamp>#<recipe_id>`

#### AI Suggestions
- **PK**: `USER#<user_id>`
- **SK**: `SUGGESTION#<timestamp>`

---

## 📊 Storage Estimates (1,000 users)

| Item | Count | Size | Total |
|------|-------|------|-------|
| Users | 1,000 | 1 KB | 1 MB |
| Ingredients | 20,000 (20/user) | 0.3 KB | 6 MB |
| Recipes | 5,000 (5/user) | 3 KB | 15 MB |
| AI Suggestions | 10,000 (10/user) | 2 KB | 20 MB |
| **TOTAL** | | | **~42 MB** |

**DynamoDB Cost**: ~$10-15/month (1,000 users)

---

## ✅ Ưu điểm so với schema phức tạp

- 🚀 **Nhanh**: Deploy trong 2 tuần thay vì 2 tháng
- 💰 **Rẻ**: $60-80/tháng thay vì $133-168
- 🧪 **Đơn giản**: Dễ demo, dễ debug
- 📊 **Đủ**: Chứng minh được AI suggestion core feature
