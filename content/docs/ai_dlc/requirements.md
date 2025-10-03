# Requirements Document - Smart Cooking MVP

## Introduction

Smart Cooking App là nền tảng ứng dụng nấu ăn thông minh sử dụng AI để cá nhân hóa gợi ý công thức dựa trên nguyên liệu có sẵn, tối ưu hóa việc sử dụng nguyên liệu để giảm lãng phí thực phẩm, và kết nối cộng đồng người yêu thích nấu ăn. Hệ thống sử dụng kiến trúc serverless trên AWS với chiến lược flexible mix giữa database và AI generation để tối ưu chi phí và chất lượng.

## Requirements

### Requirement 1: User Authentication & Profile Management

**User Story:** As a home cook, I want to create and manage my account with personal preferences, so that I can receive personalized recipe suggestions and track my cooking journey.

#### Acceptance Criteria

1. WHEN a user registers with email, password, username, and full name THEN the system SHALL validate email format, check password strength (min 8 characters), verify username uniqueness, and send verification email via Cognito
2. WHEN a user logs in with valid credentials THEN the system SHALL authenticate via Cognito and return JWT token with redirect to dashboard
3. WHEN a user updates their profile with full name, date of birth, gender, country, and avatar THEN the system SHALL validate date of birth (user >= 13 years old), gender enum (male, female, other), and country code
4. WHEN a user sets cooking preferences including preferred cooking methods, favorite cuisines, allergies, and dietary restrictions THEN the system SHALL save these preferences for AI personalization
5. IF a user provides invalid profile data THEN the system SHALL return specific validation errors with guidance

### Requirement 2: Ingredient Management System

**User Story:** As a user, I want to input and manage my available ingredients with validation and auto-correction, so that I can get accurate recipe suggestions based on what I actually have.

#### Acceptance Criteria

1. WHEN a user adds an ingredient by name THEN the system SHALL validate against master ingredients database with fuzzy search and auto-correction for similar matches
2. WHEN an ingredient is not found THEN the system SHALL provide suggestions for similar ingredients and log the invalid ingredient for admin review
3. WHEN a user views their ingredient list THEN the system SHALL display all ingredients with added timestamps sorted by most recent
4. WHEN a user removes an ingredient THEN the system SHALL delete it from their personal ingredient list
5. WHEN a user validates multiple ingredients in batch THEN the system SHALL return arrays of valid, invalid, corrected, and suggested ingredients
6. IF an invalid ingredient is reported 5 or more times THEN the system SHALL notify admin for review

### Requirement 3: AI Recipe Suggestion Engine

**User Story:** As a user, I want to receive personalized recipe suggestions based on my available ingredients and preferences, so that I can cook meals that match my taste and dietary needs while using what I have.

#### Acceptance Criteria

1. WHEN a user requests 1-5 recipe suggestions with minimum 2 ingredients THEN the system SHALL query approved recipes from database matching ingredients and categories, calculate gap between requested and available recipes, and generate remaining recipes using AI
2. WHEN generating AI suggestions THEN the system SHALL use user context including age, gender, country, preferred cooking methods, favorite cuisines, and allergies for personalization
3. WHEN user has allergies specified THEN the system SHALL absolutely avoid those ingredients in all suggestions
4. WHEN AI generation is requested THEN the system SHALL ensure category diversity (xào, canh, hấp, chiên) and complete within 60 seconds timeout
5. WHEN invalid ingredients are provided THEN the system SHALL log them to CloudWatch, provide alternative suggestions, and still generate recipes with valid ingredients
6. IF AI generation fails or times out THEN the system SHALL return database recipes only with appropriate error message

### Requirement 4: Cooking History & Rating System

**User Story:** As a user, I want to track my cooking history and rate recipes I've made, so that I can build a personal cookbook and help improve the community recipe database through my feedback.

#### Acceptance Criteria

1. WHEN a user starts cooking a recipe THEN the system SHALL create a cooking history entry with status 'cooking' and return history ID
2. WHEN a user completes cooking THEN the system SHALL update status to 'completed' and prompt for rating
3. WHEN a user rates a recipe with 1-5 stars and optional comment THEN the system SHALL save the rating, update recipe's average rating, and check for auto-approval
4. WHEN a recipe's average rating reaches 4.0 or higher THEN the system SHALL automatically set is_approved=true, is_public=true, and notify the user that their recipe was added to the database
5. WHEN a user views their cooking history THEN the system SHALL display all cooking sessions with recipe details, status, cook date, personal rating, and notes sorted by newest first
6. WHEN a user marks a recipe as favorite THEN the system SHALL update their cooking history and allow filtering by favorites

