---
title: "20 - Backend Implementation"
weight: 20
---

# Backend Implementation - Smart Cooking App

##    Lambda Functions Architecture

### Function Overview

```yaml
Lambda Functions (11 total):
  1. auth-handler (256MB, 10s)
  2. recipe-crud (512MB, 30s) 
  3. ai-suggestion (1024MB, 60s) ⭐ Core
  4. user-profile (256MB, 10s)
  5. cooking-history (256MB, 10s) ⭐ New
  6. rating-handler (256MB, 10s) ⭐ New
  7. ingredient-validator (256MB, 10s) ⭐ New
  8. social-friends (256MB, 10s)
  9. posts-handler (512MB, 30s)
  10. notifications (256MB, 10s)
  11. admin-ops (512MB, 30s)

Runtime: Node.js 20 (all functions)
Deployment: AWS SAM or CDK
```

##    Core Lambda Functions

### 1. AI Suggestion Engine (ai-suggestion) ⭐

**Purpose**: Flexible DB/AI mix for recipe suggestions

```javascript
// Main handler
exports.handler = async (event, context) => {
  const { ingredients, recipe_count = 1 } = JSON.parse(event.body);
  const userId = event.requestContext.authorizer.claims.sub;

  try {
    // Step 1: Validate ingredients
    const validation = await validateIngredients(ingredients);
    if (validation.invalid.length === ingredients.length) {
      return errorResponse(400, 'all_ingredients_invalid', validation);
    }

    // Step 2: Get user context for AI personalization
    const userContext = await getUserContext(userId);

    // Step 3: Query DB for approved recipes (diverse categories)
    const dbRecipes = await queryDiverseRecipes(
      validation.valid, 
      userContext, 
      recipe_count
    );

    // Step 4: Calculate AI gap
    const aiGap = Math.max(0, recipe_count - dbRecipes.length);

    // Step 5: Generate AI recipes if needed
    const aiRecipes = aiGap > 0 
      ? await generateAIRecipes(validation.valid, userContext, aiGap)
      : [];

    // Step 6: Save suggestion history
    await saveSuggestionHistory({
      userId,
      ingredients: validation.valid,
      requestedCount: recipe_count,
      dbCount: dbRecipes.length,
      aiCount: aiRecipes.length,
      invalidIngredients: validation.invalid
    });

    return successResponse({
      suggestions: [...dbRecipes, ...aiRecipes],
      stats: {
        requested: recipe_count,
        from_database: dbRecipes.length,
        from_ai: aiRecipes.length
      },
      warnings: validation.warnings
    });

  } catch (error) {
    console.error('AI Suggestion Error:', error);
    return errorResponse(500, 'ai_generation_failed');
  }
};

// Validate ingredients with master table
async function validateIngredients(ingredients) {
  const valid = [];
  const invalid = [];
  const warnings = [];

  for (const ingredient of ingredients) {
    const normalized = normalizeText(ingredient);
    
    // Check exact match
    const exactMatch = await ddb.query({
      TableName: process.env.DYNAMODB_TABLE,
      IndexName: 'GSI2',
      KeyConditionExpression: 'GSI2PK = :pk AND GSI2SK = :sk',
      ExpressionAttributeValues: {
        ':pk': 'INGREDIENT#SEARCH',
        ':sk': `NAME#${normalized}`
      }
    });

    if (exactMatch.Items?.length > 0) {
      valid.push(exactMatch.Items[0].name);
    } else {
      // Fuzzy search
      const similar = await fuzzySearchIngredients(normalized);
      
      if (similar.length > 0 && similar[0].confidence >= 0.8) {
        // Auto-correct
        valid.push(similar[0].name);
        warnings.push({
          original: ingredient,
          corrected: similar[0].name,
          confidence: similar[0].confidence
        });
      } else {
        // Invalid - log and report
        invalid.push(ingredient);
        await logInvalidIngredient(ingredient, userId);
        warnings.push({
          ingredient,
          message: 'Nguyên liệu không hợp lệ',
          suggestions: similar.slice(0, 5).map(s => s.name),
          reported: true
        });
      }
    }
  }

  return { valid, invalid, warnings };
}

