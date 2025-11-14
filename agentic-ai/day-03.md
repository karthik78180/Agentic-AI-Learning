# Day 3: Prompt Engineering for Agents

## Overview
Prompt engineering is the foundation of effective AI agents. Today you'll learn advanced techniques to make your agents more reliable, accurate, and capable through better prompting strategies.

## Learning Objectives
- Master core prompt engineering techniques
- Implement chain-of-thought reasoning
- Create structured outputs with JSON mode
- Use role-based prompting effectively
- Handle edge cases and errors through prompting
- Build reusable prompt templates

## Theoretical Concepts

### Why Prompt Engineering Matters for Agents
Unlike simple chatbots, agents must:
- Break down complex tasks into steps
- Make decisions about tool usage
- Maintain consistency across multiple iterations
- Handle errors and unexpected situations
- Produce structured, parseable outputs

Good prompting makes agents **reliable** and **predictable**.

### Core Prompt Engineering Principles

#### 1. Clarity and Specificity
```
❌ Bad: "Help me with data"
✅ Good: "Analyze the CSV file sales_2024.csv and create a summary with total revenue, top 3 products, and monthly trends"
```

#### 2. Context and Constraints
```
❌ Bad: "Write code"
✅ Good: "Write a Python function that validates email addresses. Use regex, include error handling, add type hints, and write docstrings"
```

#### 3. Format Specification
```
❌ Bad: "Give me the info"
✅ Good: "Return the information in JSON format with keys: title, description, priority (1-5), and tags (array)"
```

### Advanced Techniques

#### Chain-of-Thought (CoT) Prompting
Encourage step-by-step reasoning:

```python
prompt = """
Solve this problem step by step:

Problem: A company has 150 employees. 60% work remotely, and 30% of remote workers are in different time zones. How many employees are remote AND in different time zones?

Let's approach this systematically:
1. First, calculate the number of remote workers
2. Then, calculate how many of those are in different time zones
3. Show your work for each step
"""
```

#### Few-Shot Learning
Provide examples to guide behavior:

```python
system_prompt = """
You are a data extraction agent. Extract structured information from text.

Examples:

Input: "John Smith ordered 3 laptops on March 15th"
Output: {"customer": "John Smith", "item": "laptops", "quantity": 3, "date": "2024-03-15"}

Input: "Sarah bought 2 phones yesterday"
Output: {"customer": "Sarah", "item": "phones", "quantity": 2, "date": "2024-03-14"}

Now process the following:
"""
```

#### ReAct Pattern (Reason + Act)
Structure agent thinking:

```python
react_prompt = """
You are an agent that solves problems using available tools.

For each step, use this format:
Thought: [Your reasoning about what to do next]
Action: [The action to take - tool name and parameters]
Observation: [Result of the action]
... (repeat Thought/Action/Observation as needed)
Thought: I now know the final answer
Final Answer: [Your final response]

Available tools:
- search(query): Search the web
- calculator(expression): Calculate math expressions
- get_weather(location): Get weather information

Question: What's the temperature difference between New York and Tokyo today?
"""
```

## Hands-On Exercises

### Exercise 1: Chain-of-Thought Implementation

```python
# cot_agent.py
from openai import OpenAI

client = OpenAI()

def cot_solver(problem):
    """Solve problems using chain-of-thought prompting"""
    prompt = f"""
    Solve the following problem using step-by-step reasoning.
    Think through each step carefully before moving to the next.

    Problem: {problem}

    Solution:
    Step 1: [Identify what we know and what we need to find]
    Step 2: [Break down the problem into smaller parts]
    Step 3: [Solve each part]
    Step 4: [Combine results for final answer]

    Let's begin:
    """

    response = client.chat.completions.create(
        model="gpt-4-turbo-preview",
        messages=[{"role": "user", "content": prompt}],
        temperature=0.3  # Lower for more consistent reasoning
    )

    return response.choices[0].message.content

# Test
problem = "A train leaves Station A at 9 AM traveling at 60 mph. Another train leaves Station B (180 miles away) at 10 AM traveling at 90 mph toward Station A. When do they meet?"
print(cot_solver(problem))
```

### Exercise 2: Structured Output with JSON Mode

