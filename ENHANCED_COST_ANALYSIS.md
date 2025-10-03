# COST ANALYSIS - ENHANCED FEATURES

##    Chi Phí So Sánh (1,000 users)

### Version Comparison

| Component | MVP Basic | Enhanced | Delta | Note |
|-----------|-----------|----------|-------|------|
| **DynamoDB** | $12 | $15 | +$3 | +GSI2 (categories) |
| **Bedrock AI** | $15 | $12-30 | ±$15 | Flexible (see below) |
| **CloudWatch** | $5 | $7 | +$2 | Invalid ingredient logs |
| **API Gateway** | $6 | $6 | $0 | Same endpoints |
| **Lambda** | $12 | $14 | +$2 | Enhanced logic |
| **Other** | $23 | $23 | $0 | Amplify, CloudFront, Cognito |
| **TOTAL** | **$73** | **$77-95** | **+$4-22** | Depends on usage |

---

##    Bedrock Cost Analysis (Key Variable)

### Scenario 1: DB đủ recipes (Best Case)
```
1,000 users × 10 requests/tháng = 10,000 requests

Average recipe count per request: 3 món
DB coverage: 100% (DB có đủ recipes)

AI calls: 0
Bedrock cost: $0 ✅

Total cost: $77/tháng
```

### Scenario 2: Mixed DB/AI (Average Case)
```
10,000 requests × 3 món average = 30,000 món requested

DB coverage: 60%
→ DB recipes: 18,000 món (free)
→ AI recipes: 12,000 món (cost money)

Bedrock cost calculation:
- Input tokens: 200 tokens/request × 12,000 = 2.4M tokens
- Output tokens: 800 tokens/request × 12,000 = 9.6M tokens
- Cost (Haiku): (2.4M × $0.25 + 9.6M × $1.25) / 1M = $12.60

Total cost: $77 + $13 = $90/tháng
```

### Scenario 3: DB rỗng (Worst Case)
```
10,000 requests × 3 món = 30,000 món
DB coverage: 0% (cold start, empty DB)
→ AI recipes: 30,000 món

Bedrock cost:
- Input: 6M tokens × $0.25/1M = $1.50
- Output: 24M tokens × $1.25/1M = $30.00
- Total: $31.50

Total cost: $77 + $32 = $109/tháng ⚠️
```

### Scenario 4: User request 5 món (Power Users)
```
Assuming 20% users request 5 món, 80% request 1 món:
- 200 users × 10 req × 5 món = 10,000 món
- 800 users × 10 req × 1 món = 8,000 món
- Total: 18,000 món

DB coverage: 50%
→ AI: 9,000 món

Bedrock cost:
- Input: 1.8M × $0.25 = $0.45
- Output: 7.2M × $1.25 = $9.00
- Total: $9.45

Total cost: $77 + $9 = $86/tháng
```

---

##    Cost Optimization Strategies

### Strategy 1: Freemium Tiers
```javascript
// Free tier
{
  recipe_count_limit: 1,  // Chỉ 1 món
  requests_per_day: 5
}
// Cost impact: Minimal AI usage

// Premium tier ($2.99/month)
{
  recipe_count_limit: 5,  // Tối đa 5 món
  requests_per_day: unlimited
}
// Cost impact: Higher but controlled
```

**Break-even analysis**:
- Free users (900): 900 × 10 req × 1 món = 9,000 món
- Premium users (100): 100 × 30 req × 3 món = 9,000 món
- Total: 18,000 món × 50% AI = $9 Bedrock
- Revenue: 100 × $2.99 = $299
- Profit after costs: $299 - $86 = $213 ✅

### Strategy 2: Intelligent DB Building
```javascript
// Auto-save popular AI recipes to DB
async function autoApprovePopularAIRecipe(recipe) {
  const usageCount = await getRecipeUsageCount(recipe.id);

  if (usageCount >= 10) {  // Used by 10+ users
    await saveToDatabase({
      ...recipe,
      is_approved: true,
      approval_type: 'auto_popular'
    });

    // Future requests: DB hit instead of AI! 🎉
  }
}
```

**Cost reduction over time**:
```
Month 1: DB coverage 0% → Cost $109
Month 2: DB coverage 20% → Cost $95
Month 3: DB coverage 40% → Cost $87
Month 4: DB coverage 60% → Cost $81
Month 6: DB coverage 80% → Cost $77 ✅
```