// Query diverse recipes by categories
async function queryDiverseRecipes(ingredients, userContext, count) {
  const categories = ['stir-fry', 'soup', 'steam', 'grill', 'bake'];
  const results = [];

  // Try to get recipes from different categories
  for (const method of categories) {
    if (results.length >= count) break;

    const recipes = await ddb.query({
      TableName: process.env.DYNAMODB_TABLE,
      IndexName: 'GSI2',
      KeyConditionExpression: 'GSI2PK = :pk',
      FilterExpression: 'is_approved = :approved AND contains(ingredients_list, :ing)',
      ExpressionAttributeValues: {
        ':pk': `METHOD#${method}`,
        ':approved': true,
        ':ing': ingredients[0] // Main ingredient
      },
      Limit: 1
    });

    if (recipes.Items?.length > 0) {
      results.push({
        ...recipes.Items[0],
        source: 'database',
        match_score: calculateMatchScore(recipes.Items[0], ingredients)
      });
    }
  }

  return results;
}

// Generate AI recipes with Bedrock
async function generateAIRecipes(ingredients, userContext, count) {
  const recipes = [];
  const usedMethods = ['stir-fry', 'soup', 'steam', 'grill', 'bake'];

  for (let i = 0; i < count; i++) {
    const method = usedMethods[i % usedMethods.length];
    
    try {
      const recipe = await callBedrock(ingredients, userContext, method);
      recipes.push({
        ...recipe,
        recipe_id: `ai-gen-${generateUUID()}`,
        source: 'ai',
        is_new: true,
        is_approved: false
      });
    } catch (error) {
      console.error(`AI generation failed for ${method}:`, error);
      // Continue with other recipes
    }
  }

  return recipes;
}
```

### 2. Cooking History Handler (cooking-history) ⭐

**Purpose**: Track personal cooking sessions and favorites

```javascript
exports.handler = async (event, context) => {
  const { httpMethod, pathParameters } = event;
  const userId = event.requestContext.authorizer.claims.sub;

  switch (httpMethod) {
    case 'POST':
      return await startCooking(event, userId);
    case 'PUT':
      return await completeCooking(event, userId, pathParameters.id);
    case 'GET':
      return await getCookingHistory(event, userId);
    case 'DELETE':
      return await deleteCookingHistory(userId, pathParameters.id);
    default:
      return errorResponse(405, 'method_not_allowed');
  }
};

// Start cooking session
async function startCooking(event, userId) {
  const { recipe_id, suggestion_id } = JSON.parse(event.body);
  const historyId = generateUUID();

  await ddb.put({
    TableName: process.env.DYNAMODB_TABLE,
    Item: {
      PK: `USER#${userId}`,
      SK: `COOKING#${new Date().toISOString()}#${historyId}`,
      history_id: historyId,
      user_id: userId,
      recipe_id,
      suggestion_id,
      status: 'cooking',
      created_at: new Date().toISOString(),
      updated_at: new Date().toISOString()
    }
  });

  return successResponse({
    history_id: historyId,
    message: 'Bắt đầu nấu ăn!'
  });
}

// Complete cooking session
async function completeCooking(event, userId, historyId) {
  await ddb.update({
    TableName: process.env.DYNAMODB_TABLE,
    Key: {
      PK: `USER#${userId}`,
      SK: `COOKING#${historyId}`
    },
    UpdateExpression: 'SET #status = :status, cook_date = :date, updated_at = :updated',
    ExpressionAttributeNames: {
      '#status': 'status'
    },
    ExpressionAttributeValues: {
      ':status': 'completed',
      ':date': new Date().toISOString(),
      ':updated': new Date().toISOString()
    }
  });

  return successResponse({
    message: 'Hoàn thành! Vui lòng đánh giá món ăn.',
    show_rating_form: true
  });
}

