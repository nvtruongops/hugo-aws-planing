---
title: "22 - DevOps & Deployment"
weight: 22
---

# DevOps & Deployment - Smart Cooking App

## 🚀 Deployment Strategy

### Infrastructure as Code (IaC)

```yaml
Tool: AWS CDK (TypeScript)
Alternative: AWS SAM
Reason: Better TypeScript integration, more flexible than SAM

Structure:
  - cdk/
    ├── lib/
    │   ├── database-stack.ts      # DynamoDB tables
    │   ├── lambda-stack.ts        # Lambda functions
    │   ├── api-stack.ts           # API Gateway
    │   ├── auth-stack.ts          # Cognito
    │   ├── storage-stack.ts       # S3 buckets
    │   ├── monitoring-stack.ts    # CloudWatch, X-Ray
    │   └── main-stack.ts          # Main orchestration
    ├── bin/
    │   └── app.ts                 # CDK app entry
    └── package.json
```

### Environment Strategy

```yaml
Environments:
  - dev: Development (personal)
  - staging: Pre-production testing
  - prod: Production

Naming Convention:
  - Resources: smart-cooking-{env}-{resource}
  - Example: smart-cooking-prod-api-gateway
```

## 📦 CI/CD Pipeline

### Frontend Pipeline (AWS Amplify)

```yaml
# amplify.yml
version: 1
applications:
  - frontend:
      phases:
        preBuild:
          commands:
            - npm ci
        build:
          commands:
            - npm run build
            - npm start &
            - sleep 5
            - pkill -f "node"
      artifacts:
        baseDirectory: .
        files:
          - '**/*'
        excludePaths:
          - node_modules/**/*
          - .git/**/*
      cache:
        paths:
          - node_modules/**/*
      
      customHeaders:
        - pattern: '**/*'
          headers:
            - key: 'Strict-Transport-Security'
              value: 'max-age=31536000; includeSubDomains'
            - key: 'X-Content-Type-Options'
              value: 'nosniff'
            - key: 'X-Frame-Options'
              value: 'DENY'

environments:
  main:
    environmentVariables:
      API_URL: https://api.smartcooking.app
      NODE_ENV: production
      PORT: 3000
  develop:
    environmentVariables:
      API_URL: https://api-dev.smartcooking.app
      NODE_ENV: development
      PORT: 3000
```

### Backend Pipeline (GitHub Actions)

```yaml
# .github/workflows/deploy.yml
name: Deploy Backend

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  AWS_REGION: us-east-1

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Run linting
        run: npm run lint

  deploy-dev:
    if: github.ref == 'refs/heads/develop'
    needs: test
    runs-on: ubuntu-latest
    environment: development
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Deploy CDK stack
        run: |
          cd cdk
          npm ci
          npx cdk deploy --all --require-approval never
        env:
          ENVIRONMENT: dev

  deploy-prod:
    if: github.ref == 'refs/heads/main'
    needs: test
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID_PROD }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY_PROD }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Deploy CDK stack
        run: |
          cd cdk
          npm ci
          npx cdk deploy --all --require-approval never
        env:
          ENVIRONMENT: prod
```

## 🏗️ Infrastructure Code

### Main CDK Stack

```typescript
// cdk/lib/main-stack.ts
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import { DatabaseStack } from './database-stack';
import { LambdaStack } from './lambda-stack';
import { ApiStack } from './api-stack';
import { AuthStack } from './auth-stack';
import { StorageStack } from './storage-stack';
import { MonitoringStack } from './monitoring-stack';

export class MainStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const env = process.env.ENVIRONMENT || 'dev';

    // Database
    const databaseStack = new DatabaseStack(this, 'Database', {
      environment: env
    });

    // Storage
    const storageStack = new StorageStack(this, 'Storage', {
      environment: env
    });

    // Auth
    const authStack = new AuthStack(this, 'Auth', {
      environment: env
    });

    // Lambda Functions
    const lambdaStack = new LambdaStack(this, 'Lambda', {
      environment: env,
      table: databaseStack.table,
      userPool: authStack.userPool,
      buckets: storageStack.buckets
    });

    // API Gateway
    const apiStack = new ApiStack(this, 'Api', {
      environment: env,
      lambdaFunctions: lambdaStack.functions,
      userPool: authStack.userPool
    });

    // Monitoring
    new MonitoringStack(this, 'Monitoring', {
      environment: env,
      api: apiStack.api,
      lambdaFunctions: lambdaStack.functions
    });

    // Outputs
    new cdk.CfnOutput(this, 'ApiUrl', {
      value: apiStack.api.url,
      description: 'API Gateway URL'
    });

    new cdk.CfnOutput(this, 'UserPoolId', {
      value: authStack.userPool.userPoolId,
      description: 'Cognito User Pool ID'
    });
  }
}
```

