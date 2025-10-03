---
title: "41 - Privacy & Data Protection"
weight: 41
---

# Privacy & Data Protection - Smart Cooking App

## Privacy Overview

### Privacy Principles
1. **Data Minimization**: Collect only necessary data
2. **User Control**: Users control their data visibility
3. **Transparency**: Clear data usage policies
4. **Security**: Protect data with encryption
5. **Compliance**: GDPR-ready implementation

### Privacy Levels
- **Public**: Visible to everyone
- **Friends**: Visible to accepted friends only
- **Private**: Visible to user only

## Privacy Settings System

### Privacy Configuration
```yaml
Default Privacy Settings:
  profile_visibility: public
  email_visibility: private
  date_of_birth_visibility: friends
  gender_visibility: public
  country_visibility: public
  recipes_visibility: public
  ingredients_visibility: friends
  preferences_visibility: friends
  cooking_history_visibility: private
  posts_visibility: public
```

### Privacy Settings Schema
```javascript
// DynamoDB Privacy Settings Item
{
  PK: "USER#<user_id>",
  SK: "PRIVACY",
  entity_type: "PRIVACY_SETTINGS",
  
  // Profile visibility
  profile_visibility: "public|friends|private",
  email_visibility: "private", // Always private
  date_of_birth_visibility: "friends",
  gender_visibility: "public",
  country_visibility: "public",
  
  // Content visibility
  recipes_visibility: "public",
  ingredients_visibility: "friends",
  preferences_visibility: "friends",
  cooking_history_visibility: "private",
  posts_visibility: "public",
  
  // Advanced settings
  allow_friend_requests: true,
  show_online_status: false,
  allow_recipe_sharing: true,
  
  created_at: "2025-01-20T10:00:00Z",
  updated_at: "2025-01-20T10:00:00Z"
}
```

## Privacy Filtering Implementation

### Backend Privacy Filter
```javascript
// shared/privacy-filter.js
class PrivacyFilter {
  constructor(ddb) {
    this.ddb = ddb;
  }

  async filterUserData(targetUserId, requestingUserId, userData) {
    // Self-access - return all data
    if (targetUserId === requestingUserId) {
      return userData;
    }

    // Get privacy settings
    const privacySettings = await this.getPrivacySettings(targetUserId);
    
    // Check friendship status
    const friendshipStatus = await this.getFriendshipStatus(targetUserId, requestingUserId);
    
    // Apply privacy filters
    return this.applyPrivacyRules(userData, privacySettings, friendshipStatus);
  }

  async getPrivacySettings(userId) {
    const result = await this.ddb.get({
      TableName: process.env.DYNAMODB_TABLE,
      Key: {
        PK: `USER#${userId}`,
        SK: 'PRIVACY'
      }
    });

    return result.Item || this.getDefaultPrivacySettings();
  }

  async getFriendshipStatus(userId1, userId2) {
    const result = await this.ddb.get({
      TableName: process.env.DYNAMODB_TABLE,
      Key: {
        PK: `USER#${userId1}`,
        SK: `FRIEND#${userId2}`
      }
    });

    return result.Item?.status || 'none';
  }

  applyPrivacyRules(userData, privacySettings, friendshipStatus) {
    const filteredData = { ...userData };

    // Helper function to check visibility
    const canView = (visibility) => {
      switch (visibility) {
        case 'public':
          return true;
        case 'friends':
          return friendshipStatus === 'accepted';
        case 'private':
          return false;
        default:
          return false;
      }
    };

    // Filter profile fields
    if (!canView(privacySettings.email_visibility)) {
      delete filteredData.email;
    }
    
    if (!canView(privacySettings.date_of_birth_visibility)) {
      delete filteredData.date_of_birth;
    }
    
    if (!canView(privacySettings.gender_visibility)) {
      delete filteredData.gender;
    }
    
    if (!canView(privacySettings.country_visibility)) {
      delete filteredData.country;
    }

    // Filter preferences
    if (!canView(privacySettings.preferences_visibility)) {
      delete filteredData.preferences;
    }

    // Filter ingredients
    if (!canView(privacySettings.ingredients_visibility)) {
      delete filteredData.ingredients;
    }

    return filteredData;
  }

  getDefaultPrivacySettings() {
    return {
      profile_visibility: 'public',
      email_visibility: 'private',
      date_of_birth_visibility: 'friends',
      gender_visibility: 'public',
      country_visibility: 'public',
      recipes_visibility: 'public',
      ingredients_visibility: 'friends',
      preferences_visibility: 'friends',
      cooking_history_visibility: 'private',
      posts_visibility: 'public'
    };
  }
}