```python
# structured_output.py
import json
from openai import OpenAI

client = OpenAI()

def extract_structured_data(text):
    """Extract structured data using JSON mode"""
    prompt = f"""
    Extract the following information from the text and return as JSON:
    - name: person's full name
    - email: email address
    - phone: phone number
    - company: company name
    - role: job title
    - interests: array of interests mentioned

    Text: {text}

    Return only valid JSON, no additional text.
    """

    response = client.chat.completions.create(
        model="gpt-4-turbo-preview",
        messages=[{"role": "user", "content": prompt}],
        response_format={"type": "json_object"},
        temperature=0
    )

    return json.loads(response.choices[0].message.content)

# Test
sample_text = """
Hi, I'm Dr. Jane Smith, Chief Data Scientist at TechCorp Inc.
You can reach me at jane.smith@techcorp.com or call 555-0123.
I'm passionate about machine learning, hiking, and classical music.
"""

result = extract_structured_data(sample_text)
print(json.dumps(result, indent=2))
```

### Exercise 3: Role-Based Agent Prompting

```python
# role_based_agent.py
class RoleBasedAgent:
    """Agent with specialized roles"""

    def __init__(self, client):
        self.client = client
        self.roles = {
            "analyst": """You are a data analyst. Approach problems methodically:
                1. Understand the data and question
                2. Identify relevant metrics
                3. Perform analysis step-by-step
                4. Present findings with key insights
                Be precise and data-driven.""",

            "developer": """You are an expert software engineer. When solving problems:
                1. Understand requirements clearly
                2. Design before coding
                3. Write clean, documented code
                4. Consider edge cases and errors
                5. Suggest tests
                Follow best practices and design patterns.""",

            "critic": """You are a critical reviewer. Your job is to:
                1. Identify flaws and weaknesses
                2. Suggest improvements
                3. Check for edge cases
                4. Verify logic and assumptions
                Be constructive but thorough."""
        }

    def execute(self, role, task):
        """Execute task with specific role"""
        if role not in self.roles:
            raise ValueError(f"Unknown role: {role}")

        messages = [
            {"role": "system", "content": self.roles[role]},
            {"role": "user", "content": task}
        ]

        response = self.client.chat.completions.create(
            model="gpt-4-turbo-preview",
            messages=messages,
            temperature=0.4
        )

        return response.choices[0].message.content

# Usage
client = OpenAI()
agent = RoleBasedAgent(client)

task = "Design a function to find the most frequent word in a text file"

print("=== Developer Response ===")
print(agent.execute("developer", task))

print("\n=== Critic Response ===")
dev_output = agent.execute("developer", task)
print(agent.execute("critic", f"Review this solution:\n{dev_output}"))
```

### Exercise 4: Prompt Templates with Variables

```python
# prompt_templates.py
from string import Template

class PromptTemplate:
    """Reusable prompt templates"""

    RESEARCH_TEMPLATE = Template("""
    You are a research agent. Your task is to research: $topic

    Requirements:
    - Depth: $depth (brief/moderate/comprehensive)
    - Focus areas: $focus_areas
    - Output format: $output_format

    Please provide well-structured, accurate information with sources where applicable.
    """)

    CODE_REVIEW_TEMPLATE = Template("""
    You are a code review agent. Review the following $language code:

    ```$language
    $code
    ```

    Review checklist:
    1. Code quality and readability
    2. Potential bugs or errors
    3. Performance considerations
    4. Security issues
    5. Best practices adherence

    Provide specific, actionable feedback.
    """)

    DATA_ANALYSIS_TEMPLATE = Template("""
    You are a data analysis agent. Analyze the following dataset:

    Data: $data_description
    Question: $question
    Context: $context

    Provide:
    1. Summary statistics
    2. Key insights
    3. Visualizations to create (describe)
    4. Recommendations

    Be thorough and data-driven.
    """)

    @classmethod
    def research(cls, topic, depth="moderate", focus_areas="general overview", output_format="markdown"):
        return cls.RESEARCH_TEMPLATE.substitute(
            topic=topic,
            depth=depth,
            focus_areas=focus_areas,
            output_format=output_format
        )

    @classmethod
    def code_review(cls, code, language="python"):
        return cls.CODE_REVIEW_TEMPLATE.substitute(
            code=code,
            language=language
        )

    @classmethod
    def data_analysis(cls, data_description, question, context=""):
        return cls.DATA_ANALYSIS_TEMPLATE.substitute(
            data_description=data_description,
            question=question,
            context=context
        )

# Usage
prompt = PromptTemplate.research(
    topic="Transformer architecture in deep learning",
    depth="comprehensive",
    focus_areas="attention mechanisms, applications, recent advances"
)
print(prompt)
```

