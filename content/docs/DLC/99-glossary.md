---
title: "99 - Glossary"
weight: 99
---

# Glossary - Smart Cooking App

## Technical Terms

### A

**API Gateway**
- AWS service that creates, publishes, maintains, monitors, and secures REST APIs
- Acts as front door for backend services

**Auto-Approval**
- Automated system that approves AI-generated recipes when average rating ≥ 4.0 stars
- Reduces manual moderation workload

**AWS Amplify**
- Full-stack development platform for building web and mobile applications
- Used for frontend hosting and CI/CD

**AWS Bedrock**
- Fully managed service for building AI applications with foundation models
- Used for Claude 3 Haiku AI recipe generation

### B

**Bedrock Model**
- Foundation model available through AWS Bedrock
- Claude 3 Haiku: Cost-effective model for recipe generation

### C

**CDK (Cloud Development Kit)**
- Infrastructure as Code framework for defining AWS resources
- Uses TypeScript for type safety

**CloudFront**
- AWS Content Delivery Network (CDN) service
- Caches static assets globally for faster access

**CloudWatch**
- AWS monitoring and observability service
- Collects metrics, logs, and events

**Cognito**
- AWS identity and access management service
- Handles user authentication and authorization

**Cold Start**
- Initial delay when Lambda function executes after being idle
- Mitigated with provisioned concurrency

### D

**DynamoDB**
- AWS NoSQL database service
- Uses single-table design for optimal performance

**DB Coverage**
- Percentage of recipe requests served from database vs AI generation
- Target: 0% → 80% over time for cost optimization

### E

**EJS (Embedded JavaScript)**
- Template engine for generating HTML with JavaScript
- Used for server-side rendering

**Express.js**
- Web application framework for Node.js
- Handles routing and middleware

### F

**Flexible Mix Strategy**
- Algorithm that combines database recipes with AI-generated recipes
- Optimizes cost by reducing AI API calls

**Fuzzy Search**
- Approximate string matching for ingredient validation
- Handles typos and variations in ingredient names

### G

**GSI (Global Secondary Index)**
- Alternative query patterns for DynamoDB tables
- Enables efficient data access patterns

### I

**IAM (Identity and Access Management)**
- AWS service for managing access to resources
- Implements least privilege principle

**Invalid Ingredient Handling**
- System for managing unrecognized ingredients
- Provides suggestions and logs for admin review

### J

**JWT (JSON Web Token)**
- Secure token format for authentication
- Contains user claims and expiration

### L

**Lambda Function**
- AWS serverless compute service
- Executes code without managing servers

**Least Privilege**
- Security principle of granting minimum necessary permissions
- Applied to all IAM roles and policies

### M

**Master Ingredients**
- Curated database of valid cooking ingredients
- Used for validation and fuzzy matching

**MVP (Minimum Viable Product)**
- Initial version with core features only
- Target: 1,000 users in 3 months

### N

**Node.js**
- JavaScript runtime for server-side development
- Used for frontend web application

### P

**PITR (Point-in-Time Recovery)**
- DynamoDB backup feature for data protection
- Enables restoration to any point within retention period

**Privacy Filtering**
- System that respects user privacy settings
- Controls data visibility based on relationship status

### R

**Rate Limiting**
- Controls request frequency to prevent abuse
- Implemented at WAF and application levels

**Recipe Auto-Creation**
- Process where highly-rated AI recipes become permanent
- Triggered when average rating ≥ 4.0 stars

### S

**Single-Table Design**
- DynamoDB pattern using one table for all entities
- Optimizes performance and cost

**Serverless**
- Architecture pattern without server management
- Auto-scales and charges only for usage

### T

**TTL (Time To Live)**
- Automatic data expiration in DynamoDB
- Used for cleaning up temporary data

### U

**User Pool**
- Cognito directory for user management
- Handles registration, authentication, and user attributes

### V

**Validation**
- Process of checking ingredient names against master list
- Includes fuzzy matching and auto-correction

### W

**WAF (Web Application Firewall)**
- AWS security service protecting against web exploits
- Blocks malicious requests and implements rate limiting

### X

**X-Ray**
- AWS distributed tracing service
- Tracks requests across multiple services

## 🏢 Business Terms

### A

**Approval Rate**
- Percentage of AI recipes that achieve ≥4.0 star rating
- Target: 40%+ for sustainable growth

### C

**Cost per User**
- Monthly operational cost divided by active users
- Target: <$0.20 (MVP), <$0.03 (Scale)

### D

**DAU (Daily Active Users)**
- Number of unique users active each day
- Key engagement metric

### M

**MAU (Monthly Active Users)**
- Number of unique users active each month
- Primary growth metric

### R

**Recipe Database Coverage**
- Percentage of suggestions served from approved recipes vs AI
- Growth target: 0% → 80% over 12 months

**Retention Rate**
- Percentage of users who return after initial visit
- Target: 60% (30-day retention)

### U

**User Journey**
- Complete flow from registration to recipe rating
- Critical path for product success

## Security Terms

### E

**Encryption at Rest**
- Data protection when stored in databases/files
- All data encrypted using AWS KMS

**Encryption in Transit**
- Data protection during transmission
- HTTPS/TLS enforced everywhere

### G

**GDPR (General Data Protection Regulation)**
- EU privacy regulation for data protection
- Implemented through privacy controls and data rights

### M

**MFA (Multi-Factor Authentication)**
- Additional security layer beyond password
- Optional feature for enhanced security

### O

**OWASP Top 10**
- List of most critical web application security risks
- Used as security checklist

## Performance Terms

### L

**Latency**
- Time delay between request and response
- Target: <500ms for API endpoints

### P

**P95 Latency**
- 95th percentile response time
- Key performance indicator

### T

**Throughput**
- Number of requests processed per unit time
- Measured in requests per second

### U

**Uptime**
- Percentage of time system is operational
- Target: 99.5% monthly uptime

## Related Documents

- [00 - Overview](00-overview.md)
- [99 - References](99-references.md)