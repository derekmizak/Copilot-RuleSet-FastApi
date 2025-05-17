# GitHub Copilot: Python Core Principles

Code-focused guidelines for Python 3.12+ development covering core principles, language specifics, and documentation practices.

> **Note**: These instructions focus on actual coding practices that you'll use while writing Python code, not the broader development lifecycle.

**Keywords**: #python #core #principles #documentation #docstrings #typing

---

## 🧠 Core Principles

* **DRY**: Don't Repeat Yourself. Single source of truth for knowledge.
* **KISS**: Keep It Simple, Stupid. Avoid needless complexity.
* **YAGNI**: You Ain't Gonna Need It. Implement only necessary features.
* **SOLID**:
  * **S**RP: Single Responsibility. Modules/classes/funcs do one thing.
  * **O**CP: Open/Closed. Extendable, not modifiable.
  * **L**SP: Liskov Substitution. Subtypes replaceable for base types.
  * **I**SP: Interface Segregation. Small, specific interfaces.
  * **D**IP: Dependency Inversion. Depend on abstractions, not concretions.

---

## 🐍 Python Specifics (Python 3.12+)

* **Functions**: SRP. Clear purpose. Leverage new pattern matching features.
* **Docstrings**: PEP 257 compliant. Format recommendations:
  * **Google Style**: Clean, readable format with sections for Args, Returns, Raises.
  * **NumPy Style**: Detailed documentation style with standardized sections.
  * **reStructuredText (Sphinx)**: For projects requiring elaborate documentation.
* **Comments**: Explain *why*, not *what* for complex logic. Group related code with section comments.
* **Naming**: PEP 8. `snake_case` (vars/funcs), `PascalCase` (classes), `UPPER_CASE` (consts). Descriptive.
* **Type Hints**: PEP 484/585/604. Always use. Leverage new union types (|) and type guards.
* **Structure**: Logical modules/packages. PEP 8 for formatting. Use `__init__.py` for clear API definitions.
* **Error Handling**: Specific exceptions. No bare `except:`. Add context with `exception groups` (Python 3.11+).
* **Context Managers (`with`)**: Resource management (files, locks, connections). Use `contextlib` when appropriate.
* **Comprehensions/Generators**: Prefer for concise, efficient iteration/creation. Use assignment expressions (:=) judiciously.
* **Package Management**: Use UV for fast, reliable package installation and dependency resolution.
* **Virtual Environments**: Always use (uv, venv, or poetry depending on project complexity).

---

## 📝 In-Code Documentation

* **Function/Method Docstrings**:
  * Document purpose, parameters, return values, and exceptions
  * Follow a consistent style (Google, NumPy, or reStructuredText)
  * Include examples for complex functions
  * Document all public APIs
  * Example (Google style):

    ```python
    def calculate_discount(price: float, discount_rate: float) -> float:
        """Calculate the final price after applying a discount.
        
        Args:
            price: Original price in dollars
            discount_rate: Discount as a decimal between 0 and 1
            
        Returns:
            Discounted price
            
        Raises:
            ValueError: If discount_rate is not between 0 and 1
        """
        if not 0 <= discount_rate <= 1:
            raise ValueError("Discount rate must be between 0 and 1")
        return price * (1 - discount_rate)
    ```

* **Code Comments**:
  * Explain *why*, not *what* for complex logic
  * Use comment blocks to group related code sections
  * Keep comments current when changing code
  * Add references to external resources when relevant
  * Format TODO comments: `# TODO: Brief description [JIRA-123]`

* **Type Annotations**:
  * Use type hints for all function parameters and return values
  * Leverage modern Python 3.12+ typing features
  * Use Union types with the pipe operator: `str | None`
  * Add TypedDict for dictionary structures
  * Use Protocols for duck typing

---

## See Also
- [Python Code Quality Practices](/python/python-code-quality.md)
- [Python Performance](/python/python-performance.md)
