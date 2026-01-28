---
name: ai-prompt-engineer
description: Master prompt engineering for Claude - create effective prompts, skills, agents, and system instructions following Anthropic's official best practices.
---

# AI Prompt Engineering Skill

Apply these patterns when creating prompts, skills, agents, and system instructions for Claude. Based on Anthropic's official documentation.

## Core Principles

### The Golden Rule
> Show your prompt to a colleague with minimal context. If they're confused, Claude will be too.

### When to Use Prompt Engineering
- Resource efficient (no GPU training)
- Cost effective (uses base model)
- Survives model updates
- Instant iteration
- Minimal data needs
- Preserves general knowledge

## Technique Priority Order

Apply these techniques from most to least broadly effective:

| Priority | Technique | When to Use |
|----------|-----------|-------------|
| 1 | Be clear and direct | Always |
| 2 | Use examples (multishot) | When format/style matters |
| 3 | Let Claude think (CoT) | Complex reasoning tasks |
| 4 | Use XML tags | Multi-part prompts |
| 5 | Give Claude a role | Specialized expertise needed |
| 6 | Prefill response | Control output format |
| 7 | Chain prompts | Multi-step workflows |
| 8 | Long context tips | Large documents (20K+ tokens) |

---

## 1. Be Clear and Direct

### Provide Context
Tell Claude:
- What the task results will be used for
- Who the audience is
- Where this fits in a workflow
- What success looks like

### Be Specific
```
# Bad
"Write a report"

# Good
"Write a 500-word technical report for senior engineers.
Focus on: performance metrics, bottlenecks, recommendations.
Format: Executive summary, then numbered sections.
Tone: Professional, data-driven, no marketing language."
```

### Use Sequential Steps
```
Instructions:
1. Read the customer feedback in <feedback> tags
2. Identify the primary issue category
3. Rate sentiment (Positive/Neutral/Negative)
4. Assign priority (High/Medium/Low)
5. Output as JSON with keys: category, sentiment, priority, summary
```

---

## 2. Use Examples (Multishot Prompting)

### Why Examples Work
- Reduce misinterpretation
- Enforce consistent structure
- Boost performance on complex tasks

### Example Structure
```xml
<examples>
  <example>
    <input>The dashboard loads slowly and the export button is missing</input>
    <output>
      Category: UI/UX, Performance
      Sentiment: Negative
      Priority: High
    </output>
  </example>
  <example>
    <input>Love the Salesforce integration! Would be great to add Hubspot</input>
    <output>
      Category: Integration, Feature Request
      Sentiment: Positive
      Priority: Medium
    </output>
  </example>
</examples>

Now analyze: {{FEEDBACK}}
```

### Best Practices
- Include 3-5 diverse examples
- Cover edge cases
- Vary examples to avoid unintended patterns
- Wrap in `<example>` tags for clarity

---

## 3. Chain of Thought (CoT)

### When to Use CoT
- Complex math or logic
- Multi-step analysis
- Decisions with many factors
- Tasks humans would think through

### CoT Levels

**Basic**: Add "Think step-by-step"
```
Analyze this contract for risks. Think step-by-step.
```

**Guided**: Specify thinking steps
```
Think before answering:
1. First, identify the key stakeholders
2. Then, list their competing interests
3. Finally, propose a balanced solution
```

**Structured**: Use XML tags to separate thinking from answer
```
Think through this problem in <thinking> tags.
Then provide your final answer in <answer> tags.
```

### CoT Trade-offs
| Pros | Cons |
|------|------|
| Higher accuracy | Increased latency |
| Better coherence | More output tokens |
| Easier debugging | Not always needed |

---

## 4. XML Tags

### Why Use XML Tags
- Clearly separate prompt sections
- Reduce misinterpretation
- Easy to parse outputs
- Flexible modification

### Common Tag Patterns
```xml
<context>Background information here</context>

<instructions>
1. Step one
2. Step two
</instructions>

<examples>
  <example>...</example>
</examples>

<data>{{INPUT_DATA}}</data>

<output_format>
Describe expected output structure
</output_format>
```