### Lambda Stack

```typescript
// cdk/lib/lambda-stack.ts
import * as cdk from 'aws-cdk-lib';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as dynamodb from 'aws-cdk-lib/aws-dynamodb';
import * as cognito from 'aws-cdk-lib/aws-cognito';
import * as s3 from 'aws-cdk-lib/aws-s3';
import { Construct } from 'constructs';

interface LambdaStackProps {
  environment: string;
  table: dynamodb.Table;
  userPool: cognito.UserPool;
  buckets: { [key: string]: s3.Bucket };
}

export class LambdaStack extends Construct {
  public readonly functions: { [key: string]: lambda.Function };

  constructor(scope: Construct, id: string, props: LambdaStackProps) {
    super(scope, id);

    const commonEnvironment = {
      DYNAMODB_TABLE: props.table.tableName,
      USER_POOL_ID: props.userPool.userPoolId,
      ENVIRONMENT: props.environment
    };

    // AI Suggestion Function (Most important)
    const aiSuggestionFunction = new lambda.Function(this, 'AiSuggestion', {
      functionName: `smart-cooking-${props.environment}-ai-suggestion`,
      runtime: lambda.Runtime.NODEJS_20_X,
      handler: 'index.handler',
      code: lambda.Code.fromAsset('lambda-functions/ai-suggestion'),
      memorySize: 1024,
      timeout: cdk.Duration.seconds(60),
      environment: {
        ...commonEnvironment,
        BEDROCK_MODEL_ID: 'anthropic.claude-3-haiku-20240307-v1:0'
      },
      tracing: lambda.Tracing.ACTIVE
    });

    // Cooking History Function
    const cookingHistoryFunction = new lambda.Function(this, 'CookingHistory', {
      functionName: `smart-cooking-${props.environment}-cooking-history`,
      runtime: lambda.Runtime.NODEJS_20_X,
      handler: 'index.handler',
      code: lambda.Code.fromAsset('lambda-functions/cooking-history'),
      memorySize: 256,
      timeout: cdk.Duration.seconds(10),
      environment: commonEnvironment,
      tracing: lambda.Tracing.ACTIVE
    });

    // Rating Handler Function
    const ratingHandlerFunction = new lambda.Function(this, 'RatingHandler', {
      functionName: `smart-cooking-${props.environment}-rating-handler`,
      runtime: lambda.Runtime.NODEJS_20_X,
      handler: 'index.handler',
      code: lambda.Code.fromAsset('lambda-functions/rating-handler'),
      memorySize: 256,
      timeout: cdk.Duration.seconds(10),
      environment: commonEnvironment,
      tracing: lambda.Tracing.ACTIVE
    });

    // Ingredient Validator Function
    const ingredientValidatorFunction = new lambda.Function(this, 'IngredientValidator', {
      functionName: `smart-cooking-${props.environment}-ingredient-validator`,
      runtime: lambda.Runtime.NODEJS_20_X,
      handler: 'index.handler',
      code: lambda.Code.fromAsset('lambda-functions/ingredient-validator'),
      memorySize: 256,
      timeout: cdk.Duration.seconds(10),
      environment: commonEnvironment,
      tracing: lambda.Tracing.ACTIVE
    });

    // Grant permissions
    props.table.grantReadWriteData(aiSuggestionFunction);
    props.table.grantReadWriteData(cookingHistoryFunction);
    props.table.grantReadWriteData(ratingHandlerFunction);
    props.table.grantReadWriteData(ingredientValidatorFunction);

    // Grant Bedrock permissions to AI function
    aiSuggestionFunction.addToRolePolicy(
      new cdk.aws_iam.PolicyStatement({
        effect: cdk.aws_iam.Effect.ALLOW,
        actions: ['bedrock:InvokeModel'],
        resources: ['arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-haiku-*']
      })
    );

    this.functions = {
      aiSuggestion: aiSuggestionFunction,
      cookingHistory: cookingHistoryFunction,
      ratingHandler: ratingHandlerFunction,
      ingredientValidator: ingredientValidatorFunction
    };
  }
}
```

### Database Stack

