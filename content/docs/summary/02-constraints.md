---
title: "02 - Constraints"
weight: 3
---

# Giới Hạn Dự Án - Smart Cooking App

##    Budget Constraints

### Development Budget
- **Initial Investment**: $100 - 500
  - Development time: 3 months
  - Testing & QA
  - Initial infrastructure setup

### Operational Budget (Monthly)
- **MVP Phase (Month 1-3)**:
  - Target: $135-160/month (1,000 users)
  - Maximum: $200/month (hard limit)
  - Buffer: 20% for unexpected costs

- **Growth Phase (Month 4-12)**:
  - Target: $150-400/month (scaling with users)
  - Maximum: $500/month
  - Cost per user: Must decrease as scale increases

### Cost Breakdown Limits
| Service | MVP Budget | Max Budget | Notes |
|---------|------------|------------|-------|
| AWS Amplify | $15 | $20 | Hosting + CI/CD |
| CloudFront | $8 | $15 | CDN |
| API Gateway | $10 | $20 | REST API |
| Lambda | $25 | $40 | 11 functions |
| Bedrock AI | $12-30 | $50 | Flexible mix strategy |
| DynamoDB | $45 | $60 | On-demand |
| S3 | $10 | $20 | Images |
| CloudWatch | $12 | $20 | Logging |
| WAF | $6 | $10 | Security |
| Other | $2 | $5 | Secrets, misc |

### Budget Alarms
- Warning at: $140/month (MVP), $180/month (Growth)
- Critical at: $170/month (MVP), $450/month (Growth)
- Hard stop at: $200/month (MVP), $500/month (Growth)

## Time Constraints

### Project Timeline
- **Total Duration**: 3 months (MVP)
- **Start Date**: Week 1
- **MVP Launch**: Week 12
- **Beta Period**: Week 13-14

### Phase Breakdown
| Phase | Duration | Deadline | Deliverables |
|-------|----------|----------|--------------|
| Planning & Design | 2 weeks | Week 2 | Architecture, DB design, Cost analysis |
| Core Development | 6 weeks | Week 8 | Auth, Ingredients, AI Engine, Rating |
| Social Features | 4 weeks | Week 12 | Privacy, Friends, Posts, Notifications |
| Testing & Launch | 2 weeks | Week 14 | Bug fixes, Beta release |

### Critical Path Items
1. **Week 3-4**: DynamoDB schema + Lambda infrastructure (BLOCKER)
2. **Week 5-6**: AI suggestion engine + Master ingredients (BLOCKER)
3. **Week 7-8**: Rating system + Auto-approval (BLOCKER)
4. **Week 9-10**: Privacy + Friend system
5. **Week 11-12**: Posts + Notifications

### Development Velocity
- **Sprint Length**: 1 week
- **Story Points**: 20-25 per sprint (solo developer)
- **Buffer**: 20% for unexpected issues
- **No Overtime**: Sustainable pace, 40 hours/week

## Technology Constraints

### AWS Service Limits

#### Lambda Constraints
- **Memory**: 256MB-1024MB per function
- **Timeout**: 10-60 seconds (AI: 60s, others: 10-30s)
- **Concurrent Executions**: 1,000 default (can request increase)
- **Package Size**: 50MB compressed, 250MB uncompressed
- **Environment Variables**: 4KB total

#### DynamoDB Constraints
- **Item Size**: 400KB maximum
- **Partition Key**: 2048 bytes
- **Sort Key**: 1024 bytes
- **GSI**: 20 per table (using 3-4)
- **Query Result**: 1MB per query
- **Batch Write**: 25 items per batch

#### API Gateway Constraints
- **Timeout**: 29 seconds maximum
- **Payload Size**: 10MB request/response
- **Rate Limit**: 10,000 requests/second (default)
- **Burst**: 5,000 requests

#### Cognito Constraints
- **User Attributes**: 50 custom attributes max
- **Free Tier**: 50,000 MAU
- **Token Expiration**: 1 hour (access), 30 days (refresh)

#### Bedrock AI Constraints
- **Model**: Claude 3 Haiku (cost-effective) or Sonnet (quality)
- **Input Token Limit**: 200K tokens (Claude 3.5)
- **Output Token Limit**: 8K tokens recommended
- **Rate Limit**: 200 requests/minute (default)
- **Timeout**: 60 seconds per request

#### S3 Constraints
- **Object Size**: 5TB maximum
- **PUT Rate**: 3,500 requests/second per prefix
- **GET Rate**: 5,500 requests/second per prefix
- **Free Tier**: 5GB storage (12 months)

### Technology Stack Fixed Choices
- **Backend Runtime**: Node.js 20 (LTS) - FIXED
- **Database**: DynamoDB (no RDS/PostgreSQL for MVP)
- **AI Provider**: Amazon Bedrock (no OpenAI, no self-hosted)
- **Frontend**: Next.js (React-based)
- **Hosting**: AWS Amplify (no EC2, no custom servers)
- **CDN**: CloudFront (integrated with Amplify)
- **Auth**: Cognito (no custom auth, no Auth0)

