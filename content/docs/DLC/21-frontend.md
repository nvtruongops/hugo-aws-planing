---
title: "21 - Frontend Implementation"
weight: 21
---

# Frontend Implementation - Smart Cooking App

## 🎨 Technology Stack

```yaml
Framework: Node.js + Express.js
Template Engine: EJS
Styling: Tailwind CSS + Custom Components
State Management: Client-side JavaScript
Authentication: AWS Cognito SDK
HTTP Client: Axios (client) + AWS SDK (server)
Deployment: AWS Amplify Hosting
```

## 📁 Project Structure

```
smart-cooking-webapp/
├── src/
│   ├── routes/
│   │   ├── index.js             # Landing page
│   │   ├── auth.js              # Auth routes (login, register, verify)
│   │   ├── dashboard.js         # Dashboard & ingredients
│   │   ├── recipes.js           # Recipe routes (detail, search, suggestions)
│   │   ├── cooking.js           # Cooking history & sessions
│   │   ├── social.js            # Social feed, friends, profiles
│   │   └── settings.js          # Profile & privacy settings
│   ├── views/
│   │   ├── layouts/
│   │   │   └── main.ejs         # Main layout template
│   │   ├── auth/
│   │   │   ├── login.ejs        # Login page
│   │   │   ├── register.ejs     # Registration
│   │   │   └── verify.ejs       # Email verification
│   │   ├── dashboard/
│   │   │   ├── index.ejs        # User dashboard
│   │   │   └── ingredients.ejs  # Manage ingredients
│   │   ├── recipes/
│   │   │   ├── detail.ejs       # Recipe detail
│   │   │   ├── search.ejs       # Recipe search
│   │   │   └── suggestions.ejs  # AI suggestions
│   │   ├── cooking/
│   │   │   ├── history.ejs      # Cooking history
│   │   │   └── session.ejs      # Cooking session
│   │   ├── social/
│   │   │   ├── feed.ejs         # Social feed
│   │   │   ├── friends.ejs      # Friends list
│   │   │   └── profile.ejs      # User profile
│   │   └── settings/
│   │       ├── profile.ejs      # Profile settings
│   │       └── privacy.ejs      # Privacy settings
│   ├── middleware/
│   │   ├── auth.js              # Authentication middleware
│   │   ├── validation.js        # Input validation
│   │   └── errorHandler.js      # Error handling
│   ├── services/
│   │   ├── apiClient.js         # API client
│   │   ├── authService.js       # Auth helpers
│   │   └── utils.js             # Utilities
│   └── public/
│       ├── css/
│       │   └── styles.css       # Global styles
│       ├── js/
│       │   ├── auth.js          # Auth frontend logic
│       │   ├── recipes.js       # Recipe frontend logic
│       │   └── ingredients.js   # Ingredients frontend logic
│       └── images/
├── app.js                       # Express app setup
├── server.js                    # Server entry point
└── package.json                 # Dependencies
```

## 🔐 Authentication Implementation

### Auth Service (Node.js)

```javascript
// src/services/authService.js
const { CognitoIdentityProviderClient, InitiateAuthCommand, SignUpCommand } = require('@aws-sdk/client-cognito-identity-provider');
const jwt = require('jsonwebtoken');

class AuthService {
  constructor() {
    this.cognitoClient = new CognitoIdentityProviderClient({
      region: process.env.AWS_REGION
    });
    this.clientId = process.env.COGNITO_CLIENT_ID;
  }

  async login(email, password) {
    try {
      const command = new InitiateAuthCommand({
        AuthFlow: 'USER_PASSWORD_AUTH',
        ClientId: this.clientId,
        AuthParameters: {
          USERNAME: email,
          PASSWORD: password
        }
      });

      const result = await this.cognitoClient.send(command);
      
      return {
        accessToken: result.AuthenticationResult.AccessToken,
        refreshToken: result.AuthenticationResult.RefreshToken,
        idToken: result.AuthenticationResult.IdToken
      };
    } catch (error) {
      console.error('Login error:', error);
      throw new Error('Đăng nhập thất bại');
    }
  }

  async register(userData) {
    try {
      const command = new SignUpCommand({
        ClientId: this.clientId,
        Username: userData.email,
        Password: userData.password,
        UserAttributes: [
          { Name: 'email', Value: userData.email },
          { Name: 'name', Value: userData.fullName },
          { Name: 'custom:username', Value: userData.username }
        ]
      });

      const result = await this.cognitoClient.send(command);
      return result;
    } catch (error) {
      console.error('Registration error:', error);
      throw new Error('Đăng ký thất bại');
    }
  }

  verifyToken(token) {
    try {
      // Verify JWT token (simplified - in production use proper JWT verification)
      const decoded = jwt.decode(token);
      return decoded;
    } catch (error) {
      throw new Error('Token không hợp lệ');
    }
  }

  extractUserFromToken(token) {
    const decoded = this.verifyToken(token);
    return {
      id: decoded.sub,
      email: decoded.email,
      username: decoded['custom:username'],
      fullName: decoded.name,
      role: decoded['custom:role'] || 'user'
    };
  }
}

module.exports = new AuthService();
```

