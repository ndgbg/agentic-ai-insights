# Skills, Scripts, and Knowledge: The Three Pillars of Agent Capability

Understanding the difference between skills, scripts, and knowledge is fundamental to designing effective agent systems.

## The Three Pillars

### Skills (What Agents Can Do)
Tools and actions agents can execute.

**Examples:**
- Search a database
- Send an email
- Calculate statistics
- Call an API
- Generate a report

**Implementation:**
```python
@tool
def search_database(query: str) -> list:
    """Skill: Search customer database"""
    return db.query(query)

@tool
def send_email(to: str, subject: str, body: str) -> bool:
    """Skill: Send email notification"""
    return email_service.send(to, subject, body)
```

### Scripts (How Agents Do It)
Predefined workflows and procedures.

**Examples:**
- Customer onboarding process
- Incident response playbook
- Data analysis pipeline
- Code review checklist

**Implementation:**
```python
def onboarding_script(customer_email: str) -> dict:
    """Script: Complete customer onboarding"""
    
    # Step 1: Create account
    account = create_account(customer_email)
    
    # Step 2: Send welcome email
    send_email(customer_email, "Welcome!", welcome_template)
    
    # Step 3: Schedule follow-up
    schedule_followup(customer_email, days=3)
    
    return {"status": "complete", "account_id": account.id}
```

### Knowledge (What Agents Know)
Information agents can retrieve and reason about.

**Examples:**
- Product documentation
- Company policies
- Historical data
- Best practices
- Domain expertise

**Implementation:**
```python
# Knowledge stored in vector database
knowledge_base = VectorDB()

@tool
def search_knowledge(query: str) -> list:
    """Knowledge: Search documentation"""
    return knowledge_base.search(query, top_k=5)
```

## The Interplay

### Skills + Scripts = Automation

**Without Scripts (Ad-hoc):**
```python
# Agent decides what to do
agent = Agent()
agent.add_tool(search_database)
agent.add_tool(send_email)
agent.add_tool(create_ticket)

# Agent figures out the workflow
result = agent("Handle this customer complaint")
```

**With Scripts (Structured):**
```python
def complaint_handling_script(complaint: str) -> dict:
    """Defined workflow for complaints"""
    
    # Always follow this sequence
    customer = search_database(complaint.email)
    ticket = create_ticket(complaint, priority="high")
    send_email(complaint.email, "We're on it!", ticket.id)
    notify_manager(ticket.id)
    
    return {"ticket_id": ticket.id}
```

**When to use:**
- ✅ Scripts: Compliance-critical, must follow exact steps
- ✅ Skills: Flexibility needed, agent decides approach

### Skills + Knowledge = Intelligence

**Without Knowledge (Limited):**
```python
@tool
def answer_question(question: str) -> str:
    """Can only answer what's in training data"""
    return llm.generate(question)
```

**With Knowledge (Grounded):**
```python
@tool
def answer_question(question: str) -> str:
    """Retrieves relevant knowledge first"""
    
    # Get relevant context
    context = search_knowledge(question)
    
    # Answer based on retrieved knowledge
    prompt = f"Context: {context}\n\nQuestion: {question}\n\nAnswer:"
    return llm.generate(prompt)
```

**When to use:**
- ✅ Knowledge: Factual accuracy required, domain-specific info
- ✅ No Knowledge: General reasoning, creative tasks

### Scripts + Knowledge = Expertise

**Without Knowledge (Generic):**
```python
def diagnose_issue_script(symptoms: str) -> dict:
    """Generic troubleshooting"""
    
    # Fixed steps, no context
    check_logs()
    restart_service()
    notify_team()
```

**With Knowledge (Expert):**
```python
def diagnose_issue_script(symptoms: str) -> dict:
    """Expert troubleshooting with historical data"""
    
    # Search for similar past issues
    similar_issues = search_knowledge(f"symptoms: {symptoms}")
    
    # Apply solutions that worked before
    for issue in similar_issues:
        if issue.solution_success_rate > 0.8:
            apply_solution(issue.solution)
            if verify_fixed():
                return {"status": "resolved", "solution": issue.solution}
    
    # Escalate if no known solution
    escalate_to_expert(symptoms, similar_issues)
```

**When to use:**
- ✅ Scripts + Knowledge: Complex domains, learning from history
- ✅ Scripts alone: Simple, well-defined processes

### Skills + Scripts + Knowledge = Autonomous Agents

