# Day 27: Code Generation Agents

## Overview
Specialize in building agents that write, test, and debug code autonomously.

## Learning Objectives
- Build code generation agents
- Implement code testing and validation
- Create debugging agents
- Design code review systems
- Handle multiple programming languages

## Code Generation Agent

```python
class CodeGenerationAgent:
    """Agent that generates and validates code"""

    def generate_code(self, specification):
        """Generate code from specification"""

        prompt = f"""Generate Python code for this specification:

{specification}

Requirements:
- Include type hints
- Add docstrings
- Handle errors
- Follow PEP 8
- Include example usage

Return only the code, no explanations.
"""

        code = self.llm.call(prompt)
        return self.clean_code(code)

    def generate_tests(self, code):
        """Generate tests for code"""

        prompt = f"""Generate pytest tests for this code:

{code}

Include:
- Test happy path
- Test edge cases
- Test error handling
- Use fixtures where appropriate
"""

        return self.llm.call(prompt)

    def validate_code(self, code):
        """Validate code works"""

        # Syntax check
        try:
            compile(code, '<string>', 'exec')
        except SyntaxError as e:
            return False, f"Syntax error: {e}"

        # Run tests
        tests = self.generate_tests(code)
        result = self.run_tests(code, tests)

        return result.success, result.output

    def iterative_fix(self, spec, code, error):
        """Fix code based on error"""

        prompt = f"""Fix this code:

Specification: {spec}

Current Code:
{code}

Error:
{error}

Provide corrected code.
"""

        fixed_code = self.llm.call(prompt)
        return fixed_code

    def generate_with_validation(self, spec, max_attempts=3):
        """Generate and validate code"""

        for attempt in range(max_attempts):
            code = self.generate_code(spec)

            success, message = self.validate_code(code)

            if success:
                return code

            # Fix and retry
            code = self.iterative_fix(spec, code, message)

        return None
```

## Code Review Agent

```python
class CodeReviewAgent:
    """Agent that reviews code quality"""

    def review(self, code):
        """Comprehensive code review"""

        reviews = {
            "quality": self.review_quality(code),
            "security": self.review_security(code),
            "performance": self.review_performance(code),
            "best_practices": self.review_best_practices(code)
        }

        return self.synthesize_review(reviews)

    def review_quality(self, code):
        """Review code quality"""
        prompt = f"""Review this code for quality:

{code}

Check:
- Readability
- Maintainability
- Documentation
- Naming conventions

Provide specific suggestions.
"""
        return self.llm.call(prompt)
```

## Multi-Language Support

```python
class LanguageAgnosticAgent:
    """Support multiple programming languages"""

    LANGUAGES = {
        "python": {"ext": ".py", "test_framework": "pytest"},
        "javascript": {"ext": ".js", "test_framework": "jest"},
        "java": {"ext": ".java", "test_framework": "junit"}
    }

    def detect_language(self, code):
        """Detect programming language"""
        # Use LLM or heuristics
        pass

    def generate_for_language(self, spec, language):
        """Generate code in specific language"""
        config = self.LANGUAGES.get(language)

        prompt = f"""Generate {language} code for:

{spec}

Follow {language} best practices and idioms.
"""

        return self.llm.call(prompt)
```

## Resources
- [GitHub Copilot](https://github.com/features/copilot)
- [Cursor](https://cursor.sh/)
- [Aider](https://github.com/paul-gauthier/aider)

## Daily Challenge
Build a code generation agent that:
1. Takes a feature description
2. Generates code
3. Creates tests
4. Runs tests
5. Fixes failures automatically
6. Generates documentation

---

**Progress**: 27/30 days completed

[← Previous: Day 26](./day-26.md) | [Back to Overview](../README.md) | [Next: Day 28 →](./day-28.md)