### Exercise 5: Error Handling Through Prompting

```python
# robust_prompting.py
def create_robust_prompt(user_input, max_retries=3):
    """Create a prompt with built-in error handling"""
    prompt = f"""
    Process the following user request. Follow these guidelines:

    1. If the request is unclear, ask for clarification
    2. If required information is missing, list what's needed
    3. If the request is impossible, explain why and suggest alternatives
    4. If you're uncertain, state your assumptions

    User request: {user_input}

    Before responding:
    - Verify you have all necessary information
    - Check if the request is feasible
    - Consider edge cases

    Response format:
    Status: [CLEAR/NEEDS_CLARIFICATION/IMPOSSIBLE/UNCERTAIN]
    Reasoning: [Your analysis]
    Response: [Your answer or questions]
    """
    return prompt

# Test with various inputs
test_cases = [
    "Analyze the data",  # Unclear
    "Sort this list: [3, 1, 4, 1, 5]",  # Clear
    "Make me a sandwich",  # Impossible for an AI
]

client = OpenAI()
for test in test_cases:
    prompt = create_robust_prompt(test)
    response = client.chat.completions.create(
        model="gpt-4-turbo-preview",
        messages=[{"role": "user", "content": prompt}]
    )
    print(f"Input: {test}")
    print(f"Output:\n{response.choices[0].message.content}\n")
    print("-" * 80)
```

## Key Takeaways
1. Clear, specific prompts produce better agent outputs
2. Chain-of-thought improves reasoning for complex tasks
3. Few-shot examples guide agent behavior effectively
4. Structured outputs (JSON) enable reliable parsing
5. Role-based prompting creates specialized agent behavior
6. Prompt templates ensure consistency and reusability

## Prompt Engineering Best Practices

### DO:
- Be specific about format and structure
- Use examples for complex tasks
- Include error handling instructions
- Test prompts with edge cases
- Version control your prompts
- Use lower temperature for consistency

### DON'T:
- Be vague or ambiguous
- Assume the model knows context
- Skip validation instructions
- Ignore output format needs
- Hardcode prompts in multiple places

## Resources
- **Papers**:
  - "Chain-of-Thought Prompting Elicits Reasoning in LLMs" (Wei et al., 2022)
  - "ReAct: Synergizing Reasoning and Acting" (Yao et al., 2022)
  - "Large Language Models are Zero-Shot Reasoners" (Kojima et al., 2022)
- **Guides**:
  - [OpenAI Prompt Engineering Guide](https://platform.openai.com/docs/guides/prompt-engineering)
  - [Anthropic Prompt Engineering](https://docs.anthropic.com/claude/docs/prompt-engineering)
  - [Prompt Engineering Guide](https://www.promptingguide.ai/)
- **Tools**:
  - [LangChain PromptTemplate](https://python.langchain.com/docs/modules/model_io/prompts/)

## Daily Challenge

**Build a Multi-Stage Reasoning Agent**

Create an agent that:
1. Accepts a complex question or problem
2. Uses chain-of-thought to break it down
3. Identifies if external tools/data are needed
4. Reasons through each step
5. Produces a structured JSON output with:
   - reasoning_steps: array of thought processes
   - tools_needed: array of tools that would help
   - confidence: 0-100 score
   - answer: final response
   - assumptions: any assumptions made

Test with various question types:
- Factual: "What's the capital of France?"
- Analytical: "Compare pros/cons of React vs Vue"
- Mathematical: "If I invest $1000 at 5% annual interest, how much in 10 years?"
- Impossible: "What will the stock market do tomorrow?"

## Reflection Questions
1. How does chain-of-thought improve agent reliability?
2. When should you use high vs low temperature?
3. What's the trade-off between detailed prompts and token usage?
4. How can prompt engineering reduce hallucinations?

## Tomorrow's Preview
Day 4: Function Calling & Tool Use - Learn how agents interact with external tools, APIs, and functions to take actions in the real world.

---

**Progress**: 3/30 days completed

[← Previous: Day 2](./day-02.md) | [Back to Overview](../README.md) | [Next: Day 4 →](./day-04.md)
