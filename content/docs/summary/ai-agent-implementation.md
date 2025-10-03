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
- ✅ User nhập danh sách nguyên liệu
- ✅ System check nguyên liệu hợp lệ
- ✅ Gợi ý 4 món từ database + 1 món AI tạo mới
- ✅ Auto-approval công thức dựa trên rating (>= 4 sao)
- ✅ Error handling cho nguyên liệu không hợp lệ

**Thời gian**: 2 tuần (Tuần 3-4 trong deployment plan)

## 🎯 LUỒNG NGƯỜI DÙNG

### Flow 1: Gợi Ý Món Ăn Thông Minh (Smart Suggestion)

```
┌─────────────────────────────────────────────────────────┐
│                    ĐẦU VÀO NGƯỜI DÙNG                   │
├─────────────────────────────────────────────────────────┤
│  Người dùng A:                                          │
│  • Nam, 1990 (34 tuổi), Vietnam                        │
│  • Thích ăn: canh, món hấp                             │
│  • Nguyên liệu nhập: Cá rô, bông súng, gừng, hành     │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│              VALIDATE & PROCESS                         │
├─────────────────────────────────────────────────────────┤
│  1. Check nguyên liệu hợp lệ:                          │
│     ✅ "Cá rô" → Có trong database                      │
│     ✅ "Bông súng" → Có trong database                  │
│     ✅ "Gừng" → Có trong database                       │
│     ✅ "Hành" → Có trong database                       │
│                                                         │
│  2. Tìm công thức:                                     │
│     - Query DynamoDB: 4 món match nguyên liệu          │
│     - Gọi AI Bedrock: 1 món mới sáng tạo               │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│              KẾT QUẢ GỢI Ý (5 MÓN)                     │
├─────────────────────────────────────────────────────────┤
│  📦 Từ Database (4 món):                               │
│  1. Canh cá rô nấu bông súng                           │
│  2. Cá rô kho gừng                                      │
│  3. Canh bông súng                                      │
│  4. Cá rô chiên                                         │
│                                                         │
│  🤖 AI Generated (1 món mới):                          │
│  5. Cá rô hấp bông súng gừng hành                      │
│     (Món độc đáo từ AI)                                │
└─────────────────────────────────────────────────────────┘
```

### Flow 2: Auto-Approval Công Thức (Rating-based)

```
┌─────────────────────────────────────────────────────────┐
│              USER NẤU MÓN AI TẠO                        │
├─────────────────────────────────────────────────────────┤
│  1. User chọn món: "Cá rô hấp bông súng gừng hành"    │
│  2. Làm theo hướng dẫn                                 │
│  3. Click "Hoàn thành"                                 │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│              ĐÁNH GIÁ CÔNG THỨC                         │
├─────────────────────────────────────────────────────────┤
│  Popup hiển thị:                                        │
│  • Rating: ⭐⭐⭐⭐⭐ (1-5 sao)                           │
│  • Comment: "Món này rất ngon!"                        │
│  • Submit                                               │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│              XỬ LÝ RATING                               │
├─────────────────────────────────────────────────────────┤
│  IF rating >= 4 sao:                                   │
│    ✅ Set recipe.is_approved = true                     │
│    ✅ Add to public recipes database                    │
│    ✅ Notify: "Công thức đã được thêm vào database!"   │
│                                                         │
│  ELSE (rating < 4 sao):                                │
│    ℹ️ Notify: "Cảm ơn đánh giá của bạn!"              │
│    ❌ Không thêm vào database                           │
└─────────────────────────────────────────────────────────┘
```

### Flow 3: Error Handling - Nguyên Liệu Không Hợp Lệ