### Strategy 3: Smart Caching
```javascript
// Cache AI recipes in DynamoDB
{
  PK: "CACHE#thịt_gà_cà_chua_xào",  // ingredients hash
  SK: "METHOD#xào",
  recipe: {...},
  hit_count: 45,
  last_accessed: "2025-10-03",
  ttl: 30 days
}

// Reuse if same ingredients + category
const cacheHit = await checkCache(ingredientsHash, category);
if (cacheHit) {
  return cacheHit.recipe;  // FREE! No Bedrock call
}
```

**Cost savings**:
- Cache hit rate: 30% (conservative)
- AI calls reduced: 12,000 → 8,400 (30% less)
- Savings: $13 → $9 = **$4/month**

---

##    Cost Projection (6 months)

### With Optimization Strategies

| Month | Users | DB Coverage | AI Calls | Bedrock $ | Total $ | Note |
|-------|-------|-------------|----------|-----------|---------|------|
| 1 | 100 | 0% | 3,000 | $3 | $80 | Cold start |
| 2 | 500 | 10% | 13,500 | $14 | $91 | Building DB |
| 3 | 1,000 | 30% | 21,000 | $21 | $98 | Growth |
| 4 | 1,000 | 50% | 15,000 | $15 | $92 | DB maturing |
| 5 | 1,000 | 70% | 9,000 | $9 | $86 | Optimized |
| 6 | 1,000 | 80% | 6,000 | $6 | $83 | Stable ✅ |

**Average cost**: ~$88/month (vs $109 worst case)

---

##    Cost by Feature

### Invalid Ingredient Reporting
```
CloudWatch Logs: 10,000 validations/month
- Most ingredients valid: 80%
- Invalid: 20% = 2,000 logs

CloudWatch cost:
- Ingestion: 2,000 × 1KB × $0.50/GB = $0.001
- Storage: Negligible
- Total: ~$0.01/month

Admin notifications (report >= 5):
- DynamoDB writes: ~50/month
- Cost: $0.025

Total: ~$0.03/month (negligible)
```

### Recipe Categories
```
Additional DynamoDB storage:
- cooking_method + meal_type: ~100 bytes per recipe
- 5,000 recipes × 100 bytes = 0.5 MB
- Cost: $0.0001/month (negligible)

GSI2 (category index):
- Storage: Same as main table
- Reads: Same pricing
- Cost: ~$0.50/month

Total: ~$0.50/month
```

### Flexible Recipe Count
```
Variable Bedrock cost (covered above)

Additional DynamoDB:
- User preferences: 1,000 users × 50 bytes = 50 KB
- Negligible cost

Total: $0-18/month (depends on AI usage)
```

---

##    Final Cost Summary

### Conservative Estimate (1,000 users)
```
Infrastructure: $65
DynamoDB: $15 (+GSI2, reports)
Bedrock AI: $15 (60% DB coverage)
CloudWatch: $7 (enhanced logging)
Lambda: $14 (enhanced logic)

TOTAL: ~$90/month
```

### Best Case (DB mature)
```
TOTAL: ~$77/month
(Same as MVP basic!)
```

### Worst Case (DB empty, all 5 món)
```
TOTAL: ~$109/month
(Still acceptable for 1,000 users)
```

---

## ✅ ROI with Enhanced Features

### Freemium Model
```
Revenue:
- 100 premium users × $2.99 = $299/month
- Ads (optional): $50/month
- Total: $349/month

Cost: $90/month (average)

Profit: $259/month
ROI: 288% ✅
```

### Premium Features Justification
```
Free tier:
- 1 món per request
- 5 requests/day
→ Good for casual users

Premium ($2.99/month):
- 5 món per request
- Unlimited requests
- Priority support
→ Worth it for serious home cooks!
```

---

##    Recommendations

1. ✅ **Implement all 4 features** - Cost increase acceptable
2. ✅ **Use freemium tier** - Control AI costs
3. ✅ **Auto-build DB** - Reduce AI dependency over time
4. ✅ **Set billing alarms**: $85, $100, $120
5. ✅ **Monitor DB coverage** - Target 80% by month 6

**Expected stable cost**: **$77-90/month** (1,000 users) ✅
