---
title: "11 - Database Design"
weight: 11
---

# Thiết Kế Database - Smart Cooking App

## Database Strategy

### Technology Choice: Amazon DynamoDB

**Why DynamoDB over RDS/PostgreSQL?**

| Criteria | DynamoDB | RDS PostgreSQL |
|----------|----------|----------------|
| **Cost (MVP)** | $40-55/month | $30-50/month (t3.micro) |
| **Cost (Scale)** | Scales with usage | Fixed + storage growth |
| **Scaling** | Auto-scaling, zero config | Manual scaling, downtime |
| **Maintenance** | Zero maintenance | Patching, backups, monitoring |
| **Latency** | Single-digit ms | 10-50ms |
| **Lambda Integration** | Native, optimized | Connection pooling needed |
| **Serverless** | True serverless | Always running |
| **Learning Curve** | High (single-table) | Low (SQL familiar) |

**Decision**: DynamoDB for MVP
- Serverless architecture alignment
- Better cost structure for variable load
- Lower operational overhead for solo developer

## Single-Table Design

### Table Name: `smart-cooking-data`

**Design Philosophy:**
- One table to rule them all (single-table design)
- Composite primary key (PK + SK)
- GSI (Global Secondary Indexes) for alternate access patterns
- Denormalization where needed for performance

### Primary Key Structure

```
PK (Partition Key): Entity type + ID
SK (Sort Key): Sub-entity type + timestamp/ID

Example:
  USER#123      | PROFILE
  USER#123      | PREFERENCES
  RECIPE#456    | METADATA
  RECIPE#456    | INGREDIENT#001
```

## Entity Designs

### 1. Users

#### User Profile
```json
{
  "PK": "USER#<user_id>",
  "SK": "PROFILE",
  "entity_type": "USER_PROFILE",
  "user_id": "uuid-123",
  "email": "user@example.com",
  "username": "cookmaster",
  "full_name": "John Doe",
  "date_of_birth": "1990-01-15",
  "gender": "male",
  "country": "Vietnam",
  "avatar_url": "https://s3.../avatar.jpg",
  "role": "user",
  "is_active": true,
  "created_at": "2025-01-15T10:00:00Z",
  "updated_at": "2025-01-15T10:00:00Z",
  "last_login": "2025-01-20T15:30:00Z",

  "GSI1PK": "ROLE#user",
  "GSI1SK": "USER#2025-01-15T10:00:00Z"
}
```

**Access Patterns:**
- Get user by ID: `PK=USER#<id>, SK=PROFILE`
- List all admins: `GSI1PK=ROLE#admin`

#### User Preferences
```json
{
  "PK": "USER#<user_id>",
  "SK": "PREFERENCES",
  "entity_type": "USER_PREFERENCES",
  "dietary_restrictions": ["vegetarian"],
  "allergies": ["shrimp", "crab"],
  "favorite_cuisines": ["Vietnamese", "Italian"],
  "preferred_cooking_methods": ["stir-fry", "steam", "soup"],
  "preferred_recipe_count": 3,
  "spice_level": "medium",
  "created_at": "2025-01-15T10:00:00Z",
  "updated_at": "2025-01-20T11:00:00Z"
}
```

**Access Patterns:**
- Get preferences: `PK=USER#<id>, SK=PREFERENCES`

#### Privacy Settings
```json
{
  "PK": "USER#<user_id>",
  "SK": "PRIVACY",
  "entity_type": "PRIVACY_SETTINGS",
  "profile_visibility": "public",
  "email_visibility": "private",
  "date_of_birth_visibility": "friends",
  "gender_visibility": "public",
  "country_visibility": "public",
  "recipes_visibility": "public",
  "ingredients_visibility": "friends",
  "preferences_visibility": "friends",
  "created_at": "2025-01-15T10:00:00Z",
  "updated_at": "2025-01-15T10:00:00Z"
}
```

**Access Patterns:**
- Get privacy settings: `PK=USER#<id>, SK=PRIVACY`

### 2. User Ingredients

```json
{
  "PK": "USER#<user_id>",
  "SK": "INGREDIENT#<normalized_name>",
  "entity_type": "USER_INGREDIENT",
  "ingredient_id": "uuid-456",
  "ingredient_name": "Thịt gà",
  "normalized_name": "thit ga",
  "added_at": "2025-01-20T09:00:00Z"
}
```

**Access Patterns:**
- List user ingredients: `PK=USER#<id>, SK begins_with INGREDIENT#`
- Check specific ingredient: `PK=USER#<id>, SK=INGREDIENT#<name>`