```
┌─────────────────────────────────────────────────────────┐
│              USER NHẬP NGUYÊN LIỆU SAI                  │
├─────────────────────────────────────────────────────────┤
│  Input: "abc xyz", "123", "thit ga"                    │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│              VALIDATE TỪNG NGUYÊN LIỆU                  │
├─────────────────────────────────────────────────────────┤
│  1. "abc xyz" → ❌ Không tồn tại                        │
│     → Fuzzy search: Tìm nguyên liệu gần đúng           │
│     → Gợi ý: "Bạn có muốn: [gà, cá, bò]?"             │
│                                                         │
│  2. "123" → ❌ Format không hợp lệ                      │
│     → Error: "Tên nguyên liệu không hợp lệ"            │
│                                                         │
│  3. "thit ga" → ✅ Fuzzy match: "thịt gà"              │
│     → Auto-correct: "Bạn muốn dùng 'thịt gà'?"         │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│              ERROR RESPONSES                            │
├─────────────────────────────────────────────────────────┤
│  Case 1: Nguyên liệu không tồn tại                     │
│  → HTTP 400: {                                          │
│      "error": "ingredient_not_found",                   │
│      "message": "Không tìm thấy nguyên liệu: abc xyz", │
│      "suggestions": ["gà", "cá", "bò"]                 │
│    }                                                    │
│                                                         │
│  Case 2: Format không hợp lệ                           │
│  → HTTP 400: {                                          │
│      "error": "invalid_format",                         │
│      "message": "Tên nguyên liệu không hợp lệ: 123"   │
│    }                                                    │
│                                                         │
│  Case 3: Quá ít nguyên liệu                            │
│  → HTTP 400: {                                          │
│      "error": "insufficient_ingredients",               │
│      "message": "Cần ít nhất 2 nguyên liệu"           │
│    }                                                    │
│                                                         │
│  Case 4: Fuzzy match thành công                        │
│  → HTTP 200: {                                          │
│      "corrected": true,                                 │
│      "original": "thit ga",                             │
│      "matched": "thịt gà",                              │
│      "confidence": 0.85                                 │
│    }                                                    │
└─────────────────────────────────────────────────────────┘
```

## 🏗️ TRIỂN KHAI KỸ THUẬT

### Backend: Node.js 20 Lambda Functions

**Lambda AI Suggestion Engine:**
```javascript
// Sử dụng AWS SDK v3 cho Bedrock
const { BedrockRuntimeClient, InvokeModelCommand } = require('@aws-sdk/client-bedrock-runtime');
const { DynamoDBClient } = require('@aws-sdk/client-dynamodb');
const { DynamoDBDocumentClient, QueryCommand, PutCommand } = require('@aws-sdk/lib-dynamodb');

const ddb = DynamoDBDocumentClient.from(new DynamoDBClient({}));

// Flow mới:
// 1. Validate ingredients với master_ingredients table
// 2. Query 4 recipes từ DynamoDB (is_approved=true)
// 3. Generate 1 recipe mới bằng Bedrock AI
// 4. Return 5 suggestions (4 DB + 1 AI)

async function suggestRecipes(userIngredients, userPreferences) {
  // Step 1: Validate ingredients với master_ingredients
  const validatedIngredients = await validateIngredientsWithMaster(userIngredients);

  if (!validatedIngredients.isValid) {
    return {
      statusCode: 400,
      body: {
        error: 'ingredient_not_found',
        message: 'Một số nguyên liệu không hợp lệ',
        invalid_ingredients: validatedIngredients.invalidItems,
        suggestions: validatedIngredients.suggestions
      }
    };
  }

  // Step 2: Query 4 approved recipes từ DynamoDB
  const dbRecipes = await queryApprovedRecipesByIngredients(
    validatedIngredients.ingredients,
    4
  );

  // Step 3: Generate 1 AI recipe
  const aiRecipe = await generateAIRecipe(
    validatedIngredients.ingredients,
    userPreferences
  );

  // Step 4: Save AI suggestion to history
  await saveAISuggestion({
    userId: userPreferences.userId,
    ingredients: validatedIngredients.ingredients,
    aiRecipe,
    dbRecipes
  });

  // Step 5: Return combined results
  return {
    statusCode: 200,
    body: {
      recipes: [...dbRecipes, aiRecipe],
      stats: {
        database_recipes: dbRecipes.length,
        ai_recipes: 1
      }
    }
  };
}

// Validate ingredients với master_ingredients table
async function validateIngredientsWithMaster(ingredients) {
  const validated = [];
  const invalid = [];
  const suggestions = [];

  for (const ing of ingredients) {
    // Normalize input
    const normalized = normalizeText(ing);

    // Check exact match trong master_ingredients
    const exactMatch = await ddb.send(new QueryCommand({
      TableName: 'smart-cooking-data',
      IndexName: 'GSI1',
      KeyConditionExpression: 'GSI1PK = :pk AND begins_with(GSI1SK, :sk)',
      ExpressionAttributeValues: {
        ':pk': `INGREDIENT#SEARCH`,
        ':sk': `NAME#${normalized}`
      }
    }));

    if (exactMatch.Items && exactMatch.Items.length > 0) {
      validated.push(exactMatch.Items[0].name);
    } else {
      // Fuzzy search for similar ingredients
      const similar = await fuzzySearchIngredients(normalized);
      invalid.push(ing);
      suggestions.push({
        original: ing,
        similar: similar.slice(0, 5) // Top 5 suggestions
      });
    }
  }

  return {
    isValid: invalid.length === 0,
    ingredients: validated,
    invalidItems: invalid,
    suggestions
  };
}

