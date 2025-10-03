---
title: "42 - Compliance & Governance"
weight: 42
---

# Compliance & Governance - Smart Cooking App

## Compliance Overview

### Compliance Standards
- **GDPR**: EU General Data Protection Regulation
- **CCPA**: California Consumer Privacy Act (future)
- **SOC 2 Type II**: Security controls (future)
- **ISO 27001**: Information security (future)

### Governance Framework
- **Data Governance**: Clear data handling policies
- **Security Governance**: Regular security assessments
- **Privacy Governance**: User privacy protection
- **Operational Governance**: Incident response procedures

## GDPR Compliance Implementation

### Legal Basis for Processing
```yaml
Data Processing Legal Basis:
  User Registration: Contract (Art. 6(1)(b))
  AI Personalization: Legitimate Interest (Art. 6(1)(f))
  Marketing Communications: Consent (Art. 6(1)(a))
  Legal Compliance: Legal Obligation (Art. 6(1)(c))
  Security Monitoring: Legitimate Interest (Art. 6(1)(f))
```

### Data Subject Rights Implementation

#### Right to Access (Art. 15)
```javascript
// GET /user/data-export
async function exportUserData(userId) {
  const exportData = {
    personalData: {
      profile: await getUserProfile(userId),
      preferences: await getUserPreferences(userId),
      privacySettings: await getPrivacySettings(userId)
    },
    activityData: {
      cookingHistory: await getCookingHistory(userId),
      aiSuggestions: await getAISuggestions(userId),
      recipes: await getUserRecipes(userId),
      posts: await getUserPosts(userId)
    },
    socialData: {
      friends: await getUserFriends(userId),
      comments: await getUserComments(userId),
      reactions: await getUserReactions(userId)
    },
    metadata: {
      exportDate: new Date().toISOString(),
      dataRetentionPeriod: "Until account deletion",
      legalBasis: "Contract and Legitimate Interest",
      dataController: "Smart Cooking App",
      contactEmail: "privacy@smartcooking.app"
    }
  };

  return {
    statusCode: 200,
    headers: {
      'Content-Type': 'application/json',
      'Content-Disposition': 'attachment; filename="user-data-export.json"'
    },
    body: JSON.stringify(exportData, null, 2)
  };
}
```

#### Right to Rectification (Art. 16)
```javascript
// PUT /user/profile - Allow users to correct their data
async function updateUserProfile(event, userId) {
  const updates = JSON.parse(event.body);
  
  // Validate and sanitize updates
  const allowedFields = [
    'full_name', 'date_of_birth', 'gender', 'country',
    'dietary_restrictions', 'allergies', 'favorite_cuisines'
  ];
  
  const validatedUpdates = {};
  for (const [key, value] of Object.entries(updates)) {
    if (allowedFields.includes(key)) {
      validatedUpdates[key] = sanitizeInput(value);
    }
  }

  // Log data rectification for audit
  await logDataRectification(userId, validatedUpdates);

  // Update the data
  await updateUserData(userId, validatedUpdates);

  return successResponse({
    message: 'Profile updated successfully',
    updatedFields: Object.keys(validatedUpdates),
    updatedAt: new Date().toISOString()
  });
}
```