module.exports = PrivacyFilter;
```

### Privacy-Aware API Endpoints
```javascript
// user-profile Lambda with privacy filtering
exports.handler = async (event, context) => {
  const { httpMethod, pathParameters } = event;
  const requestingUserId = event.requestContext.authorizer.claims.sub;
  const targetUserId = pathParameters?.userId || requestingUserId;

  const privacyFilter = new PrivacyFilter(ddb);

  switch (httpMethod) {
    case 'GET':
      return await getUserProfile(targetUserId, requestingUserId, privacyFilter);
    case 'PUT':
      return await updateUserProfile(event, requestingUserId);
    default:
      return errorResponse(405, 'method_not_allowed');
  }
};

async function getUserProfile(targetUserId, requestingUserId, privacyFilter) {
  try {
    // Get user profile
    const profileResult = await ddb.get({
      TableName: process.env.DYNAMODB_TABLE,
      Key: {
        PK: `USER#${targetUserId}`,
        SK: 'PROFILE'
      }
    });

    if (!profileResult.Item) {
      return errorResponse(404, 'user_not_found');
    }

    // Get user preferences
    const preferencesResult = await ddb.get({
      TableName: process.env.DYNAMODB_TABLE,
      Key: {
        PK: `USER#${targetUserId}`,
        SK: 'PREFERENCES'
      }
    });

    // Get user ingredients
    const ingredientsResult = await ddb.query({
      TableName: process.env.DYNAMODB_TABLE,
      KeyConditionExpression: 'PK = :pk AND begins_with(SK, :sk)',
      ExpressionAttributeValues: {
        ':pk': `USER#${targetUserId}`,
        ':sk': 'INGREDIENT#'
      }
    });

    const userData = {
      ...profileResult.Item,
      preferences: preferencesResult.Item,
      ingredients: ingredientsResult.Items || []
    };

    // Apply privacy filtering
    const filteredData = await privacyFilter.filterUserData(
      targetUserId, 
      requestingUserId, 
      userData
    );

    return successResponse(filteredData);

  } catch (error) {
    console.error('Get user profile error:', error);
    return errorResponse(500, 'internal_error');
  }
}
```

## Privacy Settings Management

### Update Privacy Settings API
```javascript
// PUT /user/privacy endpoint
async function updatePrivacySettings(event, userId) {
  const updates = JSON.parse(event.body);
  
  // Validate privacy settings
  const validSettings = [
    'profile_visibility',
    'date_of_birth_visibility',
    'gender_visibility',
    'country_visibility',
    'recipes_visibility',
    'ingredients_visibility',
    'preferences_visibility',
    'cooking_history_visibility',
    'posts_visibility',
    'allow_friend_requests',
    'show_online_status',
    'allow_recipe_sharing'
  ];

  const validVisibility = ['public', 'friends', 'private'];
  
  // Filter and validate updates
  const filteredUpdates = {};
  for (const [key, value] of Object.entries(updates)) {
    if (validSettings.includes(key)) {
      if (key.endsWith('_visibility') && !validVisibility.includes(value)) {
        return errorResponse(400, 'invalid_visibility', `Invalid visibility: ${value}`);
      }
      if (typeof value === 'boolean' || validVisibility.includes(value)) {
        filteredUpdates[key] = value;
      }
    }
  }

  // Email is always private (cannot be changed)
  if (filteredUpdates.email_visibility) {
    delete filteredUpdates.email_visibility;
  }

  try {
    // Build update expression
    const updateExpressions = [];
    const expressionAttributeValues = {};
    
    for (const [key, value] of Object.entries(filteredUpdates)) {
      updateExpressions.push(`${key} = :${key}`);
      expressionAttributeValues[`:${key}`] = value;
    }
    
    expressionAttributeValues[':updated_at'] = new Date().toISOString();
    updateExpressions.push('updated_at = :updated_at');

    await ddb.update({
      TableName: process.env.DYNAMODB_TABLE,
      Key: {
        PK: `USER#${userId}`,
        SK: 'PRIVACY'
      },
      UpdateExpression: `SET ${updateExpressions.join(', ')}`,
      ExpressionAttributeValues: expressionAttributeValues
    });

    return successResponse({
      message: 'Privacy settings updated successfully',
      updated_fields: Object.keys(filteredUpdates)
    });

  } catch (error) {
    console.error('Update privacy settings error:', error);
    return errorResponse(500, 'update_failed');
  }
}
```

## Frontend Privacy Controls

### Privacy Settings Page
```javascript
// src/public/js/privacy.js
class PrivacySettings {
  constructor() {
    this.settings = null;
    this.isLoading = true;
    this.isSaving = false;
    this.init();
  }

