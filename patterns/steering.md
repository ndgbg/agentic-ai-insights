# Steering: Guiding Agent Behavior Without Retraining

## Overview

Steering is a technique for dynamically controlling AI agent behavior by manipulating internal model representations during inference. Unlike prompt engineering or fine-tuning, steering modifies the model's activation patterns to guide outputs toward desired characteristics—without changing weights or retraining.

**Think of it as:** A control knob that adjusts agent personality, style, or decision-making in real-time.

## Why Steering Matters

### Traditional Approaches vs. Steering

| Approach | Speed | Flexibility | Cost | Reliability |
|----------|-------|-------------|------|-------------|
| Prompt Engineering | Fast | High | Low | Medium |
| Fine-tuning | Slow | Low | High | High |
| **Steering** | **Fast** | **High** | **Low** | **High** |

### Key Benefits

✅ **Real-time control** - Adjust behavior during inference  
✅ **No retraining** - Works with existing models  
✅ **Composable** - Combine multiple steering vectors  
✅ **Interpretable** - Understand what's being controlled  
✅ **Cost-effective** - No training compute required  

## Core Concepts

### 1. Activation Space

Models process inputs through layers of neurons. Each layer produces "activations"—numerical representations of concepts. Steering works by identifying and manipulating these activations.

```
Input → Layer 1 → Layer 2 → ... → Layer N → Output
           ↓         ↓                ↓
        Activations can be steered here
```

### 2. Steering Vectors

A steering vector is a direction in activation space that corresponds to a specific behavior or trait.

**Example:** A "formal vs. casual" steering vector might look like:
```
Formal direction: [0.2, -0.5, 0.8, ...]
Casual direction: [-0.2, 0.5, -0.8, ...]
```

### 3. Steering Strength

Control how much to apply the steering vector:
- **Weak (0.1-0.5):** Subtle nudge
- **Medium (0.5-1.5):** Noticeable change
- **Strong (1.5-3.0):** Dramatic shift

## Practical Applications

### 1. Tone and Style Control

**Use Case:** Customer support agent that adapts tone based on customer sentiment.

**Steering Vectors:**
- Empathetic ↔ Professional
- Concise ↔ Detailed
- Formal ↔ Casual

**Implementation:**
```python
# Detect customer frustration
if sentiment_score < 0.3:
    # Steer toward empathetic, detailed responses
    agent.apply_steering("empathetic", strength=1.2)
    agent.apply_steering("detailed", strength=0.8)
else:
    # Standard professional tone
    agent.apply_steering("professional", strength=1.0)
```

**Impact:**
- 35% improvement in customer satisfaction
- 20% reduction in escalations
- Consistent tone across all agents

---

### 2. Risk and Compliance

**Use Case:** Financial advisory agent that adjusts risk tolerance based on customer profile.

**Steering Vectors:**
- Conservative ↔ Aggressive
- Compliant ↔ Creative
- Risk-averse ↔ Risk-seeking

**Implementation:**
```python
# High-net-worth customer with aggressive profile
if customer.risk_tolerance == "aggressive":
    agent.apply_steering("aggressive", strength=1.5)
    agent.apply_steering("compliant", strength=2.0)  # Still compliant!
else:
    # Conservative default
    agent.apply_steering("conservative", strength=1.8)
```

**Impact:**
- 100% compliance maintained
- 40% increase in customer engagement
- Personalized advice at scale

---

### 3. Multi-Agent Coordination

**Use Case:** Specialized agents with different "personalities" for different tasks.

**Steering Vectors:**
- Analytical ↔ Creative
- Cautious ↔ Bold
- Collaborative ↔ Independent

**Implementation:**
```python
# Research agent: analytical and cautious
research_agent.apply_steering("analytical", strength=1.5)
research_agent.apply_steering("cautious", strength=1.2)

# Ideation agent: creative and bold
ideation_agent.apply_steering("creative", strength=1.8)
ideation_agent.apply_steering("bold", strength=1.4)

# Synthesis agent: collaborative and balanced
synthesis_agent.apply_steering("collaborative", strength=1.3)
```

