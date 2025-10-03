---
title: "32 - Monitoring & Observability"
weight: 32
---

# Monitoring & Observability - Smart Cooking App

##    Monitoring Strategy

### Observability Pillars
1. **Metrics**: Quantitative measurements (CloudWatch)
2. **Logs**: Event records (CloudWatch Logs)
3. **Traces**: Request flow tracking (X-Ray)
4. **Alerts**: Proactive notifications (SNS)

### Monitoring Objectives
- **Performance**: API response time < 500ms (p95)
- **Reliability**: 99.5% uptime target
- **Cost Control**: Budget alerts and optimization
- **User Experience**: Error rate < 1%

## 🔍 CloudWatch Metrics

### Application Metrics

#### API Gateway Metrics
```yaml
Key Metrics:
  - Count: Total requests per minute
  - Latency: Response time (Average, p95, p99)
  - 4XXError: Client errors (authentication, validation)
  - 5XXError: Server errors (Lambda failures, timeouts)
  - IntegrationLatency: Backend processing time

Alarms:
  - API Latency > 1000ms (p95) for 5 minutes
  - Error Rate > 5% for 3 minutes
  - Request Count > 1000/minute (scale alert)
```

#### Lambda Function Metrics
```yaml
Per Function Tracking:
  - Invocations: Function call count
  - Duration: Execution time
  - Errors: Function failures
  - Throttles: Concurrency limit hits
  - ColdStarts: New container initializations

Critical Functions:
  ai-suggestion:
    - Duration < 30s (p95)
    - Error rate < 2%
    - Cold start rate < 10%
  
  cooking-history:
    - Duration < 5s (p95)
    - Error rate < 1%
  
  rating-handler:
    - Duration < 3s (p95)
    - Error rate < 0.5%
```

#### DynamoDB Metrics
```yaml
Table Metrics:
  - ConsumedReadCapacityUnits
  - ConsumedWriteCapacityUnits
  - ThrottledRequests (should be 0)
  - SuccessfulRequestLatency
  - SystemErrors

GSI Metrics:
  - GSI1/GSI2/GSI3 capacity consumption
  - Query performance per index
  - Hot partition detection

Alarms:
  - Throttled requests > 0
  - Read/Write capacity > 80%
  - Query latency > 100ms (p95)
```

### Business Metrics

#### User Engagement
```yaml
Custom Metrics:
  - Daily Active Users (DAU)
  - AI Suggestions per User per Day
  - Recipe Completion Rate
  - Rating Submission Rate
  - Social Interaction Rate

Tracking Method:
  - Lambda functions emit custom metrics
  - CloudWatch custom namespace: SmartCooking/Business
  - Dimensions: Environment, UserTier, Feature
```

#### AI Performance
```yaml
AI-Specific Metrics:
  - AI Generation Success Rate
  - DB vs AI Mix Ratio
  - Recipe Auto-Approval Rate
  - Invalid Ingredient Report Rate
  - Average AI Response Time

Implementation:
```javascript
// Custom metric emission in Lambda
const cloudwatch = new CloudWatchClient({});

await cloudwatch.send(new PutMetricDataCommand({
  Namespace: 'SmartCooking/AI',
  MetricData: [{
    MetricName: 'RecipeGenerationTime',
    Value: duration,
    Unit: 'Milliseconds',
    Dimensions: [{
      Name: 'Environment',
      Value: process.env.ENVIRONMENT
    }]
  }]
}));
```

##    Logging Strategy

### Log Structure
```json
{
  "timestamp": "2025-01-20T10:30:00.000Z",
  "level": "INFO|WARN|ERROR",
  "requestId": "uuid-123",
  "userId": "user-456",
  "functionName": "ai-suggestion",
  "event": "recipe_generated",
  "message": "AI recipe generated successfully",
  "metadata": {
    "ingredients": ["chicken", "tomato"],
    "recipeCount": 3,
    "dbRecipes": 2,
    "aiRecipes": 1,
    "duration": 2500
  },
  "error": {
    "name": "ValidationError",
    "message": "Invalid ingredient",
    "stack": "..."
  }
}
```

### Log Groups Configuration
```yaml
CloudWatch Log Groups:
  /aws/lambda/smart-cooking-prod-ai-suggestion:
    Retention: 7 days
    Subscription: Error alerts to SNS
  
  /aws/lambda/smart-cooking-prod-cooking-history:
    Retention: 7 days
  
  /aws/lambda/smart-cooking-prod-rating-handler:
    Retention: 7 days
  
  /aws/apigateway/smart-cooking-api:
    Retention: 3 days
    Format: CLF (Common Log Format)
  
  /aws/waf/smart-cooking:
    Retention: 30 days
    Content: Blocked requests, rate limits

