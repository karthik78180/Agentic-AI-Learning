# Day 2: LLM Fundamentals & API Basics

## Overview
Today you'll master the foundation of agentic AI: Large Language Models. Understanding how LLMs work, their capabilities, limitations, and how to interact with them effectively is crucial for building robust agents.

## Learning Objectives
- Understand how LLMs work at a high level
- Compare different LLM providers and models
- Master API interactions and parameters
- Learn about tokens, context windows, and costs
- Handle streaming and async responses
- Understand model selection for different use cases

## Theoretical Concepts

### How LLMs Work
LLMs are neural networks trained to predict the next token (word/subword) in a sequence:
- **Training**: Trained on massive text corpora (books, websites, code)
- **Architecture**: Based on Transformer architecture (attention mechanisms)
- **Prediction**: Generate text by repeatedly predicting next tokens
- **Context**: Use previous tokens to inform predictions

### Key LLM Concepts

#### Tokens
- Chunks of text (roughly 0.75 words in English)
- Different models use different tokenization
- Costs are calculated per token
- Context windows measured in tokens

```python
# Example: "Hello, world!" might be ~3 tokens
# "Artificial Intelligence" might be ~3-4 tokens
```

#### Context Window
- Maximum tokens the model can process at once
- Includes input (prompt) + output (completion)
- Larger windows = more expensive but more context

| Model | Context Window |
|-------|----------------|
| GPT-3.5-Turbo | 16K tokens |
| GPT-4 | 128K tokens |
| Claude 3 Sonnet | 200K tokens |
| Gemini 1.5 Pro | 1M tokens |

#### Temperature & Sampling
- **Temperature** (0.0-2.0): Controls randomness
  - 0.0 = Deterministic (same output each time)
  - 1.0 = Balanced creativity
  - 2.0 = Very random/creative
- **Top-p**: Nucleus sampling (alternative to temperature)
- **Max Tokens**: Limit response length

### Major LLM Providers

#### OpenAI
- **Models**: GPT-4, GPT-4-Turbo, GPT-3.5-Turbo
- **Strengths**: Strong reasoning, function calling, widespread adoption
- **Best for**: General purpose agents, code generation

#### Anthropic
- **Models**: Claude 3 Opus, Sonnet, Haiku
- **Strengths**: Long context, safety, nuanced understanding
- **Best for**: Complex reasoning, analysis, safety-critical applications

#### Google
- **Models**: Gemini Pro, Gemini Ultra
- **Strengths**: Multimodal, massive context windows
- **Best for**: Processing large documents, multimodal tasks

#### Open Source
- **Models**: Llama 3, Mixtral, Phi-3
- **Strengths**: Cost-effective, customizable, privacy
- **Best for**: Self-hosted solutions, fine-tuning

## Hands-On Exercises

### Exercise 1: Multi-Provider Setup
Set up API access for multiple providers.

```python
# config.py
import os
from dotenv import load_dotenv

load_dotenv()

class LLMConfig:
    OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
    ANTHROPIC_API_KEY = os.getenv("ANTHROPIC_API_KEY")
    GOOGLE_API_KEY = os.getenv("GOOGLE_API_KEY")

    # Model selection
    OPENAI_MODEL = "gpt-4-turbo-preview"
    ANTHROPIC_MODEL = "claude-3-sonnet-20240229"
    GOOGLE_MODEL = "gemini-pro"
```

### Exercise 2: Comparing Model Outputs
Test the same prompt across different models.

```python
# compare_models.py
import os
from openai import OpenAI
from anthropic import Anthropic
import google.generativeai as genai

def compare_models(prompt):
    """Compare outputs from different LLM providers"""
    results = {}

    # OpenAI
    openai_client = OpenAI()
    response = openai_client.chat.completions.create(
        model="gpt-4-turbo-preview",
        messages=[{"role": "user", "content": prompt}],
        temperature=0.7
    )
    results["OpenAI GPT-4"] = response.choices[0].message.content

    # Anthropic Claude
    anthropic_client = Anthropic()
    response = anthropic_client.messages.create(
        model="claude-3-sonnet-20240229",
        max_tokens=1024,
        messages=[{"role": "user", "content": prompt}]
    )
    results["Anthropic Claude"] = response.content[0].text

    # Display results
    print(f"Prompt: {prompt}\n")
    print("=" * 80)
    for model, output in results.items():
        print(f"\n{model}:")
        print("-" * 80)
        print(output)
        print("=" * 80)

    return results

# Test
compare_models("Explain quantum computing in simple terms.")
```

### Exercise 3: Understanding Token Usage
Track and analyze token consumption.