```typescript
// cdk/lib/database-stack.ts
import * as cdk from 'aws-cdk-lib';
import * as dynamodb from 'aws-cdk-lib/aws-dynamodb';
import { Construct } from 'constructs';

interface DatabaseStackProps {
  environment: string;
}

export class DatabaseStack extends Construct {
  public readonly table: dynamodb.Table;

  constructor(scope: Construct, id: string, props: DatabaseStackProps) {
    super(scope, id);

    // Single table design
    this.table = new dynamodb.Table(this, 'SmartCookingTable', {
      tableName: `smart-cooking-${props.environment}-data`,
      partitionKey: {
        name: 'PK',
        type: dynamodb.AttributeType.STRING
      },
      sortKey: {
        name: 'SK',
        type: dynamodb.AttributeType.STRING
      },
      billingMode: dynamodb.BillingMode.ON_DEMAND,
      encryption: dynamodb.TableEncryption.AWS_MANAGED,
      pointInTimeRecovery: props.environment === 'prod',
      removalPolicy: props.environment === 'prod' 
        ? cdk.RemovalPolicy.RETAIN 
        : cdk.RemovalPolicy.DESTROY,

      // Global Secondary Indexes
      globalSecondaryIndexes: [
        {
          indexName: 'GSI1',
          partitionKey: {
            name: 'GSI1PK',
            type: dynamodb.AttributeType.STRING
          },
          sortKey: {
            name: 'GSI1SK',
            type: dynamodb.AttributeType.STRING
          }
        },
        {
          indexName: 'GSI2',
          partitionKey: {
            name: 'GSI2PK',
            type: dynamodb.AttributeType.STRING
          },
          sortKey: {
            name: 'GSI2SK',
            type: dynamodb.AttributeType.STRING
          }
        },
        {
          indexName: 'GSI3',
          partitionKey: {
            name: 'GSI3PK',
            type: dynamodb.AttributeType.STRING
          },
          sortKey: {
            name: 'GSI3SK',
            type: dynamodb.AttributeType.STRING
          }
        }
      ]
    });

    // Add TTL attribute for auto-cleanup
    this.table.addGlobalSecondaryIndex({
      indexName: 'TTL-Index',
      partitionKey: {
        name: 'ttl',
        type: dynamodb.AttributeType.NUMBER
      }
    });
  }
}
```

## 📊 Monitoring & Logging

### CloudWatch Dashboard

```typescript
// cdk/lib/monitoring-stack.ts
import * as cdk from 'aws-cdk-lib';
import * as cloudwatch from 'aws-cdk-lib/aws-cloudwatch';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as apigateway from 'aws-cdk-lib/aws-apigateway';
import * as sns from 'aws-cdk-lib/aws-sns';
import { Construct } from 'constructs';

interface MonitoringStackProps {
  environment: string;
  api: apigateway.RestApi;
  lambdaFunctions: { [key: string]: lambda.Function };
}

export class MonitoringStack extends Construct {
  constructor(scope: Construct, id: string, props: MonitoringStackProps) {
    super(scope, id);

    // SNS Topic for alerts
    const alertTopic = new sns.Topic(this, 'AlertTopic', {
      topicName: `smart-cooking-${props.environment}-alerts`
    });

    // CloudWatch Dashboard
    const dashboard = new cloudwatch.Dashboard(this, 'Dashboard', {
      dashboardName: `smart-cooking-${props.environment}-dashboard`
    });

    // API Gateway Metrics
    dashboard.addWidgets(
      new cloudwatch.GraphWidget({
        title: 'API Gateway Requests',
        left: [
          new cloudwatch.Metric({
            namespace: 'AWS/ApiGateway',
            metricName: 'Count',
            dimensionsMap: {
              ApiName: props.api.restApiName
            },
            statistic: 'Sum'
          })
        ]
      }),
      new cloudwatch.GraphWidget({
        title: 'API Gateway Latency',
        left: [
          new cloudwatch.Metric({
            namespace: 'AWS/ApiGateway',
            metricName: 'Latency',
            dimensionsMap: {
              ApiName: props.api.restApiName
            },
            statistic: 'Average'
          })
        ]
      })
    );

    // Lambda Metrics
    Object.entries(props.lambdaFunctions).forEach(([name, func]) => {
      dashboard.addWidgets(
        new cloudwatch.GraphWidget({
          title: `${name} Function Metrics`,
          left: [
            func.metricInvocations(),
            func.metricErrors(),
            func.metricDuration()
          ]
        })
      );

      // Alarms
      const errorAlarm = new cloudwatch.Alarm(this, `${name}ErrorAlarm`, {
        alarmName: `smart-cooking-${props.environment}-${name}-errors`,
        metric: func.metricErrors({
          period: cdk.Duration.minutes(5)
        }),
        threshold: 10,
        evaluationPeriods: 2
      });

      errorAlarm.addAlarmAction(
        new cdk.aws_cloudwatch_actions.SnsAction(alertTopic)
      );

      const durationAlarm = new cloudwatch.Alarm(this, `${name}DurationAlarm`, {
        alarmName: `smart-cooking-${props.environment}-${name}-duration`,
        metric: func.metricDuration({
          period: cdk.Duration.minutes(5),
          statistic: 'Average'
        }),
        threshold: name === 'aiSuggestion' ? 30000 : 5000, // 30s for AI, 5s for others
        evaluationPeriods: 3
      });

      durationAlarm.addAlarmAction(
        new cdk.aws_cloudwatch_actions.SnsAction(alertTopic)
      );
    });

    // Cost Alarm
    const costAlarm = new cloudwatch.Alarm(this, 'CostAlarm', {
      alarmName: `smart-cooking-${props.environment}-cost`,
      metric: new cloudwatch.Metric({
        namespace: 'AWS/Billing',
        metricName: 'EstimatedCharges',
        dimensionsMap: {
          Currency: 'USD'
        },
        statistic: 'Maximum',
        period: cdk.Duration.hours(6)
      }),
      threshold: props.environment === 'prod' ? 200 : 50,
      evaluationPeriods: 1
    });

    costAlarm.addAlarmAction(
      new cdk.aws_cloudwatch_actions.SnsAction(alertTopic)
    );
  }
}
```

