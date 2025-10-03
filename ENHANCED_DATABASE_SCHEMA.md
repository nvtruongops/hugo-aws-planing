# DATABASE SCHEMA - ENHANCED VERSION

##    Cập Nhật Schema Cho Features Mới

### THAY ĐỔI 1: Thêm field vào `users`

```sql
Table users {
  user_id varchar(36) [pk]
  email varchar(255) [unique]
  username varchar(50)
  full_name varchar(100)
  role enum('user', 'admin') [default: 'user']

  -- NEW: Preferences
  preferred_recipe_count int [default: 1, note: '1-5 món']
  preferred_cooking_methods json [note: 'Array: ["xào", "hấp", "canh"]']

  created_at timestamp
}
```

**DynamoDB**:
```javascript
{
  PK: "USER#user-123",
  SK: "PROFILE",
  email: "user@example.com",
  preferred_recipe_count: 3,  // NEW
  preferred_cooking_methods: ["xào", "canh", "hấp"],  // NEW
  created_at: "2025-10-03"
}
```

---

### THAY ĐỔI 2: Thêm categories vào `recipes`

```sql
Table recipes {
  recipe_id varchar(36) [pk]
  user_id varchar(36) [ref: > users.user_id]
  title varchar(200)
  description text

  -- NEW: Categories
  cooking_method varchar(50) [note: 'xào, hấp, luộc, chiên, nướng, kho, canh, trộn']
  meal_type varchar(50) [note: 'món chính, món phụ, canh, khai vị']

  ingredients json
  instructions json
  prep_time int [note: 'phút']
  cook_time int [note: 'phút']
  servings int [note: 'số người ăn']

  is_ai_generated boolean [default: true]
  is_favorite boolean [default: false]
  is_approved boolean [default: false]
  average_rating decimal(2,1)

  created_at timestamp
}
```

**DynamoDB với GSI**:
```javascript
// Recipe item
{
  PK: "RECIPE#recipe-123",
  SK: "METADATA",
  title: "Gà xào sả ớt",
  cooking_method: "xào",  // NEW
  meal_type: "món chính",  // NEW

  // GSI2 for category search
  GSI2PK: "METHOD#xào",
  GSI2SK: "RECIPE#2025-10-03"
}

// Query by cooking method
const xaoRecipes = await ddb.query({
  IndexName: 'GSI2',
  KeyConditionExpression: 'GSI2PK = :method',
  ExpressionAttributeValues: {
    ':method': 'METHOD#xào'
  }
});
```

---

### THAY ĐỔI 3: Thêm `invalid_ingredients_reports` (Optional - Post MVP)

```sql
Table invalid_ingredients_reports {
  report_id varchar(36) [pk]
  ingredient_name varchar(256) [not null]
  user_id varchar(36) [ref: > users.user_id]
  report_count int [default: 1, note: 'Số lần user này report']
  total_reports int [default: 1, note: 'Tổng reports từ tất cả users']
  status enum('pending', 'reviewed', 'added_to_master', 'spam') [default: 'pending']
  admin_notes text
  created_at timestamp
  updated_at timestamp

  indexes {
    (ingredient_name, user_id) [unique]
    total_reports [note: 'Để query top invalid ingredients']
    status
  }
}
```

**DynamoDB Design**:
```javascript
// Report item
{
  PK: "INVALID#thịt_gấu_trúc",  // normalized ingredient name
  SK: "REPORT#user-123",
  report_id: "report-001",
  ingredient_name: "thịt gấu trúc",
  user_id: "user-123",
  user_report_count: 3,  // User này đã report 3 lần
  created_at: "2025-10-03"
}

// Aggregate count (for admin dashboard)
{
  PK: "INVALID#thịt_gấu_trúc",
  SK: "AGGREGATE",
  total_reports: 47,  // Tổng 47 users đã report
  status: "pending",
  first_reported: "2025-09-01",
  last_reported: "2025-10-03"
}

// Query: Top invalid ingredients
await ddb.query({
  IndexName: 'GSI1',
  KeyConditionExpression: 'GSI1PK = :status',
  ExpressionAttributeValues: {
    ':status': 'STATUS#pending'
  },
  ScanIndexForward: false,  // Descending by total_reports
  Limit: 20
});
```

---

### THAY ĐỔI 4: Cập nhật `ai_suggestions`