**Full Integration:**
```python
class CustomerSupportAgent:
    def __init__(self):
        # Skills
        self.add_tool(search_database)
        self.add_tool(search_knowledge)
        self.add_tool(send_email)
        self.add_tool(create_ticket)
        
        # Scripts
        self.scripts = {
            "complaint": complaint_handling_script,
            "refund": refund_processing_script,
            "technical": technical_support_script
        }
        
        # Knowledge
        self.knowledge = VectorDB("support_docs")
    
    def handle_request(self, request: str) -> dict:
        # Use knowledge to understand request
        context = self.knowledge.search(request)
        
        # Classify and route to appropriate script
        request_type = self.classify(request, context)
        
        if request_type in self.scripts:
            # Follow defined workflow
            return self.scripts[request_type](request)
        else:
            # Use skills flexibly
            return self.agent(request)
```

## Design Patterns

### Pattern 1: Skills-First (Maximum Flexibility)

**When:** Unpredictable tasks, creative work, exploration

```python
agent = Agent()
agent.add_tool(search_web)
agent.add_tool(analyze_data)
agent.add_tool(generate_report)

# Agent decides how to accomplish goal
result = agent("Research competitors and create analysis")
```

**Pros:**
- Flexible and adaptive
- Handles novel situations
- Creative problem-solving

**Cons:**
- Non-deterministic
- Harder to debug
- May be inefficient

### Pattern 2: Scripts-First (Maximum Control)

**When:** Compliance, safety-critical, well-defined processes

```python
def loan_approval_script(application: dict) -> dict:
    """Strict workflow for loan approval"""
    
    # Must follow exact sequence
    credit_score = check_credit(application.ssn)
    income_verified = verify_income(application.income_docs)
    debt_ratio = calculate_debt_ratio(application)
    
    # Deterministic decision
    if credit_score > 700 and income_verified and debt_ratio < 0.4:
        return {"approved": True}
    else:
        return {"approved": False, "reason": "Does not meet criteria"}
```

**Pros:**
- Predictable and consistent
- Easy to audit
- Meets compliance requirements

**Cons:**
- Inflexible
- Can't handle edge cases
- Requires updates for new scenarios

### Pattern 3: Knowledge-First (Maximum Accuracy)

**When:** Factual accuracy critical, domain expertise required

```python
@tool
def medical_diagnosis_assistant(symptoms: str) -> dict:
    """Knowledge-grounded medical assistance"""
    
    # Retrieve relevant medical knowledge
    relevant_conditions = medical_knowledge.search(symptoms)
    relevant_studies = research_papers.search(symptoms)
    
    # Ground response in retrieved knowledge
    prompt = f"""
    Based on these medical sources:
    {relevant_conditions}
    {relevant_studies}
    
    Symptoms: {symptoms}
    
    Provide differential diagnosis with source citations.
    """
    
    return llm.generate(prompt)
```

**Pros:**
- Factually accurate
- Citable sources
- Reduces hallucinations

**Cons:**
- Limited to available knowledge
- Requires knowledge maintenance
- Slower (retrieval overhead)

### Pattern 4: Hybrid (Balanced)

**When:** Most production use cases

```python
class HybridAgent:
    def handle_task(self, task: str) -> dict:
        # Step 1: Check if there's a script for this
        if task_type := self.classify_task(task):
            if task_type in self.scripts:
                return self.scripts[task_type](task)
        
        # Step 2: Retrieve relevant knowledge
        knowledge = self.knowledge_base.search(task)
        
        # Step 3: Use skills with knowledge context
        agent = Agent()
        agent.add_context(knowledge)
        return agent(task)
```

**Pros:**
- Flexible where needed
- Structured where required
- Grounded in knowledge

**Cons:**
- More complex to build
- Requires careful design
- Higher maintenance

## Implementation Guide

### Building Skills

**1. Identify Core Actions**
```python
# What can your agent DO?
skills = [
    "search_database",
    "send_notification",
    "create_record",
    "update_status",
    "generate_report"
]
```

**2. Implement as Tools**
```python
@tool
def search_database(query: str, filters: dict = None) -> list:
    """
    Search database with optional filters.
    
    Args:
        query: Search query string
        filters: Optional filters (e.g., {"status": "active"})
    
    Returns:
        List of matching records
    """
    return db.search(query, filters)
```

**3. Test Thoroughly**
```python
def test_search_database():
    results = search_database("customer", {"status": "active"})
    assert len(results) > 0
    assert all(r["status"] == "active" for r in results)
```

### Building Scripts

**1. Document Workflows**
```
Customer Onboarding:
1. Validate email
2. Create account
3. Send welcome email
4. Schedule follow-up
5. Notify sales team
```