### Auth Middleware

```javascript
// src/middleware/auth.js
const authService = require('../services/authService');

const requireAuth = (req, res, next) => {
  const token = req.headers.authorization?.replace('Bearer ', '') || req.cookies.accessToken;
  
  if (!token) {
    return res.redirect('/auth/login');
  }

  try {
    const user = authService.extractUserFromToken(token);
    req.user = user;
    res.locals.user = user;
    next();
  } catch (error) {
    res.clearCookie('accessToken');
    return res.redirect('/auth/login');
  }
};

const optionalAuth = (req, res, next) => {
  const token = req.headers.authorization?.replace('Bearer ', '') || req.cookies.accessToken;
  
  if (token) {
    try {
      const user = authService.extractUserFromToken(token);
      req.user = user;
      res.locals.user = user;
    } catch (error) {
      res.clearCookie('accessToken');
    }
  }
  
  next();
};

module.exports = { requireAuth, optionalAuth };
```

### Auth Routes

```javascript
// src/routes/auth.js
const express = require('express');
const authService = require('../services/authService');
const { body, validationResult } = require('express-validator');

const router = express.Router();

// Login page
router.get('/login', (req, res) => {
  if (req.cookies.accessToken) {
    return res.redirect('/dashboard');
  }
  res.render('auth/login', { 
    title: 'Đăng nhập',
    error: req.query.error 
  });
});

// Login handler
router.post('/login', [
  body('email').isEmail().withMessage('Email không hợp lệ'),
  body('password').isLength({ min: 8 }).withMessage('Mật khẩu phải có ít nhất 8 ký tự')
], async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.render('auth/login', {
      title: 'Đăng nhập',
      error: errors.array()[0].msg,
      email: req.body.email
    });
  }

  try {
    const { email, password } = req.body;
    const tokens = await authService.login(email, password);
    
    // Set HTTP-only cookie
    res.cookie('accessToken', tokens.accessToken, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      maxAge: 24 * 60 * 60 * 1000 // 24 hours
    });
    
    res.redirect('/dashboard');
  } catch (error) {
    res.render('auth/login', {
      title: 'Đăng nhập',
      error: error.message,
      email: req.body.email
    });
  }
});

// Register page
router.get('/register', (req, res) => {
  if (req.cookies.accessToken) {
    return res.redirect('/dashboard');
  }
  res.render('auth/register', { 
    title: 'Đăng ký tài khoản',
    error: req.query.error 
  });
});

// Register handler
router.post('/register', [
  body('email').isEmail().withMessage('Email không hợp lệ'),
  body('password').isLength({ min: 8 }).withMessage('Mật khẩu phải có ít nhất 8 ký tự'),
  body('fullName').notEmpty().withMessage('Họ tên không được để trống'),
  body('username').isLength({ min: 3 }).withMessage('Username phải có ít nhất 3 ký tự')
], async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.render('auth/register', {
      title: 'Đăng ký tài khoản',
      error: errors.array()[0].msg,
      formData: req.body
    });
  }

  try {
    const userData = req.body;
    await authService.register(userData);
    
    res.render('auth/verify', {
      title: 'Xác thực email',
      email: userData.email,
      message: 'Vui lòng kiểm tra email để xác thực tài khoản'
    });
  } catch (error) {
    res.render('auth/register', {
      title: 'Đăng ký tài khoản',
      error: error.message,
      formData: req.body
    });
  }
});

// Logout
router.post('/logout', (req, res) => {
  res.clearCookie('accessToken');
  res.redirect('/');
});

module.exports = router;
```