  async init() {
    await this.loadPrivacySettings();
    this.renderSettings();
    this.bindEvents();
  }

  async loadPrivacySettings() {
    try {
      const response = await apiClient.get('/user/privacy');
      this.settings = response;
      this.isLoading = false;
    } catch (error) {
      this.showError('Failed to load privacy settings');
      this.isLoading = false;
    }
  }

  renderSettings() {
    const container = document.getElementById('privacy-settings');
    
    if (this.isLoading) {
      container.innerHTML = '<div class="loading">Loading privacy settings...</div>';
      return;
    }

    if (!this.settings) {
      container.innerHTML = '<div class="error">Failed to load privacy settings</div>';
      return;
    }

    const visibilityOptions = [
      { value: 'public', label: 'Public - Everyone can see' },
      { value: 'friends', label: 'Friends - Only friends can see' },
      { value: 'private', label: 'Private - Only you can see' }
    ];

    container.innerHTML = `
      <div class="privacy-form">
        <div class="section">
          <h3>Profile Privacy</h3>
          <div class="form-grid">
            <div class="form-group">
              <label>Date of Birth</label>
              <select id="date_of_birth_visibility" value="${this.settings.date_of_birth_visibility}">
                ${visibilityOptions.map(opt => 
                  `<option value="${opt.value}" ${this.settings.date_of_birth_visibility === opt.value ? 'selected' : ''}>
                    ${opt.label}
                  </option>`
                ).join('')}
              </select>
            </div>
            
            <div class="form-group">
              <label>Gender</label>
              <select id="gender_visibility" value="${this.settings.gender_visibility}">
                ${visibilityOptions.map(opt => 
                  `<option value="${opt.value}" ${this.settings.gender_visibility === opt.value ? 'selected' : ''}>
                    ${opt.label}
                  </option>`
                ).join('')}
              </select>
            </div>
            
            <div class="form-group">
              <label>Country</label>
              <select id="country_visibility" value="${this.settings.country_visibility}">
                ${visibilityOptions.map(opt => 
                  `<option value="${opt.value}" ${this.settings.country_visibility === opt.value ? 'selected' : ''}>
                    ${opt.label}
                  </option>`
                ).join('')}
              </select>
            </div>
          </div>
        </div>

        <div class="section">
          <h3>Content Privacy</h3>
          <div class="form-grid">
            <div class="form-group">
              <label>Recipes</label>
              <select id="recipes_visibility" value="${this.settings.recipes_visibility}">
                ${visibilityOptions.map(opt => 
                  `<option value="${opt.value}" ${this.settings.recipes_visibility === opt.value ? 'selected' : ''}>
                    ${opt.label}
                  </option>`
                ).join('')}
              </select>
            </div>
            
            <div class="form-group">
              <label>Ingredients</label>
              <select id="ingredients_visibility" value="${this.settings.ingredients_visibility}">
                ${visibilityOptions.map(opt => 
                  `<option value="${opt.value}" ${this.settings.ingredients_visibility === opt.value ? 'selected' : ''}>
                    ${opt.label}
                  </option>`
                ).join('')}
              </select>
            </div>
            
            <div class="form-group">
              <label>Food Preferences</label>
              <select id="preferences_visibility" value="${this.settings.preferences_visibility}">
                ${visibilityOptions.map(opt => 
                  `<option value="${opt.value}" ${this.settings.preferences_visibility === opt.value ? 'selected' : ''}>
                    ${opt.label}
                  </option>`
                ).join('')}
              </select>
            </div>
            
            <div class="form-group">
              <label>Posts</label>
              <select id="posts_visibility" value="${this.settings.posts_visibility}">
                ${visibilityOptions.map(opt => 
                  `<option value="${opt.value}" ${this.settings.posts_visibility === opt.value ? 'selected' : ''}>
                    ${opt.label}
                  </option>`
                ).join('')}
              </select>
            </div>
          </div>
        </div>

        <div class="section">
          <h3>Social Settings</h3>
          <div class="form-group">
            <label class="checkbox-label">
              <input type="checkbox" id="allow_friend_requests" ${this.settings.allow_friend_requests ? 'checked' : ''}>
              Allow Friend Requests
              <small>Let others send you friend requests</small>
            </label>
          </div>
          
          <div class="form-group">
            <label class="checkbox-label">
              <input type="checkbox" id="show_online_status" ${this.settings.show_online_status ? 'checked' : ''}>
              Show Online Status
              <small>Let friends see when you're online</small>
            </label>
          </div>
          
          <div class="form-group">
            <label class="checkbox-label">
              <input type="checkbox" id="allow_recipe_sharing" ${this.settings.allow_recipe_sharing ? 'checked' : ''}>
              Allow Recipe Sharing
              <small>Let others share your recipes</small>
            </label>
          </div>
        </div>

        <div class="form-actions">
          <button id="save-privacy-settings" class="btn btn-primary" ${this.isSaving ? 'disabled' : ''}>
            ${this.isSaving ? 'Saving...' : 'Save Privacy Settings'}
          </button>
        </div>
      </div>
    `;
  }