**2. Implement as Functions**
```python
def customer_onboarding_script(email: str, name: str) -> dict:
    """Complete customer onboarding workflow"""
    
    # Step 1: Validate
    if not validate_email(email):
        return {"error": "Invalid email"}
    
    # Step 2: Create account
    account = create_account(email, name)
    
    # Step 3: Send welcome
    send_welcome_email(email, name)
    
    # Step 4: Schedule follow-up
    schedule_followup(account.id, days=3)
    
    # Step 5: Notify sales
    notify_sales_team(account.id)
    
    return {"status": "complete", "account_id": account.id}
```

**3. Add Error Handling**
```python
def customer_onboarding_script(email: str, name: str) -> dict:
    """Robust onboarding with error handling"""
    
    results = {"steps_completed": []}
    
    try:
        if not validate_email(email):
            return {"error": "Invalid email"}
        results["steps_completed"].append("validation")
        
        account = create_account(email, name)
        results["steps_completed"].append("account_creation")
        results["account_id"] = account.id
        
        send_welcome_email(email, name)
        results["steps_completed"].append("welcome_email")
        
        schedule_followup(account.id, days=3)
        results["steps_completed"].append("followup_scheduled")
        
        notify_sales_team(account.id)
        results["steps_completed"].append("sales_notified")
        
        results["status"] = "complete"
        return results
        
    except Exception as e:
        results["status"] = "partial_failure"
        results["error"] = str(e)
        return results
```

### Building Knowledge

**1. Identify Knowledge Sources**
```python
knowledge_sources = [
    "product_documentation",
    "support_articles",
    "past_tickets",
    "company_policies",
    "best_practices"
]
```

**2. Ingest and Index**
```python
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.embeddings import OpenAIEmbeddings

def ingest_knowledge(documents: list) -> VectorDB:
    """Ingest documents into knowledge base"""
    
    # Split into chunks
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=500,
        chunk_overlap=50
    )
    chunks = splitter.split_documents(documents)
    
    # Generate embeddings
    embeddings = OpenAIEmbeddings()
    
    # Store in vector database
    vector_db = VectorDB()
    vector_db.add_documents(chunks, embeddings)
    
    return vector_db
```

**3. Implement Retrieval**
```python
@tool
def search_knowledge(query: str, top_k: int = 5) -> list:
    """Search knowledge base"""
    
    # Semantic search
    results = knowledge_base.similarity_search(query, k=top_k)
    
    # Return with metadata
    return [
        {
            "content": doc.page_content,
            "source": doc.metadata["source"],
            "relevance": doc.metadata["score"]
        }
        for doc in results
    ]
```

## Decision Framework

### When to Add Skills

✅ **Add skills when:**
- Agents need to interact with external systems
- Actions have side effects (create, update, delete)
- Real-time data access required
- Integration with existing tools needed

❌ **Don't add skills when:**
- Pure reasoning task (no external action)
- Information already in context
- Action is unsafe without human approval

### When to Add Scripts

✅ **Add scripts when:**
- Workflow must be consistent
- Compliance requirements exist
- Process is well-defined
- Efficiency is critical

❌ **Don't add scripts when:**
- Task is highly variable
- Creativity is needed
- Process is still evolving
- Flexibility is more important than consistency

### When to Add Knowledge

✅ **Add knowledge when:**
- Factual accuracy is critical
- Domain expertise required
- Information changes frequently
- Need to cite sources

❌ **Don't add knowledge when:**
- General reasoning sufficient
- Information is in training data
- Real-time data needed (use skills instead)
- Knowledge is too dynamic to maintain

## Best Practices

### Skills
1. **Single responsibility** - One skill, one action
2. **Clear descriptions** - Agents rely on them
3. **Type safety** - Validate inputs
4. **Error handling** - Return structured errors
5. **Idempotency** - Safe to retry

### Scripts
1. **Document steps** - Clear workflow
2. **Error recovery** - Handle partial failures
3. **Logging** - Track execution
4. **Versioning** - Track changes
5. **Testing** - Validate each step

### Knowledge
1. **Keep updated** - Stale knowledge is worse than none
2. **Chunk appropriately** - Balance context and precision
3. **Add metadata** - Source, date, confidence
4. **Monitor quality** - Track retrieval accuracy
5. **Version control** - Track knowledge changes

## Conclusion

Effective agents balance skills (what they can do), scripts (how they do it), and knowledge (what they know). The right mix depends on your use case:

- **Exploratory tasks** → Skills-heavy
- **Compliance-critical** → Scripts-heavy
- **Domain expertise** → Knowledge-heavy
- **Production systems** → Balanced hybrid

Start simple, measure results, and evolve your architecture based on real-world performance.