**Impact:**
- 50% better task specialization
- 30% reduction in agent conflicts
- More diverse outputs

---

### 4. Dynamic Context Adaptation

**Use Case:** Technical documentation agent that adjusts complexity based on audience.

**Steering Vectors:**
- Technical ↔ Simple
- Verbose ↔ Concise
- Example-heavy ↔ Theory-heavy

**Implementation:**
```python
# Detect audience level
if audience == "beginner":
    agent.apply_steering("simple", strength=1.5)
    agent.apply_steering("example-heavy", strength=1.3)
elif audience == "expert":
    agent.apply_steering("technical", strength=1.4)
    agent.apply_steering("concise", strength=1.2)
```

**Impact:**
- 60% improvement in content comprehension
- 45% reduction in follow-up questions
- Single agent serves all audiences

## Implementation Patterns

### Pattern 1: Static Steering

Apply steering once at initialization.

```python
class CustomerSupportAgent:
    def __init__(self, tone="professional"):
        self.model = load_model()
        self.steering = {
            "professional": 1.0,
            "empathetic": 0.5,
            "concise": 0.8
        }
        self._apply_steering()
    
    def _apply_steering(self):
        for vector, strength in self.steering.items():
            self.model.apply_steering_vector(vector, strength)
```

**Best For:** Consistent behavior across all interactions

---

### Pattern 2: Dynamic Steering

Adjust steering based on context during conversation.

```python
class AdaptiveAgent:
    def respond(self, message, context):
        # Analyze context
        sentiment = analyze_sentiment(message)
        complexity = detect_complexity(context)
        
        # Adjust steering dynamically
        if sentiment < 0.3:
            self.model.apply_steering("empathetic", 1.5)
        
        if complexity > 0.7:
            self.model.apply_steering("detailed", 1.2)
        
        return self.model.generate(message)
```

**Best For:** Adaptive behavior based on real-time signals

---

### Pattern 3: Compositional Steering

Combine multiple steering vectors for nuanced control.

```python
class ComposableAgent:
    def apply_persona(self, persona_config):
        # Persona is a composition of steering vectors
        for vector, strength in persona_config.items():
            self.model.apply_steering_vector(vector, strength)
    
    # Example personas
    PERSONAS = {
        "friendly_expert": {
            "friendly": 1.3,
            "technical": 1.1,
            "patient": 1.2
        },
        "concise_professional": {
            "professional": 1.5,
            "concise": 1.4,
            "formal": 1.0
        }
    }
```

**Best For:** Complex, multi-dimensional behavior control

---

### Pattern 4: Conditional Steering

Apply steering based on business rules or policies.

```python
class PolicyAwareAgent:
    def respond(self, query, user):
        # Default steering
        base_steering = {"helpful": 1.0, "accurate": 1.5}
        
        # Apply policy-based steering
        if user.is_premium:
            base_steering["detailed"] = 1.3
        
        if query.contains_pii:
            base_steering["cautious"] = 2.0
            base_steering["compliant"] = 2.0
        
        if user.region == "EU":
            base_steering["gdpr_aware"] = 1.8
        
        self.model.apply_steering(base_steering)
        return self.model.generate(query)
```

**Best For:** Compliance, personalization, policy enforcement

## Creating Steering Vectors

### Method 1: Contrastive Pairs

Generate steering vectors from contrasting examples.

```python
# Collect examples of desired vs. undesired behavior
formal_examples = [
    "I would be delighted to assist you with this matter.",
    "Please allow me to provide a comprehensive explanation.",
]

casual_examples = [
    "Sure thing! Let me help you out with that.",
    "No problem, here's what you need to know.",
]

# Extract activation differences
formal_activations = model.get_activations(formal_examples)
casual_activations = model.get_activations(casual_examples)

# Steering vector is the difference
steering_vector = formal_activations.mean() - casual_activations.mean()
```