// Get cooking history
async function getCookingHistory(event, userId) {
  const { favorites } = event.queryStringParameters || {};

  let filterExpression = undefined;
  if (favorites === 'true') {
    filterExpression = 'is_favorite = :favorite';
  }

  const result = await ddb.query({
    TableName: process.env.DYNAMODB_TABLE,
    KeyConditionExpression: 'PK = :pk AND begins_with(SK, :sk)',
    FilterExpression: filterExpression,
    ExpressionAttributeValues: {
      ':pk': `USER#${userId}`,
      ':sk': 'COOKING#',
      ...(filterExpression && { ':favorite': true })
    },
    ScanIndexForward: false // Newest first
  });

  return successResponse({
    history: result.Items || []
  });
}
```

### 3. Rating Handler (rating-handler) ⭐

**Purpose**: Auto-approval system based on ratings

```javascript
exports.handler = async (event, context) => {
  const { recipe_id } = event.pathParameters;
  const { rating, comment, history_id } = JSON.parse(event.body);
  const userId = event.requestContext.authorizer.claims.sub;

  // Validate rating
  if (rating < 1 || rating > 5) {
    return errorResponse(400, 'invalid_rating', 'Rating must be 1-5');
  }

  try {
    // Step 1: Save rating
    await saveRating(recipe_id, userId, rating, comment, history_id);

    // Step 2: Update cooking history
    if (history_id) {
      await updateCookingHistory(userId, history_id, rating);
    }

    // Step 3: Calculate average rating
    const avgRating = await calculateAverageRating(recipe_id);
    const ratingCount = await getRatingCount(recipe_id);

    // Step 4: Auto-approval check
    if (avgRating >= 4.0) {
      await approveRecipe(recipe_id, avgRating, ratingCount);
      
      return successResponse({
        success: true,
        rating_saved: true,
        average_rating: avgRating,
        rating_count: ratingCount,
        auto_approved: true,
        message: 'Công thức đã được thêm vào database!'
      });
    }

    return successResponse({
      success: true,
      rating_saved: true,
      average_rating: avgRating,
      rating_count: ratingCount,
      auto_approved: false,
      message: 'Cảm ơn đánh giá của bạn!'
    });

  } catch (error) {
    console.error('Rating Error:', error);
    return errorResponse(500, 'rating_failed');
  }
};

// Auto-approve recipe
async function approveRecipe(recipeId, avgRating, ratingCount) {
  await ddb.update({
    TableName: process.env.DYNAMODB_TABLE,
    Key: {
      PK: `RECIPE#${recipeId}`,
      SK: 'METADATA'
    },
    UpdateExpression: `
      SET is_approved = :approved, 
          is_public = :public,
          approval_type = :type,
          average_rating = :rating,
          rating_count = :count,
          approved_at = :approved_at,
          updated_at = :updated_at
    `,
    ExpressionAttributeValues: {
      ':approved': true,
      ':public': true,
      ':type': 'auto_rating',
      ':rating': avgRating,
      ':count': ratingCount,
      ':approved_at': new Date().toISOString(),
      ':updated_at': new Date().toISOString()
    }
  });
}
```

### 4. Ingredient Validator (ingredient-validator) ⭐

**Purpose**: Validate ingredients and fuzzy search

```javascript
exports.handler = async (event, context) => {
  const { ingredients } = JSON.parse(event.body);
  const userId = event.requestContext.authorizer.claims.sub;

  const validation = await validateIngredientsWithMaster(ingredients, userId);

  return successResponse(validation);
};

// Fuzzy search for similar ingredients
async function fuzzySearchIngredients(searchTerm) {
  // Simple fuzzy search implementation
  const allIngredients = await ddb.query({
    TableName: process.env.DYNAMODB_TABLE,
    IndexName: 'GSI1',
    KeyConditionExpression: 'GSI1PK = :pk',
    ExpressionAttributeValues: {
      ':pk': 'INGREDIENT#ACTIVE'
    }
  });

  const results = [];
  
  for (const item of allIngredients.Items || []) {
    const similarity = calculateSimilarity(searchTerm, item.normalized_name);
    if (similarity >= 0.3) {
      results.push({
        name: item.name,
        confidence: similarity
      });
    }
  }

  return results.sort((a, b) => b.confidence - a.confidence);
}

// Log invalid ingredient for admin review
async function logInvalidIngredient(ingredient, userId) {
  const normalized = normalizeText(ingredient);

  // Log to CloudWatch
  console.log(JSON.stringify({
    level: 'WARN',
    event: 'invalid_ingredient',
    user_id: userId,
    ingredient,
    normalized,
    timestamp: new Date().toISOString()
  }));

  // Increment report count in database
  try {
    await ddb.update({
      TableName: process.env.DYNAMODB_TABLE,
      Key: {
        PK: `INVALID_INGREDIENT#${normalized}`,
        SK: `USER#${userId}`
      },
      UpdateExpression: 'ADD report_count :inc SET updated_at = :updated',
      ExpressionAttributeValues: {
        ':inc': 1,
        ':updated': new Date().toISOString()
      }
    });
  } catch (error) {
    // Create new report entry
    await ddb.put({
      TableName: process.env.DYNAMODB_TABLE,
      Item: {
        PK: `INVALID_INGREDIENT#${normalized}`,
        SK: `USER#${userId}`,
        ingredient_name: ingredient,
        normalized_name: normalized,
        report_count: 1,
        created_at: new Date().toISOString(),
        updated_at: new Date().toISOString()
      }
    });
  }
}
```

##    Shared Utilities

### DynamoDB Helper

```javascript
// shared/dynamodb.js
const { DynamoDBClient } = require('@aws-sdk/client-dynamodb');
const { DynamoDBDocumentClient, QueryCommand, PutCommand, UpdateCommand } = require('@aws-sdk/lib-dynamodb');