### Output Parsing Tags
```xml
Think in <thinking> tags, then answer in <answer> tags.
```

```xml
Put your analysis in <analysis> tags.
Put recommendations in <recommendations> tags.
```

### Best Practices
- Be consistent with tag names
- Nest tags for hierarchy: `<outer><inner></inner></outer>`
- Reference tags in instructions: "Using the data in <data> tags..."

---

## 5. System Prompts (Roles)

### When to Use Roles
- Specialized expertise needed
- Consistent persona across conversation
- Specific behavioral guidelines

### Role Structure
```
You are a senior solutions architect with 15 years of experience
in distributed systems. You specialize in:
- Microservices architecture
- Event-driven systems
- Cloud-native design (AWS, GCP)

When reviewing architectures:
1. First identify single points of failure
2. Then assess scalability bottlenecks
3. Finally, recommend specific improvements

Communication style:
- Be direct and technical
- Use diagrams when helpful
- Cite industry standards
```

### Role Components
| Component | Purpose |
|-----------|---------|
| Identity | Who Claude is |
| Expertise | What they know |
| Behavior | How they work |
| Constraints | What they don't do |
| Style | How they communicate |

---

## 6. Prefill Claude's Response

### When to Prefill
- Force specific output format
- Skip preamble text
- Maintain character in roleplay
- Start with specific structure

### JSON Output
```python
messages = [
    {"role": "user", "content": "Extract product info as JSON: {{DESCRIPTION}}"},
    {"role": "assistant", "content": "{"}  # Prefill forces JSON
]
```

### Character Maintenance
```python
messages = [
    {"role": "user", "content": "What do you think of this code?"},
    {"role": "assistant", "content": "[Senior Code Reviewer]"}  # Stay in character
]
```

### Rules
- No trailing whitespace in prefill
- Not available with extended thinking
- A little prefill goes a long way

---

## 7. Prompt Chaining

### When to Chain
- Multi-step analysis
- Content creation pipelines
- Data processing workflows
- Self-correction needed

### Chain Patterns

**Content Pipeline**
```
Research → Outline → Draft → Edit → Format
```

**Analysis Pipeline**
```
Extract → Transform → Analyze → Visualize
```

**Decision Pipeline**
```
Gather info → List options → Analyze each → Recommend
```

**Self-Correction Chain**
```
Generate → Review → Refine → Re-review
```

### Chaining Best Practices
1. Identify distinct subtasks
2. Use XML tags to pass outputs between prompts
3. Each subtask has single, clear objective
4. Run independent subtasks in parallel

---

## 8. Long Context Tips

### Document Placement
- Put long documents (20K+ tokens) at the TOP
- Place query/instructions at the END
- Can improve quality by up to 30%

### Multi-Document Structure
```xml
<documents>
  <document index="1">
    <source>annual_report_2023.pdf</source>
    <document_content>{{REPORT_CONTENT}}</document_content>
  </document>
  <document index="2">
    <source>competitor_analysis.xlsx</source>
    <document_content>{{ANALYSIS_CONTENT}}</document_content>
  </document>
</documents>

Based on the documents above, analyze...
```

### Grounding with Quotes
```
First, find and quote relevant passages from the documents
in <quotes> tags. Then provide your analysis in <analysis> tags.
```

---

## Creating Effective Skills

### Skill Structure
```markdown
---
name: skill-name
description: One-line description of what this skill does
---

# Skill Title

Brief overview of when to use this skill.

## Core Concepts
Key principles and patterns

## Patterns & Examples
Code examples, templates

## Best Practices
Do's and don'ts

## Checklists
Verification items
```

### Skill Design Principles
1. **Focused**: One domain/technology per skill
2. **Actionable**: Provide concrete patterns, not theory
3. **Scannable**: Use tables, bullet points, code blocks
4. **Complete**: Cover common scenarios
5. **Opinionated**: Give clear recommendations

