# Data Analysis Agent: Natural Language to Insights

## Business Problem

**Challenge:** Business users struggle to get insights from data:
- Dependency on data analysts (3-5 day turnaround)
- SQL knowledge required
- Ad-hoc requests bottleneck data team
- Insights delayed, decisions slower
- $300K/year in analyst time for repetitive queries

**Impact:**
- Business decisions delayed by days
- Data team overwhelmed with requests
- Self-service BI tools underutilized (too complex)
- Missed opportunities due to slow insights

## Agentic AI Solution

### Architecture

```
Natural Language Query → SQL Agent → Execution → Visualization Agent → Insights
                           ↓                          ↓
                      Schema Understanding      Chart Generation
```

**Key Components:**
1. **SQL Generation Agent**: Converts questions to SQL
2. **Execution Agent**: Runs queries safely
3. **Visualization Agent**: Creates charts
4. **Insights Agent**: Explains findings

### Why Agentic AI vs Traditional BI?

| Capability | Traditional BI | Agentic AI |
|------------|---------------|------------|
| Natural language | Limited | ✅ |
| Complex queries | Manual SQL | Automated |
| Explanations | ❌ | ✅ |
| Follow-up questions | ❌ | ✅ |
| Learning curve | High | Low |

## Implementation

### Phase 1: SQL Generation (Weeks 1-3)
- Deploy SQL generation agent
- Connect to data warehouse
- Handle top 50 query patterns

**Metrics:**
- Query success rate: 85%
- Generation time: <5 seconds
- Accuracy: 90%

### Phase 2: Visualization (Weeks 4-5)
- Auto-generate charts
- Recommend visualization types
- Export to dashboards

**Metrics:**
- Chart relevance: 88%
- User satisfaction: 4.3/5

### Phase 3: Insights & Recommendations (Weeks 6-8)
- Explain findings in plain English
- Suggest follow-up analyses
- Detect anomalies

**Metrics:**
- Insight quality: 4.5/5
- Follow-up rate: 65%

## Business Impact

### Quantitative Results

**Efficiency Gains:**
- Query turnaround: 3 days → 30 seconds (99.8% faster)
- Analyst time saved: 60% (focus on complex analysis)
- Self-service adoption: 15% → 75%

**Cost Savings:**
- Analyst time saved: $180K/year
- Faster decisions: $500K/year value
- Reduced BI tool complexity: $50K/year
**Total Annual Value: $730K**

**Usage Metrics:**
- Queries/day: 50 → 500 (10x increase)
- Unique users: 20 → 200 (10x increase)
- Data-driven decisions: +300%

### Qualitative Benefits

- **Democratized Data**: Everyone can ask questions
- **Faster Decisions**: Real-time insights
- **Analyst Satisfaction**: Focus on strategic work
- **Data Culture**: More people using data

## Example Interactions

### Simple Query

**User:** "What were our sales last month?"

**Agent:**
```sql
SELECT SUM(amount) as total_sales
FROM sales
WHERE date >= '2026-01-01' AND date < '2026-02-01'
```

**Result:** $1.2M in sales

**Visualization:** Bar chart showing daily sales

**Insight:** "Sales increased 15% compared to December, driven by new product launch."

### Complex Query

**User:** "Which products have declining sales but increasing returns?"

**Agent:**
```sql
WITH sales_trend AS (
  SELECT product_id,
         SUM(CASE WHEN month = '2026-01' THEN amount END) as current_sales,
         SUM(CASE WHEN month = '2025-12' THEN amount END) as prev_sales
  FROM sales
  GROUP BY product_id
),
returns_trend AS (
  SELECT product_id,
         COUNT(CASE WHEN month = '2026-01' THEN 1 END) as current_returns,
         COUNT(CASE WHEN month = '2025-12' THEN 1 END) as prev_returns
  FROM returns
  GROUP BY product_id
)
SELECT p.name, s.current_sales, s.prev_sales, r.current_returns, r.prev_returns
FROM sales_trend s
JOIN returns_trend r ON s.product_id = r.product_id
JOIN products p ON s.product_id = p.id
WHERE s.current_sales < s.prev_sales
  AND r.current_returns > r.prev_returns
```

**Visualization:** Scatter plot (sales change vs return change)