## 🍳 Core Components

### AI Suggestion Card

```typescript
// components/recipes/SuggestionCard.tsx
'use client';

import { useState } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';
import { Clock, Users, Star, Sparkles } from 'lucide-react';
import { Recipe } from '@/types/recipe';

interface SuggestionCardProps {
  recipe: Recipe;
  onStartCooking: (recipeId: string) => void;
}

export default function SuggestionCard({ recipe, onStartCooking }: SuggestionCardProps) {
  const [isStarting, setIsStarting] = useState(false);

  const handleStartCooking = async () => {
    setIsStarting(true);
    try {
      await onStartCooking(recipe.recipe_id);
    } finally {
      setIsStarting(false);
    }
  };

  return (
    <Card className="h-full">
      <CardHeader>
        <div className="flex items-start justify-between">
          <CardTitle className="text-lg">{recipe.name}</CardTitle>
          <div className="flex gap-1">
            {recipe.source === 'ai' && (
              <Badge variant="secondary" className="flex items-center gap-1">
                <Sparkles className="h-3 w-3" />
                AI
              </Badge>
            )}
            {recipe.is_new && (
              <Badge variant="outline">Mới</Badge>
            )}
          </div>
        </div>
      </CardHeader>

      <CardContent className="space-y-4">
        <div className="flex items-center gap-4 text-sm text-muted-foreground">
          <div className="flex items-center gap-1">
            <Clock className="h-4 w-4" />
            {recipe.prep_time_minutes + recipe.cook_time_minutes} phút
          </div>
          <div className="flex items-center gap-1">
            <Users className="h-4 w-4" />
            {recipe.servings} người
          </div>
          {recipe.average_rating && (
            <div className="flex items-center gap-1">
              <Star className="h-4 w-4 fill-yellow-400 text-yellow-400" />
              {recipe.average_rating}
            </div>
          )}
        </div>

        <div className="space-y-2">
          <h4 className="font-medium">Nguyên liệu:</h4>
          <div className="flex flex-wrap gap-1">
            {recipe.ingredients?.slice(0, 3).map((ing, index) => (
              <Badge key={index} variant="outline" className="text-xs">
                {ing.ingredient_name}
              </Badge>
            ))}
            {recipe.ingredients?.length > 3 && (
              <Badge variant="outline" className="text-xs">
                +{recipe.ingredients.length - 3} khác
              </Badge>
            )}
          </div>
        </div>

        <div className="space-y-2">
          <h4 className="font-medium">Phương pháp:</h4>
          <Badge variant="secondary">{recipe.cooking_method}</Badge>
        </div>

        {recipe.source === 'ai' && (
          <div className="text-sm text-muted-foreground bg-blue-50 p-2 rounded">
            💡 Món mới từ AI - Hãy thử và đánh giá để giúp cộng đồng!
          </div>
        )}

        <Button 
          onClick={handleStartCooking} 
          className="w-full"
          disabled={isStarting}
        >
          {isStarting ? 'Đang bắt đầu...' : 'Bắt đầu nấu'}
        </Button>
      </CardContent>
    </Card>
  );
}
```

### Ingredient Input Component