  bindEvents() {
    const saveButton = document.getElementById('save-privacy-settings');
    if (saveButton) {
      saveButton.addEventListener('click', () => this.saveSettings());
    }

    // Bind change events for all form elements
    const formElements = document.querySelectorAll('#privacy-settings select, #privacy-settings input');
    formElements.forEach(element => {
      element.addEventListener('change', (e) => this.updateSetting(e.target.id, e.target.value || e.target.checked));
    });
  }

  updateSetting(key, value) {
    if (this.settings) {
      this.settings[key] = value;
    }
  }

  async saveSettings() {
    if (!this.settings || this.isSaving) return;

    this.isSaving = true;
    this.updateSaveButton();

    try {
      await apiClient.put('/user/privacy', this.settings);
      this.showSuccess('Privacy settings updated successfully');
    } catch (error) {
      this.showError('Failed to update privacy settings');
    } finally {
      this.isSaving = false;
      this.updateSaveButton();
    }
  }

  updateSaveButton() {
    const saveButton = document.getElementById('save-privacy-settings');
    if (saveButton) {
      saveButton.disabled = this.isSaving;
      saveButton.textContent = this.isSaving ? 'Saving...' : 'Save Privacy Settings';
    }
  }

  showSuccess(message) {
    this.showNotification(message, 'success');
  }