### 3. Master Ingredients

```json
{
  "PK": "INGREDIENT#<ingredient_id>",
  "SK": "METADATA",
  "entity_type": "MASTER_INGREDIENT",
  "ingredient_id": "uuid-789",
  "name": "Thịt gà",
  "normalized_name": "thit ga",
  "category": "meat",
  "aliases": ["gà", "chicken", "thịt gà ta"],
  "is_active": true,
  "created_at": "2025-01-01T00:00:00Z",
  "updated_at": "2025-01-01T00:00:00Z",

  "GSI1PK": "CATEGORY#meat",
  "GSI1SK": "INGREDIENT#thit ga",
  "GSI2PK": "INGREDIENT#SEARCH",
  "GSI2SK": "NAME#thit ga"
}
```

**Access Patterns:**
- Get ingredient by ID: `PK=INGREDIENT#<id>, SK=METADATA`
- Search by normalized name: `GSI2PK=INGREDIENT#SEARCH, GSI2SK begins_with NAME#<term>`
- List by category: `GSI1PK=CATEGORY#<category>`

### 4. Recipes

#### Recipe Metadata
```json
{
  "PK": "RECIPE#<recipe_id>",
  "SK": "METADATA",
  "entity_type": "RECIPE",
  "recipe_id": "uuid-101",
  "user_id": "uuid-123",
  "title": "Gà xào sả ớt",
  "normalized_title": "ga xao sa ot",
  "description": "Món gà xào thơm ngon...",
  "cuisine_type": "Vietnamese",
  "cooking_method": "stir-fry",
  "meal_type": "main",
  "prep_time_minutes": 15,
  "cook_time_minutes": 20,
  "servings": 2,
  "calories_per_serving": 350,
  "instructions": [
    {
      "step_number": 1,
      "description": "Ướp gà với sả ớt",
      "duration": "10 phút"
    }
  ],
  "is_public": true,
  "is_ai_generated": true,
  "is_approved": true,
  "approval_type": "auto_rating",
  "average_rating": 4.5,
  "rating_count": 12,
  "ai_cache_hit_count": 0,
  "created_at": "2025-01-20T10:00:00Z",
  "updated_at": "2025-01-22T14:00:00Z",
  "approved_at": "2025-01-22T14:00:00Z",

  "GSI1PK": "USER#uuid-123",
  "GSI1SK": "RECIPE#2025-01-20T10:00:00Z",
  "GSI2PK": "METHOD#stir-fry",
  "GSI2SK": "RECIPE#4.5#2025-01-20T10:00:00Z"
}
```

**Access Patterns:**
- Get recipe: `PK=RECIPE#<id>, SK=METADATA`
- List user's recipes: `GSI1PK=USER#<user_id>, GSI1SK begins_with RECIPE#`
- Search by cooking method: `GSI2PK=METHOD#<method>, Sort by rating DESC`
- Browse approved recipes: `GSI2PK=APPROVED, GSI2SK begins_with RECIPE#`

#### Recipe Ingredients
```json
{
  "PK": "RECIPE#<recipe_id>",
  "SK": "INGREDIENT#001",
  "entity_type": "RECIPE_INGREDIENT",
  "ingredient_name": "Thịt gà",
  "quantity": "300g",
  "unit": "gram",
  "is_optional": false
}
```

**Access Patterns:**
- List recipe ingredients: `PK=RECIPE#<id>, SK begins_with INGREDIENT#`

### 5. Cooking History

```json
{
  "PK": "USER#<user_id>",
  "SK": "COOKING#2025-01-20T15:30:00Z#<history_id>",
  "entity_type": "COOKING_HISTORY",
  "history_id": "uuid-202",
  "user_id": "uuid-123",
  "recipe_id": "uuid-101",
  "suggestion_id": "uuid-303",
  "status": "completed",
  "personal_rating": 5,
  "personal_notes": "Rất ngon, sẽ nấu lại!",
  "is_favorite": true,
  "cook_date": "2025-01-20T18:00:00Z",
  "created_at": "2025-01-20T15:30:00Z",
  "updated_at": "2025-01-20T18:05:00Z",

  "GSI1PK": "USER#uuid-123#FAVORITE",
  "GSI1SK": "COOKING#2025-01-20T18:00:00Z"
}
```

**Access Patterns:**
- List cooking history: `PK=USER#<id>, SK begins_with COOKING#, Sort DESC`
- List favorites only: `GSI1PK=USER#<id>#FAVORITE, Sort by cook_date DESC`

### 6. Recipe Ratings