```typescript
// components/ingredients/IngredientInput.tsx
'use client';

import { useState } from 'react';
import { useForm } from 'react-hook-form';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Alert, AlertDescription } from '@/components/ui/alert';
import { Badge } from '@/components/ui/badge';
import { X, Plus, AlertTriangle } from 'lucide-react';
import { useIngredientStore } from '@/lib/stores/ingredientStore';

export default function IngredientInput() {
  const [inputValue, setInputValue] = useState('');
  const [validationResult, setValidationResult] = useState<any>(null);
  const { addIngredient, validateIngredients } = useIngredientStore();

  const handleValidate = async () => {
    if (!inputValue.trim()) return;

    try {
      const result = await validateIngredients([inputValue.trim()]);
      setValidationResult(result);
    } catch (error) {
      console.error('Validation error:', error);
    }
  };

  const handleAdd = async (ingredientName: string) => {
    try {
      await addIngredient(ingredientName);
      setInputValue('');
      setValidationResult(null);
    } catch (error) {
      console.error('Add ingredient error:', error);
    }
  };

  const handleAcceptCorrection = (corrected: string) => {
    handleAdd(corrected);
  };

  return (
    <div className="space-y-4">
      <div className="flex gap-2">
        <Input
          placeholder="Nhập tên nguyên liệu..."
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          onKeyPress={(e) => e.key === 'Enter' && handleValidate()}
        />
        <Button onClick={handleValidate} disabled={!inputValue.trim()}>
          <Plus className="h-4 w-4" />
        </Button>
      </div>

      {validationResult && (
        <div className="space-y-3">
          {/* Valid ingredients */}
          {validationResult.valid?.map((ingredient: string, index: number) => (
            <Alert key={index} className="border-green-200 bg-green-50">
              <AlertDescription className="flex items-center justify-between">
                <span>✅ "{ingredient}" hợp lệ</span>
                <Button size="sm" onClick={() => handleAdd(ingredient)}>
                  Thêm
                </Button>
              </AlertDescription>
            </Alert>
          ))}

          {/* Corrected ingredients */}
          {validationResult.corrected?.map((item: any, index: number) => (
            <Alert key={index} className="border-yellow-200 bg-yellow-50">
              <AlertTriangle className="h-4 w-4" />
              <AlertDescription className="flex items-center justify-between">
                <div>
                  <p>Bạn có muốn dùng "{item.matched}" thay vì "{item.original}"?</p>
                  <p className="text-xs text-muted-foreground">
                    Độ chính xác: {Math.round(item.confidence * 100)}%
                  </p>
                </div>
                <div className="flex gap-2">
                  <Button 
                    size="sm" 
                    variant="outline"
                    onClick={() => setValidationResult(null)}
                  >
                    Bỏ qua
                  </Button>
                  <Button 
                    size="sm" 
                    onClick={() => handleAcceptCorrection(item.matched)}
                  >
                    Chấp nhận
                  </Button>
                </div>
              </AlertDescription>
            </Alert>
          ))}

          {/* Invalid ingredients with suggestions */}
          {validationResult.suggestions?.map((item: any, index: number) => (
            <Alert key={index} variant="destructive">
              <X className="h-4 w-4" />
              <AlertDescription>
                <p>❌ Không tìm thấy "{item.original}"</p>
                {item.similar?.length > 0 && (
                  <div className="mt-2">
                    <p className="text-xs">Có thể bạn muốn:</p>
                    <div className="flex flex-wrap gap-1 mt-1">
                      {item.similar.slice(0, 5).map((suggestion: string, i: number) => (
                        <Badge 
                          key={i} 
                          variant="outline" 
                          className="cursor-pointer hover:bg-gray-100"
                          onClick={() => handleAdd(suggestion)}
                        >
                          {suggestion}
                        </Badge>
                      ))}
                    </div>
                  </div>
                )}
              </AlertDescription>
            </Alert>
          ))}
        </div>
      )}
    </div>
  );
}
```

### Cooking History Card