Log Filtering:
  - ERROR level: Immediate SNS alert
  - WARN level: Daily digest
  - Invalid ingredients: Weekly admin report
```

### Structured Logging Implementation
```javascript
// shared/logger.js
class Logger {
  constructor(functionName, requestId) {
    this.functionName = functionName;
    this.requestId = requestId;
  }

  info(message, metadata = {}) {
    this.log('INFO', message, metadata);
  }

  warn(message, metadata = {}) {
    this.log('WARN', message, metadata);
  }

  error(message, error, metadata = {}) {
    this.log('ERROR', message, {
      ...metadata,
      error: {
        name: error.name,
        message: error.message,
        stack: error.stack
      }
    });
  }

  log(level, message, metadata) {
    const logEntry = {
      timestamp: new Date().toISOString(),
      level,
      requestId: this.requestId,
      functionName: this.functionName,
      message,
      ...metadata
    };

    console.log(JSON.stringify(logEntry));
  }
}

// Usage in Lambda
exports.handler = async (event, context) => {
  const logger = new Logger(context.functionName, context.awsRequestId);
  
  logger.info('Processing AI suggestion request', {
    userId: event.requestContext.authorizer.claims.sub,
    ingredients: JSON.parse(event.body).ingredients
  });
};
```

## 🔍 Distributed Tracing (X-Ray)

### Tracing Configuration
```yaml
Services Traced:
  - API Gateway: 100% sampling
  - Lambda (AI Suggestion): 100% sampling
  - Lambda (Others): 10% sampling
  - DynamoDB: Auto-instrumented
  - Bedrock: Auto-instrumented

Trace Segments:
  1. API Gateway → Lambda
  2. Lambda → DynamoDB Query
  3. Lambda → Bedrock API
  4. Lambda → DynamoDB Write
  5. Lambda → Response

Annotations:
  - user_id: For user-specific analysis
  - recipe_count: Request size
  - db_recipes_found: DB hit rate
  - ai_recipes_generated: AI usage
  - total_duration: End-to-end timing
```

### X-Ray Implementation
```javascript
// Enable X-Ray tracing in Lambda
const AWSXRay = require('aws-xray-sdk-core');
const AWS = AWSXRay.captureAWS(require('aws-sdk'));

exports.handler = async (event, context) => {
  // Create subsegment for AI generation
  const segment = AWSXRay.getSegment();
  const subsegment = segment.addNewSubsegment('ai-generation');
  
  try {
    subsegment.addAnnotation('user_id', userId);
    subsegment.addAnnotation('recipe_count', recipeCount);
    
    const aiRecipes = await generateAIRecipes(ingredients);
    
    subsegment.addMetadata('ai_response', {
      recipes_generated: aiRecipes.length,
      total_tokens: aiRecipes.reduce((sum, r) => sum + r.tokens, 0)
    });
    
    subsegment.close();
    return aiRecipes;
  } catch (error) {
    subsegment.addError(error);
    subsegment.close();
    throw error;
  }
};
```

##    Alerting System

### Alert Categories

#### Critical Alerts (Immediate Response)
```yaml
System Down:
  - API Gateway 5XX > 50% for 2 minutes
  - Lambda function errors > 90% for 1 minute
  - DynamoDB throttling > 10 events in 5 minutes

Performance Degradation:
  - API latency > 2s (p95) for 5 minutes
  - Lambda timeout rate > 10% for 3 minutes
  - AI generation failure rate > 20% for 5 minutes

Security Issues:
  - WAF blocked requests > 1000 in 5 minutes
  - Unusual authentication failures > 100 in 1 minute
  - Cost spike > 200% of daily average
```

#### Warning Alerts (Monitor & Plan)
```yaml
Capacity Concerns:
  - DynamoDB capacity > 70% for 10 minutes
  - Lambda concurrent executions > 80% for 5 minutes
  - Daily cost > budget threshold

Quality Issues:
  - Recipe approval rate < 30% for 1 hour
  - Invalid ingredient reports > 50 per hour
  - User error rate > 5% for 10 minutes
```

### SNS Alert Configuration
```yaml
Topics:
  smart-cooking-prod-critical:
    Subscribers:
      - Email: admin@smartcooking.app
      - SMS: +84xxx (for critical only)
    
  smart-cooking-prod-warnings:
    Subscribers:
      - Email: team@smartcooking.app
      - Slack: #alerts channel

Alert Format:
  Subject: "[CRITICAL] Smart Cooking - API Gateway High Error Rate"
  Body: |
    Alert: API Gateway 5XX Error Rate
    Environment: Production
    Current Value: 15.2%
    Threshold: 5%
    Duration: 3 minutes
    
    Dashboard: https://console.aws.amazon.com/cloudwatch/...
    Runbook: https://wiki.smartcooking.app/runbooks/api-errors
