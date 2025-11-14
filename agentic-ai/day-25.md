# Day 25: Security & Safety Considerations

## Overview
Build secure and safe AI agents that protect user data and prevent misuse.

## Learning Objectives
- Implement input validation and sanitization
- Prevent prompt injection attacks
- Secure API keys and credentials
- Implement rate limiting
- Content filtering
- Audit logging for security

## Security Threats

### 1. Prompt Injection
```python
def validate_input(user_input):
    """Detect and prevent prompt injection"""
    dangerous_patterns = [
        "ignore previous instructions",
        "disregard system prompt",
        "new instructions:",
        "override:"
    ]

    for pattern in dangerous_patterns:
        if pattern.lower() in user_input.lower():
            raise SecurityException("Potential prompt injection detected")

    return user_input
```

### 2. Data Leakage Prevention
```python
def sanitize_output(output):
    """Remove sensitive information from output"""
    # Remove email addresses
    output = re.sub(r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b', '[EMAIL]', output)

    # Remove API keys
    output = re.sub(r'sk-[a-zA-Z0-9]{48}', '[API_KEY]', output)

    # Remove credit cards
    output = re.sub(r'\b\d{4}[-\s]?\d{4}[-\s]?\d{4}[-\s]?\d{4}\b', '[CARD]', output)

    return output
```

### 3. Rate Limiting
```python
from datetime import datetime, timedelta

class RateLimiter:
    def __init__(self, max_requests=100, window_minutes=60):
        self.max_requests = max_requests
        self.window = timedelta(minutes=window_minutes)
        self.requests = {}

    def allow_request(self, user_id):
        now = datetime.now()

        if user_id not in self.requests:
            self.requests[user_id] = []

        # Remove old requests
        self.requests[user_id] = [
            req_time for req_time in self.requests[user_id]
            if now - req_time < self.window
        ]

        # Check limit
        if len(self.requests[user_id]) >= self.max_requests:
            return False

        self.requests[user_id].append(now)
        return True
```

### 4. Secure Configuration
```python
import os
from cryptography.fernet import Fernet

class SecureConfig:
    def __init__(self):
        self.encryption_key = os.getenv("ENCRYPTION_KEY")
        self.fernet = Fernet(self.encryption_key)

    def encrypt_secret(self, secret):
        return self.fernet.encrypt(secret.encode())

    def decrypt_secret(self, encrypted):
        return self.fernet.decrypt(encrypted).decode()
```

## Content Safety

```python
class ContentModerator:
    """Filter unsafe content"""

    def __init__(self):
        self.banned_topics = [
            "violence",
            "illegal activities",
            # Add more
        ]

    def is_safe(self, text):
        # Check against OpenAI moderation API
        response = client.moderations.create(input=text)
        return not response.results[0].flagged

    def filter_output(self, output):
        if not self.is_safe(output):
            return "I cannot provide that information."
        return output
```

## Best Practices

1. **Never Trust User Input**: Always validate
2. **Principle of Least Privilege**: Minimal permissions
3. **Encrypt Secrets**: Never store in plain text
4. **Audit Everything**: Log security events
5. **Regular Updates**: Keep dependencies current
6. **Content Moderation**: Filter unsafe outputs

## Resources
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OpenAI Safety Best Practices](https://platform.openai.com/docs/guides/safety-best-practices)

## Daily Challenge
Add comprehensive security measures to your agent including input validation, rate limiting, and audit logging.

---

**Progress**: 25/30 days completed

[← Previous: Day 24](./day-24.md) | [Back to Overview](../README.md) | [Next: Day 26 →](./day-26.md)
