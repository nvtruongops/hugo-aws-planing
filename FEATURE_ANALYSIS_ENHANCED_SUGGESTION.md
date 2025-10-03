# PHÂN TÍCH YÊU CẦU MỚI - ENHANCED AI SUGGESTION

##    4 Yêu Cầu Chính

### 1. ✅ Số Món Muốn Nấu (1-5 món)
**Đánh giá**: EXCELLENT - Cải thiện UX đáng kể

**Lý do nên làm**:
- ✅ User muốn nấu nhiều món cho bữa ăn gia đình
- ✅ Tăng engagement (nhiều suggestions = nhiều interactions)
- ✅ Tăng Bedrock usage nhưng vẫn control được (max 5)

**Implementation**:
```javascript
// User preferences
{
  preferred_recipe_count: 3,  // Default: 1, Max: 5
  last_updated: timestamp
}

// API Request
POST /ai/suggest
{
  "ingredients": [...],
  "recipe_count": 3  // Optional, default 1
}
```

**Trade-off**:
- ⚠️ Chi phí AI tăng 3-5x nếu user request nhiều món
- ✅ Có thể giới hạn: Free tier (1 món), Premium (5 món)

---

### 2. ✅ Flexible Mix (DB + AI linh hoạt)
**Đánh giá**: EXCELLENT - Giảm chi phí AI đáng kể

**Lý do nên làm**:
- ✅ Database có ít recipes → AI fill gap
- ✅ Tiết kiệm Bedrock cost (ưu tiên DB trước)
- ✅ Kết quả đa dạng hơn

**Logic**:
```javascript
async function getFlexibleSuggestions(ingredients, count = 5) {
  // Step 1: Query DB recipes
  const dbRecipes = await queryApprovedRecipes(ingredients, count);

  // Step 2: Calculate gap
  const gap = count - dbRecipes.length;

  // Step 3: Generate AI recipes if needed
  let aiRecipes = [];
  if (gap > 0) {
    aiRecipes = await generateAIRecipes(ingredients, gap);
  }

  return {
    recipes: [...dbRecipes, ...aiRecipes],
    stats: {
      from_database: dbRecipes.length,
      from_ai: aiRecipes.length,
      total: dbRecipes.length + aiRecipes.length
    }
  };
}
```

**Examples**:
```
User requests: 5 món
DB có: 2 món match
→ Return: 2 DB + 3 AI ✅

User requests: 5 món
DB có: 7 món match
→ Return: 5 DB + 0 AI (save cost!) ✅

User requests: 5 món
DB có: 0 món match
→ Return: 0 DB + 5 AI ✅
```

---

### 3. ⚠️ Invalid Ingredient Reporting
**Đánh giá**: GOOD nhưng cần đơn giản hóa cho MVP

**Phân tích**:
- ✅ Phát hiện spam/abuse
- ✅ Cải thiện master_ingredients database
- ⚠️ Phức tạp: Cần admin dashboard

**Recommendation cho MVP**:

**Option A: Simple Logging (MVP - Recommended)**
```javascript
// Chỉ log, không block user
async function validateIngredient(ingredient) {
  const isValid = await checkMasterIngredients(ingredient);

  if (!isValid) {
    // Log to CloudWatch
    console.warn('Invalid ingredient', {
      ingredient,
      user_id: userId,
      timestamp: new Date()
    });

    // Return warning nhưng vẫn cho AI process
    return {
      valid: false,
      warning: `"${ingredient}" không có trong database. AI sẽ cố gắng xử lý.`,
      suggestions: await fuzzySearch(ingredient)
    };
  }

  return { valid: true };
}
```

**Option B: Report System (Post-MVP)**
```javascript
// Thêm table: invalid_ingredients_reports
{
  PK: "INGREDIENT#invalid",
  SK: "REPORT#<timestamp>#<user_id>",
  ingredient_name: "abc xyz",
  user_id: "user-123",
  report_count: 1,
  status: "pending", // pending | reviewed | added
  created_at: timestamp
}

// Auto-escalate to admin
if (reportCount >= 5) {
  await notifyAdmin({
    type: 'invalid_ingredient_spike',
    ingredient: ingredientName,
    count: reportCount
  });
}
```