```python
# token_tracker.py
import tiktoken

def count_tokens(text, model="gpt-4"):
    """Count tokens in text for a given model"""
    encoding = tiktoken.encoding_for_model(model)
    return len(encoding.encode(text))

def estimate_cost(input_tokens, output_tokens, model="gpt-4"):
    """Estimate API cost"""
    # Prices as of 2024 (per 1K tokens)
    prices = {
        "gpt-4": {"input": 0.03, "output": 0.06},
        "gpt-4-turbo": {"input": 0.01, "output": 0.03},
        "gpt-3.5-turbo": {"input": 0.0005, "output": 0.0015},
        "claude-3-sonnet": {"input": 0.003, "output": 0.015}
    }

    if model not in prices:
        return None

    cost = (input_tokens / 1000 * prices[model]["input"] +
            output_tokens / 1000 * prices[model]["output"])
    return cost

# Example usage
prompt = "Write a detailed essay about artificial intelligence."
input_tokens = count_tokens(prompt)
estimated_output = 1000  # tokens

print(f"Input tokens: {input_tokens}")
print(f"Estimated output tokens: {estimated_output}")
print(f"Estimated cost: ${estimate_cost(input_tokens, estimated_output, 'gpt-4'):.4f}")
```

### Exercise 4: Streaming Responses
Implement streaming for real-time output.

```python
# streaming_example.py
from openai import OpenAI

client = OpenAI()

def stream_response(prompt):
    """Stream LLM response in real-time"""
    stream = client.chat.completions.create(
        model="gpt-4-turbo-preview",
        messages=[{"role": "user", "content": prompt}],
        stream=True
    )

    print("Streaming response:\n")
    for chunk in stream:
        if chunk.choices[0].delta.content:
            print(chunk.choices[0].delta.content, end="", flush=True)

    print("\n")

stream_response("Write a haiku about AI agents.")
```

### Exercise 5: Async API Calls
Make parallel API calls for efficiency.

```python
# async_llm.py
import asyncio
from openai import AsyncOpenAI

client = AsyncOpenAI()

async def get_completion(prompt, model="gpt-4-turbo-preview"):
    """Async LLM completion"""
    response = await client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}]
    )
    return response.choices[0].message.content

async def parallel_completions(prompts):
    """Run multiple completions in parallel"""
    tasks = [get_completion(prompt) for prompt in prompts]
    results = await asyncio.gather(*tasks)
    return results

# Example
prompts = [
    "What is machine learning?",
    "What is deep learning?",
    "What is reinforcement learning?"
]

results = asyncio.run(parallel_completions(prompts))
for prompt, result in zip(prompts, results):
    print(f"Q: {prompt}")
    print(f"A: {result}\n")
```

## Key Takeaways
1. LLMs predict next tokens based on training data and context
2. Different models have different strengths (reasoning, context, speed, cost)
3. Tokens are the unit of measurement for cost and context
4. Temperature controls randomness/creativity in outputs
5. Streaming and async calls improve user experience and efficiency
6. Always track token usage for cost management

## Advanced Concepts

### System Prompts
System messages set the agent's behavior and persona:

```python
messages = [
    {"role": "system", "content": "You are an expert Python developer..."},
    {"role": "user", "content": "How do I sort a dictionary?"}
]
```

### Few-Shot Learning
Provide examples to guide the model:

```python
messages = [
    {"role": "system", "content": "Convert sentences to emojis"},
    {"role": "user", "content": "I love pizza"},
    {"role": "assistant", "content": "😍🍕"},
    {"role": "user", "content": "The weather is sunny"},
    {"role": "assistant", "content": "☀️😊"},
    {"role": "user", "content": "I'm going to the beach"}
]
```

## Resources
- **Documentation**:
  - [OpenAI API Reference](https://platform.openai.com/docs/api-reference)
  - [Anthropic API Documentation](https://docs.anthropic.com/claude/reference)
  - [Google AI Studio](https://ai.google.dev/)
- **Tools**:
  - [Tiktoken (OpenAI tokenizer)](https://github.com/openai/tiktoken)
  - [LangChain Model Comparison](https://python.langchain.com/docs/integrations/llms/)
- **Cost Calculators**:
  - [OpenAI Pricing](https://openai.com/pricing)
  - [Anthropic Pricing](https://www.anthropic.com/pricing)

## Daily Challenge

**Build a Smart Model Router**

Create a system that:
1. Takes a user query
2. Analyzes the complexity (simple/medium/complex)
3. Routes to appropriate model:
   - Simple → GPT-3.5 or Claude Haiku
   - Medium → GPT-4 or Claude Sonnet
   - Complex → GPT-4 with higher token limit or Claude Opus
4. Tracks total cost across queries
5. Provides cost optimization suggestions

Bonus: Add retry logic and error handling.

## Reflection Questions
1. How does context window size affect agent capabilities?
2. When would you choose Claude over GPT-4, or vice versa?
3. How can you minimize API costs while maintaining quality?
4. What role does temperature play in agent reliability?

## Tomorrow's Preview
Day 3: Prompt Engineering for Agents - Learn advanced techniques to get the best performance from LLMs, including chain-of-thought, role prompting, and structured outputs.

---

**Progress**: 2/30 days completed

[← Previous: Day 1](./day-01.md) | [Back to Overview](../README.md) | [Next: Day 3 →](./day-03.md)