```json
{
  "PK": "RECIPE#<recipe_id>",
  "SK": "RATING#<user_id>",
  "entity_type": "RECIPE_RATING",
  "rating_id": "uuid-404",
  "recipe_id": "uuid-101",
  "user_id": "uuid-123",
  "history_id": "uuid-202",
  "rating": 5,
  "comment": "Món này rất ngon!",
  "is_verified_cook": true,
  "created_at": "2025-01-20T18:05:00Z",
  "updated_at": "2025-01-20T18:05:00Z",

  "GSI1PK": "USER#uuid-123",
  "GSI1SK": "RATING#2025-01-20T18:05:00Z"
}
```

**Access Patterns:**
- Get all ratings for recipe: `PK=RECIPE#<id>, SK begins_with RATING#`
- Get user's ratings: `GSI1PK=USER#<id>, SK begins_with RATING#`
- Check if user rated: `PK=RECIPE#<id>, SK=RATING#<user_id>`

### 7. AI Suggestions

```json
{
  "PK": "USER#<user_id>",
  "SK": "SUGGESTION#2025-01-20T10:00:00Z",
  "entity_type": "AI_SUGGESTION",
  "suggestion_id": "uuid-303",
  "user_id": "uuid-123",
  "recipe_ids": ["uuid-101", "uuid-102", "uuid-103"],
  "cache_ids": [],
  "prompt_text": "Gợi ý món với gà, cà chua, hành",
  "ingredients_used": ["thịt gà", "cà chua", "hành tây"],
  "requested_recipe_count": 3,
  "recipes_from_db": 2,
  "recipes_from_ai": 1,
  "invalid_ingredients": [],
  "ai_response": {...},
  "was_from_cache": false,
  "was_accepted": true,
  "feedback_rating": 5,
  "feedback_comment": "Gợi ý rất hay!",
  "created_at": "2025-01-20T10:00:00Z"
}
```

**Access Patterns:**
- List user's suggestions: `PK=USER#<id>, SK begins_with SUGGESTION#, Sort DESC`
- Get specific suggestion: `PK=USER#<id>, SK=SUGGESTION#<timestamp>`

### 8. Friendships

```json
{
  "PK": "USER#<user_id>",
  "SK": "FRIEND#<friend_id>",
  "entity_type": "FRIENDSHIP",
  "friendship_id": "uuid-505",
  "user_id": "uuid-123",
  "friend_id": "uuid-456",
  "status": "accepted",
  "requested_at": "2025-01-18T10:00:00Z",
  "responded_at": "2025-01-18T14:00:00Z",

  "GSI1PK": "USER#uuid-456",
  "GSI1SK": "FRIEND#uuid-123"
}
```

**Access Patterns:**
- List user's friends: `PK=USER#<id>, SK begins_with FRIEND#, Filter status=accepted`
- Reverse lookup (who friended me): `GSI1PK=USER#<id>, SK begins_with FRIEND#`
- Check friendship status: `PK=USER#<id>, SK=FRIEND#<friend_id>`

### 9. Posts

```json
{
  "PK": "POST#<post_id>",
  "SK": "METADATA",
  "entity_type": "POST",
  "post_id": "uuid-606",
  "user_id": "uuid-123",
  "recipe_id": "uuid-101",
  "content": "Vừa nấu món này, rất ngon!",
  "images": ["https://s3.../post1.jpg"],
  "is_public": true,
  "likes_count": 15,
  "comments_count": 3,
  "created_at": "2025-01-20T20:00:00Z",
  "updated_at": "2025-01-20T20:00:00Z",

  "GSI1PK": "USER#uuid-123",
  "GSI1SK": "POST#2025-01-20T20:00:00Z",
  "GSI3PK": "FEED#PUBLIC",
  "GSI3SK": "POST#2025-01-20T20:00:00Z"
}
```

**Access Patterns:**
- Get post: `PK=POST#<id>, SK=METADATA`
- List user's posts: `GSI1PK=USER#<id>, SK begins_with POST#`
- Public feed: `GSI3PK=FEED#PUBLIC, Sort DESC`

### 10. Comments

```json
{
  "PK": "POST#<post_id>",
  "SK": "COMMENT#2025-01-20T21:00:00Z#<comment_id>",
  "entity_type": "COMMENT",
  "comment_id": "uuid-707",
  "post_id": "uuid-606",
  "user_id": "uuid-456",
  "parent_comment_id": null,
  "content": "Trông rất ngon!",
  "created_at": "2025-01-20T21:00:00Z",
  "updated_at": "2025-01-20T21:00:00Z",

  "GSI1PK": "USER#uuid-456",
  "GSI1SK": "COMMENT#2025-01-20T21:00:00Z"
}
```