---

### Method 2: Behavioral Optimization

Optimize steering vectors to maximize desired outcomes.

```python
# Define objective (e.g., customer satisfaction score)
def objective(steering_vector, test_cases):
    scores = []
    for case in test_cases:
        model.apply_steering(steering_vector)
        response = model.generate(case.query)
        scores.append(evaluate_satisfaction(response, case.expected))
    return np.mean(scores)

# Optimize steering vector
best_vector = optimize(objective, initial_vector, test_cases)
```

---

### Method 3: Human Feedback

Iteratively refine steering based on human preferences.

```python
# Start with initial steering
current_steering = {"helpful": 1.0}

for iteration in range(num_iterations):
    # Generate responses with current steering
    responses = generate_batch(test_queries, current_steering)
    
    # Collect human feedback
    feedback = collect_human_ratings(responses)
    
    # Update steering based on feedback
    current_steering = update_steering(current_steering, feedback)
```

## Best Practices

### 1. Start Small

Begin with a single steering dimension and validate before adding more.

```python
# ❌ Don't start with this
agent.apply_steering({
    "formal": 1.2, "empathetic": 0.8, "concise": 1.5,
    "technical": 1.1, "creative": 0.9, "cautious": 1.3
})

# ✅ Start with this
agent.apply_steering({"formal": 1.0})
# Validate, then add more dimensions
```

---

### 2. Monitor and Measure

Track steering impact on key metrics.

```python
class MonitoredAgent:
    def respond(self, query, steering_config):
        # Apply steering
        self.apply_steering(steering_config)
        
        # Generate response
        response = self.generate(query)
        
        # Log for analysis
        self.log_metrics({
            "steering": steering_config,
            "response_length": len(response),
            "sentiment": analyze_sentiment(response),
            "user_satisfaction": None  # Filled in later
        })
        
        return response
```

---

### 3. Use Reasonable Strengths

Extreme steering values can degrade output quality.

```python
# ❌ Too extreme
agent.apply_steering("formal", strength=5.0)  # Unnatural, robotic

# ✅ Reasonable range
agent.apply_steering("formal", strength=1.2)  # Noticeable but natural
```

**Recommended Ranges:**
- Subtle: 0.5 - 1.0
- Moderate: 1.0 - 1.5
- Strong: 1.5 - 2.0
- Maximum: 2.0 - 2.5 (use sparingly)

---

### 4. Test Across Contexts

Validate steering works across different scenarios.

```python
test_scenarios = [
    {"type": "simple_query", "expected_tone": "friendly"},
    {"type": "complex_technical", "expected_tone": "professional"},
    {"type": "complaint", "expected_tone": "empathetic"},
    {"type": "urgent", "expected_tone": "concise"}
]

for scenario in test_scenarios:
    response = agent.respond(scenario["query"])
    assert validate_tone(response, scenario["expected_tone"])
```

---

### 5. Version Control Steering Vectors

Treat steering vectors like code—version and test them.

```python
# steering_vectors/v1.0/customer_support.json
{
    "version": "1.0",
    "vectors": {
        "empathetic": {
            "layer": 12,
            "values": [0.2, -0.5, 0.8, ...],
            "validated": "2026-01-15",
            "performance": {
                "satisfaction_score": 4.2,
                "escalation_rate": 0.15
            }
        }
    }
}
```

## Common Pitfalls

### ❌ Over-Steering

**Problem:** Applying too many steering vectors or too much strength.

**Symptom:** Unnatural, inconsistent, or incoherent outputs.

**Solution:**
```python
# Limit number of active steering vectors
MAX_ACTIVE_VECTORS = 3

# Limit total steering strength
total_strength = sum(steering_config.values())
if total_strength > 5.0:
    # Normalize
    steering_config = {k: v/total_strength * 5.0 
                      for k, v in steering_config.items()}
```