#### Right to Erasure (Art. 17)
```javascript
// DELETE /user/account - Complete data deletion
async function deleteUserAccount(userId) {
  try {
    // 1. Anonymize user-generated content (recipes, comments)
    await anonymizeUserContent(userId);
    
    // 2. Delete personal data from DynamoDB
    await deletePersonalData(userId);
    
    // 3. Delete files from S3
    await deleteUserFiles(userId);
    
    // 4. Disable Cognito user
    await disableCognitoUser(userId);
    
    // 5. Log deletion for compliance
    await logDataDeletion(userId);

    return successResponse({
      message: 'Account and all personal data have been deleted',
      deletedAt: new Date().toISOString(),
      retainedData: 'Anonymized recipes and comments (no personal identifiers)'
    });
  } catch (error) {
    console.error('Account deletion error:', error);
    return errorResponse(500, 'deletion_failed');
  }
}

async function anonymizeUserContent(userId) {
  // Keep recipes but remove personal identifiers
  await ddb.update({
    TableName: process.env.DYNAMODB_TABLE,
    Key: { PK: `RECIPE#${recipeId}`, SK: 'METADATA' },
    UpdateExpression: 'SET created_by = :anon, user_id = :anon',
    ExpressionAttributeValues: {
      ':anon': 'anonymous-user'
    }
  });
  
  // Similar for comments, posts, etc.
}
```

#### Right to Data Portability (Art. 20)
```javascript
// GET /user/data-portability
async function getPortableData(userId) {
  const portableData = {
    format: "JSON",
    version: "1.0",
    exportDate: new Date().toISOString(),
    user: {
      profile: await getUserProfile(userId),
      preferences: await getUserPreferences(userId),
      ingredients: await getUserIngredients(userId)
    },
    recipes: await getUserRecipes(userId),
    cookingHistory: await getCookingHistory(userId),
    // Exclude system-generated data like AI suggestions
    // Include only user-created content
  };

  return {
    statusCode: 200,
    headers: {
      'Content-Type': 'application/json',
      'Content-Disposition': 'attachment; filename="portable-data.json"'
    },
    body: JSON.stringify(portableData, null, 2)
  };
}
```

#### Right to Object (Art. 21)
```javascript
// PUT /user/data-processing-objection
async function handleDataProcessingObjection(userId, objectionType) {
  switch (objectionType) {
    case 'ai_personalization':
      // Disable AI personalization
      await updateUserPreferences(userId, { 
        ai_personalization_enabled: false 
      });
      break;
      
    case 'marketing':
      // Disable marketing communications
      await updateUserPreferences(userId, { 
        marketing_emails_enabled: false 
      });
      break;
      
    case 'analytics':
      // Disable analytics tracking
      await updateUserPreferences(userId, { 
        analytics_tracking_enabled: false 
      });
      break;
  }

  await logDataProcessingObjection(userId, objectionType);
  
  return successResponse({
    message: `Data processing objection recorded for: ${objectionType}`,
    effectiveDate: new Date().toISOString()
  });
}
```

## Data Protection Impact Assessment (DPIA)

### DPIA Summary
```yaml
Processing Activity: AI Recipe Suggestion System
Risk Level: Medium
DPIA Required: Yes (automated decision-making with profiling)

Data Categories:
  - Personal identifiers (email, username)
  - Demographic data (age, gender, country)
  - Behavioral data (cooking preferences, ingredient usage)
  - Health data (allergies, dietary restrictions)

Risks Identified:
  1. AI bias in recipe suggestions
  2. Inference of health conditions from dietary data
  3. Potential data breaches
  4. Unauthorized access to personal preferences

Mitigation Measures:
  1. Regular AI model bias testing
  2. Encryption of sensitive data
  3. Access controls and audit logging
  4. User consent and transparency
```

### Privacy by Design Implementation
```javascript
// Privacy-preserving AI suggestions
class PrivacyAwareAI {
  async generateSuggestions(userId, ingredients) {
    // 1. Data minimization - only use necessary data
    const userContext = await this.getMinimalUserContext(userId);
    
    // 2. Purpose limitation - only for recipe suggestions
    const suggestions = await this.generateRecipes(ingredients, userContext);
    
    // 3. Storage limitation - TTL for AI suggestions
    await this.saveSuggestionsWithTTL(userId, suggestions, 90); // 90 days
    
    // 4. Transparency - log what data was used
    await this.logAIDataUsage(userId, userContext);
    
    return suggestions;
  }

  async getMinimalUserContext(userId) {
    // Only get data necessary for AI personalization
    const profile = await getUserProfile(userId);
    const preferences = await getUserPreferences(userId);
    
    return {
      age_range: this.getAgeRange(profile.date_of_birth), // Not exact age
      gender: profile.gender,
      country: profile.country,
      allergies: preferences.allergies, // Critical for safety
      cooking_methods: preferences.preferred_cooking_methods,
      cuisines: preferences.favorite_cuisines
    };
  }