### Log Aggregation

```typescript
// Lambda function logging setup
const logger = {
  info: (message: string, meta: any = {}) => {
    console.log(JSON.stringify({
      timestamp: new Date().toISOString(),
      level: 'INFO',
      message,
      requestId: context.awsRequestId,
      functionName: context.functionName,
      ...meta
    }));
  },

  error: (message: string, error: Error, meta: any = {}) => {
    console.error(JSON.stringify({
      timestamp: new Date().toISOString(),
      level: 'ERROR',
      message,
      error: {
        name: error.name,
        message: error.message,
        stack: error.stack
      },
      requestId: context.awsRequestId,
      functionName: context.functionName,
      ...meta
    }));
  }
};
```

## 🔒 Security & Compliance

### Security Scanning

```yaml
# .github/workflows/security.yml
name: Security Scan

on:
  push:
    branches: [main, develop]
  schedule:
    - cron: '0 2 * * 1' # Weekly on Monday

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run npm audit
        run: npm audit --audit-level high
      
      - name: Run Snyk security scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high
      
      - name: Run OWASP ZAP scan
        uses: zaproxy/action-full-scan@v0.4.0
        with:
          target: 'https://api-dev.smartcooking.app'
```

### Environment Variables Management

```typescript
// Environment-specific configurations
const config = {
  dev: {
    apiUrl: 'https://api-dev.smartcooking.app',
    cognitoUserPoolId: 'us-east-1_dev123',
    bedrockModel: 'anthropic.claude-3-haiku-20240307-v1:0',
    logLevel: 'DEBUG'
  },
  prod: {
    apiUrl: 'https://api.smartcooking.app',
    cognitoUserPoolId: 'us-east-1_prod456',
    bedrockModel: 'anthropic.claude-3-haiku-20240307-v1:0',
    logLevel: 'INFO'
  }
};

export const getConfig = () => {
  const env = process.env.ENVIRONMENT || 'dev';
  return config[env as keyof typeof config];
};
```

## 📈 Performance Optimization

### Lambda Optimization

```typescript
// Shared connection pool for DynamoDB
let ddbClient: DynamoDBDocumentClient;

export const getDynamoDBClient = () => {
  if (!ddbClient) {
    const client = new DynamoDBClient({
      region: process.env.AWS_REGION || 'us-east-1'
    });
    ddbClient = DynamoDBDocumentClient.from(client);
  }
  return ddbClient;
};

// Warm-up function
export const warmUp = async () => {
  if (process.env.AWS_LAMBDA_FUNCTION_NAME) {
    // Pre-initialize connections
    getDynamoDBClient();
    
    // Pre-load master ingredients cache
    await loadMasterIngredientsCache();
  }
};
```

### Caching Strategy

```typescript
// In-memory cache for master ingredients
let masterIngredientsCache: Map<string, any> | null = null;

export const getMasterIngredient = async (normalizedName: string) => {
  if (!masterIngredientsCache) {
    await loadMasterIngredientsCache();
  }
  
  return masterIngredientsCache?.get(normalizedName);
};

const loadMasterIngredientsCache = async () => {
  const result = await ddb.query({
    TableName: process.env.DYNAMODB_TABLE,
    IndexName: 'GSI1',
    KeyConditionExpression: 'GSI1PK = :pk',
    ExpressionAttributeValues: {
      ':pk': 'INGREDIENT#ACTIVE'
    }
  });

  masterIngredientsCache = new Map();
  result.Items?.forEach(item => {
    masterIngredientsCache?.set(item.normalized_name, item);
  });
};
```

## Related Documents

- [10 - Architecture](10-architecture.md)
- [20 - Backend](20-backend.md)
- [21 - Frontend](21-frontend.md)
- [32 - Monitoring](32-monitoring.md)