# Prompt Engineering for Product Managers

How to write effective prompts when building prototypes with AI coding assistants like Kiro, Cursor, and Claude.

## The New PM Skill

As a PM, you're used to writing:
- User stories
- Requirements docs
- Product specs

Now add to that list:
- **Prompts for AI agents**

The quality of your prompts directly determines the quality of your prototypes.

## Two Approaches

### Vibe Coding
Conversational, iterative, exploratory. Good for:
- Early exploration
- Learning what's possible
- Quick experiments
- Unclear requirements

**Example:**
```
"Build me a dashboard"
→ See what it creates
→ "Make it show sales data"
→ Iterate until it feels right
```

### Spec-Driven Development
Structured, detailed, intentional. Good for:
- Clear requirements
- Complex features
- Handoff to engineering
- Production-quality code

**Example:**
```
"Create a sales dashboard with:
- Revenue by region (bar chart)
- Top 10 products (table)
- Month-over-month growth (line chart)
- Filters: date range, region, product category
- Export to CSV functionality"
```

**Both are valid. Use the right approach for your goal.**

## The Anatomy of a Good Prompt

### Bad Prompt
```
"Build a customer portal"
```

**Problems:**
- Too vague
- No context
- Unclear scope
- Missing requirements

### Good Prompt
```
"Create a customer portal where users can:
1. View their order history (last 12 months)
2. Track current orders with shipping status
3. Download invoices as PDF
4. Update their email and shipping address
5. Submit support tickets

Technical requirements:
- Authentication required
- Mobile responsive
- Load orders from PostgreSQL database
- Send email notifications for ticket submissions"
```

**Why it's better:**
- Specific features listed
- Clear scope
- Technical context
- Success criteria

## The 5 Elements of Effective Prompts

### 1. Context
Tell the AI what you're building and why.

```
❌ "Create a form"

✅ "I'm building a B2B SaaS product. Create a lead capture form for our marketing site that sales will use to qualify prospects."
```

**Why it matters:** Context helps the AI make better decisions about validation, fields, and UX.

### 2. User Story
Describe who will use it and what they need.

```
❌ "Build a search feature"

✅ "As a customer support agent, I need to search for customers by email, phone, or order number so I can quickly pull up their account during calls."
```

**Why it matters:** User stories clarify the actual need, not just the feature.

### 3. Specific Requirements
List exactly what you need.

```
❌ "Make it work with our database"

✅ "Connect to PostgreSQL database at $DATABASE_URL. Query the 'customers' table which has columns: id, email, phone, created_at, status."
```

**Why it matters:** Specificity reduces back-and-forth iterations.

### 4. Constraints
State what NOT to do or what to avoid.

```
✅ "Don't use external libraries beyond the standard ones. Keep it simple - no complex frameworks."

✅ "Must work on mobile devices. No features that require desktop-only interactions."

✅ "Don't store sensitive data in logs. Mask credit card numbers in all outputs."
```

**Why it matters:** Constraints prevent the AI from making wrong assumptions.

### 5. Success Criteria
Define what "done" looks like.

```
✅ "Success means:
- Form submits without errors
- Data appears in database within 1 second
- User sees confirmation message
- Email notification sent to sales team"
```

**Why it matters:** Clear success criteria help you validate the output.

## Prompt Patterns for Common PM Tasks

### Pattern 1: CRUD Application

```
"Create a [RESOURCE] management system where users can:
- Create new [RESOURCE] with fields: [LIST FIELDS]
- View all [RESOURCES] in a table with columns: [LIST COLUMNS]
- Edit existing [RESOURCES]
- Delete [RESOURCES] with confirmation
- Search/filter by [CRITERIA]

Data should be stored in [DATABASE TYPE].
Include basic validation: [LIST VALIDATIONS]."
```

**Example:**
```
"Create a project management system where users can:
- Create new projects with fields: name, description, start_date, end_date, status, owner
- View all projects in a table with columns: name, owner, status, start_date
- Edit existing projects
- Delete projects with confirmation
- Search/filter by status and owner

Data should be stored in PostgreSQL.
Include basic validation: name required, dates must be valid, end_date after start_date."
```

### Pattern 2: API Integration

```
"Create an integration with [SERVICE] that:
- Authenticates using [AUTH METHOD]
- Fetches [DATA] from [ENDPOINT]
- Transforms the data to [FORMAT]
- Stores in [DESTINATION]
- Handles errors by [ERROR HANDLING]
- Runs [FREQUENCY]"
```

**Example:**
```
"Create an integration with Stripe that:
- Authenticates using API key from environment variable
- Fetches all successful payments from the last 24 hours
- Transforms the data to include: customer_email, amount, currency, created_at
- Stores in PostgreSQL 'payments' table
- Handles errors by logging to console and sending Slack notification
- Runs every hour via cron job"
```

### Pattern 3: Dashboard/Analytics