**Insight:** "3 products show this pattern. Product X has 25% sales decline and 40% return increase - potential quality issue."

**Recommendation:** "Investigate Product X quality. Check recent reviews and manufacturing changes."

### Follow-up Questions

**User:** "Show me reviews for Product X"

**Agent:** (Automatically understands context, queries reviews table)

**User:** "When did returns start increasing?"

**Agent:** (Generates time-series query, shows trend)

## Technical Stack

**Platform:** AWS Bedrock AgentCore
- Runtime for agent hosting
- Code Interpreter for SQL execution
- Memory for conversation context

**Model:** Claude 3 Sonnet
- Strong SQL generation
- Data interpretation
- Natural language explanations

**Data Infrastructure:**
- Snowflake (data warehouse)
- dbt (data modeling)
- Tableau (visualization export)

**Integrations:**
- Slack (query from Slack)
- Email (scheduled reports)
- API (embed in apps)

## Agent Implementation

### SQL Generation Agent

```python
def generate_sql(question, schema, conversation_history):
    # Understand schema
    relevant_tables = identify_tables(question, schema)
    
    # Generate SQL
    sql = llm.generate(
        prompt=f"""Given this question: {question}
        And these tables: {relevant_tables}
        Generate SQL query."""
    )
    
    # Validate SQL
    if not validate_sql(sql, schema):
        sql = refine_sql(sql, schema)
    
    return sql
```

### Visualization Agent

```python
def create_visualization(data, query_intent):
    # Determine chart type
    chart_type = recommend_chart(data, query_intent)
    
    # Generate chart
    if chart_type == "bar":
        chart = create_bar_chart(data)
    elif chart_type == "line":
        chart = create_line_chart(data)
    elif chart_type == "scatter":
        chart = create_scatter_plot(data)
    
    return chart
```

### Insights Agent

```python
def generate_insights(data, query, chart):
    # Analyze data
    patterns = detect_patterns(data)
    anomalies = detect_anomalies(data)
    trends = detect_trends(data)
    
    # Generate explanation
    insight = llm.generate(
        prompt=f"""Explain these findings:
        Data: {data}
        Patterns: {patterns}
        Anomalies: {anomalies}
        In simple business terms."""
    )
    
    # Suggest follow-ups
    recommendations = suggest_next_questions(data, query)
    
    return {
        "insight": insight,
        "recommendations": recommendations
    }
```

## Safety & Governance

### Query Safety

**Read-Only Access:**
- Agent can only SELECT
- No INSERT, UPDATE, DELETE
- No schema modifications

**Query Limits:**
- Max execution time: 30 seconds
- Max rows returned: 10,000
- Cost limits per query

**Data Access Control:**
- Row-level security enforced
- User sees only their data
- Audit log of all queries

### Data Privacy

- PII detection and masking
- Sensitive data flagging
- Compliance with regulations

## Lessons Learned

### What Worked

✅ **Start with common queries**
- Build confidence quickly
- Clear value demonstration

✅ **Show SQL to users**
- Transparency builds trust
- Users learn SQL patterns

✅ **Conversational follow-ups**
- Natural exploration
- Higher engagement

### Challenges

⚠️ **Schema complexity**
- 500+ tables overwhelming
- Required schema documentation
- Created "business glossary"

⚠️ **Ambiguous questions**
- "Sales" could mean revenue or units
- Added clarification prompts

⚠️ **Performance**
- Some queries too slow
- Added query optimization agent

## Decision Framework

### When to Use This Pattern

✅ High volume of ad-hoc queries
✅ Non-technical users need data access
✅ Data team bottlenecked
✅ Well-structured data warehouse

### When NOT to Use

❌ Highly complex, custom analyses
❌ Real-time operational queries
❌ Poorly documented schema
❌ Strict regulatory requirements

## Getting Started

### Week 1: Schema Preparation
- Document tables and columns
- Create business glossary
- Define access controls

### Week 2: Pilot
- Deploy for data team
- Test on historical questions
- Measure accuracy

### Week 3-4: Limited Rollout
- 20 business users
- Gather feedback
- Iterate on prompts

### Month 2+: Scale
- Expand to all users
- Add more data sources
- Advanced features

## Cost Analysis

### Initial Investment
- Platform setup: $10K
- Schema documentation: $15K
- Integration: $25K
- Training: $5K
**Total: $55K**

