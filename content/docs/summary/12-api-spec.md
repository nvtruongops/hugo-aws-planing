---
title: "12 - API Specification"
weight: 12
---

# API Specification - Smart Cooking App

## API Overview

### Base Configuration

```yaml
API Name: Smart Cooking API
Base URL: https://api.smartcooking.app
Version: v1
Protocol: HTTPS only (TLS 1.2+)
Format: JSON
Authentication: JWT Bearer Token (Cognito)
```

### API Gateway Configuration

```yaml
Type: REST API
Stage: prod
Endpoint Type: Regional
CORS: Enabled
  - Origins: ['https://smartcooking.app', 'http://localhost:3000']
  - Methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS']
  - Headers: ['Content-Type', 'Authorization', 'X-Request-ID']
  - Credentials: true

Throttling:
  Rate Limit: 1000 requests/second
  Burst Limit: 2000 requests

Timeout: 29 seconds (API Gateway max)
```

## Authentication

### Authentication Flow

```http
POST /auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "SecurePass123"
}

Response 200 OK:
{
  "access_token": "eyJraWQiOiJ...",
  "refresh_token": "eyJjdHkiOiJ...",
  "id_token": "eyJraWQiOiJ...",
  "expires_in": 3600,
  "token_type": "Bearer"
}
```

### Authenticated Requests

```http
GET /api/v1/user/profile
Authorization: Bearer eyJraWQiOiJ...
Content-Type: application/json
```

## API Endpoints

### 1. Authentication & User Management

#### POST /auth/register
**Description**: Register new user

**Request:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123",
  "username": "cookmaster",
  "full_name": "John Doe"
}
```

**Response 201 Created:**
```json
{
  "user_id": "uuid-123",
  "email": "user@example.com",
  "username": "cookmaster",
  "message": "Verification email sent"
}
```

**Errors:**
- 400: Invalid email format, weak password
- 409: Email or username already exists

---

#### POST /auth/login
**Description**: Login user

**Request:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123"
}
```

**Response 200 OK:**
```json
{
  "access_token": "eyJraWQiOiJ...",
  "refresh_token": "eyJjdHkiOiJ...",
  "expires_in": 3600,
  "user": {
    "user_id": "uuid-123",
    "email": "user@example.com",
    "username": "cookmaster",
    "role": "user"
  }
}
```

**Errors:**
- 401: Invalid credentials
- 403: Account not verified

---

#### GET /user/profile
**Auth**: Required

**Response 200 OK:**
```json
{
  "user_id": "uuid-123",
  "email": "user@example.com",
  "username": "cookmaster",
  "full_name": "John Doe",
  "date_of_birth": "1990-01-15",
  "gender": "male",
  "country": "Vietnam",
  "avatar_url": "https://s3.../avatar.jpg",
  "role": "user",
  "created_at": "2025-01-15T10:00:00Z"
}
```

---

#### PUT /user/profile
**Auth**: Required

**Request:**
```json
{
  "full_name": "John Smith",
  "date_of_birth": "1990-01-15",
  "gender": "male",
  "country": "Vietnam",
  "avatar_url": "https://s3.../new-avatar.jpg"
}
```

**Response 200 OK:**
```json
{
  "message": "Profile updated successfully",
  "user": { /* updated user object */ }
}
```

---

#### PUT /user/preferences
**Auth**: Required

**Request:**
```json
{
  "dietary_restrictions": ["vegetarian"],
  "allergies": ["shrimp", "crab"],
  "favorite_cuisines": ["Vietnamese", "Italian"],
  "preferred_cooking_methods": ["stir-fry", "steam", "soup"],
  "preferred_recipe_count": 3,
  "spice_level": "medium"
}
```

**Response 200 OK:**
```json
{
  "message": "Preferences updated",
  "preferences": { /* updated preferences */ }
}
```

---

### 2. Ingredient Management

#### GET /user/ingredients
**Auth**: Required

**Response 200 OK:**
```json
{
  "ingredients": [
    {
      "ingredient_id": "uuid-456",
      "ingredient_name": "Thịt gà",
      "normalized_name": "thit ga",
      "added_at": "2025-01-20T09:00:00Z"
    },
    {
      "ingredient_id": "uuid-457",
      "ingredient_name": "Cà chua",
      "normalized_name": "ca chua",
      "added_at": "2025-01-20T09:05:00Z"
    }
  ],
  "count": 2
}
```