**Khuyến nghị**:
- ✅ MVP: Dùng Option A (simple logging)
- ✅ Post-MVP: Nâng cấp lên Option B khi có admin dashboard

---

### 4. ✅ Recipe Categories (Loại món)
**Đánh giá**: EXCELLENT - Tăng chất lượng AI suggestions

**Lý do nên làm**:
- ✅ Đa dạng món ăn (không toàn xào/chiên)
- ✅ Balanced meal (có canh, có rau, có món chính)
- ✅ AI prompt tốt hơn

**Categories**:
```javascript
const RECIPE_CATEGORIES = {
  cooking_method: [
    'xào',      // stir-fry
    'hấp',      // steam
    'luộc',     // boil
    'chiên',    // fry
    'nướng',    // grill
    'kho',      // braise
    'rim',      // simmer
    'trộn',     // salad
    'canh',     // soup
    'lẩu'       // hotpot
  ],
  meal_type: [
    'món chính',    // main dish
    'món phụ',      // side dish
    'canh',         // soup
    'khai vị',      // appetizer
    'tráng miệng'   // dessert
  ]
};
```

**AI Prompt với categories**:
```javascript
async function generateAIRecipeWithCategory(ingredients, category) {
  const prompt = `
Bạn là đầu bếp chuyên nghiệp. Tạo công thức nấu ăn với:
- Nguyên liệu: ${ingredients.join(', ')}
- Phương pháp nấu: ${category.cooking_method}
- Loại món: ${category.meal_type}

Yêu cầu:
- Sử dụng tối đa nguyên liệu đã cho
- Hướng dẫn chi tiết, dễ hiểu
- Thời gian nấu thực tế
  `;

  return await bedrock.invoke(prompt);
}
```

**Smart Distribution**:
```javascript
// User request 5 món
// → AI tự động phân bổ đa dạng
const categoryDistribution = [
  { cooking_method: 'xào', meal_type: 'món chính' },
  { cooking_method: 'canh', meal_type: 'canh' },
  { cooking_method: 'hấp', meal_type: 'món phụ' },
  { cooking_method: 'trộn', meal_type: 'món phụ' },
  { cooking_method: 'kho', meal_type: 'món chính' }
];
```

---

##    TỔNG HỢP ĐÁNH GIÁ

| Feature | Priority | Complexity | MVP | Cost Impact |
|---------|----------|------------|-----|-------------|
| **Số món (1-5)** | ⭐⭐⭐⭐⭐ HIGH | 🟢 LOW | ✅ YES | +20-50% AI |
| **Flexible Mix** | ⭐⭐⭐⭐⭐ HIGH | 🟢 LOW | ✅ YES | -30% AI (save!) |
| **Invalid Reports** | ⭐⭐⭐ MEDIUM | 🟡 MEDIUM | ⚠️ SIMPLE | +$5/month |
| **Categories** | ⭐⭐⭐⭐ HIGH | 🟢 LOW | ✅ YES | $0 |

---

## ✅ KHUYẾN NGHỊ TRIỂN KHAI

### Phase 1: MVP (2 tuần)
1. ✅ **Số món (1-5)** - Must have
2. ✅ **Flexible Mix** - Must have (tiết kiệm cost)
3. ✅ **Categories** - Should have (improve AI quality)
4. ⚠️ **Invalid Reports** - Nice to have (simple logging only)

### Phase 2: Post-MVP (4 tuần sau)
1. ✅ Full reporting system với admin dashboard
2. ✅ Advanced category preferences (user chọn yêu thích canh/xào/hấp)
3. ✅ Smart recipe balancing (tự động cân bằng dinh dưỡng)

---

##    Chi Phí Ước Tính

### Trước (1,000 users):
- AI cost: $15/tháng (1 recipe per request)

### Sau (1,000 users):
- **Best case** (DB đủ): $15/tháng (không tăng) ✅
- **Average case** (50% DB, 50% AI): $22/tháng (+$7)
- **Worst case** (DB rỗng, request 5 món): $45/tháng (+$30)

**Mitigation**:
- ✅ Free tier: 1 món
- ✅ Premium tier: 5 món (giới hạn cost)
- ✅ Prioritize DB recipes (auto save cost)

**Kết luận**: Features tốt, nên implement!   