### Third-Party Dependencies
- **Minimal External APIs**: Avoid external dependencies
- **No Payment Gateway in MVP**: Defer to Phase 3
- **No Email Service**: Use Cognito built-in email
- **No SMS Service**: Defer to Phase 3

## Resource Constraints

### Team Size
- **Solo Developer**: 1 full-stack developer
- **No Designer**: Use free UI libraries (Tailwind, shadcn/ui)
- **No QA Team**: Self-testing + beta users
- **No DevOps**: Serverless = minimal ops

### Skills & Expertise
- **Required Skills**:
  - JavaScript/TypeScript/Node.js
  - React/Node.js
  - AWS services (Lambda, DynamoDB, Cognito)
  - Serverless architecture
  - AI/ML: Basic understanding (learning curve)
  - DynamoDB single-table design (learning curve)

### Learning Curve Budget
- **Amazon Bedrock**: 1 week to learn API
- **DynamoDB Single-Table Design**: 1 week to master
- **AWS X-Ray Tracing**: 2-3 days
- **Total**: ~2.5 weeks learning time (included in 3-month timeline)

## Security Constraints

### Compliance Requirements
- **GDPR-Ready**: EU users data protection
  - Right to access
  - Right to deletion
  - Data portability
  - Consent management

- **Privacy Policy**: Must have before launch
- **Terms of Service**: Must have before launch
- **Cookie Policy**: If using analytics

### Data Retention
- **User Data**: Keep until user deletes account
- **AI Suggestions**: TTL 90 days (optional cleanup)
- **Cooking History**: Keep for 1 year, then archive
- **Logs**: CloudWatch retention 7 days (cost optimization)

### Security Standards
- **Password Policy**: Enforced by Cognito
  - Min 8 characters
  - 1 uppercase, 1 lowercase, 1 number
  - No common passwords
- **MFA**: Optional (not in MVP)
- **API Keys**: Stored in AWS Secrets Manager
- **Encryption**: All data encrypted at rest and in transit

## Data Constraints

### Data Volume Limits (MVP)
- **Users**: 1,000 active users (target)
- **Recipes**: 5,000 total recipes
- **Master Ingredients**: 5,000 ingredients
- **AI Suggestions**: 10,000 per month
- **Cooking History**: 50,000 entries
- **Posts**: 10,000 posts (Phase 2)
- **Comments**: 30,000 comments (Phase 2)

### Storage Limits
- **DynamoDB**: 25GB free tier, then $0.25/GB
- **S3**: 5GB free tier (12 months), then $0.023/GB
- **User Avatars**: Max 5MB per image
- **Recipe Images**: Max 10MB per image
- **Post Images**: Max 5MB per image (Phase 2)

### Data Quality Constraints
- **Master Ingredients**: Manually curated (start with 500, grow to 5,000)
- **Invalid Ingredients**: Auto-report system, admin review
- **Recipe Quality**: Community-driven (rating >= 4.0 for approval)
- **Duplicate Recipes**: Fuzzy matching to prevent duplicates

## Geographic Constraints

### Target Region
- **Primary**: Vietnam (Vietnamese users)
- **Secondary**: Southeast Asia
- **Language**: Vietnamese + English (MVP)

### AWS Region
- **Primary Region**: us-east-1 (N. Virginia)
  - Reason: Lowest cost, most services available
  - Bedrock available
  - Amplify supported
- **Secondary Region**: None (MVP)
- **CloudFront**: Global CDN (auto-distributed)

### Latency Expectations
- **Vietnam to us-east-1**: ~200-250ms base latency
- **API Response**: +100-500ms processing
- **Total**: ~300-750ms acceptable for MVP
- **Optimization**: CloudFront caching reduces latency for static assets

## Operational Constraints

### Monitoring & Alerting
- **Free Tier Usage**: Maximize AWS Free Tier
- **CloudWatch Alarms**: Max 10 alarms (free tier)
- **Log Retention**: 7 days (cost limit)
- **X-Ray Tracing**: Sample 10% of requests (cost limit)

### Support & Maintenance
- **No 24/7 Support**: Best-effort during business hours
- **Response Time**: 48 hours for non-critical issues
- **Downtime Window**: Sunday 2-4 AM UTC (acceptable for maintenance)

### Backup & Recovery
- **DynamoDB**: Point-in-time recovery enabled (cost: ~$20/month)
- **RTO** (Recovery Time Objective): 4 hours
- **RPO** (Recovery Point Objective): 1 hour (PITR)
- **S3**: Versioning enabled for critical buckets

## Platform Constraints