```
"Create a dashboard showing:
1. [METRIC 1]: [VISUALIZATION TYPE] showing [DATA]
2. [METRIC 2]: [VISUALIZATION TYPE] showing [DATA]
3. [METRIC 3]: [VISUALIZATION TYPE] showing [DATA]

Filters: [LIST FILTERS]
Data source: [SOURCE]
Update frequency: [FREQUENCY]
Export options: [FORMATS]"
```

**Example:**
```
"Create a sales dashboard showing:
1. Total Revenue: Large number card showing sum of all orders this month
2. Revenue by Region: Bar chart showing revenue grouped by region
3. Top Products: Table showing top 10 products by revenue with columns: name, units_sold, revenue

Filters: Date range (default: last 30 days), Region (dropdown)
Data source: PostgreSQL 'orders' table
Update frequency: Real-time (query on page load)
Export options: CSV, PDF"
```

### Pattern 4: Workflow Automation

```
"Create a workflow that:
1. Triggers when [EVENT]
2. Checks if [CONDITION]
3. If true: [ACTION 1]
4. If false: [ACTION 2]
5. Logs [WHAT] to [WHERE]
6. Notifies [WHO] via [METHOD]"
```

**Example:**
```
"Create a workflow that:
1. Triggers when a new order is created
2. Checks if order amount > $1000
3. If true: Send approval request email to manager
4. If false: Auto-approve and send confirmation to customer
5. Logs all decisions to 'approval_log' table with timestamp, order_id, decision, approver
6. Notifies sales team via Slack #orders channel"
```

## Iterative Prompting

Start broad, then refine.

### Iteration 1: Get Something Working
```
"Create a customer feedback form with name, email, rating, and comments"
```

### Iteration 2: Add Validation
```
"Add validation:
- Name required, min 2 characters
- Email must be valid format
- Rating required, 1-5 stars
- Comments optional, max 500 characters"
```

### Iteration 3: Improve UX
```
"Make the rating field use star icons instead of a dropdown. Show character count for comments field."
```

### Iteration 4: Add Functionality
```
"After submission, send email notification to support@company.com with the feedback details"
```

**Key insight:** Each iteration builds on the previous. Don't try to get everything perfect in one prompt.

## Common Mistakes

### Mistake 1: Too Vague
```
❌ "Make it better"
❌ "Fix the bugs"
❌ "Improve the design"
```

**Fix:** Be specific about what "better" means.
```
✅ "Increase the font size of headings to 24px"
✅ "Fix the bug where form submits even when email is invalid"
✅ "Change the button color to match our brand blue (#0066CC)"
```

### Mistake 2: Too Much at Once
```
❌ "Create a complete e-commerce platform with products, cart, checkout, payments, shipping, inventory management, customer accounts, admin dashboard, analytics, and email notifications"
```

**Fix:** Break into smaller pieces.
```
✅ "First, create a product catalog page showing products from the database"
→ Then: "Add a shopping cart"
→ Then: "Add checkout flow"
→ etc.
```

### Mistake 3: Assuming Context
```
❌ "Connect to the database"
```

**Fix:** Provide all necessary context.
```
✅ "Connect to PostgreSQL database at $DATABASE_URL. The 'products' table has columns: id, name, price, description, image_url, stock_quantity"
```

### Mistake 4: No Success Criteria
```
❌ "Build a search feature"
```

**Fix:** Define what success looks like.
```
✅ "Build a search feature that returns results in under 1 second, highlights matching text, and shows 'No results found' message when appropriate"
```

### Mistake 5: Ignoring Errors
```
❌ [AI generates code with errors]
"Okay, now add another feature..."
```

**Fix:** Address errors before moving forward.
```
✅ "The code has an error on line 23 where it tries to access 'user.email' but user might be null. Fix this by checking if user exists first."
```

## Advanced Techniques

### Technique 1: Provide Examples

```
"Create a function that formats currency. Examples:
- formatCurrency(1234.56) → '$1,234.56'
- formatCurrency(0.5) → '$0.50'
- formatCurrency(1000000) → '$1,000,000.00'"
```

**Why it works:** Examples clarify edge cases and expected behavior.

### Technique 2: Reference Existing Code

```
"Create a new endpoint similar to the existing /api/users endpoint, but for products. Follow the same pattern: GET for list, POST for create, PUT for update, DELETE for remove."
```

**Why it works:** Maintains consistency with existing codebase.

### Technique 3: Specify Technology

```
"Use React with TypeScript. For state management, use React hooks (useState, useEffect). For styling, use Tailwind CSS. For API calls, use fetch with async/await."
```

**Why it works:** Ensures the AI uses your preferred stack.

### Technique 4: Request Explanations

```
"Create a function to calculate shipping cost. Include comments explaining the logic for each step."
```

**Why it works:** Helps you understand the code for future modifications.

### Technique 5: Ask for Alternatives

```
"Show me two different approaches for implementing user authentication: one using JWT tokens and one using session cookies. Explain the tradeoffs of each."
```

**Why it works:** Helps you make informed decisions.

## Debugging with Prompts

### When Something Doesn't Work

**Bad approach:**
```
❌ "It's broken, fix it"
```