  getAgeRange(birthDate) {
    if (!birthDate) return null;
    const age = new Date().getFullYear() - new Date(birthDate).getFullYear();
    return `${Math.floor(age / 10) * 10}-${Math.floor(age / 10) * 10 + 9}`;
  }
}
```

## Consent Management

### Consent Categories
```yaml
Consent Types:
  essential: 
    description: "Essential for app functionality"
    required: true
    legal_basis: "Contract"
    
  ai_personalization:
    description: "AI recipe personalization"
    required: false
    legal_basis: "Consent"
    
  analytics:
    description: "Usage analytics and improvements"
    required: false
    legal_basis: "Legitimate Interest"
    
  marketing:
    description: "Marketing communications"
    required: false
    legal_basis: "Consent"
```

### Consent Management Implementation
```javascript
// Consent management system
class ConsentManager {
  async recordConsent(userId, consentData) {
    const consentRecord = {
      PK: `USER#${userId}`,
      SK: `CONSENT#${new Date().toISOString()}`,
      entity_type: 'CONSENT_RECORD',
      user_id: userId,
      consents: consentData,
      ip_address: this.hashIP(consentData.ip_address),
      user_agent: consentData.user_agent,
      timestamp: new Date().toISOString(),
      version: '1.0' // Consent form version
    };

    await ddb.put({
      TableName: process.env.DYNAMODB_TABLE,
      Item: consentRecord
    });

    // Update current consent status
    await this.updateCurrentConsent(userId, consentData);
  }

  async updateCurrentConsent(userId, consentData) {
    await ddb.put({
      TableName: process.env.DYNAMODB_TABLE,
      Item: {
        PK: `USER#${userId}`,
        SK: 'CONSENT_CURRENT',
        entity_type: 'CURRENT_CONSENT',
        essential: true, // Always true
        ai_personalization: consentData.ai_personalization || false,
        analytics: consentData.analytics || false,
        marketing: consentData.marketing || false,
        updated_at: new Date().toISOString()
      }
    });
  }

  async withdrawConsent(userId, consentType) {
    // Record consent withdrawal
    await this.recordConsent(userId, {
      [consentType]: false,
      action: 'withdraw',
      timestamp: new Date().toISOString()
    });

    // Take action based on consent type
    switch (consentType) {
      case 'ai_personalization':
        await this.disableAIPersonalization(userId);
        break;
      case 'marketing':
        await this.unsubscribeMarketing(userId);
        break;
      case 'analytics':
        await this.disableAnalytics(userId);
        break;
    }
  }

  hashIP(ipAddress) {
    // Hash IP for privacy while maintaining audit trail
    return crypto.createHash('sha256').update(ipAddress).digest('hex');
  }
}
```

## 🔍 Audit & Compliance Monitoring

### Audit Logging
```javascript
// Comprehensive audit logging
class AuditLogger {
  async logDataAccess(userId, dataType, accessor, purpose) {
    const auditLog = {
      PK: `AUDIT#${new Date().toISOString().split('T')[0]}`,
      SK: `ACCESS#${new Date().toISOString()}#${generateUUID()}`,
      entity_type: 'AUDIT_LOG',
      event_type: 'data_access',
      user_id: userId,
      data_type: dataType,
      accessed_by: accessor,
      purpose: purpose,
      timestamp: new Date().toISOString(),
      ip_address: this.hashIP(this.getClientIP()),
      user_agent: this.getUserAgent()
    };

    await ddb.put({
      TableName: process.env.DYNAMODB_TABLE,
      Item: auditLog
    });
  }