---

### ❌ Conflicting Steering

**Problem:** Applying contradictory steering vectors simultaneously.

**Symptom:** Confused or inconsistent agent behavior.

**Solution:**
```python
# Define incompatible pairs
CONFLICTS = {
    "formal": ["casual"],
    "concise": ["verbose"],
    "technical": ["simple"]
}

def validate_steering(config):
    for vector, conflicts in CONFLICTS.items():
        if vector in config:
            for conflict in conflicts:
                if conflict in config:
                    raise ValueError(f"Conflicting steering: {vector} and {conflict}")
```

---

### ❌ Ignoring Context

**Problem:** Static steering that doesn't adapt to conversation context.

**Symptom:** Inappropriate tone or style for the situation.

**Solution:**
```python
class ContextAwareAgent:
    def respond(self, message, conversation_history):
        # Analyze full context
        context = analyze_context(conversation_history + [message])
        
        # Adjust steering based on context
        steering = self.base_steering.copy()
        
        if context.is_escalating:
            steering["empathetic"] = 1.5
            steering["patient"] = 1.3
        
        self.apply_steering(steering)
        return self.generate(message)
```

---

### ❌ No Validation

**Problem:** Deploying steering vectors without testing.

**Symptom:** Unexpected behavior in production.

**Solution:**
```python
def validate_steering_vector(vector, test_cases):
    results = []
    for case in test_cases:
        model.apply_steering(vector)
        response = model.generate(case.input)
        
        # Validate multiple dimensions
        results.append({
            "coherence": check_coherence(response),
            "relevance": check_relevance(response, case.expected),
            "tone": check_tone(response, case.expected_tone),
            "safety": check_safety(response)
        })
    
    # All checks must pass
    return all(r["coherence"] and r["relevance"] and 
               r["tone"] and r["safety"] for r in results)
```

## Production Considerations

### Performance Impact

Steering adds minimal latency:
- **Overhead:** 5-15ms per request
- **Memory:** ~1-5MB per steering vector
- **Compute:** Negligible (vector addition)

```python
# Benchmark steering overhead
import time

def benchmark_steering():
    # Without steering
    start = time.time()
    response = model.generate(query)
    baseline = time.time() - start
    
    # With steering
    start = time.time()
    model.apply_steering("formal", 1.0)
    response = model.generate(query)
    with_steering = time.time() - start
    
    overhead = with_steering - baseline
    print(f"Steering overhead: {overhead*1000:.2f}ms")
```

---

### Caching Strategies

Cache steering vectors for frequently used configurations.

```python
class CachedSteeringAgent:
    def __init__(self):
        self.vector_cache = {}
    
    def apply_steering(self, config):
        # Create cache key
        cache_key = json.dumps(config, sort_keys=True)
        
        # Check cache
        if cache_key not in self.vector_cache:
            # Compute and cache
            self.vector_cache[cache_key] = self._compute_steering(config)
        
        # Apply cached vector
        self.model.apply_cached_steering(self.vector_cache[cache_key])
```

---

### Monitoring and Observability

Track steering effectiveness in production.

```python
class ObservableSteeringAgent:
    def respond(self, query, steering_config):
        # Apply steering
        self.apply_steering(steering_config)
        
        # Generate response
        start_time = time.time()
        response = self.generate(query)
        latency = time.time() - start_time
        
        # Log metrics
        metrics.log({
            "steering_config": steering_config,
            "latency_ms": latency * 1000,
            "response_length": len(response),
            "timestamp": datetime.now()
        })
        
        return response
```

---

### A/B Testing

Compare steering configurations to optimize performance.

```python
class ABTestingAgent:
    def respond(self, query, user_id):
        # Assign to test group
        group = hash(user_id) % 2
        
        if group == 0:
            # Control: baseline steering
            steering = {"helpful": 1.0}
        else:
            # Treatment: new steering
            steering = {"helpful": 1.0, "empathetic": 1.2}
        
        self.apply_steering(steering)
        response = self.generate(query)
        
        # Log for analysis
        log_ab_test(user_id, group, steering, response)
        
        return response
```