**Good approach:**
```
✅ "When I click the submit button, I get this error: 'Cannot read property email of undefined'. This happens on line 45 where we access user.email. The user object is coming from the API call on line 30. Can you add error handling to check if user exists before accessing user.email?"
```

**Template:**
```
"When I [ACTION], I expect [EXPECTED RESULT] but instead [ACTUAL RESULT]. 

Error message: [ERROR]
Location: [FILE/LINE]
Context: [WHAT YOU WERE TRYING TO DO]

Please fix by [SUGGESTED APPROACH]."
```

## Prompts for Different Stages

### Exploration Stage
```
"Show me 3 different ways to implement a recommendation engine. For each approach, explain:
- How it works
- Pros and cons
- Complexity level
- Data requirements"
```

### Prototyping Stage
```
"Build a basic recommendation engine using collaborative filtering. Use sample data for now. Focus on getting it working, not on optimization."
```

### Refinement Stage
```
"The recommendation engine works but is slow with large datasets. Optimize it by:
1. Adding caching for frequently accessed data
2. Limiting results to top 10 instead of all matches
3. Using database indexes on the user_id and product_id columns"
```

### Handoff Stage
```
"Document this recommendation engine code with:
- Overview of how it works
- API endpoints and parameters
- Database schema requirements
- Performance characteristics
- Known limitations
- Suggestions for production deployment"
```

## Measuring Prompt Quality

### Good prompts result in:
- ✅ Code that works on first try (or second)
- ✅ Minimal back-and-forth iterations
- ✅ Code that matches your mental model
- ✅ Clear understanding of what was built

### Poor prompts result in:
- ❌ Many iterations to get it right
- ❌ Code that doesn't match expectations
- ❌ Confusion about what was built
- ❌ Need to start over

**Track your prompt effectiveness:**
- How many iterations to working code?
- How often do you need to clarify?
- How often does it match your vision?

## Prompt Templates Library

### Template: New Feature
```
"Create a [FEATURE] that allows users to [ACTION].

User story: As a [ROLE], I want to [GOAL] so that [BENEFIT].

Requirements:
- [REQUIREMENT 1]
- [REQUIREMENT 2]
- [REQUIREMENT 3]

Technical details:
- [TECH DETAIL 1]
- [TECH DETAIL 2]

Success criteria:
- [CRITERIA 1]
- [CRITERIA 2]"
```

### Template: Bug Fix
```
"There's a bug where [DESCRIPTION].

Steps to reproduce:
1. [STEP 1]
2. [STEP 2]
3. [STEP 3]

Expected: [EXPECTED BEHAVIOR]
Actual: [ACTUAL BEHAVIOR]

Error message: [ERROR]

Please fix by [APPROACH]."
```

### Template: Refactor
```
"Refactor the [COMPONENT] to:
- [IMPROVEMENT 1]
- [IMPROVEMENT 2]
- [IMPROVEMENT 3]

Keep the same functionality but improve:
- [ASPECT 1]
- [ASPECT 2]

Don't change:
- [CONSTRAINT 1]
- [CONSTRAINT 2]"
```

### Template: Integration
```
"Integrate with [SERVICE] to [PURPOSE].

Authentication: [METHOD]
Endpoints needed: [LIST]
Data to send: [FIELDS]
Data to receive: [FIELDS]
Error handling: [APPROACH]
Rate limits: [LIMITS]"
```

## Best Practices Summary

1. **Start with context** - What are you building and why?
2. **Be specific** - List exact requirements
3. **Provide examples** - Show expected inputs/outputs
4. **State constraints** - What NOT to do
5. **Define success** - What does "done" look like?
6. **Iterate incrementally** - Build in small steps
7. **Address errors immediately** - Don't pile on features
8. **Request explanations** - Understand what was built
9. **Test thoroughly** - Validate each iteration
10. **Document learnings** - What prompts worked well?

## Practice Exercise

Try improving this prompt:

**Before:**
```
"Make a login page"
```

**After (your turn):**
```
[Write a better version using the techniques from this guide]
```

**Suggested improvement:**
```
"Create a login page with:
- Email field (required, must be valid email format)
- Password field (required, min 8 characters, masked input)
- 'Remember me' checkbox
- 'Forgot password?' link
- Submit button

On submit:
- Validate fields client-side
- POST to /api/auth/login with email and password
- On success: redirect to /dashboard
- On error: show error message below form
- Store auth token in localStorage if 'remember me' checked

Design: Clean, centered on page, mobile responsive"
```

## Conclusion

Prompt engineering is a skill. Like any skill, you get better with practice.

**Start simple:**
- Write clear, specific prompts
- Iterate based on results
- Learn what works for you

**Get advanced:**
- Develop your own templates
- Build a prompt library
- Share learnings with your team

**The PM who writes great prompts ships better prototypes faster.**

---

**Tools to practice with:**
- Kiro IDE (kiro.dev)
- Cursor (cursor.sh)
- Claude (claude.ai)
- GitHub Copilot

**Last Updated:** January 2026