---

#### POST /user/ingredients
**Auth**: Required

**Request:**
```json
{
  "ingredient_name": "Thịt gà"
}
```

**Response 201 Created:**
```json
{
  "ingredient_id": "uuid-456",
  "ingredient_name": "Thịt gà",
  "normalized_name": "thit ga",
  "added_at": "2025-01-20T09:00:00Z",
  "message": "Ingredient added"
}
```

**Response 200 OK (Auto-corrected):**
```json
{
  "ingredient_id": "uuid-456",
  "ingredient_name": "Thịt gà",
  "original_input": "thit ga",
  "corrected": true,
  "confidence": 0.85,
  "message": "Ingredient auto-corrected and added"
}
```

**Errors:**
- 400: Invalid ingredient with suggestions
```json
{
  "error": "ingredient_not_found",
  "message": "Không tìm thấy nguyên liệu: abc xyz",
  "suggestions": ["gà", "cá", "bò", "tôm"]
}
```

---

#### DELETE /user/ingredients/{ingredient_id}
**Auth**: Required

**Response 200 OK:**
```json
{
  "message": "Ingredient removed"
}
```

---

#### POST /ingredients/validate
**Auth**: Required
**Description**: Validate multiple ingredients at once

**Request:**
```json
{
  "ingredients": ["thịt gà", "abc xyz", "thit ca"]
}
```