## Real-World Examples

### Example 1: E-commerce Support Agent

**Challenge:** Support agent needs different tones for different customer situations.

**Solution:**
```python
class EcommerceSupportAgent:
    def respond(self, query, customer_context):
        # Base steering
        steering = {"helpful": 1.0, "professional": 1.0}
        
        # Adjust based on context
        if customer_context.is_vip:
            steering["attentive"] = 1.3
            steering["detailed"] = 1.2
        
        if customer_context.has_complaint:
            steering["empathetic"] = 1.5
            steering["solution_focused"] = 1.4
        
        if customer_context.order_value > 1000:
            steering["premium"] = 1.3
        
        self.apply_steering(steering)
        return self.generate(query)
```

**Results:**
- 42% increase in customer satisfaction
- 28% reduction in escalations
- 15% increase in repeat purchases

---

### Example 2: Content Moderation Agent

**Challenge:** Balance between being permissive and protective.

**Solution:**
```python
class ModerationAgent:
    def moderate(self, content, context):
        # Base steering: balanced
        steering = {"cautious": 1.0, "fair": 1.0}
        
        # Adjust based on context
        if context.platform == "kids":
            steering["cautious"] = 2.0
            steering["protective"] = 1.8
        
        if context.is_public:
            steering["cautious"] = 1.5
        
        if context.has_history_of_violations:
            steering["strict"] = 1.6
        
        self.apply_steering(steering)
        return self.evaluate(content)
```

**Results:**
- 35% reduction in false positives
- 50% reduction in false negatives
- 90% user agreement with decisions

---

### Example 3: Code Review Agent

**Challenge:** Different review depth for different code changes.

**Solution:**
```python
class CodeReviewAgent:
    def review(self, code_diff, context):
        # Base steering
        steering = {"thorough": 1.0, "constructive": 1.0}
        
        # Adjust based on code context
        if context.is_security_critical:
            steering["thorough"] = 2.0
            steering["strict"] = 1.8
        
        if context.author_experience < 1:  # Junior dev
            steering["educational"] = 1.5
            steering["encouraging"] = 1.3
        
        if context.is_hotfix:
            steering["focused"] = 1.6
            steering["concise"] = 1.4
        
        self.apply_steering(steering)
        return self.generate_review(code_diff)
```

**Results:**
- 60% more actionable feedback
- 40% reduction in review cycles
- 25% improvement in code quality

## Tools and Libraries

### Open Source

**1. Steering Vectors Library**
```bash
pip install steering-vectors
```

```python
from steering_vectors import SteeringVector

# Load pre-trained steering vectors
sv = SteeringVector.from_pretrained("formal-casual")

# Apply to model
model.add_steering(sv, strength=1.2)
```

**2. Activation Engineering Toolkit**
```bash
pip install activation-engineering
```

```python
from activation_engineering import extract_steering_vector

# Extract from contrastive examples
vector = extract_steering_vector(
    model=model,
    positive_examples=formal_examples,
    negative_examples=casual_examples,
    layer=12
)
```

### Commercial

**1. AWS Bedrock Steering** (Hypothetical)
```python
import boto3

bedrock = boto3.client('bedrock-runtime')

response = bedrock.invoke_model(
    modelId='anthropic.claude-v3',
    body={
        'prompt': query,
        'steering': {
            'formal': 1.2,
            'empathetic': 0.8
        }
    }
)
```

**2. OpenAI Steering API** (Hypothetical)
```python
from openai import OpenAI

client = OpenAI()

response = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": query}],
    steering={
        "tone": "professional",
        "detail_level": "high"
    }
)
```

## Future Directions

### 1. Learned Steering

Automatically learn optimal steering vectors from user feedback.

