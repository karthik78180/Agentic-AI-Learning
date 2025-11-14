# Day 20: Agent Evaluation & Testing

## Overview
Learn to evaluate and test AI agents systematically. Ensure your agents are reliable, accurate, and production-ready.

## Learning Objectives
- Define evaluation metrics for agents
- Build test suites for agents
- Implement benchmarking systems
- Use human evaluation effectively
- Create regression test frameworks
- Monitor agent performance in production

## Evaluation Metrics

### Task Success Metrics
- **Task Completion Rate**: % of tasks completed successfully
- **Accuracy**: Correctness of outputs
- **Precision & Recall**: For classification/retrieval tasks
- **F1 Score**: Harmonic mean of precision and recall

### Efficiency Metrics
- **Token Usage**: Total tokens consumed
- **API Costs**: Total cost per task
- **Latency**: Time to complete tasks
- **Tool Call Efficiency**: Unnecessary tool calls

### Quality Metrics
- **Relevance**: Output relevance to query
- **Coherence**: Logical flow of reasoning
- **Hallucination Rate**: Frequency of false information
- **Citation Accuracy**: Correct source attribution

## Testing Framework

```python
# agent_testing.py
import unittest
from typing import Dict, List

class AgentTestCase:
    def __init__(self, input: str, expected_output: str = None,
                 expected_tools: List[str] = None):
        self.input = input
        self.expected_output = expected_output
        self.expected_tools = expected_tools

class AgentTester:
    """Framework for testing agents"""

    def __init__(self, agent):
        self.agent = agent
        self.results = []

    def run_test(self, test_case: AgentTestCase) -> Dict:
        """Run a single test"""
        result = {
            "input": test_case.input,
            "passed": False,
            "output": None,
            "errors": []
        }

        try:
            output = self.agent.process(test_case.input)
            result["output"] = output

            # Check expected output
            if test_case.expected_output:
                if test_case.expected_output in output:
                    result["passed"] = True
                else:
                    result["errors"].append("Output mismatch")

            # Check tool usage
            if test_case.expected_tools:
                tools_used = self.agent.get_tools_used()
                if set(tools_used) == set(test_case.expected_tools):
                    result["passed"] = True
                else:
                    result["errors"].append(
                        f"Tool mismatch: expected {test_case.expected_tools}, "
                        f"got {tools_used}"
                    )

        except Exception as e:
            result["errors"].append(str(e))

        self.results.append(result)
        return result

    def run_suite(self, test_cases: List[AgentTestCase]):
        """Run test suite"""
        print(f"Running {len(test_cases)} tests...\n")

        passed = 0
        for i, test_case in enumerate(test_cases, 1):
            result = self.run_test(test_case)
            status = "✓" if result["passed"] else "✗"
            print(f"{status} Test {i}: {test_case.input[:50]}...")

            if result["passed"]:
                passed += 1

        print(f"\nResults: {passed}/{len(test_cases)} passed")
        return self.results

# Usage
test_cases = [
    AgentTestCase(
        input="What is 2+2?",
        expected_tools=["calculator"]
    ),
    AgentTestCase(
        input="Search for Python tutorials",
        expected_tools=["web_search"]
    )
]

tester = AgentTester(my_agent)
results = tester.run_suite(test_cases)
```

## Benchmarking

```python
# benchmark.py
import time
from typing import List

class AgentBenchmark:
    def __init__(self, agent):
        self.agent = agent

    def benchmark(self, queries: List[str]) -> Dict:
        """Benchmark agent performance"""
        metrics = {
            "total_time": 0,
            "total_tokens": 0,
            "total_cost": 0,
            "success_count": 0,
            "failure_count": 0
        }

        for query in queries:
            start = time.time()

            try:
                result = self.agent.process(query)
                metrics["success_count"] += 1

                # Track metrics
                metrics["total_time"] += time.time() - start
                metrics["total_tokens"] += self.agent.last_token_count
                metrics["total_cost"] += self.agent.last_cost

            except Exception as e:
                metrics["failure_count"] += 1

        # Calculate averages
        total = len(queries)
        metrics["avg_time"] = metrics["total_time"] / total
        metrics["avg_tokens"] = metrics["total_tokens"] / total
        metrics["avg_cost"] = metrics["total_cost"] / total
        metrics["success_rate"] = metrics["success_count"] / total

        return metrics
```

## Human Evaluation

```python
class HumanEvaluator:
    """Collect human ratings for agent outputs"""

    def evaluate(self, query: str, output: str) -> Dict:
        print(f"\nQuery: {query}")
        print(f"Output: {output}\n")

        ratings = {}
        ratings["relevance"] = int(input("Relevance (1-5): "))
        ratings["accuracy"] = int(input("Accuracy (1-5): "))
        ratings["helpfulness"] = int(input("Helpfulness (1-5): "))

        return ratings
```

## Best Practices

1. **Automated Testing**: Run tests on every change
2. **Diverse Test Cases**: Cover edge cases and failures
3. **Regression Testing**: Ensure fixes don't break existing functionality
4. **Performance Monitoring**: Track metrics over time
5. **A/B Testing**: Compare different agent versions
6. **Human Validation**: Regular human review of outputs

## Resources
- [LangChain Evaluation](https://python.langchain.com/docs/guides/evaluation/)
- [OpenAI Evals](https://github.com/openai/evals)
- [Anthropic Evaluation Guide](https://docs.anthropic.com/claude/docs/test-and-evaluate)

## Daily Challenge
Create a comprehensive test suite for your research assistant with:
- 20+ test cases covering different scenarios
- Automated evaluation metrics
- Benchmark comparing different prompts/models
- Human evaluation workflow

---

**Progress**: 20/30 days completed | 2/3 Complete!

[← Previous: Day 19](./day-19.md) | [Back to Overview](../README.md) | [Next: Day 21 →](./day-21.md)
