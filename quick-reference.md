# Agentic AI Quick Reference

## When to Use Agentic AI

✅ **Good Fit:**
- Repetitive, multi-step tasks
- Need for reasoning and decision-making
- Integration with multiple tools/APIs
- 24/7 availability required
- High volume of similar requests

❌ **Not a Good Fit:**
- Simple classification (<5 categories)
- Real-time requirements (<100ms)
- Deterministic, rule-based logic
- Cost-sensitive, high-volume (>1M/day)
- Highly regulated without human oversight

## Framework Quick Comparison

| Need | Use This |
|------|----------|
| Enterprise, managed | AWS Bedrock AgentCore |
| Custom workflows | LangGraph |
| Quick prototype | CrewAI |
| Code generation | AutoGen |
| Full control | LangChain |

## Cost Estimates

| Scale | Monthly Cost |
|-------|-------------|
| Pilot (100 queries/day) | $500-$1K |
| Small (1K queries/day) | $2K-$5K |
| Medium (10K queries/day) | $10K-$20K |
| Large (100K queries/day) | $50K-$100K |

## Common Patterns

### Single Agent
```python
agent = Agent()
agent.add_tool(search_tool)
agent.add_tool(action_tool)
result = agent("Do something")
```

### Multi-Agent
```python
researcher = Agent()
writer = Agent()
reviewer = Agent()

research = researcher("Gather info")
draft = writer(research)
final = reviewer(draft)
```

### Agent with Memory
```python
agent = Agent()
agent.memory.add("user_preference", "concise")
result = agent("Answer this", context=agent.memory)
```

## Typical Metrics

| Metric | Target |
|--------|--------|
| Accuracy | >85% |
| Response Time | <2s |
| Uptime | >99.5% |
| User Satisfaction | >4/5 |
| Cost per Query | <$0.10 |

## Common Tools

```python
@tool
def search_database(query: str) -> list:
    """Search for records"""
    return db.query(query)

@tool
def send_notification(user: str, message: str) -> bool:
    """Send notification"""
    return notify(user, message)

@tool
def get_user_info(user_id: str) -> dict:
    """Get user details"""
    return users.get(user_id)
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Low accuracy | Improve prompts, add examples |
| Slow responses | Cache results, optimize tools |
| High costs | Use cheaper models, batch requests |
| Hallucinations | Ground in retrieved data, add citations |
| Tool failures | Add retries, fallbacks, error handling |

## Security Checklist

- [ ] Input validation on all tools
- [ ] Rate limiting per user
- [ ] Audit logging enabled
- [ ] Secrets in environment variables
- [ ] API keys rotated regularly
- [ ] Human review for sensitive actions

## Deployment Checklist

- [ ] Monitoring and alerting
- [ ] Error tracking
- [ ] Cost tracking
- [ ] Performance benchmarks
- [ ] Rollback plan
- [ ] Documentation
- [ ] Team training

## Quick Wins

1. **FAQ Automation** - 2 weeks, 60% deflection
2. **Alert Triage** - 3 weeks, 80% noise reduction
3. **Code Review** - 4 weeks, 85% time savings
4. **Data Queries** - 2 weeks, 10x faster insights

## Resources

- [Getting Started Guide](./getting-started.md)
- [Use Cases](./use-cases/)
- [Framework Comparison](./comparisons/framework-comparison.md)
- [Cost Analysis](./comparisons/cost-analysis.md)