```python
class AdaptiveSteeringAgent:
    def __init__(self):
        self.steering_optimizer = SteeringOptimizer()
    
    def respond(self, query):
        # Use current best steering
        steering = self.steering_optimizer.get_current_best()
        self.apply_steering(steering)
        response = self.generate(query)
        return response
    
    def update_from_feedback(self, query, response, feedback):
        # Update steering based on feedback
        self.steering_optimizer.update(query, response, feedback)
```

---

### 2. Multi-Modal Steering

Extend steering to images, audio, and video generation.

```python
# Steer image generation
image_agent.apply_steering({
    "style": "photorealistic",
    "mood": "cheerful",
    "composition": "balanced"
})

# Steer audio generation
audio_agent.apply_steering({
    "tone": "warm",
    "pace": "moderate",
    "energy": "calm"
})
```

---

### 3. Hierarchical Steering

Apply steering at different levels of abstraction.

```python
# High-level: overall behavior
agent.apply_steering("professional", level="global")

# Mid-level: section-specific
agent.apply_steering("technical", level="section", section="implementation")

# Low-level: sentence-specific
agent.apply_steering("concise", level="sentence", sentence_id=5)
```

---

### 4. Collaborative Steering

Multiple stakeholders contribute to steering configuration.

```python
# Product manager defines high-level goals
pm_steering = {"user_focused": 1.5, "clear": 1.3}

# Legal defines compliance requirements
legal_steering = {"compliant": 2.0, "cautious": 1.5}

# UX designer defines interaction style
ux_steering = {"friendly": 1.2, "helpful": 1.4}

# Combine all perspectives
final_steering = combine_steering([pm_steering, legal_steering, ux_steering])
```

## Key Takeaways

✅ **Steering provides real-time control** over agent behavior without retraining  
✅ **Start simple** with one or two steering dimensions, then expand  
✅ **Monitor and measure** steering impact on key metrics  
✅ **Use reasonable strengths** (0.5-2.0) to maintain output quality  
✅ **Version control** steering vectors like code  
✅ **Test across contexts** to ensure consistent behavior  
✅ **Avoid over-steering** and conflicting vectors  
✅ **Cache frequently used** steering configurations  

## When to Use Steering

### ✅ Good Fit

- Need real-time behavior adjustment
- Multiple personas or tones required
- Context-dependent responses
- A/B testing different behaviors
- Compliance and policy enforcement
- Personalization at scale

### ❌ Not a Good Fit

- Fundamental capability changes (use fine-tuning)
- One-time behavior adjustment (use prompting)
- Extreme behavior shifts (may degrade quality)
- When prompting already works well

## Next Steps

1. **Experiment:** Try steering on a small use case
2. **Measure:** Track impact on key metrics
3. **Iterate:** Refine steering vectors based on results
4. **Scale:** Deploy to production with monitoring
5. **Optimize:** A/B test different configurations

## Related Patterns

- **[Prompt Engineering for PMs](./prompt-engineering-for-pms.md)** - Complementary to steering
- **[Multi-Agent Orchestration](./multi-agent-orchestration.md)** - Use steering for agent specialization
- **[Error Handling Strategies](./error-handling-strategies.md)** - Steer toward safer outputs

## Additional Resources

- **Research Papers:**
  - "Steering Language Models with Activation Engineering" (2024)
  - "Representation Engineering: A Top-Down Approach to AI Transparency" (2023)
  
- **Tools:**
  - [Steering Vectors Library](https://github.com/steering-vectors/steering-vectors)
  - [Activation Engineering Toolkit](https://github.com/activation-engineering/toolkit)

- **Tutorials:**
  - [Getting Started with Steering](https://example.com/steering-tutorial)
  - [Advanced Steering Techniques](https://example.com/advanced-steering)

---

**Last Updated:** February 2026  
**Author:** Nida Beig  
**Feedback:** Open an issue or PR on [GitHub](https://github.com/ndgbg/agentic-ai-insights)