### Ongoing Costs
- AgentCore Runtime: $500/month
- Model invocations: $1,000/month (500 queries/day)
- Maintenance: $2,000/month
**Total: $3,500/month = $42K/year**

### ROI
- Annual value: $730K
- Annual costs: $42K
- **Net benefit: $688K/year**
- **ROI: 1,638%**
- **Payback: <1 month**

## Metrics to Track

**Usage:**
- Queries per day
- Unique users
- Query success rate
- Follow-up rate

**Quality:**
- SQL accuracy
- User satisfaction
- Insight relevance

**Business:**
- Time to insight
- Decisions influenced
- Analyst time saved

## Advanced Features

### Anomaly Detection
```
Agent: "I noticed sales dropped 30% yesterday. 
        This is unusual. Investigating..."
```

### Predictive Analytics
```
User: "Will we hit Q1 target?"
Agent: "Based on current trend, projected at 95% of target. 
        Need $50K more in next 2 weeks."
```

### Automated Reports
```
Schedule: "Every Monday, send sales summary to leadership"
Agent: Generates report, sends via email
```

## Next Steps

1. **Add more data sources**: CRM, marketing, product
2. **Predictive models**: Forecasting, recommendations
3. **Automated insights**: Proactive anomaly alerts
4. **Natural language dashboards**: "Create dashboard for sales"

---

**Status:** Production at financial services company  
**Timeline:** 8 weeks from concept to production  
**Team Size:** 1 PM, 2 engineers, 1 data analyst  
**Last Updated:** January 2026

## Real-World Examples

### Example 1: Sales Performance Analysis

**Business User Question:** "Which regions had the highest revenue growth last quarter?"

**Agent Process:**
1. **SQL Agent** generates:
```sql
SELECT region, 
       SUM(revenue) as current_quarter,
       LAG(SUM(revenue)) OVER (PARTITION BY region ORDER BY quarter) as previous_quarter,
       (SUM(revenue) - LAG(SUM(revenue)) OVER (PARTITION BY region ORDER BY quarter)) / 
       LAG(SUM(revenue)) OVER (PARTITION BY region ORDER BY quarter) * 100 as growth_pct
FROM sales
WHERE quarter IN ('Q3-2025', 'Q4-2025')
GROUP BY region, quarter
ORDER BY growth_pct DESC;
```

2. **Visualization Agent** creates bar chart showing growth by region

3. **Insights Agent** explains:
   - "West region grew 45% ($2.1M → $3.0M)"
   - "Driven by 3 new enterprise customers"
   - "East region declined 12% - investigate"

**Follow-up:** "Why did East region decline?"
- Agent analyzes customer churn data
- Identifies pricing competition
- Suggests retention strategies

**Time:** 30 seconds (vs 2 days waiting for analyst)

### Example 2: Customer Churn Prediction

**Question:** "Show me customers at risk of churning"

**Agent Actions:**
1. Analyzes usage patterns (declining activity)
2. Checks support ticket sentiment (negative)
3. Reviews payment history (late payments)
4. Generates risk score for each customer

**Output:**
- List of 47 at-risk customers
- Risk factors for each
- Recommended interventions
- Expected revenue impact: $280K

**Business Action:** Sales team reaches out proactively, saves 60% of at-risk accounts

### Example 3: Inventory Optimization

**Question:** "Which products are overstocked?"

**Agent Analysis:**
1. Calculates inventory turnover ratio
2. Compares to historical averages
3. Factors in seasonal trends
4. Identifies slow-moving items

**Insights:**
- 23 products with >90 days inventory
- $450K tied up in excess stock
- Recommends clearance sale
- Suggests adjusted reorder points

**Result:** Freed up $300K in working capital

### Example 4: Marketing Campaign ROI

**Question:** "What's the ROI of our email campaigns?"

**Agent Calculation:**
```
Revenue from email clicks: $125K
Campaign cost: $15K
ROI: 733%

Breakdown by segment:
- Enterprise: 1,200% ROI
- SMB: 400% ROI
- Free users: -20% ROI (cost > revenue)
```

**Recommendation:** "Focus budget on Enterprise segment, reduce free user campaigns"

**Business Impact:** Reallocated $50K budget, increased overall ROI by 40%

---

**Last Updated:** January 2026