### MVP Platform Support
- **Web App**: Priority 1
  - Desktop: Chrome, Firefox, Safari, Edge (latest 2 versions)
  - Mobile Web: iOS Safari, Android Chrome

### Browser Compatibility
- **Modern Browsers Only**: ES6+ required
- **No IE11 Support**: Not supporting legacy browsers
- **Mobile-First Design**: Responsive from 320px width

## Scaling Constraints

### MVP Scale Targets
- **Concurrent Users**: 100 (MVP), 1,000 (Month 6)
- **Requests/Second**: 10 (MVP), 100 (Month 6)
- **Database Reads**: 10K/day (MVP), 100K/day (Month 6)
- **AI Calls**: 1K/month (MVP), 10K/month (Month 6)

### Scaling Strategy
- **Vertical Scaling**: Increase Lambda memory (256MB → 1024MB)
- **Horizontal Scaling**: Serverless auto-scales
- **Database Scaling**: DynamoDB on-demand (auto-scales)
- **Cost Scaling**: Must stay below $0.20/user/month (MVP)

### Scaling Blockers
- **AI Cost**: Most expensive component (30-50% of total cost)
- **Mitigation**: Flexible DB/AI mix, cache AI responses
- **Target**: Reduce AI cost by 50% through DB coverage growth

## Risk Constraints

### Technical Risks
1. **DynamoDB Single-Table Design Complexity**: High learning curve
   - Mitigation: 1 week study period, consulting documentation
2. **Bedrock API Rate Limits**: Could block AI suggestions
   - Mitigation: Implement queue system, retry logic
3. **Cold Start Latency**: Lambda cold starts can be slow
   - Mitigation: Provisioned concurrency for critical functions

### Business Risks
1. **Low User Adoption**: < 500 users in 3 months
   - Mitigation: Beta testing, marketing strategy
2. **High Cost Overrun**: > $200/month
   - Mitigation: Budget alarms, cost monitoring dashboard
3. **AI Recipe Quality**: Low approval rate (< 30%)
   - Mitigation: Prompt engineering, A/B testing

### Legal Risks
1. **GDPR Non-Compliance**: EU user data violations
   - Mitigation: Privacy policy, data deletion API, consent management
2. **User-Generated Content**: Inappropriate content
   - Mitigation: Content moderation (Phase 2), reporting system

## Quality Constraints

### Testing Requirements
- **Unit Tests**: >= 70% coverage for critical paths
- **Integration Tests**: API endpoint testing
- **Manual Testing**: Full user journey testing before launch
- **Performance Testing**: Load testing with 100 concurrent users
- **Security Testing**: OWASP top 10 vulnerabilities check

### Code Quality
- **ESLint**: Enforce code standards
- **Prettier**: Code formatting
- **TypeScript**: Type safety (preferred, not required for MVP)
- **Code Reviews**: Self-review + checklist

### Documentation
- **API Documentation**: OpenAPI/Swagger spec
- **Architecture Diagrams**: Mermaid diagrams in Markdown
- **Deployment Guide**: Step-by-step AWS setup
- **User Guide**: Basic usage instructions

## Integration Constraints

### No External Integrations in MVP
- No payment gateway (Stripe/PayPal) - Phase 3
- No email marketing (Mailchimp) - Phase 3
- No analytics (Google Analytics) - Use CloudWatch
- No social login (Google/Facebook) - Cognito only
- No third-party recipe APIs - Self-contained

### Future Integrations (Post-MVP)
- Stripe for payments
- SendGrid for transactional emails
- Mixpanel/Amplitude for analytics
- Firebase Cloud Messaging for push notifications

## Summary of Critical Constraints

### Hard Constraints (Cannot Change)
1. Budget: $200/month maximum (MVP)
2. Timeline: 3 months to MVP launch
3. Team: Solo developer
4. Technology: AWS serverless stack
5. Region: us-east-1 (AWS limitations)

### Soft Constraints (Can Negotiate)
1. User Target: 1,000 users (can be 500-1,500)
2. Features: Social features can defer to Phase 2
3. Performance: 500ms API can be 750ms if needed
4. Test Coverage: 70% can be 50% for MVP

### Constraints Impact Matrix
| Constraint | Impact | Mitigation | Priority |
|------------|--------|------------|----------|
| Budget | HIGH | Cost alarms, flexible AI mix | P0 |
| Timeline | HIGH | Agile sprints, MVP focus | P0 |
| Solo Dev | MEDIUM | Serverless, managed services | P0 |
| Learning Curve | MEDIUM | 2.5 weeks learning budget | P1 |
| AI Cost | HIGH | DB coverage growth strategy | P0 |
| Scaling | LOW | Serverless auto-scaling | P2 |

## Related Documents

- [00 - Overview](00-overview.md)
- [01 - Requirements](01-requirements.md)
- [30 - Cost Analysis](30-cost-analysis.md)
- [31 - Scaling](31-scaling.md)