// Fuzzy search cho nguyên liệu tương tự
async function fuzzySearchIngredients(searchTerm) {
  // Scan master_ingredients với filter
  const result = await ddb.send(new QueryCommand({
    TableName: 'smart-cooking-data',
    IndexName: 'GSI1',
    KeyConditionExpression: 'GSI1PK = :pk',
    FilterExpression: 'contains(normalized_name, :term) OR contains(aliases, :term)',
    ExpressionAttributeValues: {
      ':pk': `INGREDIENT#ACTIVE`,
      ':term': searchTerm
    }
  }));

  return result.Items.map(item => item.name);
}

// Query approved recipes matching ingredients
async function queryApprovedRecipesByIngredients(ingredients, limit) {
  const recipes = await ddb.send(new QueryCommand({
    TableName: 'smart-cooking-data',
    IndexName: 'GSI2',
    KeyConditionExpression: 'GSI2PK = :pk',
    FilterExpression: 'is_approved = :approved',
    ExpressionAttributeValues: {
      ':pk': `RECIPES#APPROVED`,
      ':approved': true
    },
    Limit: limit,
    ScanIndexForward: false // Sắp xếp theo rating cao nhất
  }));

  return recipes.Items || [];
}

// Normalize text (bỏ dấu, lowercase)
function normalizeText(text) {
  return text
    .toLowerCase()
    .normalize('NFD')
    .replace(/[\u0300-\u036f]/g, '')
    .trim();
}
```

**Lambda Recipe Rating Handler:**
```javascript
// Auto-approval dựa trên rating (>= 4 sao)
async function handleRecipeRating(recipeId, userId, rating, comment, historyId) {
  // Validate rating (1-5)
  if (rating < 1 || rating > 5) {
    return {
      statusCode: 400,
      body: { error: 'Invalid rating. Must be 1-5.' }
    };
  }

  // Step 1: Save rating to recipe_ratings table
  await ddb.send(new PutCommand({
    TableName: 'smart-cooking-data',
    Item: {
      PK: `RECIPE#${recipeId}`,
      SK: `RATING#${userId}`,
      rating_id: generateUUID(),
      recipe_id: recipeId,
      user_id: userId,
      rating,
      comment,
      history_id: historyId,
      is_verified_cook: historyId ? true : false,
      created_at: new Date().toISOString(),
      updated_at: new Date().toISOString()
    }
  }));

  // Step 2: Update user_cooking_history với personal_rating
  if (historyId) {
    await ddb.send(new PutCommand({
      TableName: 'smart-cooking-data',
      Item: {
        PK: `USER#${userId}`,
        SK: `COOKING#${historyId}`,
        personal_rating: rating,
        updated_at: new Date().toISOString()
      }
    }));
  }

  // Step 3: Calculate average rating cho recipe
  const avgRating = await calculateAverageRating(recipeId);
  const ratingCount = await getRatingCount(recipeId);

  // Step 4: Auto-approve if >= 4.0 stars
  if (avgRating >= 4.0) {
    await ddb.send(new PutCommand({
      TableName: 'smart-cooking-data',
      Item: {
        PK: `RECIPE#${recipeId}`,
        SK: 'METADATA',
        is_approved: true,
        is_public: true,
        approval_type: 'auto_rating',
        average_rating: avgRating,
        rating_count: ratingCount,
        approved_at: new Date().toISOString(),
        updated_at: new Date().toISOString()
      }
    }));

    return {
      statusCode: 200,
      body: {
        success: true,
        rating_saved: true,
        average_rating: avgRating,
        rating_count: ratingCount,
        auto_approved: true,
        message: 'Công thức đã được thêm vào database tổng!'
      }
    };
  }

  // Step 5: Rating < 4.0 - chỉ cảm ơn
  return {
    statusCode: 200,
    body: {
      success: true,
      rating_saved: true,
      average_rating: avgRating,
      rating_count: ratingCount,
      auto_approved: false,
      message: 'Cảm ơn đánh giá của bạn!'
    }
  };
}