  showError(message) {
    this.showNotification(message, 'error');
  }

  showNotification(message, type) {
    const notification = document.createElement('div');
    notification.className = `notification ${type}`;
    notification.textContent = message;
    document.body.appendChild(notification);
    
    setTimeout(() => {
      notification.remove();
    }, 3000);
  }
}

// Initialize privacy settings when page loads
document.addEventListener('DOMContentLoaded', () => {
  new PrivacySettings();
});
```

## AI Data Usage Policy

### AI Privacy Policy
```markdown
## AI Agent Data Usage Policy

### Data Used for AI Personalization
The Smart Cooking AI uses the following information to provide personalized recipe suggestions:

**Personal Information Used:**
- Age range (derived from birth year) - for nutritional recommendations
- Gender - for portion size recommendations  
- Country - for regional cuisine preferences
- Food preferences (cooking methods, favorite cuisines)
- Allergies - **CRITICAL** for food safety (absolutely avoided)

**Data NOT Used:**
- Full name or email address
- Exact location or address
- Payment information
- Browsing history outside the app
- Personal messages or comments

### Purpose & Usage
- **Primary Purpose**: Personalize recipe suggestions only
- **No Advertising**: We do not use your data for advertising
- **No Third-Party Sharing**: Your data is never sold or shared
- **No Profiling**: We don't create profiles beyond cooking preferences

### User Control
- You can update your preferences anytime
- You can delete your data anytime
- AI suggestions are optional - you can use the app without them
- You control what information is shared through privacy settings

### Data Retention
- AI suggestion history: 90 days (automatic cleanup)
- Personal preferences: Until you delete your account
- Anonymized usage patterns: May be kept for service improvement

### Your Rights
- Right to access your data
- Right to correct your data  
- Right to delete your data
- Right to data portability
- Right to object to processing
```

## GDPR Compliance

### Data Subject Rights Implementation
```javascript
// GDPR compliance endpoints
exports.handler = async (event, context) => {
  const { httpMethod, resource } = event;
  const userId = event.requestContext.authorizer.claims.sub;

  switch (resource) {
    case '/user/data-export':
      return await exportUserData(userId);
    case '/user/data-delete':
      return await deleteUserData(userId);
    case '/user/data-portability':
      return await getPortableData(userId);
    default:
      return errorResponse(404, 'not_found');
  }
};

// Right to Access - Export all user data
async function exportUserData(userId) {
  try {
    const userData = {
      profile: await getUserProfile(userId),
      preferences: await getUserPreferences(userId),
      ingredients: await getUserIngredients(userId),
      cookingHistory: await getCookingHistory(userId),
      aiSuggestions: await getAISuggestions(userId),
      posts: await getUserPosts(userId),
      friends: await getUserFriends(userId),
      privacySettings: await getPrivacySettings(userId)
    };

    return successResponse({
      message: 'Data export completed',
      data: userData,
      exportDate: new Date().toISOString(),
      format: 'JSON'
    });
  } catch (error) {
    return errorResponse(500, 'export_failed');
  }
}

// Right to Erasure - Delete all user data
async function deleteUserData(userId) {
  try {
    // Delete from DynamoDB
    await deleteAllUserItems(userId);
    
    // Delete from S3 (user images)
    await deleteUserImages(userId);
    
    // Disable Cognito user
    await disableCognitoUser(userId);

    return successResponse({
      message: 'All user data has been deleted',
      deletedAt: new Date().toISOString()
    });
  } catch (error) {
    return errorResponse(500, 'deletion_failed');
  }
}
```

## Related Documents

- [40 - Auth](40-auth.md)
- [13 - Security](13-security.md)
- [42 - Compliance](42-compliance.md)
- [11 - Database](11-database.md)