### Requirement 5: Recipe Database & Search

**User Story:** As a user, I want to browse and search approved recipes from the community database, so that I can discover new dishes and see detailed cooking instructions.

#### Acceptance Criteria

1. WHEN a user views recipe details THEN the system SHALL display title, description, cuisine type, cooking method, ingredients with quantities, step-by-step instructions, nutritional info, average rating, and approval status
2. WHEN a user searches recipes THEN the system SHALL search in title and description with filters for cuisine type, cooking method, and meal type, returning paginated results of approved recipes only
3. WHEN displaying search results THEN the system SHALL allow sorting by rating, created date, and popularity
4. WHEN a recipe is auto-approved through rating system THEN the system SHALL make it publicly searchable and viewable
5. IF no recipes match search criteria THEN the system SHALL suggest alternative search terms or broader filters

### Requirement 6: Master Ingredients Database

**User Story:** As a system administrator, I want to maintain a curated database of valid ingredients with fuzzy matching capabilities, so that users can input ingredients naturally while maintaining data quality.

#### Acceptance Criteria

1. WHEN the system validates an ingredient THEN it SHALL check against master ingredients database with exact match first, then fuzzy search for similar matches
2. WHEN fuzzy search finds similar ingredients THEN the system SHALL suggest auto-correction with confidence score
3. WHEN an invalid ingredient is reported multiple times THEN the system SHALL flag it for admin review to potentially add to master database
4. WHEN master ingredients database is updated THEN the system SHALL maintain Vietnamese and English names for each ingredient
5. IF an ingredient has multiple variations or spellings THEN the system SHALL normalize to the canonical form

### Requirement 7: System Performance & Reliability

**User Story:** As a user, I want the application to be fast, reliable, and available when I need to cook, so that I can have a smooth cooking experience without technical interruptions.

#### Acceptance Criteria

1. WHEN a user makes API requests (non-AI) THEN the system SHALL respond within 500ms for 95% of requests
2. WHEN AI recipe generation is requested THEN the system SHALL complete within 5 seconds per recipe with loading indicators shown
3. WHEN the system experiences high load THEN it SHALL auto-scale serverless functions to maintain performance
4. WHEN system errors occur THEN the system SHALL log to CloudWatch, maintain <1% error rate, and provide graceful error messages to users
5. WHEN the system is operational THEN it SHALL maintain 99.5% uptime monthly with monitoring and alerting in place

### Requirement 8: Data Privacy & Security

**User Story:** As a user, I want my personal data to be secure and used transparently for AI personalization, so that I can trust the system with my information while getting personalized experiences.

#### Acceptance Criteria

1. WHEN AI generates suggestions THEN it SHALL use only age, gender, country, cooking preferences, cuisines, and allergies for personalization, never using email, full name, or address
2. WHEN user data is stored THEN the system SHALL encrypt all data at rest and in transit using AWS managed encryption
3. WHEN users access the system THEN all API calls SHALL be authenticated via Cognito JWT tokens with proper authorization
4. WHEN API requests are made THEN the system SHALL validate and sanitize all inputs to prevent injection attacks
5. IF a user wants to delete their account THEN the system SHALL provide data deletion capability in compliance with GDPR requirements

### Requirement 9: Cost Optimization & Scalability

**User Story:** As a product owner, I want the system to operate cost-effectively while scaling with user growth, so that the business remains sustainable as it grows.

#### Acceptance Criteria

1. WHEN the system serves 1,000 users THEN the monthly operational cost SHALL remain under $160 with budget alarms at $140
2. WHEN recipe suggestions are requested THEN the system SHALL use flexible mix strategy, prioritizing database recipes over AI generation to reduce costs
3. WHEN database coverage grows THEN the system SHALL automatically reduce AI generation costs by 30-70% through increased database usage
4. WHEN system usage increases THEN serverless architecture SHALL auto-scale without manual intervention
5. IF costs exceed budget thresholds THEN the system SHALL trigger alerts and implement cost-saving measures like caching and rate limiting