```sql
Table ai_suggestions {
  suggestion_id varchar(36) [pk]
  user_id varchar(36) [ref: > users.user_id]

  -- Request info
  prompt_ingredients json
  requested_count int [default: 1, note: '1-5 món']
  requested_categories json [note: 'User chọn loại món']

  -- Response info
  ai_response json
  recipes_from_db int [default: 0, note: 'Số món từ DB']
  recipes_from_ai int [default: 0, note: 'Số món AI tạo']

  -- Validation warnings
  invalid_ingredients json [note: 'Nguyên liệu không hợp lệ']

  created_at timestamp
}
```

**DynamoDB**:
```javascript
{
  PK: "USER#user-123",
  SK: "SUGGESTION#2025-10-03T10:30:00",
  suggestion_id: "sugg-001",

  // Request
  prompt_ingredients: ["thịt gà", "cà chua", "xyz123"],
  requested_count: 3,

  // Response
  recipes_from_db: 1,
  recipes_from_ai: 2,

  // Validation
  invalid_ingredients: ["xyz123"],

  created_at: "2025-10-03T10:30:00"
}
```

---

##    Storage Estimates (Updated)

| Item | Count | Size | Total |
|------|-------|------|-------|
| Users (with preferences) | 1,000 | 1.2 KB | 1.2 MB |
| Ingredients | 20,000 | 0.3 KB | 6 MB |
| Recipes (with categories) | 5,000 | 3.5 KB | 17.5 MB |
| AI Suggestions (enhanced) | 10,000 | 2.5 KB | 25 MB |
| Invalid Reports (optional) | 500 | 0.5 KB | 0.25 MB |
| **TOTAL** | | | **~50 MB** |

**DynamoDB Cost**: ~$12-18/month (tăng ~$2-3 so với $10-15)

---

##    GSI Design Summary

### GSI1: User-based queries
- **PK**: `USER#<user_id>` hoặc `ROLE#<role>`
- **SK**: Various (RECIPE#, SUGGESTION#, etc.)
- **Use cases**: User's recipes, suggestions, role filtering

### GSI2: Category-based queries (NEW)
- **PK**: `METHOD#<cooking_method>` hoặc `MEALTYPE#<meal_type>`
- **SK**: `RECIPE#<timestamp>`
- **Use cases**: Find recipes by cooking method/meal type

### GSI3: Invalid ingredients tracking (Optional)
- **PK**: `STATUS#<status>`
- **SK**: `TOTALREPORTS#<count>#<ingredient>`
- **Use cases**: Admin dashboard - top reported ingredients

---

## ✅ Migration Plan (Nếu đã có data)

### Step 1: Add new fields (non-breaking)
```javascript
// User preferences - set default
await ddb.update({
  TableName: 'smart-cooking-mvp',
  Key: { PK: 'USER#user-123', SK: 'PROFILE' },
  UpdateExpression: 'SET preferred_recipe_count = :count, preferred_cooking_methods = :methods',
  ExpressionAttributeValues: {
    ':count': 1,
    ':methods': []
  }
});
```

### Step 2: Backfill recipe categories (AI-powered)
```javascript
// Dùng AI để classify existing recipes
for (const recipe of existingRecipes) {
  const category = await classifyRecipe(recipe.title, recipe.instructions);

  await ddb.update({
    Key: { PK: `RECIPE#${recipe.id}`, SK: 'METADATA' },
    UpdateExpression: 'SET cooking_method = :method, meal_type = :type',
    ExpressionAttributeValues: {
      ':method': category.cooking_method,
      ':type': category.meal_type
    }
  });
}
```

### Step 3: Create GSI2 (category index)
```javascript
// CloudFormation / Terraform
await ddb.updateTable({
  TableName: 'smart-cooking-mvp',
  GlobalSecondaryIndexUpdates: [{
    Create: {
      IndexName: 'GSI2',
      KeySchema: [
        { AttributeName: 'GSI2PK', KeyType: 'HASH' },
        { AttributeName: 'GSI2SK', KeyType: 'RANGE' }
      ],
      Projection: { ProjectionType: 'ALL' }
    }
  }]
});
```

---

##    Kết luận

**Schema updates**:
- ✅ Minimal changes (backward compatible)
- ✅ Cost tăng nhẹ (+$2-3/tháng)
- ✅ Enable tất cả 4 features mới
- ✅ Có thể deploy incremental (không cần migration lớn)
