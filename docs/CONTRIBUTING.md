# Contributing Guide

Thank you for your interest in contributing to the Personalized Holiday Management Agent project! This document provides guidelines and instructions for contributing.

## Table of Contents

1. [Code of Conduct](#code-of-conduct)
2. [Getting Started](#getting-started)
3. [Development Workflow](#development-workflow)
4. [Coding Standards](#coding-standards)
5. [Commit Guidelines](#commit-guidelines)
6. [Pull Request Process](#pull-request-process)
7. [Reporting Issues](#reporting-issues)
8. [Feature Requests](#feature-requests)

---

## Code of Conduct

### Our Pledge

We are committed to providing a welcoming and inspiring community for all. Please read and adhere to our Code of Conduct:

- **Be Respectful**: Treat everyone with respect and kindness
- **Be Inclusive**: Welcome people of all backgrounds
- **Be Collaborative**: Work together constructively
- **Be Professional**: Keep discussions focused and productive

---

## Getting Started

### 1. Fork the Repository

Click the "Fork" button on GitHub to create your own copy.

```bash
# Clone your fork
git clone https://github.com/YOUR_USERNAME/Personalized-Holiday-Management-Agent.git
cd Personalized-Holiday-Management-Agent

# Add upstream remote
git remote add upstream https://github.com/original-owner/Personalized-Holiday-Management-Agent.git
```

### 2. Set Up Development Environment

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Install development tools
pip install pytest pytest-cov black flake8 mypy
```

### 3. Create a Branch

```bash
# Fetch latest from upstream
git fetch upstream

# Create feature branch
git checkout -b feature/your-feature-name

# Or for bug fixes
git checkout -b fix/issue-description
```

---

## Development Workflow

### 1. Make Your Changes

- Keep changes focused and single-purpose
- Write clean, readable code
- Add comments for complex logic
- Update documentation if needed

### 2. Test Your Changes

```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_planner.py

# Run with coverage
pytest --cov=holiday_management

# Check code style
black --check .
flake8 .
mypy .
```

### 3. Format Code

```bash
# Auto-format with Black
black holiday_management/ app.py

# Fix some linting issues automatically
flake8 --extend-ignore=E203,W503 holiday_management/
```

### 4. Commit Changes

```bash
git add .
git commit -m "feat: Add new feature description"
```

See [Commit Guidelines](#commit-guidelines) for message format.

### 5. Push and Create Pull Request

```bash
# Push to your fork
git push origin feature/your-feature-name

# Go to GitHub and create Pull Request
```

---

## Coding Standards

### Python Style Guide

Follow PEP 8 with these tools:

#### Black (Code Formatter)
```bash
black .
```

#### Flake8 (Linter)
```bash
flake8 .
```

#### MyPy (Type Checker)
```bash
mypy .
```

### Naming Conventions

```python
# Classes: PascalCase
class HolidayPlanner:
    pass

# Functions: snake_case
def plan_itinerary():
    pass

# Constants: UPPER_SNAKE_CASE
MAX_DAYS = 30

# Private members: _leading_underscore
def _helper_function():
    pass
```

### Code Examples

#### Good Example
```python
"""Plan a holiday itinerary based on user requirements."""

from typing import List, Dict
from pydantic import BaseModel

class ItineraryRequest(BaseModel):
    destination: str
    duration_days: int
    interests: List[str]

def plan_itinerary(request: ItineraryRequest) -> Dict[str, any]:
    """
    Create an itinerary plan.
    
    Args:
        request: Holiday planning request with constraints
        
    Returns:
        Dictionary with planned activities
        
    Raises:
        ValueError: If invalid duration specified
    """
    if request.duration_days < 1:
        raise ValueError("Duration must be at least 1 day")
    
    # Implementation
    return {}
```

#### Bad Example
```python
# No docstring
def plan(r):
    # Not descriptive
    if r.d < 1:
        raise ValueError("Bad")
    return {}
```

### Documentation

Always include docstrings:

```python
def function_name(param1: str, param2: int) -> Dict[str, str]:
    """
    Brief description of what the function does.
    
    Longer description if needed, explaining the logic,
    special cases, or important details.
    
    Args:
        param1: Description of param1
        param2: Description of param2
        
    Returns:
        Description of return value
        
    Raises:
        ValueError: When param2 is negative
        TypeError: When param1 is not a string
        
    Example:
        >>> result = function_name("test", 42)
        >>> print(result)
        {...}
    """
    pass
```

### Type Hints

Always use type hints:

```python
from typing import List, Optional, Dict, Tuple

# Functions
def process_requests(items: List[str]) -> Dict[str, any]:
    pass

# Classes
class Agent:
    name: str
    model_client: Optional[object] = None
    
    def process(self, request: str) -> str:
        pass

# Variables
results: List[Dict[str, any]] = []
```

---

## Commit Guidelines

Follow Conventional Commits format:

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

- **feat**: New feature
- **fix**: Bug fix
- **docs**: Documentation changes
- **style**: Code style (no functional change)
- **refactor**: Code refactoring
- **perf**: Performance improvements
- **test**: Test additions/changes
- **chore**: Build/dependency updates

### Examples

```bash
# Feature
git commit -m "feat(agents): Add researcher agent implementation"

# Bug fix
git commit -m "fix(teams): Resolve agent message passing issue"

# Documentation
git commit -m "docs: Update API documentation"

# With body
git commit -m "feat(planner): Enhance itinerary generation
  
This commit improves the planning algorithm to better
handle multi-destination trips by considering travel
time between locations.

Closes #123"
```

---

## Pull Request Process

### 1. Before Submitting

- [ ] Fork the repository
- [ ] Create feature branch
- [ ] Make focused changes
- [ ] Test thoroughly
- [ ] Update documentation
- [ ] Commit with clear messages

### 2. PR Description Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Related Issues
Closes #123

## Changes Made
- Change 1
- Change 2
- Change 3

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] Manual testing completed

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex logic
- [ ] Documentation updated
- [ ] No new warnings generated
- [ ] Tests pass locally
```

### 3. PR Review Process

- Maintainers will review your PR
- Address feedback and suggestions
- Update code as needed
- Ensure CI/CD passes
- Wait for approval to merge

### 4. Common Feedback

| Feedback | Action |
|----------|--------|
| "Please add tests" | Write unit/integration tests |
| "Update documentation" | Add/update docstrings and docs |
| "Code style issues" | Run `black` and `flake8` |
| "Merge conflicts" | Resolve conflicts and rebase |

---

## Reporting Issues

### Bug Reports

Use GitHub Issues with this template:

```markdown
## Bug Description
Clear description of the bug

## Steps to Reproduce
1. Step 1
2. Step 2
3. Step 3

## Expected Behavior
What should happen

## Actual Behavior
What actually happens

## Environment
- OS: [e.g., Windows 10, macOS 12]
- Python: [e.g., 3.11]
- Package version: [e.g., 1.0.0]

## Error Messages
```
Paste error logs here
```

## Screenshots
If applicable, add screenshots
```

### Example Issue

```markdown
## Bug: API returns 500 on invalid input

### Steps to Reproduce
1. Send POST request to `/plan` with empty content
2. Observe server response

### Expected Behavior
Should return 422 Unprocessable Entity with validation error

### Actual Behavior
Returns 500 Internal Server Error

### Environment
- OS: Windows 11
- Python: 3.11
- Version: 1.0.0

### Error Message
```
Traceback (most recent call last):
  File "app.py", line 35, in plan
    TypeError: 'NoneType' object is not subscriptable
```
```

---

## Feature Requests

### Suggesting Enhancements

Use GitHub Issues with this template:

```markdown
## Feature Description
Clear description of the desired feature

## Problem Statement
Problem this feature would solve

## Proposed Solution
Your idea for the solution

## Alternatives Considered
Other possible approaches

## Additional Context
Any other context, screenshots, or references
```

### Example Feature Request

```markdown
## Feature: Export itinerary as PDF

### Description
Allow users to download generated itineraries as PDF files

### Problem
Currently only Markdown format available, users need PDF

### Proposed Solution
Add `/export-pdf` endpoint that converts Markdown to PDF

### Alternatives
- Use browser print-to-PDF
- Email itinerary

### Additional Context
Would help users share itineraries in professional settings
```

---

## Areas to Contribute

### Code

- [ ] Bug fixes
- [ ] New agents (Tourism, Budget, etc.)
- [ ] API enhancements
- [ ] Performance optimizations
- [ ] Testing improvements

### Documentation

- [ ] README improvements
- [ ] Tutorial creation
- [ ] API documentation
- [ ] Architecture diagrams
- [ ] Example scripts

### Infrastructure

- [ ] Docker setup
- [ ] CI/CD pipeline
- [ ] Database optimization
- [ ] Deployment guides

### Community

- [ ] Answer questions
- [ ] Review pull requests
- [ ] Provide feedback on issues
- [ ] Share your projects

---

## Development Tips

### Running in Debug Mode

```bash
# Enable debug logging
DEBUG=True uvicorn app:app --reload

# Check logs
tail -f holiday_agent.log
```

### Testing Agent Logic

```python
# test_planner.py
import pytest
from holiday_management.agents.planner import planner_agent

def test_planner_agent_creation():
    assert planner_agent.name == "Holiday_Planner"
    assert planner_agent.model_client is not None
```

### Database Exploration

```bash
# Check Chroma database
python -c "from chromadb import Client; c = Client(); print(c.list_collections())"
```

### Performance Profiling

```python
import cProfile
import pstats

profiler = cProfile.Profile()
profiler.enable()

# Run your code here

profiler.disable()
stats = pstats.Stats(profiler)
stats.sort_stats('cumulative')
stats.print_stats()
```

---

## Getting Help

- **Discussions**: Start a discussion on GitHub for questions
- **Issues**: Check existing issues before creating new ones
- **Documentation**: Read docs/ folder thoroughly
- **Community**: Join our Discord/Slack (if available)

---

## Recognition

Contributors will be recognized in:
- GitHub contributors list
- Project CONTRIBUTORS file
- Release notes

Thank you for contributing! 🙏

---

## License

By contributing, you agree that your contributions will be licensed under the same license as the project (MIT).

---

Questions? Don't hesitate to ask!

📧 Email: support@example.com
💬 GitHub Discussions: [Link]
🐛 GitHub Issues: [Link]
