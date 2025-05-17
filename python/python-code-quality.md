# GitHub Copilot: Python Code Quality Practices

Guidelines for maintaining high code quality in Python 3.12+ projects, including structure, testing, and static analysis.

**Keywords**: #python #code-quality #testing #linting #static-analysis

---

## 🛠️ Code Quality Practices

* **Testable Code Writing**:
  * Write small, focused functions that are easy to test
  * Use dependency injection to make components testable
  * Keep side effects isolated for easier mocking
  * Design for testability from the start
  * Create factory functions for complex test objects

* **Code Structure**:
  * Organize related functionality into modules
  * Keep files reasonably sized (< 500 lines as a guideline)
  * Follow logical import order (stdlib, third-party, local)
  * Use clear directory structure for packages
  * Create meaningful `__init__.py` files

* **Static Analysis During Coding**:
  * Configure your editor to use Ruff or Flake8
  * Apply Black formatting automatically on save
  * Use MyPy for type checking during development
  * Enable import sorting with isort
  * Fix linting issues as you code, not after

* **Dependency Management**:
  * Use UV for dependency installation
  * Keep dependencies explicit in pyproject.toml
  * Specify version constraints appropriately
  * Keep your dependencies minimal
  * Regularly update dependencies in development

---

## See Also
- [Python Core Principles](/python/python-core-principles.md)
- [Python Performance](/python/python-performance.md)
