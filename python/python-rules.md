<!-- filepath: /python/python-rules.md -->
# GitHub Copilot: Python Rules

A collection of structured rule directives for GitHub Copilot to follow when working with Python code.

**Keywords**: #python #rules #best-practices #coding-standards

---

## Python Rule Directives

### Code Style and Quality

```
@python Rule - Follow PEP 8: When writing Python code, always follow PEP 8 style guidelines including line length, naming conventions, and indentation.
```

```
@python Rule - Type Hints: Use type hints for function parameters and return values to improve code readability and enable better static analysis.
```

```
@python Rule - Docstrings: Document all public functions, classes, and methods with docstrings following the Google style guide format.
```

```
@python Rule - F-Strings: Prefer f-strings over older string formatting methods for readability and performance.
```

### Python Best Practices

```
@python Rule - Context Managers: Use context managers (with statements) for resource management to ensure proper cleanup.
```

```
@python Rule - List Comprehensions: Use list comprehensions instead of map/filter when appropriate for readability.
```

```
@python Rule - Default Parameters: Never use mutable objects as default parameters to avoid unexpected behavior.
```

```
@python Rule - Exception Handling: Be specific about exceptions caught and provide informative error messages.
```

### Testing and Quality Assurance

```
@python Rule - Unit Testing: Write unit tests for all public functions and classes using pytest.
```

```
@python Rule - Test Coverage: Strive for at least 80% test coverage for production code.
```

```
@python Rule - Static Analysis: Use tools like mypy, pylint, or flake8 to catch common errors and enforce style.
```

### Package Management

```
@python Rule - Virtual Environments: Always use virtual environments for Python projects to manage dependencies.
```

```
@python Rule - Requirements File: Maintain an up-to-date requirements.txt or pyproject.toml file with specific version constraints.
```

### Python Performance

```
@python Rule - Generator Expressions: Use generator expressions for large datasets to save memory.
```

```
@python Rule - Profiling: Profile performance-critical code to identify bottlenecks and optimize accordingly.
```

---

## Usage Instructions

To use these rules in your GitHub Copilot instructions, copy the appropriate rule directives into your instruction file or include this file using:

```markdown
<!-- include: /python/python-rules.md -->
```