```typescript
// components/cooking/HistoryCard.tsx
'use client';

import { useState } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';
import { Star, Heart, Clock, Calendar } from 'lucide-react';
import { CookingHistory } from '@/types/recipe';
import RatingModal from './RatingModal';

interface HistoryCardProps {
  history: CookingHistory;
  onToggleFavorite: (historyId: string, isFavorite: boolean) => void;
  onRate: (historyId: string, rating: number, comment?: string) => void;
}

export default function HistoryCard({ history, onToggleFavorite, onRate }: HistoryCardProps) {
  const [showRatingModal, setShowRatingModal] = useState(false);

  const formatDate = (dateString: string) => {
    return new Date(dateString).toLocaleDateString('vi-VN');
  };

  const getStatusBadge = (status: string) => {
    const variants = {
      'completed': 'default',
      'cooking': 'secondary',
      'planned': 'outline'
    } as const;

    const labels = {
      'completed': 'Hoàn thành',
      'cooking': 'Đang nấu',
      'planned': 'Dự định'
    };

    return (
      <Badge variant={variants[status as keyof typeof variants]}>
        {labels[status as keyof typeof labels]}
      </Badge>
    );
  };

  return (
    <>
      <Card>
        <CardHeader>
          <div className="flex items-start justify-between">
            <CardTitle className="text-lg">{history.recipe_name}</CardTitle>
            <div className="flex items-center gap-2">
              {getStatusBadge(history.status)}
              <Button
                variant="ghost"
                size="sm"
                onClick={() => onToggleFavorite(history.history_id, !history.is_favorite)}
              >
                <Heart 
                  className={`h-4 w-4 ${history.is_favorite ? 'fill-red-500 text-red-500' : ''}`} 
                />
              </Button>
            </div>
          </div>
        </CardHeader>

        <CardContent className="space-y-3">
          <div className="flex items-center gap-4 text-sm text-muted-foreground">
            <div className="flex items-center gap-1">
              <Calendar className="h-4 w-4" />
              {formatDate(history.cook_date || history.created_at)}
            </div>
            {history.personal_rating && (
              <div className="flex items-center gap-1">
                <Star className="h-4 w-4 fill-yellow-400 text-yellow-400" />
                {history.personal_rating}/5
              </div>
            )}
          </div>

          {history.personal_notes && (
            <div className="text-sm bg-gray-50 p-2 rounded">
              <p className="font-medium">Ghi chú:</p>
              <p>{history.personal_notes}</p>
            </div>
          )}

          <div className="flex gap-2">
            {history.status === 'completed' && !history.personal_rating && (
              <Button 
                size="sm" 
                onClick={() => setShowRatingModal(true)}
              >
                Đánh giá món ăn
              </Button>
            )}
            
            <Button variant="outline" size="sm">
              Xem công thức
            </Button>
          </div>
        </CardContent>
      </Card>

      <RatingModal
        isOpen={showRatingModal}
        onClose={() => setShowRatingModal(false)}
        recipeName={history.recipe_name}
        onSubmit={(rating, comment) => {
          onRate(history.history_id, rating, comment);
          setShowRatingModal(false);
        }}
      />
    </>
  );
}
```

## 🔄 State Management

### Recipe Store

```typescript
// lib/stores/recipeStore.ts
import { create } from 'zustand';
import { api } from '@/lib/api';
import { Recipe, AISuggestion } from '@/types/recipe';

interface RecipeState {
  suggestions: Recipe[];
  isLoadingSuggestions: boolean;
  getSuggestions: (ingredients: string[], count?: number) => Promise<void>;
  startCooking: (recipeId: string, suggestionId?: string) => Promise<string>;
  completeCooking: (historyId: string) => Promise<void>;
  rateRecipe: (recipeId: string, rating: number, comment?: string, historyId?: string) => Promise<any>;
}

export const useRecipeStore = create<RecipeState>((set, get) => ({
  suggestions: [],
  isLoadingSuggestions: false,

  getSuggestions: async (ingredients: string[], count = 1) => {
    set({ isLoadingSuggestions: true });
    
    try {
      const response = await api.post('/ai/suggest', {
        ingredients,
        recipe_count: count
      });

      set({ 
        suggestions: response.data.suggestions,
        isLoadingSuggestions: false 
      });
    } catch (error) {
      console.error('Get suggestions error:', error);
      set({ isLoadingSuggestions: false });
      throw error;
    }
  },

  startCooking: async (recipeId: string, suggestionId?: string) => {
    const response = await api.post('/cooking/start', {
      recipe_id: recipeId,
      suggestion_id: suggestionId
    });

    return response.data.history_id;
  },

  completeCooking: async (historyId: string) => {
    await api.put(`/cooking/${historyId}/complete`);
  },

  rateRecipe: async (recipeId: string, rating: number, comment?: string, historyId?: string) => {
    const response = await api.post(`/recipes/${recipeId}/rate`, {
      rating,
      comment,
      history_id: historyId
    });

    return response.data;
  }
}));
```

## Related Documents

- [10 - Architecture](10-architecture.md)
- [20 - Backend](20-backend.md)
- [22 - DevOps](22-devops.md)
- [40 - Auth](40-auth.md)