  async logDataModification(userId, dataType, changes, reason) {
    const auditLog = {
      PK: `AUDIT#${new Date().toISOString().split('T')[0]}`,
      SK: `MODIFY#${new Date().toISOString()}#${generateUUID()}`,
      entity_type: 'AUDIT_LOG',
      event_type: 'data_modification',
      user_id: userId,
      data_type: dataType,
      changes: changes,
      reason: reason,
      timestamp: new Date().toISOString()
    };

    await ddb.put({
      TableName: process.env.DYNAMODB_TABLE,
      Item: auditLog
    });
  }

  async logDataDeletion(userId, dataType, reason) {
    const auditLog = {
      PK: `AUDIT#${new Date().toISOString().split('T')[0]}`,
      SK: `DELETE#${new Date().toISOString()}#${generateUUID()}`,
      entity_type: 'AUDIT_LOG',
      event_type: 'data_deletion',
      user_id: userId,
      data_type: dataType,
      reason: reason,
      timestamp: new Date().toISOString()
    };

    await ddb.put({
      TableName: process.env.DYNAMODB_TABLE,
      Item: auditLog
    });
  }
}
```

### Compliance Monitoring Dashboard
```javascript
// Compliance metrics for admin dashboard
async function getComplianceMetrics() {
  const metrics = {
    gdpr_requests: {
      data_exports: await countGDPRRequests('data_export'),
      data_deletions: await countGDPRRequests('data_deletion'),
      data_rectifications: await countGDPRRequests('data_rectification'),
      processing_objections: await countGDPRRequests('processing_objection')
    },
    consent_metrics: {
      ai_personalization_rate: await getConsentRate('ai_personalization'),
      marketing_consent_rate: await getConsentRate('marketing'),
      analytics_consent_rate: await getConsentRate('analytics')
    },
    data_retention: {
      ai_suggestions_cleanup: await getCleanupStats('ai_suggestions'),
      old_logs_cleanup: await getCleanupStats('audit_logs'),
      inactive_users: await getInactiveUserCount()
    },
    security_incidents: {
      failed_logins: await getFailedLoginCount(),
      suspicious_activities: await getSuspiciousActivityCount(),
      data_breaches: 0 // Manual tracking
    }
  };

  return metrics;
}
```

## Data Retention Policies

### Retention Schedule
```yaml
Data Retention Policies:
  User Profiles:
    Retention: Until account deletion
    Legal Basis: Contract
    
  AI Suggestions:
    Retention: 90 days
    Cleanup: Automatic (DynamoDB TTL)
    Legal Basis: Legitimate Interest
    
  Cooking History:
    Retention: 2 years
    Archive: S3 after 1 year
    Legal Basis: Contract
    
  Audit Logs:
    Retention: 7 years
    Legal Basis: Legal Obligation
    
  Marketing Data:
    Retention: Until consent withdrawn
    Legal Basis: Consent
    
  Analytics Data:
    Retention: 26 months (anonymized)
    Legal Basis: Legitimate Interest
```

### Automated Data Cleanup
```javascript
// Scheduled Lambda for data cleanup
exports.dataCleanupHandler = async (event, context) => {
  const cleanupTasks = [
    cleanupExpiredAISuggestions(),
    cleanupOldAuditLogs(),
    cleanupInactiveUsers(),
    cleanupExpiredSessions()
  ];

  const results = await Promise.allSettled(cleanupTasks);
  
  const summary = {
    ai_suggestions_cleaned: results[0].value || 0,
    audit_logs_archived: results[1].value || 0,
    inactive_users_notified: results[2].value || 0,
    sessions_expired: results[3].value || 0,
    cleanup_date: new Date().toISOString()
  };

  // Log cleanup summary
  console.log('Data cleanup completed:', JSON.stringify(summary));
  
  return summary;
};

async function cleanupExpiredAISuggestions() {
  // DynamoDB TTL handles this automatically
  // This function just reports the count
  const expiredCount = await countExpiredItems('ai_suggestions');
  return expiredCount;
}
```

## Related Documents

- [41 - Privacy](41-privacy.md)
- [40 - Auth](40-auth.md)
- [13 - Security](13-security.md)
- [32 - Monitoring](32-monitoring.md)