**Access Patterns:**
- List post comments: `PK=POST#<id>, SK begins_with COMMENT#, Sort ASC`
- List user's comments: `GSI1PK=USER#<id>, SK begins_with COMMENT#`

### 11. Reactions

```json
{
  "PK": "POST#<post_id>",
  "SK": "REACTION#<user_id>",
  "entity_type": "REACTION",
  "reaction_id": "uuid-808",
  "user_id": "uuid-456",
  "target_type": "post",
  "target_id": "uuid-606",
  "reaction_type": "like",
  "created_at": "2025-01-20T21:30:00Z",

  "GSI1PK": "USER#uuid-456",
  "GSI1SK": "REACTION#2025-01-20T21:30:00Z"
}
```

**Access Patterns:**
- List post reactions: `PK=POST#<id>, SK begins_with REACTION#`
- Check user's reaction: `PK=POST#<id>, SK=REACTION#<user_id>`
- List user's reactions: `GSI1PK=USER#<id>, SK begins_with REACTION#`

### 12. Notifications

```json
{
  "PK": "USER#<user_id>",
  "SK": "NOTIFICATION#2025-01-20T21:00:00Z#<notification_id>",
  "entity_type": "NOTIFICATION",
  "notification_id": "uuid-909",
  "user_id": "uuid-123",
  "actor_id": "uuid-456",
  "type": "comment",
  "target_type": "post",
  "target_id": "uuid-606",
  "content": "John commented on your post",
  "is_read": false,
  "created_at": "2025-01-20T21:00:00Z",

  "GSI1PK": "USER#uuid-123#UNREAD",
  "GSI1SK": "NOTIFICATION#2025-01-20T21:00:00Z"
}
```

**Access Patterns:**
- List notifications: `PK=USER#<id>, SK begins_with NOTIFICATION#, Sort DESC`
- List unread only: `GSI1PK=USER#<id>#UNREAD, Sort DESC`

### 13. Invalid Ingredient Reports (Optional)

```json
{
  "PK": "INVALID_INGREDIENT#<normalized_name>",
  "SK": "USER#<user_id>",
  "entity_type": "INVALID_INGREDIENT_REPORT",
  "report_id": "uuid-1010",
  "ingredient_name": "abc xyz",
  "normalized_name": "abc xyz",
  "user_id": "uuid-123",
  "user_report_count": 1,
  "total_reports": 5,
  "status": "pending",
  "admin_notes": "",
  "created_at": "2025-01-20T10:00:00Z",
  "updated_at": "2025-01-20T10:00:00Z",

  "GSI1PK": "REPORTS#PENDING",
  "GSI1SK": "TOTAL#5#2025-01-20T10:00:00Z"
}
```

**Access Patterns:**
- Check if ingredient reported: `PK=INVALID_INGREDIENT#<name>, SK=USER#<user_id>`
- List pending reports (admin): `GSI1PK=REPORTS#PENDING, Sort by total DESC`

## Global Secondary Indexes (GSI)

### GSI1: User-Based Queries
```
GSI1PK: Entity owner or type
GSI1SK: Timestamp or secondary sort key

Use Cases:
  - ROLE#admin → List all admins
  - USER#<id> → User's recipes, ratings, comments
  - USER#<id>#FAVORITE → User's favorite recipes
  - USER#<id>#UNREAD → Unread notifications
  - CATEGORY#<category> → Ingredients by category
```

**Projection**: ALL (include all attributes)

### GSI2: Search & Discovery
```
GSI2PK: Category or filter type
GSI2SK: Composite sort key (rating + timestamp)

Use Cases:
  - METHOD#stir-fry → Recipes by cooking method
  - CUISINE#Vietnamese → Recipes by cuisine
  - MEALTYPE#main → Recipes by meal type
  - INGREDIENT#SEARCH → Ingredient fuzzy search
  - APPROVED → Browse approved recipes
```

**Projection**: ALL

### GSI3: Social Feed
```
GSI3PK: Feed type
GSI3SK: Timestamp

Use Cases:
  - FEED#PUBLIC → Public posts (explore feed)
  - FEED#<user_id> → User's personalized feed
```

**Projection**: ALL

## Capacity Planning

### Read/Write Patterns (MVP - 1,000 users)