// Calculate average rating
async function calculateAverageRating(recipeId) {
  const result = await ddb.send(new QueryCommand({
    TableName: 'smart-cooking-data',
    KeyConditionExpression: 'PK = :pk AND begins_with(SK, :sk)',
    ExpressionAttributeValues: {
      ':pk': `RECIPE#${recipeId}`,
      ':sk': 'RATING#'
    }
  }));

  if (!result.Items || result.Items.length === 0) return 0;

  const total = result.Items.reduce((sum, item) => sum + item.rating, 0);
  return (total / result.Items.length).toFixed(1);
}

// Get rating count
async function getRatingCount(recipeId) {
  const result = await ddb.send(new QueryCommand({
    TableName: 'smart-cooking-data',
    KeyConditionExpression: 'PK = :pk AND begins_with(SK, :sk)',
    ExpressionAttributeValues: {
      ':pk': `RECIPE#${recipeId}`,
      ':sk': 'RATING#'
    },
    Select: 'COUNT'
  }));

  return result.Count || 0;
}
```

**Lambda Cooking History Handler:**
```javascript
// Quản lý lịch sử nấu ăn cá nhân
async function startCooking(userId, recipeId, suggestionId) {
  const historyId = generateUUID();

  await ddb.send(new PutCommand({
    TableName: 'smart-cooking-data',
    Item: {
      PK: `USER#${userId}`,
      SK: `COOKING#${new Date().toISOString()}#${historyId}`,
      history_id: historyId,
      user_id: userId,
      recipe_id: recipeId,
      suggestion_id: suggestionId,
      status: 'cooking',
      created_at: new Date().toISOString(),
      updated_at: new Date().toISOString()
    }
  }));

  return {
    statusCode: 200,
    body: {
      history_id: historyId,
      message: 'Bắt đầu nấu ăn!'
    }
  };
}

// Hoàn thành nấu ăn
async function completeCooking(historyId, userId) {
  await ddb.send(new PutCommand({
    TableName: 'smart-cooking-data',
    Item: {
      PK: `USER#${userId}`,
      SK: `COOKING#${historyId}`,
      status: 'completed',
      cook_date: new Date().toISOString(),
      updated_at: new Date().toISOString()
    }
  }));

  return {
    statusCode: 200,
    body: {
      message: 'Hoàn thành! Vui lòng đánh giá món ăn.',
      show_rating_form: true
    }
  };
}

// Lấy lịch sử nấu ăn
async function getCookingHistory(userId) {
  const result = await ddb.send(new QueryCommand({
    TableName: 'smart-cooking-data',
    KeyConditionExpression: 'PK = :pk AND begins_with(SK, :sk)',
    ExpressionAttributeValues: {
      ':pk': `USER#${userId}`,
      ':sk': 'COOKING#'
    },
    ScanIndexForward: false // Newest first
  }));

  return {
    statusCode: 200,
    body: {
      history: result.Items || []
    }
  };
}
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
- `Recipes` (PK: recipe_id) - Công thức (có thêm field is_approved, approval_type, average_rating)
- `RecipeRatings` (PK: recipe_id, SK: user_id) - Đánh giá công thức cho auto-approval
- `UserCookingHistory` (PK: user_id, SK: timestamp) - Lịch sử nấu ăn cá nhân (thay thế favorites)
- `MasterIngredients` (PK: ingredient_id) - Master list nguyên liệu để validate
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