const client = new DynamoDBClient({});
const ddb = DynamoDBDocumentClient.from(client);

module.exports = {
  query: (params) => ddb.send(new QueryCommand(params)),
  put: (params) => ddb.send(new PutCommand(params)),
  update: (params) => ddb.send(new UpdateCommand(params)),
  // Add other operations as needed
};
```

### Response Helpers

```javascript
// shared/responses.js
function successResponse(data, statusCode = 200) {
  return {
    statusCode,
    headers: {
      'Content-Type': 'application/json',
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Headers': 'Content-Type,Authorization'
    },
    body: JSON.stringify(data)
  };
}

function errorResponse(statusCode, error, message, details = {}) {
  return {
    statusCode,
    headers: {
      'Content-Type': 'application/json',
      'Access-Control-Allow-Origin': '*'
    },
    body: JSON.stringify({
      error,
      message,
      details,
      timestamp: new Date().toISOString()
    })
  };
}

module.exports = { successResponse, errorResponse };
```

### Bedrock Client

```javascript
// shared/bedrock-client.js
const { BedrockRuntimeClient, InvokeModelCommand } = require('@aws-sdk/client-bedrock-runtime');

const client = new BedrockRuntimeClient({ region: 'us-east-1' });

async function callBedrock(ingredients, userContext, cookingMethod) {
  const prompt = buildPrompt(ingredients, userContext, cookingMethod);

  const response = await client.send(new InvokeModelCommand({
    modelId: 'anthropic.claude-3-haiku-20240307-v1:0',
    contentType: 'application/json',
    accept: 'application/json',
    body: JSON.stringify({
      anthropic_version: 'bedrock-2023-05-31',
      max_tokens: 2048,
      messages: [{
        role: 'user',
        content: prompt
      }]
    })
  }));

  const responseBody = JSON.parse(new TextDecoder().decode(response.body));
  return JSON.parse(responseBody.content[0].text);
}

function buildPrompt(ingredients, userContext, cookingMethod) {
  const age = userContext.birth_year 
    ? new Date().getFullYear() - userContext.birth_year 
    : null;

  return `Bạn là đầu bếp chuyên nghiệp. Tạo công thức với:

**Nguyên liệu**: ${ingredients.join(', ')}
**Phương pháp**: ${cookingMethod}
**Người dùng**: ${age ? `${age} tuổi` : 'người lớn'}, ${userContext.gender || 'không rõ giới tính'}
**Quốc gia**: ${userContext.country || 'Việt Nam'}
**Dị ứng tránh**: ${userContext.allergies?.join(', ') || 'Không'}

Trả về JSON:
{
  "name": "Tên món",
  "cooking_method": "${cookingMethod}",
  "ingredients": [{"ingredient_name": "...", "quantity": "..."}],
  "instructions": [{"step_number": 1, "description": "..."}]
}`;
}

module.exports = { callBedrock };
```

##    Error Handling & Logging

### Centralized Error Handler

```javascript
// shared/error-handler.js
class AppError extends Error {
  constructor(message, statusCode, errorCode) {
    super(message);
    this.statusCode = statusCode;
    this.errorCode = errorCode;
    this.isOperational = true;
  }
}

function handleError(error, context) {
  console.error('Lambda Error:', {
    error: error.message,
    stack: error.stack,
    context: context.functionName,
    requestId: context.awsRequestId
  });

  if (error.isOperational) {
    return errorResponse(error.statusCode, error.errorCode, error.message);
  }

  return errorResponse(500, 'internal_server_error', 'Something went wrong');
}

module.exports = { AppError, handleError };
```

### Structured Logging

```javascript
// shared/logger.js
function log(level, message, metadata = {}) {
  console.log(JSON.stringify({
    timestamp: new Date().toISOString(),
    level,
    message,
    ...metadata
  }));
}

module.exports = {
  info: (message, meta) => log('INFO', message, meta),
  warn: (message, meta) => log('WARN', message, meta),
  error: (message, meta) => log('ERROR', message, meta)
};
```

## Related Documents

- [10 - Architecture](10-architecture.md)
- [11 - Database](11-database.md)
- [12 - API Spec](12-api-spec.md)
- [21 - Frontend](21-frontend.md)
- [22 - DevOps](22-devops.md)