```
Daily Operations (estimate):
  Users:
    - Login: 500 reads/day
    - Profile updates: 100 writes/day

  Ingredients:
    - List ingredients: 1,000 reads/day
    - Add/remove: 200 writes/day

  AI Suggestions:
    - Requests: 1,000/day (peak feature)
    - DB queries: 4,000 reads/day
    - Save suggestions: 1,000 writes/day

  Recipes:
    - View details: 3,000 reads/day
    - Create (AI): 200 writes/day

  Cooking History:
    - Start/complete: 300 writes/day
    - View history: 500 reads/day

  Ratings:
    - Submit rating: 150 writes/day
    - Query ratings: 300 reads/day

  Social (Phase 2):
    - Posts: 200 writes/day
    - Comments: 500 writes/day
    - Reactions: 1,000 writes/day
    - Feed reads: 2,000 reads/day

Total Daily:
  Reads: ~12,000/day = ~139/hour = ~0.04/second
  Writes: ~3,250/day = ~38/hour = ~0.01/second
```

### Pricing Calculation (On-Demand)

```
DynamoDB On-Demand Pricing:
  Writes: $1.25 per million write request units
  Reads: $0.25 per million read request units

Monthly (30 days):
  Reads: 12,000 × 30 = 360,000/month
    Cost: 360,000 / 1,000,000 × $0.25 = $0.09

  Writes: 3,250 × 30 = 97,500/month
    Cost: 97,500 / 1,000,000 × $1.25 = $0.12

  Storage: ~200MB = ~$0.05

  GSI (3 indexes): ~$0.20

  Backups (PITR): ~$20

Total: ~$20-25/month (MVP)

As scale grows (10,000 users):
  Reads: 3.6M/month = $0.90
  Writes: 975K/month = $1.22
  Storage: 2GB = $0.50
  GSI: $2.00
  Backups: $40

Total: ~$45-50/month
```

## Data Lifecycle Management

### TTL (Time-To-Live) Policies

```javascript
// AI Suggestions - Keep 90 days
{
  "PK": "USER#<user_id>",
  "SK": "SUGGESTION#<timestamp>",
  "ttl": Math.floor(Date.now() / 1000) + (90 * 24 * 60 * 60)
}

// Notifications - Keep 30 days
{
  "PK": "USER#<user_id>",
  "SK": "NOTIFICATION#<timestamp>",
  "ttl": Math.floor(Date.now() / 1000) + (30 * 24 * 60 * 60)
}

// Old cooking history - Archive after 1 year (optional)
{
  "PK": "USER#<user_id>",
  "SK": "COOKING#<timestamp>",
  "ttl": Math.floor(Date.now() / 1000) + (365 * 24 * 60 * 60)
}
```

### Backup Strategy

```yaml
DynamoDB Backup:
  Method: Point-in-Time Recovery (PITR)
  Retention: 35 days
  Recovery: Any point in last 35 days
  Cost: ~$20-40/month

Manual Backups:
  Frequency: Weekly (before major updates)
  Retention: 3 months
  Tool: AWS Backup service
```

## Query Optimization

### Best Practices

1. **Use Composite Sort Keys**
   ```
   Bad:  SK = "RECIPE"
   Good: SK = "RECIPE#2025-01-20T10:00:00Z"

   Why: Enables range queries and sorting
   ```

2. **Denormalize for Read Performance**
   ```json
   // Store user name in post (avoid extra query)
   {
     "PK": "POST#606",
     "SK": "METADATA",
     "user_id": "123",
     "user_name": "John Doe",  // Denormalized
     "user_avatar": "https://..." // Denormalized
   }
   ```

3. **Use GSI Sparse Indexes**
   ```json
   // Only index favorites (save cost)
   {
     "PK": "USER#123",
     "SK": "COOKING#...",
     "is_favorite": true,
     "GSI1PK": "USER#123#FAVORITE"  // Only if favorite
   }
   ```

4. **Batch Operations**
   ```javascript
   // Get multiple recipes at once
   const params = {
     RequestItems: {
       'smart-cooking-data': {
         Keys: [
           { PK: 'RECIPE#101', SK: 'METADATA' },
           { PK: 'RECIPE#102', SK: 'METADATA' },
           { PK: 'RECIPE#103', SK: 'METADATA' }
         ]
       }
     }
   };
   await docClient.batchGet(params);
   ```

### Anti-Patterns to Avoid

**Scan Operations**: Never use Scan in production
**Hot Partitions**: Avoid using user_id as PK for high-traffic items
**Large Items**: Keep items < 100KB (max 400KB)
**Unbounded Queries**: Always use pagination

## Related Documents

- [10 - Architecture](10-architecture.md)
- [12 - API Spec](12-api-spec.md)
- [20 - Backend](20-backend.md)
- [31 - Scaling](31-scaling.md)