### What Makes a Good Skill
| Element | Purpose |
|---------|---------|
| Code examples | Show exact patterns to follow |
| Tables | Quick reference for decisions |
| Checklists | Verification before completion |
| Anti-patterns | What to avoid |
| Templates | Starting points for common tasks |

---

## Creating Effective Agents

### Agent Structure
```markdown
---
name: agent-name
description: One-line description
model: sonnet  # or opus, haiku
skills:
  - skill-1
  - skill-2
---

# Agent Title

Identity and specialization.

## Your Responsibilities
What tasks this agent handles.

## Before You Start
Context loading instructions.

## Key Patterns
Domain-specific patterns to follow.

## Do Not
Explicit constraints.
```

### Agent Design Principles
1. **Clear identity**: Who is this agent?
2. **Defined scope**: What does it handle (and not handle)?
3. **Loaded context**: Which skills provide expertise?
4. **Explicit constraints**: What should it never do?

### Agent Types by Task
| Task Type | Agent Focus |
|-----------|-------------|
| Implementation | Language skills, architecture patterns |
| Specification | Domain knowledge, templates |
| Review | Checklists, best practices |
| Planning | Trade-off analysis, options |

---

## Prompt Templates

### Task Execution Template
```
You are a {{ROLE}} helping with {{TASK_TYPE}}.

<context>
{{RELEVANT_BACKGROUND}}
</context>

<instructions>
1. {{STEP_1}}
2. {{STEP_2}}
3. {{STEP_3}}
</instructions>

<output_format>
{{EXPECTED_FORMAT}}
</output_format>

<input>
{{USER_INPUT}}
</input>
```

### Analysis Template
```
Analyze the following {{ITEM_TYPE}} for {{ANALYSIS_FOCUS}}.

<item>
{{CONTENT}}
</item>

Think through your analysis in <thinking> tags:
1. First, identify {{ASPECT_1}}
2. Then, evaluate {{ASPECT_2}}
3. Finally, assess {{ASPECT_3}}

Provide your findings in <findings> tags.
Provide recommendations in <recommendations> tags.
```

### Code Review Template
```
Review this {{LANGUAGE}} code for:
- Correctness
- Performance
- Security
- Maintainability

<code>
{{CODE}}
</code>

<context>
Purpose: {{PURPOSE}}
Requirements: {{REQUIREMENTS}}
</context>

Output your review as:
1. Issues found (with severity: Critical/High/Medium/Low)
2. Specific line references
3. Suggested fixes
```

---

## Common Anti-Patterns

### Prompt Anti-Patterns
| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| Vague instructions | Unpredictable output | Be specific about format, length, style |
| No examples | Inconsistent results | Add 3-5 diverse examples |
| Everything in one prompt | Dropped steps | Chain into subtasks |
| No structure | Mixed content | Use XML tags |
| Implicit expectations | Misinterpretation | State assumptions explicitly |

### Skill Anti-Patterns
| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| Too broad | Diluted expertise | Focus on one domain |
| Theory-heavy | Not actionable | Add code examples |
| No constraints | Over-engineering | Include "Do Not" section |
| Wall of text | Hard to scan | Use tables, bullets, code |

### Agent Anti-Patterns
| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| No identity | Inconsistent behavior | Define clear role |
| Overlapping scope | Confusion | Clarify responsibilities |
| Missing skills | Incomplete knowledge | Load relevant skills |
| No constraints | Unwanted actions | Add "Do Not" section |

---

## Quality Checklist

### For Prompts
- [ ] Context is clear (purpose, audience, workflow)
- [ ] Instructions are sequential and numbered
- [ ] Output format is specified
- [ ] Examples included if format matters
- [ ] XML tags separate sections
- [ ] Edge cases addressed

### For Skills
- [ ] Focused on one domain
- [ ] Has code examples
- [ ] Includes patterns and anti-patterns
- [ ] Scannable with tables and bullets
- [ ] Has verification checklist

### For Agents
- [ ] Clear identity and expertise
- [ ] Defined responsibilities
- [ ] Required skills loaded
- [ ] Explicit constraints
- [ ] Context loading instructions