**Response 200 OK:**
```json
{
  "valid": [
    {
      "ingredient_name": "Thịt gà",
      "normalized_name": "thit ga"
    }
  ],
  "invalid": ["abc xyz"],
  "corrected": [
    {
      "original": "thit ca",
      "matched": "Thịt cá",
      "confidence": 0.9
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

---

#### GET /ingredients/search
**Auth**: Required
**Description**: Search master ingredients

**Query Params:**
- `q`: Search term (required)
- `category`: Filter by category (optional)
- `limit`: Max results (default: 20)

**Example:**
```
GET /ingredients/search?q=ga&category=meat&limit=10
```

**Response 200 OK:**
```json
{
  "results": [
    {
      "ingredient_id": "uuid-789",
      "name": "Thịt gà",
      "normalized_name": "thit ga",
      "category": "meat",
      "aliases": ["gà", "chicken"]
    },
    {
      "ingredient_id": "uuid-790",
      "name": "Gà ta",
      "normalized_name": "ga ta",
      "category": "meat",
      "aliases": ["gà đồi", "gà nhà"]
    }
  ],
  "count": 2
}
```

---

### 3. AI Recipe Suggestions 

#### POST /ai/suggest
**Auth**: Required
**Timeout**: 60 seconds

**Request:**
```json
{
  "ingredients": ["thịt gà", "cà chua", "hành tây"],
  "recipe_count": 3
}
```

**Response 200 OK (Flexible Mix):**
```json
{
  "suggestions": [
    {
      "id": "recipe-001",
      "name": "Gà xào cà chua",
      "source": "database",
      "cooking_method": "stir-fry",
      "meal_type": "main",
      "cuisine_type": "Vietnamese",
      "prep_time_minutes": 15,
      "cook_time_minutes": 20,
      "servings": 2,
      "difficulty": "easy",
      "match_score": 0.95,
      "is_approved": true,
      "average_rating": 4.5,
      "rating_count": 12,
      "ingredients": [
        {
          "ingredient_name": "Thịt gà",
          "quantity": "300g",
          "preparation": "Cắt miếng vừa"
        }
      ],
      "instructions": [
        {
          "step_number": 1,
          "description": "Ướp gà với gia vị",
          "duration": "10 phút"
        }
      ],
      "nutritional_info": {
        "calories": 350,
        "protein": "30g",
        "carbs": "20g",
        "fat": "15g"
      }
    },
    {
      "id": "recipe-002",
      "name": "Canh cà chua",
      "source": "database",
      "cooking_method": "soup",
      "meal_type": "soup",
      "match_score": 0.80,
      "is_approved": true,
      "average_rating": 4.2
    },
    {
      "id": "ai-gen-001",
      "name": "Gà hấp cà chua hành",
      "source": "ai",
      "cooking_method": "steam",
      "meal_type": "main",
      "is_new": true,
      "is_approved": false,
      "ingredients": [...],
      "instructions": [...],
      "note": "Recipe created by AI - Try it and rate!"
    }
  ],
  "stats": {
    "requested": 3,
    "from_database": 2,
    "from_ai": 1,
    "processing_time_ms": 3500
  },
  "warnings": []
}
```

**Response 200 OK (With Warnings):**
```json
{
  "suggestions": [...],
  "stats": {
    "requested": 3,
    "from_database": 0,
    "from_ai": 3
  },
  "warnings": [
    {
      "ingredient": "abc xyz",
      "message": "Nguyên liệu không hợp lệ, AI cố gắng xử lý",
      "suggestions": ["gà", "cá", "bò"],
      "reported": true
    }
  ]
}
```

**Errors:**
- 400: All ingredients invalid
```json
{
  "error": "all_ingredients_invalid",
  "message": "Tất cả nguyên liệu không hợp lệ",
  "invalid_ingredients": ["abc", "xyz"],
  "suggestions": [
    { "original": "abc", "similar": ["cá", "gà"] }
  ]
}
```
- 400: Too few ingredients
```json
{
  "error": "insufficient_ingredients",
  "message": "Cần ít nhất 2 nguyên liệu"
}
```
- 403: Rate limit exceeded (free tier)
```json
{
  "error": "rate_limit_exceeded",
  "message": "Bạn đã hết lượt gợi ý. Nâng cấp Premium?",
  "limit": 10,
  "used": 10,
  "reset_at": "2025-02-01T00:00:00Z"
}
```

---

#### GET /ai/suggestions
**Auth**: Required
**Description**: Get user's suggestion history

**Query Params:**
- `limit`: Max results (default: 20)
- `offset`: Pagination offset

**Response 200 OK:**
```json
{
  "suggestions": [
    {
      "suggestion_id": "uuid-303",
      "ingredients_used": ["thịt gà", "cà chua"],
      "requested_recipe_count": 3,
      "recipes_from_db": 2,
      "recipes_from_ai": 1,
      "was_accepted": true,
      "feedback_rating": 5,
      "created_at": "2025-01-20T10:00:00Z"
    }
  ],
  "count": 1,
  "has_more": false
}
```

---

#### POST /ai/feedback
**Auth**: Required
**Description**: Provide feedback on AI suggestion

**Request:**
```json
{
  "suggestion_id": "uuid-303",
  "was_accepted": true,
  "feedback_rating": 5,
  "feedback_comment": "Gợi ý rất hay!"
}
```

**Response 200 OK:**
```json
{
  "message": "Feedback saved. Thank you!"
}
```

---

### 4. Cooking History & Rating 

#### POST /cooking/start
**Auth**: Required

**Request:**
```json
{
  "recipe_id": "recipe-001",
  "suggestion_id": "uuid-303"
}
```

**Response 201 Created:**
```json
{
  "history_id": "uuid-202",
  "recipe_id": "recipe-001",
  "status": "cooking",
  "created_at": "2025-01-20T15:30:00Z",
  "message": "Bắt đầu nấu ăn!"
}
```

---

#### PUT /cooking/{history_id}/complete
**Auth**: Required

**Response 200 OK:**
```json
{
  "history_id": "uuid-202",
  "status": "completed",
  "cook_date": "2025-01-20T18:00:00Z",
  "message": "Hoàn thành! Vui lòng đánh giá món ăn.",
  "show_rating_form": true
}
```

---

#### GET /user/cooking-history
**Auth**: Required

**Query Params:**
- `limit`: Max results (default: 20)
- `offset`: Pagination
- `favorites`: true/false (filter favorites only)
- `status`: cooking/completed/failed

**Response 200 OK:**
```json
{
  "history": [
    {
      "history_id": "uuid-202",
      "recipe_id": "recipe-001",
      "recipe_name": "Gà xào cà chua",
      "status": "completed",
      "personal_rating": 5,
      "personal_notes": "Rất ngon!",
      "is_favorite": true,
      "cook_date": "2025-01-20T18:00:00Z",
      "created_at": "2025-01-20T15:30:00Z"
    }
  ],
  "count": 1,
  "has_more": false
}
```

---

#### DELETE /cooking/{history_id}
**Auth**: Required

**Response 200 OK:**
```json
{
  "message": "Cooking history deleted"
}
```

---

#### PUT /cooking/{history_id}/favorite
**Auth**: Required

**Request:**
```json
{
  "is_favorite": true
}
```

**Response 200 OK:**
```json
{
  "message": "Added to favorites"
}
```

---

#### POST /recipes/{recipe_id}/rate
**Auth**: Required

**Request:**
```json
{
  "rating": 5,
  "comment": "Món này rất ngon!",
  "history_id": "uuid-202"
}
```

**Response 200 OK (Auto-approved):**
```json
{
  "success": true,
  "rating_saved": true,
  "average_rating": 4.2,
  "rating_count": 13,
  "auto_approved": true,
  "message": "Công thức đã được thêm vào database!",
  "recipe": {
    "id": "recipe-001",
    "is_approved": true,
    "is_public": true,
    "approval_type": "auto_rating"
  }
}
```

**Response 200 OK (Not approved):**
```json
{
  "success": true,
  "rating_saved": true,
  "average_rating": 3.5,
  "rating_count": 8,
  "auto_approved": false,
  "message": "Cảm ơn đánh giá của bạn!"
}
```

**Errors:**
- 400: Invalid rating (must be 1-5)
- 409: Already rated this recipe

---

### 5. Recipe Management

#### GET /recipes/{recipe_id}
**Auth**: Optional (required for private recipes)

**Response 200 OK:**
```json
{
  "recipe_id": "recipe-001",
  "title": "Gà xào cà chua",
  "description": "Món gà xào thơm ngon...",
  "cuisine_type": "Vietnamese",
  "cooking_method": "stir-fry",
  "meal_type": "main",
  "prep_time_minutes": 15,
  "cook_time_minutes": 20,
  "servings": 2,
  "difficulty": "easy",
  "calories_per_serving": 350,
  "ingredients": [...],
  "instructions": [...],
  "nutritional_info": {...},
  "is_public": true,
  "is_ai_generated": false,
  "is_approved": true,
  "average_rating": 4.5,
  "rating_count": 12,
  "created_by": {
    "user_id": "uuid-123",
    "username": "cookmaster"
  },
  "created_at": "2025-01-15T10:00:00Z"
}
```

**Errors:**
- 404: Recipe not found
- 403: Recipe is private

---

#### GET /recipes/search
**Auth**: Optional

**Query Params:**
- `q`: Search query (optional)
- `cuisine_type`: Vietnamese/Italian/... (optional)
- `cooking_method`: stir-fry/steam/soup/... (optional)
- `meal_type`: main/side/soup/... (optional)
- `min_rating`: Minimum rating (optional)
- `sort`: rating/created_at/popularity (default: rating)
- `limit`: Max results (default: 20)
- `offset`: Pagination

**Example:**
```
GET /recipes/search?cuisine_type=Vietnamese&cooking_method=stir-fry&min_rating=4&sort=rating&limit=10
```

**Response 200 OK:**
```json
{
  "recipes": [
    {
      "recipe_id": "recipe-001",
      "title": "Gà xào cà chua",
      "cuisine_type": "Vietnamese",
      "cooking_method": "stir-fry",
      "average_rating": 4.5,
      "rating_count": 12,
      "prep_time_minutes": 15,
      "image_url": "https://s3.../recipe-001.jpg"
    }
  ],
  "count": 1,
  "has_more": false
}
```

---

#### POST /recipes
**Auth**: Required
**Description**: Create custom recipe (not from AI)

**Request:**
```json
{
  "title": "Món của tôi",
  "description": "Công thức tự tạo",
  "cuisine_type": "Vietnamese",
  "cooking_method": "stir-fry",
  "meal_type": "main",
  "ingredients": [...],
  "instructions": [...]
}
```

**Response 201 Created:**
```json
{
  "recipe_id": "recipe-999",
  "message": "Recipe created. It will be public after approval.",
  "is_approved": false
}
```

---

#### PUT /recipes/{recipe_id}
**Auth**: Required (own recipes only)

**Request:**
```json
{
  "title": "Updated title",
  "description": "Updated description"
}
```

**Response 200 OK:**
```json
{
  "message": "Recipe updated",
  "recipe": {...}
}
```

---

#### DELETE /recipes/{recipe_id}
**Auth**: Required (own recipes or admin)

**Response 200 OK:**
```json
{
  "message": "Recipe deleted"
}
```

---

### 6. Privacy Settings

#### GET /user/privacy
**Auth**: Required

**Response 200 OK:**
```json
{
  "profile_visibility": "public",
  "email_visibility": "private",
  "date_of_birth_visibility": "friends",
  "gender_visibility": "public",
  "country_visibility": "public",
  "recipes_visibility": "public",
  "ingredients_visibility": "friends",
  "preferences_visibility": "friends"
}
```

---

#### PUT /user/privacy
**Auth**: Required

**Request:**
```json
{
  "profile_visibility": "friends",
  "ingredients_visibility": "private"
}
```

**Response 200 OK:**
```json
{
  "message": "Privacy settings updated",
  "settings": {...}
}
```

---

### 7. Social - Friends

#### POST /friends/request
**Auth**: Required

**Request:**
```json
{
  "friend_id": "uuid-456"
}
```

**Response 201 Created:**
```json
{
  "friendship_id": "uuid-505",
  "status": "pending",
  "message": "Friend request sent"
}
```

---

#### GET /friends
**Auth**: Required

**Query Params:**
- `status`: pending/accepted/blocked

**Response 200 OK:**
```json
{
  "friends": [
    {
      "friendship_id": "uuid-505",
      "friend_id": "uuid-456",
      "username": "friend1",
      "full_name": "Friend One",
      "avatar_url": "https://s3.../avatar.jpg",
      "status": "accepted",
      "requested_at": "2025-01-18T10:00:00Z"
    }
  ],
  "count": 1
}
```

---

#### PUT /friends/{friendship_id}/accept
**Auth**: Required

**Response 200 OK:**
```json
{
  "message": "Friend request accepted",
  "friendship": {...}
}
```

---

#### PUT /friends/{friendship_id}/reject
**Auth**: Required

**Response 200 OK:**
```json
{
  "message": "Friend request rejected"
}
```

---

#### DELETE /friends/{friendship_id}
**Auth**: Required

**Response 200 OK:**
```json
{
  "message": "Friendship removed"
}
```

---

### 8. Social - Posts & Comments

#### POST /posts
**Auth**: Required

**Request:**
```json
{
  "content": "Vừa nấu món này, rất ngon!",
  "recipe_id": "recipe-001",
  "images": ["https://s3.../post1.jpg"],
  "is_public": true
}
```

**Response 201 Created:**
```json
{
  "post_id": "uuid-606",
  "message": "Post created"
}
```

---

#### GET /posts/feed
**Auth**: Required
**Description**: Get personalized feed (friends' posts)

**Query Params:**
- `limit`: Max results (default: 20)
- `offset`: Pagination

**Response 200 OK:**
```json
{
  "posts": [
    {
      "post_id": "uuid-606",
      "user_id": "uuid-456",
      "username": "friend1",
      "user_avatar": "https://s3.../avatar.jpg",
      "content": "Vừa nấu món này!",
      "recipe_id": "recipe-001",
      "recipe_name": "Gà xào cà chua",
      "images": ["https://s3.../post1.jpg"],
      "likes_count": 15,
      "comments_count": 3,
      "user_has_liked": false,
      "created_at": "2025-01-20T20:00:00Z"
    }
  ],
  "has_more": false
}
```

---

#### GET /posts/{post_id}
**Auth**: Optional

**Response 200 OK:**
```json
{
  "post_id": "uuid-606",
  "user": {...},
  "content": "...",
  "recipe": {...},
  "images": [...],
  "likes_count": 15,
  "comments_count": 3,
  "created_at": "2025-01-20T20:00:00Z"
}
```

---

#### PUT /posts/{post_id}
**Auth**: Required (own posts only)

**Request:**
```json
{
  "content": "Updated content"
}
```

**Response 200 OK:**
```json
{
  "message": "Post updated"
}
```

---

#### DELETE /posts/{post_id}
**Auth**: Required (own posts or admin)

**Response 200 OK:**
```json
{
  "message": "Post deleted"
}
```

---

#### POST /posts/{post_id}/comments
**Auth**: Required

**Request:**
```json
{
  "content": "Trông rất ngon!",
  "parent_comment_id": null
}
```

**Response 201 Created:**
```json
{
  "comment_id": "uuid-707",
  "message": "Comment added"
}
```

---

#### GET /posts/{post_id}/comments
**Auth**: Optional

**Response 200 OK:**
```json
{
  "comments": [
    {
      "comment_id": "uuid-707",
      "user_id": "uuid-456",
      "username": "commenter",
      "user_avatar": "https://s3.../avatar.jpg",
      "content": "Trông rất ngon!",
      "parent_comment_id": null,
      "created_at": "2025-01-20T21:00:00Z"
    }
  ],
  "count": 1
}
```

---

#### POST /reactions
**Auth**: Required

**Request:**
```json
{
  "target_type": "post",
  "target_id": "uuid-606",
  "reaction_type": "like"
}
```

**Response 201 Created:**
```json
{
  "reaction_id": "uuid-808",
  "message": "Reaction added"
}
```

---

#### DELETE /reactions/{reaction_id}
**Auth**: Required

**Response 200 OK:**
```json
{
  "message": "Reaction removed"
}
```

---

### 9. Notifications

#### GET /notifications
**Auth**: Required

**Query Params:**
- `unread_only`: true/false (default: false)
- `limit`: Max results (default: 20)

**Response 200 OK:**
```json
{
  "notifications": [
    {
      "notification_id": "uuid-909",
      "type": "comment",
      "actor": {
        "user_id": "uuid-456",
        "username": "friend1",
        "avatar_url": "https://s3.../avatar.jpg"
      },
      "target_type": "post",
      "target_id": "uuid-606",
      "content": "John commented on your post",
      "is_read": false,
      "created_at": "2025-01-20T21:00:00Z"
    }
  ],
  "unread_count": 5,
  "count": 10
}
```

---

#### PUT /notifications/{notification_id}/read
**Auth**: Required

**Response 200 OK:**
```json
{
  "message": "Notification marked as read"
}
```

---

#### PUT /notifications/read-all
**Auth**: Required

**Response 200 OK:**
```json
{
  "message": "All notifications marked as read",
  "count": 5
}
```

---

### 10. Admin Operations

#### GET /admin/users
**Auth**: Required (admin only)

**Query Params:**
- `role`: user/admin
- `is_active`: true/false
- `limit`, `offset`

**Response 200 OK:**
```json
{
  "users": [
    {
      "user_id": "uuid-123",
      "email": "user@example.com",
      "username": "user1",
      "role": "user",
      "is_active": true,
      "created_at": "2025-01-15T10:00:00Z"
    }
  ],
  "count": 1
}
```

---

#### PUT /admin/users/{user_id}/ban
**Auth**: Required (admin only)

**Request:**
```json
{
  "reason": "Spam content"
}
```

**Response 200 OK:**
```json
{
  "message": "User banned",
  "user_id": "uuid-123"
}
```

---

#### GET /admin/statistics
**Auth**: Required (admin only)

**Response 200 OK:**
```json
{
  "total_users": 1000,
  "active_users_last_30_days": 600,
  "total_recipes": 5000,
  "approved_recipes": 2000,
  "ai_generated_recipes": 3000,
  "total_ai_suggestions": 10000,
  "average_db_ai_mix": "60/40",
  "invalid_ingredient_reports": 50,
  "monthly_cost_usd": 155.50
}
```

---

## Response Codes

### Success Codes
- **200 OK**: Request successful
- **201 Created**: Resource created
- **204 No Content**: Successful deletion

### Client Error Codes
- **400 Bad Request**: Invalid input
- **401 Unauthorized**: Missing or invalid token
- **403 Forbidden**: Insufficient permissions
- **404 Not Found**: Resource not found
- **409 Conflict**: Resource already exists
- **429 Too Many Requests**: Rate limit exceeded

### Server Error Codes
- **500 Internal Server Error**: Unexpected error
- **503 Service Unavailable**: Service temporarily down
- **504 Gateway Timeout**: Request timeout (AI generation)

## Error Response Format

```json
{
  "error": "error_code",
  "message": "Human-readable error message",
  "details": {
    "field": "Additional context"
  },
  "timestamp": "2025-01-20T10:00:00Z",
  "request_id": "uuid-xxx"
}
```

## Rate Limiting

```yaml
Free Tier:
  AI Suggestions: 10/month
  API Requests: 1000/hour

Premium Tier:
  AI Suggestions: Unlimited
  API Requests: 10000/hour

Rate Limit Headers:
  X-RateLimit-Limit: 1000
  X-RateLimit-Remaining: 995
  X-RateLimit-Reset: 1706097600
```

## Related Documents

- [10 - Architecture](10-architecture.md)
- [11 - Database](11-database.md)
- [13 - Security](13-security.md)
- [20 - Backend](20-backend.md)