**Luồng Gợi Ý AI (Enhanced - Flexible Mix):**
```
POST /ai/suggest
Authorization: Bearer <JWT>
Body: {
  "ingredients": ["thịt gà", "cà chua", "hành"],
  "recipe_count": 3  // NEW: 1-5 món (default: 1)
}

Response (Success - Flexible mix):
{
  "suggestions": [
    {
      "id": "recipe-001",
      "name": "Gà xào cà chua",
      "source": "database",
      "cooking_method": "xào",  // NEW
      "meal_type": "món chính",  // NEW
      "match_score": 0.95,
      "is_approved": true,
      "average_rating": 4.5
    },
    {
      "id": "recipe-002",
      "name": "Canh cà chua",
      "source": "database",
      "cooking_method": "canh",  // NEW - Diverse!
      "meal_type": "canh",
      "match_score": 0.80,
      "is_approved": true,
      "average_rating": 4.2
    },
    {
      "id": "ai-gen-001",
      "name": "Gà hấp cà chua hành",
      "source": "ai",
      "cooking_method": "hấp",  // NEW - Diverse!
      "meal_type": "món chính",
      "is_new": true,
      "is_approved": false,
      "ingredients": [...],
      "instructions": [...]
    }
  ],
  "stats": {
    "requested": 3,  // NEW
    "from_database": 2,  // NEW - Flexible!
    "from_ai": 1  // NEW - Only 1 AI call needed!
  },
  "warnings": []  // NEW - Empty if all ingredients valid
}

Response (Warning - Invalid ingredient but still work):
{
  "suggestions": [...],  // Still return results
  "stats": {
    "requested": 3,
    "from_database": 0,  // Not enough valid ingredients for DB
    "from_ai": 3  // AI handles it
  },
  "warnings": [  // NEW
    {
      "ingredient": "abc xyz",
      "message": "Nguyên liệu không hợp lệ, AI sẽ cố gắng xử lý",
      "suggestions": ["gà", "cá", "bò"],
      "reported": true  // Đã log để admin review
    }
  ]
}

Response (Error - All invalid):
{
  "error": "all_ingredients_invalid",
  "message": "Tất cả nguyên liệu không hợp lệ. Vui lòng nhập lại.",
  "invalid_ingredients": ["abc", "xyz", "123"],
  "suggestions": [
    { "original": "abc", "similar": ["cá", "gà"] },
    { "original": "xyz", "similar": ["rau", "củ"] }
  ]
}
```

**Luồng Cooking History:**
```
POST /cooking/start
Authorization: Bearer <JWT>
Body: {
  "recipe_id": "ai-gen-001",
  "suggestion_id": "suggestion-123"
}

Response:
{
  "history_id": "history-001",
  "message": "Bắt đầu nấu ăn!"
}

PUT /cooking/{history_id}/complete
Authorization: Bearer <JWT>

Response:
{
  "message": "Hoàn thành! Vui lòng đánh giá món ăn.",
  "show_rating_form": true
}

GET /user/cooking-history
Authorization: Bearer <JWT>

Response:
{
  "history": [
    {
      "history_id": "history-001",
      "recipe_id": "ai-gen-001",
      "status": "completed",
      "personal_rating": 5,
      "is_favorite": true,
      "cook_date": "2025-10-03T10:30:00Z"
    },
    ...
  ]
}
```

**Luồng Rating & Auto-Approval:**
```
POST /recipes/{id}/rate
Authorization: Bearer <JWT>
Body: {
  "rating": 5,
  "comment": "Món này rất ngon!",
  "history_id": "history-001"
}

Response (Auto-approved):
{
  "success": true,
  "rating_saved": true,
  "average_rating": 4.2,
  "rating_count": 15,
  "auto_approved": true,
  "message": "Công thức đã được thêm vào database!",
  "recipe": {
    "id": "ai-gen-001",
    "is_approved": true,
    "is_public": true,
    "approval_type": "auto_rating"
  }
}

Response (Not approved):
{
  "success": true,
  "rating_saved": true,
  "average_rating": 3.5,
  "rating_count": 8,
  "auto_approved": false,
  "message": "Cảm ơn đánh giá của bạn!"
}
```

**Luồng Validate Ingredients:**
```
POST /ingredients/validate
Authorization: Bearer <JWT>
Body: {
  "ingredients": ["cá rô", "abc xyz", "thit ga"]
}

Response:
{
  "valid": ["cá rô"],
  "invalid": ["abc xyz"],
  "corrected": [
    {
      "original": "thit ga",
      "matched": "thịt gà",
      "confidence": 0.85
    }
  ],
  "suggestions": [
    {
      "original": "abc xyz",
      "similar": ["gà", "cá", "bò", "tôm", "mực"]
    }
  ]
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
GET /admin/recipes/pending - Công thức chờ approval (ít sử dụng do auto-approval)
```

**Endpoints Mới - Social Features:**
```
POST /posts - Tạo bài đăng
GET /posts/feed - Lấy newsfeed
POST /posts/{id}/comments - Bình luận
POST /reactions - Thêm reaction
GET /notifications - Lấy thông báo
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
