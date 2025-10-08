---
title: "Design Brief"
menu:
  designer:
    parent: designer
    weight: 1
---

# 🎨 SMART COOKING - DESIGN BRIEF & SPECIFICATION

## 📖 Mục lục
1. [Tổng quan dự án](#tổng-quan-dự-án)
2. [Kiến trúc hệ thống](#kiến-trúc-hệ-thống)
3. [Tính năng chính](#tính-năng-chính)
4. [Design System](#design-system)
5. [Cấu trúc trang web](#cấu-trúc-trang-web)
6. [User Flows](#user-flows)
7. [Wireframes & Components](#wireframes--components)
8. [Responsive Design](#responsive-design)
9. [Accessibility](#accessibility)
10. [Brand Guidelines](#brand-guidelines)

---

## 🎯 Tổng quan dự án

### Giới thiệu
**Smart Cooking** là ứng dụng nấu ăn thông minh sử dụng AI để gợi ý công thức dựa trên nguyên liệu có sẵn, giúp tối ưu hóa việc sử dụng nguyên liệu và giảm lãng phí thực phẩm.

### Mục tiêu
- Cung cấp trải nghiệm cá nhân hóa cho người dùng với AI suggestions
- Giảm lãng phí thực phẩm thông qua gợi ý công thức thông minh
- Xây dựng cộng đồng chia sẻ công thức nấu ăn
- Theo dõi lịch sử nấu ăn và đánh giá công thức

### Target Users
- **Primary**: Người nội trợ, người yêu thích nấu ăn (18-45 tuổi)
- **Secondary**: Sinh viên, người mới học nấu ăn
- **Tertiary**: Admin quản trị hệ thống

### User Personas

#### Persona 1: Mai - Busy Mom (32 tuổi)
- **Goal**: Nấu ăn nhanh với nguyên liệu có sẵn
- **Pain Point**: Không biết nấu gì với nguyên liệu thừa
- **Tech Savvy**: Trung bình
- **Devices**: Smartphone chính, tablet khi nấu ăn

#### Persona 2: Minh - Student (22 tuổi)
- **Goal**: Học nấu ăn tiết kiệm
- **Pain Point**: Thiếu kinh nghiệm, dễ lãng phí thực phẩm
- **Tech Savvy**: Cao
- **Devices**: Smartphone

#### Persona 3: Lan - Food Enthusiast (28 tuổi)
- **Goal**: Khám phá công thức mới, chia sẻ kinh nghiệm
- **Pain Point**: Khó tìm công thức phù hợp sở thích
- **Tech Savvy**: Cao
- **Devices**: Desktop + mobile

---

## 🎯 Tính năng chính

### Phase 1: MVP (✅ Đã hoàn thành)

#### 1. Authentication & Authorization
- **Sign Up**: Email + password với validation
- **Sign In**: Email/password login
- **Forgot Password**: Password reset flow
- **Post-Auth Setup**: Auto-create user profile
- **Role-Based Access**: User vs Admin groups

#### 2. User Profile Management
- **Profile Info**: 
  - Full name, username, email
  - Avatar, bio (max 500 chars)
  - Date of birth, gender, country
- **Preferences**:
  - Dietary restrictions (allergies, vegetarian, etc.)
  - Favorite cuisines
  - Cooking methods
  - Skill level (beginner/intermediate/expert)
  - Max cooking time (15/30/60/120 minutes)
  - Household size (1-6 people)
  - Budget level (economical/moderate/premium)
  - Health goals (weight loss, muscle gain, etc.)
- **Privacy Settings**:
  - Profile visibility (public/friends/private)
  - Email visibility
  - Date of birth visibility
  - Cooking history visibility

#### 3. Ingredient Management
- **Master Ingredients**: 508 nguyên liệu Việt Nam
- **Fuzzy Matching**: AI tự động sửa lỗi chính tả
- **Categories**: Phân loại nguyên liệu
- **Search & Filter**: Tìm kiếm và lọc nguyên liệu
- **User's Pantry**: Quản lý nguyên liệu có sẵn

#### 4. AI Recipe Suggestions
- **Input**: Danh sách nguyên liệu + số lượng công thức mong muốn (1-5)
- **AI Processing**: 
  - Validate ingredients với fuzzy matching
  - Tìm kiếm trong database trước
  - Generate công thức mới bằng AI nếu cần
- **Output**:
  - Danh sách công thức phù hợp
  - Thống kê (from DB vs from AI)
  - Warnings về nguyên liệu không hợp lệ
- **Recipe Details**:
  - Title, description
  - Cuisine type, cooking method, meal type
  - Prep time, cook time, servings
  - Ingredients list (with quantities)
  - Step-by-step instructions
  - Nutritional info
  - Rating & review count

#### 5. Cooking Session Tracking
- **Start Cooking**: Bắt đầu session với recipe
- **In Progress**: Theo dõi quá trình nấu
- **Complete**: Đánh dấu hoàn thành
- **Rating**: Rate công thức (1-5 sao)
- **Notes**: Ghi chú cá nhân
- **Favorites**: Đánh dấu yêu thích

#### 6. Recipe Management
- **Browse**: Xem danh sách công thức
- **Search**: Tìm kiếm công thức
- **Filter**: Lọc theo cuisine, method, meal type, time
- **Details**: Xem chi tiết công thức
- **Rate & Review**: Đánh giá và bình luận
- **Auto-Approval**: Tự động approve nếu rating ≥ 4.0

#### 7. Cooking History
- **History List**: Danh sách công thức đã nấu
- **Stats**: Thống kê (total sessions, favorites, avg rating)
- **Filter**: Lọc theo status, date, rating
- **Re-cook**: Nấu lại công thức cũ

#### 8. Admin Dashboard
- **Overview Stats**:
  - Total users
  - Total recipes
  - Total ingredients
  - Pending approvals
  - Today's registrations
- **Quick Actions**:
  - Manage users
  - Approve recipes
  - Add ingredients
  - View analytics
- **Recent Activity Feed**:
  - New registrations
  - New recipes
  - System events

### Phase 2: Social Features (📋 Kế hoạch)

#### 1. Friend System
- Send/accept/reject friend requests
- Friend list management
- View friend's public recipes

#### 2. Social Feed
- Post cooking experiences with photos
- Like & comment on posts
- Share recipes
- Activity feed from friends

#### 3. Notifications
- Friend requests
- New recipes from friends
- Comments on posts
- System notifications

---

## 🎨 Design System

### Color Palette

#### Primary Colors
```css
/* Blue - Main Brand Color */
--blue-50:  #EFF6FF
--blue-100: #DBEAFE
--blue-200: #BFDBFE
--blue-300: #93C5FD
--blue-400: #60A5FA
--blue-500: #3B82F6  /* Primary */
--blue-600: #2563EB
--blue-700: #1D4ED8
--blue-800: #1E40AF
--blue-900: #1E3A8A
```

#### Secondary Colors
```css
/* Purple - Accent */
--purple-50:  #FAF5FF
--purple-100: #F3E8FF
--purple-500: #A855F7
--purple-600: #9333EA  /* Accent */

/* Green - Success/Food */
--green-50:  #F0FDF4
--green-100: #DCFCE7
--green-500: #22C55E
--green-600: #16A34A

/* Orange - Warning/Cooking */
--orange-50:  #FFF7ED
--orange-100: #FFEDD5
--orange-500: #F97316
--orange-600: #EA580C

/* Red - Error/Spicy */
--red-50:  #FEF2F2
--red-100: #FEE2E2
--red-500: #EF4444
--red-600: #DC2626

/* Yellow - Rating */
--yellow-400: #FACC15
--yellow-500: #EAB308
```

#### Neutral Colors
```css
/* Gray Scale */
--gray-50:  #F9FAFB
--gray-100: #F3F4F6
--gray-200: #E5E7EB
--gray-300: #D1D5DB
--gray-400: #9CA3AF
--gray-500: #6B7280
--gray-600: #4B5563
--gray-700: #374151
--gray-800: #1F2937
--gray-900: #111827
```

### Typography

#### Font Family
```css
/* Primary Font */
--font-sans: 'Geist', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;

/* Monospace Font */
--font-mono: 'Geist Mono', 'Courier New', monospace;
```

#### Font Sizes
```css
--text-xs:   0.75rem    /* 12px */
--text-sm:   0.875rem   /* 14px */
--text-base: 1rem       /* 16px */
--text-lg:   1.125rem   /* 18px */
--text-xl:   1.25rem    /* 20px */
--text-2xl:  1.5rem     /* 24px */
--text-3xl:  1.875rem   /* 30px */
--text-4xl:  2.25rem    /* 36px */
--text-5xl:  3rem       /* 48px */
```

#### Font Weights
```css
--font-normal:    400
--font-medium:    500
--font-semibold:  600
--font-bold:      700
```

#### Line Heights
```css
--leading-none:    1
--leading-tight:   1.25
--leading-snug:    1.375
--leading-normal:  1.5
--leading-relaxed: 1.625
--leading-loose:   2
```

### Spacing System
```css
/* Tailwind default spacing scale */
--spacing-0:  0px
--spacing-1:  0.25rem   /* 4px */
--spacing-2:  0.5rem    /* 8px */
--spacing-3:  0.75rem   /* 12px */
--spacing-4:  1rem      /* 16px */
--spacing-5:  1.25rem   /* 20px */
--spacing-6:  1.5rem    /* 24px */
--spacing-8:  2rem      /* 32px */
--spacing-10: 2.5rem    /* 40px */
--spacing-12: 3rem      /* 48px */
--spacing-16: 4rem      /* 64px */
--spacing-20: 5rem      /* 80px */
--spacing-24: 6rem      /* 96px */
```

### Border Radius
```css
--rounded-none: 0
--rounded-sm:   0.125rem  /* 2px */
--rounded:      0.25rem   /* 4px */
--rounded-md:   0.375rem  /* 6px */
--rounded-lg:   0.5rem    /* 8px */
--rounded-xl:   0.75rem   /* 12px */
--rounded-2xl:  1rem      /* 16px */
--rounded-3xl:  1.5rem    /* 24px */
--rounded-full: 9999px
```

### Shadows
```css
--shadow-sm:  0 1px 2px 0 rgb(0 0 0 / 0.05)
--shadow:     0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1)
--shadow-md:  0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1)
--shadow-lg:  0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1)
--shadow-xl:  0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1)
--shadow-2xl: 0 25px 50px -12px rgb(0 0 0 / 0.25)
```

### Component Styles

#### Buttons
```css
/* Primary Button */
.btn-primary {
  background: linear-gradient(135deg, #3B82F6, #9333EA);
  color: white;
  padding: 0.75rem 1.5rem;
  border-radius: 0.5rem;
  font-weight: 600;
  transition: all 0.2s;
}
.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 15px -3px rgb(59 130 246 / 0.3);
}

/* Secondary Button */
.btn-secondary {
  background: white;
  color: #3B82F6;
  border: 2px solid #3B82F6;
  padding: 0.75rem 1.5rem;
  border-radius: 0.5rem;
  font-weight: 600;
}

/* Outline Button */
.btn-outline {
  background: transparent;
  color: #4B5563;
  border: 1px solid #D1D5DB;
  padding: 0.75rem 1.5rem;
  border-radius: 0.5rem;
}
```

#### Cards
```css
.card {
  background: white;
  border-radius: 0.75rem;
  box-shadow: 0 1px 3px 0 rgb(0 0 0 / 0.1);
  padding: 1.5rem;
  border: 1px solid #F3F4F6;
}

.card:hover {
  box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1);
}
```

#### Inputs
```css
.input {
  width: 100%;
  padding: 0.75rem 1rem;
  border: 1px solid #D1D5DB;
  border-radius: 0.5rem;
  font-size: 1rem;
  transition: all 0.2s;
}

.input:focus {
  outline: none;
  border-color: #3B82F6;
  box-shadow: 0 0 0 3px rgb(59 130 246 / 0.1);
}

.input:error {
  border-color: #EF4444;
}
```

#### Badges
```css
/* Status Badges */
.badge-success {
  background: #DCFCE7;
  color: #16A34A;
  padding: 0.25rem 0.75rem;
  border-radius: 9999px;
  font-size: 0.875rem;
  font-weight: 500;
}

.badge-warning {
  background: #FFEDD5;
  color: #EA580C;
}

.badge-info {
  background: #DBEAFE;
  color: #2563EB;
}
```

### Icons
- **Library**: Heroicons (SVG icons)
- **Style**: Outline for general use, Solid for emphasis
- **Sizes**: 16px (small), 20px (medium), 24px (large)

---

## 📱 Cấu trúc trang web

### 1. Landing Page (`/`)
**Mục đích**: Giới thiệu sản phẩm, thu hút người dùng mới

**Sections**:
- **Hero Section**
  - Headline: "Smart Cooking - AI-Powered Recipe Assistant"
  - Subheadline: "Your personalized cooking assistant"
  - CTA buttons: "Get Started" + "Sign In"
  - Background: Gradient blue to purple
  
- **Features Grid** (3 columns)
  1. Smart Recipe Discovery
  2. Meal Planning
  3. Recipe Collection
  
- **How It Works** (Step-by-step)
  
- **CTA Section**
  - Sign up encouragement

**Components**:
- Navigation bar (sticky)
- Hero banner
- Feature cards
- CTA buttons
- Footer

---

### 2. Authentication Pages

#### Login Page (`/login`)
**Layout**:
- Centered card (max-width: 400px)
- Logo at top
- Email input
- Password input
- "Forgot password?" link
- Submit button (gradient)
- "Don't have an account?" → Register link

**Validation**:
- Email format
- Required fields
- Error messages (red text)

#### Register Page (`/register`)
**Fields**:
- Email (required)
- Username (required)
- Password (required, min 8 chars, uppercase, lowercase, number, special char)
- Confirm password
- Full name (required)
- Terms & conditions checkbox

**Flow**:
- Form validation
- Submit → Auto-confirm
- Redirect to preferences setup

#### Forgot Password (`/forgot-password`)
**Steps**:
1. Enter email
2. Receive code (AWS Cognito)
3. Enter code + new password
4. Confirm → Redirect to login

---

### 3. Dashboard Page (`/dashboard`)
**Layout**: 3-column grid (responsive)

**Top Stats Cards** (4 cards in row):
- Recipes Saved (icon: bookmark)
- Friends (icon: users)
- Sessions (icon: clock)
- Favorites (icon: heart)

**Main Content** (2 columns):
- **Left Column (2/3 width)**:
  - Quick Actions (2x2 grid)
    - Add Ingredients
    - Get AI Suggestions
    - View History
    - Browse Recipes
  - Recent Activity (scrollable list)
  
- **Right Column (1/3 width)**:
  - Pro Tip card (gradient blue-purple)
  - Profile Setup progress
  - Settings shortcuts

**Components**:
- Navigation (top)
- Stat cards
- Action buttons
- Activity feed
- Progress bar

---

### 4. Profile Pages

#### View Profile (`/profile`)
**Sections**:
- **Header**:
  - Cover photo (gradient)
  - Avatar (circular, 128px)
  - Name, username, email
  - Bio
  - Stats row: Recipes | Friends | Sessions
  - Edit button
  
- **Tabs**:
  - About (default)
  - Recipes
  - Cooking History
  - Friends

**About Tab**:
- Personal info
- Preferences summary
- Privacy settings link

#### Edit Profile (`/profile/edit`)
**Form Sections**:
1. **Personal Information**
   - Full name
   - Username
   - Email (readonly)
   - Date of birth (date picker)
   - Gender (select)
   - Country (select)
   - Bio (textarea, max 500 chars)
   
2. **Avatar**
   - Upload/change avatar
   - Default gradient avatar

**Actions**:
- Save Changes (blue button)
- Cancel (gray button)

#### Preferences (`/profile/preferences`)
**Form Sections**:

1. **Dietary Restrictions** (multi-select)
   - Vegetarian
   - Vegan
   - Gluten-free
   - Dairy-free
   - Nut-free
   - Custom...

2. **Allergies** (multi-select)
   - Common allergens
   - Custom input

3. **Favorite Cuisines** (multi-select)
   - Vietnamese
   - Chinese
   - Japanese
   - Korean
   - Thai
   - Western
   - Fusion

4. **Cooking Preferences**
   - Skill level (radio buttons)
     * Beginner
     * Intermediate
     * Expert
   - Max cooking time (select)
     * 15 minutes
     * 30 minutes
     * 60 minutes
     * 120 minutes
   - Household size (number input, 1-6)
   - Budget level (radio buttons)
     * Economical
     * Moderate
     * Premium

5. **Health Goals** (multi-select)
   - Weight loss
   - Muscle gain
   - General health

**UI Elements**:
- Checkboxes for multi-select
- Radio buttons for single choice
- Dropdowns for long lists
- Number steppers

---

### 5. Ingredients Page (`/ingredients`)
**Purpose**: Quản lý nguyên liệu có sẵn

**Layout**:

**Top Section**:
- Page title: "Nguyên liệu của bạn"
- Search bar (with fuzzy matching)
- "Thêm nguyên liệu" button

**Instructions Card** (blue background):
- Hướng dẫn sử dụng (3 steps)
- Icons cho mỗi step

**Ingredients List**:
- Grid layout (responsive)
- Ingredient cards:
  - Name
  - Category badge
  - Remove button (X icon)
  
**Tips Section** (green background):
- Tips về fuzzy matching
- Examples

**Empty State**:
- Illustration
- "Chưa có nguyên liệu nào"
- CTA: Add ingredients

**Components**:
- Search input with autocomplete
- Ingredient chips/tags
- Category filters
- Validation warnings

---

### 6. AI Suggestions Page (`/ai-suggestions`)
**Purpose**: Nhận gợi ý công thức từ AI

**Workflow**:

**Step 1: Input Ingredients**
- Ingredient selector (từ pantry)
- Selected ingredients display (chips)
- Recipe count slider (1-5)
- "Get Suggestions" button (gradient)

**Step 2: Loading State**
- Loading animation (pulsing)
- Progress text:
  * "✓ Validating ingredients..."
  * "✓ Searching database..."
  * "✓ Generating AI recipes..."

**Step 3: Results**
- Stats card:
  - Requested: X
  - From database: Y
  - From AI: Z
  
- Warnings section (if any):
  - Invalid ingredients
  - Suggestions/corrections
  
- Recipe grid (responsive):
  - Recipe cards
  - Source badge (DB/AI)
  - Quick view
  - Start cooking button

**Components**:
- Ingredient picker
- Range slider
- Loading spinner
- Stats dashboard
- Warning alerts
- Recipe cards

---

### 7. Recipes Pages

#### Recipe List (`/recipes`)
**Layout**:

**Filters Sidebar** (left, collapsible on mobile):
- Search box
- Cuisine type (checkboxes)
- Cooking method (checkboxes)
- Meal type (checkboxes)
- Max time (slider)
- Rating (star filter)
- Source (DB/AI)

**Recipe Grid** (main area):
- Card layout (3-4 columns on desktop)
- Each card shows:
  - Image placeholder (gradient)
  - Title
  - Rating (stars + count)
  - Time (prep + cook)
  - Cuisine badge
  - Method badge
  - Source icon

**Pagination**:
- Page numbers
- Previous/Next buttons

#### Recipe Detail (`/recipes/[id]`)
**Layout**:

**Header Section**:
- Recipe title (large)
- Rating + review count
- Source badge (DB/AI)
- Approval status
- Action buttons:
  * Start Cooking
  * Save to Favorites
  * Share

**Info Row** (badges):
- Cuisine type
- Cooking method
- Meal type
- Prep time
- Cook time
- Servings

**Content Tabs**:
1. **Ingredients Tab**
   - List of ingredients
   - Quantity, unit, preparation
   - Checkbox for tracking
   
2. **Instructions Tab**
   - Step-by-step (numbered)
   - Duration per step
   - Clear formatting
   
3. **Nutrition Tab**
   - Nutritional info table
   - Calories, protein, carbs, fat, fiber, sodium

**Reviews Section**:
- Review list (sorted by date)
- Each review:
  - User avatar + name
  - Star rating
  - Date
  - Comment
  - Verified cook badge
  
- "Write a Review" button (if user has cooked)

**Components**:
- Image carousel (if multiple images)
- Star rating component
- Badge components
- Tab navigation
- Review cards

---

### 8. Cooking Session Page (`/cooking`)
**Purpose**: Chế độ nấu ăn tương tác

**Layout**:

**Header**:
- Recipe name
- Timer (if set)
- Exit button

**Left Panel** (ingredients):
- Ingredients checklist
- Check off as you use

**Main Area** (instructions):
- Current step (highlighted, large text)
- Previous steps (dimmed)
- Next steps (preview)
- Step number indicator
- Navigation buttons:
  * Previous Step
  * Next Step / Complete

**Actions**:
- Mark as complete
- Add rating & notes
- Save to history

**Modal on Complete**:
- Congratulations message
- Rating stars (1-5)
- Comment textarea
- Mark as favorite checkbox
- Save button

---

### 9. History Page (`/history`)
**Layout**:

**Stats Summary** (top):
- Total sessions
- Favorites count
- Average rating

**Filters**:
- Status (all/cooking/completed)
- Date range
- Rating filter

**History List**:
- Card layout
- Each entry:
  - Recipe thumbnail
  - Recipe title
  - Date cooked
  - Status badge
  - Personal rating (stars)
  - Personal notes (preview)
  - Actions:
    * View recipe
    * Cook again
    * Edit rating

**Empty State**:
- Illustration
- "No cooking history yet"
- CTA: Browse recipes

---

### 10. Admin Dashboard (`/admin`)
**Access**: Admin users only

**Layout**:

**Stats Grid** (5 cards):
- Total Users (icon: users)
- Total Recipes (icon: book)
- Total Ingredients (icon: list)
- Pending Approvals (icon: clock, orange badge)
- Today's Registrations (icon: user-plus)

**Quick Actions** (grid):
- Manage Users
- Approve Recipes
- Add Ingredients
- View Analytics

**Recent Activity Feed**:
- Scrollable list
- Activity items:
  - Icon (based on type)
  - Description
  - Timestamp
  - View details link

**Sections**:
- User management (future)
- Recipe approval (future)
- Ingredient management (future)
- Analytics (future)

---

## 🔄 User Flows

### Flow 1: New User Registration
```
1. Landing page → Click "Get Started"
2. Register page → Fill form → Submit
3. Auto-confirm account
4. Redirect to Login
5. Login → Redirect to Dashboard
6. (Optional) Complete profile → Set preferences
```

### Flow 2: Get AI Recipe Suggestions
```
1. Dashboard → "Add Ingredients" or Go to /ingredients
2. Add ingredients to pantry (with fuzzy matching)
3. Go to AI Suggestions page
4. Select ingredients from pantry
5. Choose recipe count (1-5)
6. Click "Get Suggestions"
7. View loading states
8. See results with stats & warnings
9. Browse suggested recipes
10. Click recipe → View details
11. Start cooking → Enter cooking mode
```

### Flow 3: Cooking a Recipe
```
1. Recipe detail page → Click "Start Cooking"
2. Cooking session starts
3. Go through ingredients checklist
4. Follow step-by-step instructions
5. Mark each step as complete
6. Complete final step → Click "Complete"
7. Rate recipe (1-5 stars)
8. Add personal notes
9. Mark as favorite (optional)
10. Save to history
11. Recipe auto-approved if rating ≥ 4.0
```

### Flow 4: Browse & Search Recipes
```
1. Dashboard → "Browse Recipes" or Go to /recipes
2. Apply filters (cuisine, method, time, etc.)
3. Browse recipe grid
4. Click recipe card → View details
5. Read ingredients & instructions
6. View reviews & ratings
7. Decide: Start cooking or save for later
```

### Flow 5: Admin Approval Workflow
```
1. Admin login → Auto-redirect to /admin
2. See "Pending Approvals" count
3. Click "Approve Recipes"
4. Review list of pending recipes
5. Click recipe → View full details
6. Decision:
   - Approve → Recipe becomes public
   - Reject → Recipe stays private
   - Edit → Modify recipe metadata
```

---

## 🎨 Wireframes & Components

### Component Library

#### 1. Navigation Component
**Desktop**:
```
[Logo] [Home] [Dashboard] [Recipes] [About]        [Profile ▼]
```

**Mobile**:
```
[☰ Menu]                                    [Profile]
```

**Dropdown Menu** (on profile click):
- My Profile
- Settings
- Privacy Settings
- Sign Out

**Admin users see**:
- Admin Dashboard (highlighted)

#### 2. Recipe Card Component
```
┌─────────────────────────┐
│  [Image/Gradient]       │
│  [Source Badge]         │
├─────────────────────────┤
│  Title                  │
│  ★★★★☆ (4.2) · 15 reviews│
│  ⏱ 45 mins · 🍽 4 servings│
│  [Cuisine] [Method]     │
└─────────────────────────┘
```

**States**:
- Default
- Hover (shadow increase)
- Loading (skeleton)

#### 3. Ingredient Chip Component
```
[🥕 Carrot  ×]
```

**Variants**:
- Default (gray)
- Selected (blue)
- Invalid (red with warning icon)
- Corrected (yellow with suggestion)

#### 4. Star Rating Component
```
★★★★☆  4.2 (15 reviews)
```

**Interactive**:
- Clickable for user input
- Half-star support
- Hover preview

#### 5. Button Components
```
[Primary Button]       - Gradient blue-purple
[Secondary Button]     - White with blue border
[Outline Button]       - Transparent with gray border
[Icon Button]          - Icon only, rounded
[Text Button]          - Text only, no border
```

#### 6. Input Components
```
┌─────────────────────────┐
│ Label                   │
│ ┌─────────────────────┐ │
│ │ Input text...       │ │
│ └─────────────────────┘ │
│ Helper text / Error     │
└─────────────────────────┘
```

**Types**:
- Text input
- Email input
- Password input (with show/hide)
- Textarea
- Number input with steppers
- Date picker
- Dropdown select
- Multi-select
- Autocomplete

#### 7. Card Components
```
┌─────────────────────────┐
│ Card Header             │
├─────────────────────────┤
│ Card Body               │
│ Content goes here...    │
│                         │
├─────────────────────────┤
│ Card Footer             │
└─────────────────────────┘
```

**Variants**:
- Default card
- Stat card (large number)
- Info card (with icon)
- Warning card (yellow)
- Success card (green)
- Error card (red)

#### 8. Badge Components
```
[Success] [Warning] [Info] [Error] [Default]
```

**Usage**:
- Status indicators
- Category tags
- Cuisine types
- Cooking methods

#### 9. Alert Components
```
┌─────────────────────────┐
│ ⓘ Info Message          │
│ Additional details...   │
└─────────────────────────┘
```

**Types**:
- Info (blue)
- Success (green)
- Warning (yellow)
- Error (red)

#### 10. Modal Components
```
        ┌─────────────────┐
        │ Modal Title   × │
        ├─────────────────┤
        │                 │
        │  Modal Body     │
        │                 │
        ├─────────────────┤
        │  [Cancel] [OK]  │
        └─────────────────┘
```

**Types**:
- Confirmation modal
- Form modal
- Image modal
- Info modal

#### 11. Loading States
```
┌─────────────────────────┐
│  ⟳ Loading...           │
│  ████░░░░░░░░  40%      │
└─────────────────────────┘
```

**Variants**:
- Spinner
- Progress bar
- Skeleton screens
- Pulsing placeholders

#### 12. Empty States
```
┌─────────────────────────┐
│                         │
│    [🍳 Illustration]    │
│                         │
│   No recipes found      │
│   Try different filters │
│                         │
│   [Browse All Recipes]  │
│                         │
└─────────────────────────┘
```

---

## 📐 Responsive Design

### Breakpoints
```css
/* Mobile First Approach */
--breakpoint-sm:  640px   /* Small devices */
--breakpoint-md:  768px   /* Tablets */
--breakpoint-lg:  1024px  /* Laptops */
--breakpoint-xl:  1280px  /* Desktops */
--breakpoint-2xl: 1536px  /* Large screens */
```

### Grid System
```css
/* Mobile (default): 1 column */
.grid { grid-template-columns: 1fr; }

/* Tablet: 2 columns */
@media (min-width: 768px) {
  .grid { grid-template-columns: repeat(2, 1fr); }
}

/* Desktop: 3-4 columns */
@media (min-width: 1024px) {
  .grid { grid-template-columns: repeat(3, 1fr); }
}
```

### Layout Patterns

#### Mobile (< 768px)
- Single column layout
- Stack all cards vertically
- Hamburger menu
- Bottom tab bar (optional)
- Full-width cards
- Reduced padding (16px)

#### Tablet (768px - 1024px)
- 2 column grid
- Collapsible sidebar
- Medium cards
- Standard padding (24px)

#### Desktop (> 1024px)
- 3-4 column grid
- Persistent sidebar
- Large cards
- Generous padding (32px)

### Component Responsiveness

#### Navigation
- **Mobile**: Hamburger menu → Drawer
- **Desktop**: Horizontal menu bar

#### Recipe Grid
- **Mobile**: 1 column
- **Tablet**: 2 columns
- **Desktop**: 3-4 columns

#### Filters
- **Mobile**: Bottom sheet / Modal
- **Desktop**: Fixed sidebar

#### Forms
- **Mobile**: Full width, stacked
- **Desktop**: 2 column layout

---

## ♿ Accessibility

### WCAG 2.1 AA Compliance

#### Color Contrast
- Text on background: minimum 4.5:1
- Large text (18px+): minimum 3:1
- Interactive elements: 3:1

#### Keyboard Navigation
- All interactive elements accessible via Tab
- Focus indicators visible
- Skip to main content link
- Logical tab order

#### Screen Reader Support
- Semantic HTML (`<main>`, `<nav>`, `<article>`)
- ARIA labels where needed
- Alt text for images
- Form labels properly associated

#### Focus Management
- Clear focus indicators (blue ring)
- Focus trap in modals
- Return focus after modal close

#### Form Validation
- Clear error messages
- Error summary at top
- Inline validation
- Success messages

### Inclusive Design
- Minimum touch target: 44x44px
- Readable font sizes (minimum 16px)
- High contrast mode support
- Reduced motion support

```css
/* Respect user's motion preferences */
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## 🎨 Brand Guidelines

### Brand Personality
- **Modern**: Clean, minimal design
- **Friendly**: Approachable, warm colors
- **Intelligent**: AI-powered features
- **Practical**: Focus on usability

### Voice & Tone
- **Conversational**: Natural, friendly language
- **Helpful**: Provide guidance and tips
- **Encouraging**: Motivate users to cook
- **Clear**: Simple, jargon-free

### Visual Style
- **Clean layouts**: Plenty of white space
- **Gradient accents**: Blue to purple
- **Rounded corners**: Friendly feel (8-12px)
- **Soft shadows**: Subtle depth
- **Bold CTAs**: Clear action buttons

### Iconography
- **Style**: Outlined (primary), Filled (accent)
- **Library**: Heroicons
- **Size**: Consistent sizing (16/20/24px)
- **Color**: Match text color or brand colors

### Photography/Illustrations
- **Style**: Bright, appetizing food photos
- **Tone**: Warm, inviting
- **Usage**: Recipe images, empty states
- **Fallback**: Gradient backgrounds when no image

### Animation
- **Transitions**: 200-300ms ease-in-out
- **Hover effects**: Subtle lift (2-4px)
- **Loading**: Smooth spinners, pulse effects
- **Page transitions**: Fade in/out

---

## 📊 Design Deliverables

### Required Deliverables

#### 1. High-Fidelity Mockups
- [ ] Landing page (desktop + mobile)
- [ ] Login/Register (desktop + mobile)
- [ ] Dashboard (desktop + mobile)
- [ ] Profile pages (all tabs)
- [ ] Ingredients page
- [ ] AI Suggestions page (all states)
- [ ] Recipe list & detail
- [ ] Cooking session page
- [ ] History page
- [ ] Admin dashboard
- [ ] All modals & overlays

#### 2. Component Library
- [ ] All UI components (buttons, inputs, cards, etc.)
- [ ] Component states (default, hover, active, disabled)
- [ ] Component variants
- [ ] Icon set

#### 3. Design System Documentation
- [ ] Color palette with hex codes
- [ ] Typography scale
- [ ] Spacing system
- [ ] Grid system
- [ ] Shadow system
- [ ] Animation guidelines

#### 4. Prototype
- [ ] Interactive prototype (Figma/Adobe XD)
- [ ] User flows demonstrated
- [ ] Microinteractions
- [ ] Responsive behavior

#### 5. Assets
- [ ] Logo files (SVG, PNG in multiple sizes)
- [ ] Icons (SVG format)
- [ ] Illustrations
- [ ] Sample images/photos
- [ ] Favicon

#### 6. Developer Handoff
- [ ] Design specs (measurements, spacing)
- [ ] CSS variables
- [ ] Component documentation
- [ ] Responsive breakpoints
- [ ] Animation specs

---

## 🛠️ Development Notes

### Implementation with Tailwind CSS

All designs should be implementable using Tailwind CSS utility classes. Examples:

```tsx
// Button Primary
<button className="bg-gradient-to-r from-blue-500 to-purple-600 text-white font-semibold px-6 py-3 rounded-lg hover:shadow-lg hover:-translate-y-0.5 transition-all">
  Get Started
</button>

// Card
<div className="bg-white rounded-xl shadow-sm hover:shadow-md p-6 border border-gray-100 transition-shadow">
  {/* Card content */}
</div>

// Input
<input className="w-full px-4 py-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent" />
```

### Component Structure

Follow Next.js/React component patterns:
```tsx
// components/RecipeCard.tsx
interface RecipeCardProps {
  recipe: Recipe;
  onClick?: () => void;
}

export default function RecipeCard({ recipe, onClick }: RecipeCardProps) {
  return (
    <div className="card" onClick={onClick}>
      {/* Card content */}
    </div>
  );
}
```

---

## 📞 Contact & Support

**Project Manager**: [Name]
**Designer**: [Your Name]
**Developer**: [Dev Name]

**Timeline**: [Start Date] - [End Date]
**Review Cycles**: Weekly design reviews every [Day]

---

## 📚 References

### Design Inspiration
- [Tasty](https://tasty.co) - Recipe cards & cooking mode
- [AllRecipes](https://www.allrecipes.com) - Recipe structure
- [Notion](https://notion.so) - Clean UI, card layouts
- [Linear](https://linear.app) - Modern design system

### Competitors Analysis
- Yummly (AI suggestions)
- Cookpad (social features)
- Mealime (meal planning)

### Technical References
- [Tailwind CSS Docs](https://tailwindcss.com)
- [Next.js Docs](https://nextjs.org/docs)
- [Heroicons](https://heroicons.com)
- [AWS Cognito UX](https://docs.aws.amazon.com/cognito)

---

## 📝 Changelog

### Version 1.0.0 (Current)
- Initial design brief
- Complete Phase 1 MVP specifications
- Design system defined
- Component library outlined
- User flows documented

### Future Updates
- Phase 2 social features
- Mobile app considerations
- Dark mode support
- Internationalization (i18n)

---

**Last Updated**: October 8, 2025
**Version**: 1.0.0
**Status**: Ready for Design Phase
