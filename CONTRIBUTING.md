# Contributing to pandas-airtable

Thank you for your interest in contributing to pandas-airtable! This document provides guidelines and instructions for contributing.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Setup](#development-setup)
- [Making Changes](#making-changes)
- [Testing](#testing)
- [Submitting Changes](#submitting-changes)
- [Style Guide](#style-guide)
- [Reporting Issues](#reporting-issues)

## Code of Conduct

This project adheres to a [Code of Conduct](CODE_OF_CONDUCT.md). By participating, you are expected to uphold this code. Please report unacceptable behavior to the project maintainers.

## Getting Started

1. Fork the repository on GitHub
2. Clone your fork locally
3. Set up the development environment
4. Create a branch for your changes
5. Make your changes
6. Submit a pull request

## Development Setup

### Prerequisites

- Python 3.13 or higher
- [uv](https://github.com/astral-sh/uv) package manager (recommended)
- Git

### Setting Up Your Environment

```bash
# Clone your fork
git clone https://github.com/YOUR_USERNAME/pandas-airtable.git
cd pandas-airtable

# Install dependencies with uv
uv sync

# Or with pip
pip install -e ".[dev]"
```

### Environment Variables for Testing

For integration tests, you'll need Airtable credentials:

```bash
# Create a .env file (git-ignored)
echo 'AIRTABLE_API_KEY="patXXXXXXXXXXXXXX"' >> .env
echo 'AIRTABLE_BASE_ID="appXXXXXXXXXXXXXX"' >> .env
```

## Making Changes

### Branch Naming

Use descriptive branch names:

- `feature/add-new-field-type` - For new features
- `fix/rate-limit-handling` - For bug fixes
- `docs/update-readme` - For documentation
- `refactor/batch-processing` - For code refactoring

### Commit Messages

Write clear, concise commit messages:

```
Add support for multilineText field type

- Add multilineText to AIRTABLE_FIELD_TYPES
- Update schema inference for long strings
- Add tests for multilineText handling
```

Follow these conventions:
- Use present tense ("Add feature" not "Added feature")
- Use imperative mood ("Move cursor to..." not "Moves cursor to...")
- Limit the first line to 72 characters
- Reference issues and PRs in the body when relevant

## Testing

### Running Tests

```bash
# Run all unit tests (no credentials needed)
uv run pytest tests/ --ignore=tests/integration/ -v

# Run with coverage
uv run pytest tests/ --ignore=tests/integration/ --cov=pandas_airtable --cov-report=html

# Run integration tests (requires credentials)
uv run pytest tests/integration/ -v

# Run all tests
uv run pytest tests/ -v

# Run specific test file
uv run pytest tests/test_batch.py -v

# Run specific test
uv run pytest tests/test_batch.py::TestChunkRecords::test_chunk_records_exact_batch -v
```

### Writing Tests

- Write tests for all new functionality
- Maintain or improve code coverage
- Use descriptive test names that explain what is being tested
- Mock external API calls in unit tests
- Integration tests should clean up after themselves

Example test structure:

```python
class TestFeatureName:
    """Test description."""

    def test_specific_behavior(self):
        """Test that specific behavior works as expected."""
        # Arrange
        input_data = ...

        # Act
        result = function_under_test(input_data)

        # Assert
        assert result == expected_value
```

## Submitting Changes

### Before Submitting

1. **Run linting:**
   ```bash
   uv run ruff check src/ tests/
   ```

2. **Fix any lint errors:**
   ```bash
   uv run ruff check src/ tests/ --fix
   ```

3. **Run all tests:**
   ```bash
   uv run pytest tests/ --ignore=tests/integration/ -v
   ```

4. **Update documentation** if needed

### Pull Request Process

1. Update the README.md with details of changes if applicable
2. Update the CHANGELOG.md with your changes under "Unreleased"
3. Ensure all tests pass
4. Request review from maintainers

### Pull Request Template

When creating a PR, include:

- **Summary**: Brief description of changes
- **Motivation**: Why this change is needed
- **Testing**: How you tested the changes
- **Checklist**:
  - [ ] Tests pass locally
  - [ ] Lint checks pass
  - [ ] Documentation updated (if applicable)
  - [ ] CHANGELOG.md updated

## Style Guide

### Python Style

We use [Ruff](https://github.com/astral-sh/ruff) for linting and formatting. Key points:

- Follow PEP 8
- Use type hints for function signatures
- Maximum line length: 88 characters (Black default)
- Use double quotes for strings
- Use trailing commas in multi-line structures

### Code Organization

```python
# Standard library imports
from __future__ import annotations

import logging
from typing import Any

# Third-party imports
import pandas as pd
from pyairtable import Api

# Local imports
from pandas_airtable.exceptions import AirtableError
from pandas_airtable.types import WriteResult
```

### Documentation

- Use docstrings for all public functions and classes
- Follow Google-style docstrings
- Include type hints in signatures

```python
def process_records(
    records: list[dict[str, Any]],
    batch_size: int = 10,
) -> WriteResult:
    """Process records in batches.

    Args:
        records: List of record dictionaries to process.
        batch_size: Number of records per batch (max 10).

    Returns:
        WriteResult containing operation statistics.

    Raises:
        AirtableValidationError: If records are invalid.
    """
```

## Reporting Issues

### Bug Reports

When reporting bugs, include:

1. **Environment**: Python version, OS, package versions
2. **Steps to reproduce**: Minimal code example
3. **Expected behavior**: What should happen
4. **Actual behavior**: What actually happens
5. **Error messages**: Full traceback if applicable

### Feature Requests

For feature requests, include:

1. **Use case**: Why you need this feature
2. **Proposed solution**: How it might work
3. **Alternatives**: Other approaches you've considered

## Questions?

Feel free to open an issue for any questions about contributing. We're happy to help!

## License

By contributing to pandas-airtable, you agree that your contributions will be licensed under the MIT License.