```

##    Dashboards

### Executive Dashboard
```yaml
High-Level Metrics (Daily/Weekly):
  - Total Users (Active)
  - Revenue (Premium subscriptions)
  - System Uptime %
  - Cost per User
  - Customer Satisfaction (App Store ratings)

Widgets:
  - User Growth Chart (30 days)
  - Revenue Trend (90 days)
  - Cost Breakdown Pie Chart
  - System Health Status
```

### Operations Dashboard
```yaml
Real-Time Metrics:
  - API Request Rate (requests/minute)
  - Error Rate % (last 1 hour)
  - Average Response Time (p95)
  - Active Lambda Invocations
  - DynamoDB Throttling Events

Performance Widgets:
  - API Gateway Latency (5-minute intervals)
  - Lambda Duration by Function
  - DynamoDB Query Performance
  - AI Generation Success Rate
  - Cost Trend (daily)
```

### AI Performance Dashboard
```yaml
AI-Specific Metrics:
  - DB vs AI Mix Ratio (daily trend)
  - Recipe Auto-Approval Rate
  - AI Generation Time Distribution
  - Invalid Ingredient Reports
  - Bedrock API Success Rate

Business Impact:
  - Cost Savings from DB Mix
  - User Satisfaction with AI Recipes
  - Recipe Database Growth Rate
```

### Dashboard Implementation
```typescript
// CDK Dashboard Configuration
const dashboard = new cloudwatch.Dashboard(this, 'SmartCookingDashboard', {
  dashboardName: 'smart-cooking-production'
});

// API Gateway metrics
dashboard.addWidgets(
  new cloudwatch.GraphWidget({
    title: 'API Gateway Requests',
    left: [apiGateway.metricCount()],
    right: [apiGateway.metricLatency()]
  }),
  
  new cloudwatch.GraphWidget({
    title: 'Lambda Performance',
    left: [
      aiSuggestionFunction.metricDuration(),
      cookingHistoryFunction.metricDuration()
    ],
    right: [
      aiSuggestionFunction.metricErrors(),
      cookingHistoryFunction.metricErrors()
    ]
  })
);

// Custom business metrics
dashboard.addWidgets(
  new cloudwatch.GraphWidget({
    title: 'AI Performance',
    left: [
      new cloudwatch.Metric({
        namespace: 'SmartCooking/AI',
        metricName: 'RecipeGenerationTime',
        statistic: 'Average'
      })
    ]
  })
);
```

##    Monitoring Tools & Integration

### Third-Party Integrations
```yaml
Slack Integration:
  - Critical alerts to #alerts channel
  - Daily summary to #operations channel
  - Cost reports to #finance channel

PagerDuty (Future):
  - Escalation for critical alerts
  - On-call rotation management
  - Incident response coordination

Datadog (Alternative):
  - Advanced APM capabilities
  - Custom dashboards
  - Machine learning anomaly detection
```

### Monitoring Automation
```yaml
Auto-Remediation:
  - Lambda timeout → Increase memory allocation
  - DynamoDB throttling → Enable auto-scaling
  - High error rate → Circuit breaker activation
  - Cost spike → Rate limiting activation

Health Checks:
  - Synthetic transactions every 5 minutes
  - End-to-end user journey testing
  - Database connectivity checks
  - AI service availability monitoring
```

##    Performance Baselines

### Current Baselines (MVP)
```yaml
API Performance:
  - Average Response Time: 200ms
  - P95 Response Time: 500ms
  - P99 Response Time: 1000ms
  - Error Rate: <0.5%

Lambda Performance:
  - AI Suggestion: 3-5s average
  - Other Functions: <1s average
  - Cold Start Rate: <5%
  - Memory Utilization: <70%

Database Performance:
  - Query Latency: <50ms (p95)
  - Write Latency: <25ms (p95)
  - Throttling Events: 0
  - Hot Partitions: 0
```

### Target Improvements (Scale)
```yaml
6-Month Targets:
  - API P95 Latency: <300ms
  - AI Generation: <2s average
  - Error Rate: <0.1%
  - Uptime: >99.9%

12-Month Targets:
  - API P95 Latency: <200ms
  - AI Generation: <1s average
  - Cost per User: <$0.05
  - DB Coverage: >80%
```

## Related Documents

- [30 - Cost Analysis](30-cost-analysis.md)
- [31 - Scaling](31-scaling.md)
- [10 - Architecture](10-architecture.md)
- [22 - DevOps](22-devops.md)