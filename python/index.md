# GitHub Copilot: Python Instruction Files

This directory contains specialized instruction files for Python development with GitHub Copilot. Each file focuses on a specific aspect of Python development to optimize token usage and provide more targeted guidance.

## Available Instruction Files

1. **[Python Core Principles](python-core-principles.md)**
   - Core programming principles
   - Python language specifics
   - Documentation standards
   - **Keywords**: #python #core #principles #documentation #docstrings #typing

2. **[Python Code Quality](python-code-quality.md)**
   - Code quality practices
   - Testable code writing
   - Static analysis
   - Dependency management
   - **Keywords**: #python #code-quality #testing #linting #static-analysis

3. **[Python Performance](python-performance.md)**
   - Performance optimization techniques
   - Concurrency patterns
   - Caching strategies
   - Memory optimization
   - **Keywords**: #python #performance #optimization #concurrency #caching #memory

4. **[Python Rules](python-rules.md)**
   - Structured rule directives
   - Format-specific guidance for GitHub Copilot
   - Concise coding standards
   - **Keywords**: #python #rules #directives #guidelines #standards

## Usage Guide

Choose the most appropriate instruction file(s) based on your current development focus:

- For general Python development, include **Python Core Principles**
- When focusing on code quality and testing, include **Python Code Quality**
- For optimizing performance-critical code, include **Python Performance**
- When you need specific, structured guidance for GitHub Copilot, include **Python Rules**

You can include multiple files if your task spans multiple areas.

### Using Python Rules

The **Python Rules** file contains structured directives that GitHub Copilot can follow. These rules use the format:

```markdown
@python Rule - RuleName: RuleDescription
```

Example usage in your Copilot instructions:

```markdown
<instructions>
@python Rule - Follow PEP 8: When writing Python code, always follow PEP 8 style guidelines.
@python Rule - Type Hints: Use type hints for function parameters and return values.
</instructions>
```

---

These instruction files are derived from the comprehensive [Python Best Practices](/python-best-practices.md) but are split for more efficient token usage and focused guidance.
