# pandas-airtable Development Guide

## Project Overview

This is a pandas wrapper for Airtable that provides:
- `pd.read_airtable()` - Read Airtable tables into DataFrames
- `df.airtable.to_airtable()` - Write DataFrames to Airtable tables

## Development Environment

Always use **uv** for package management:
```bash
uv sync                    # Install dependencies
uv run pytest tests/ -v    # Run tests
uv run ruff check src/     # Lint code
```

## Project Structure

```
src/pandas_airtable/
├── __init__.py        # Public API, accessor registration, pd.read_airtable attachment
├── accessor.py        # AirtableAccessor class with to_airtable() method
├── batch.py           # Batch operations (create, update, upsert, delete)
├── exceptions.py      # Custom exception hierarchy
├── reader.py          # read_airtable() function implementation
├── retry.py           # Exponential backoff and RateLimitedExecutor
├── schema.py          # Schema inference and type mapping
├── types.py           # Constants, dataclasses (WriteResult, DryRunResult, etc.)
├── utils.py           # Logging utilities, chunk_records, to_snake_case
└── py.typed           # PEP 561 marker for type hints

tests/
├── conftest.py        # Shared pytest fixtures
├── test_accessor.py   # Tests for to_airtable() accessor
├── test_batch.py      # Tests for batch operations
├── test_reader.py     # Tests for read_airtable()
├── test_retry.py      # Tests for retry/rate limiting logic
└── test_schema.py     # Tests for schema inference and conversion
```

## Key Airtable Constraints

These are enforced by the Airtable API and must be respected:

| Constraint | Value | Handled In |
|------------|-------|------------|
| Batch size limit | 10 records per request | `batch.py`, `types.py` |
| Page size limit | 100 records per request | `reader.py`, `types.py` |
| Rate limit | 5 QPS per base | `retry.py` |
| Record ID format | `rec` + 14 alphanumeric chars | - |
| Timestamp format | ISO 8601 UTC | `schema.py` |

## Testing Guidelines

- All tests use **pytest** and are located in `tests/`
- External API calls must be mocked using `unittest.mock.patch`
- Use fixtures from `conftest.py` for common test data
- Run tests: `uv run pytest tests/ -v`

### Test Coverage Targets

- reader.py: Test pagination, view/formula filters, type coercion
- accessor.py: Test all if_exists modes, validation, schema handling
- batch.py: Test chunking, partial failures, all batch operations
- schema.py: Test type inference for all pandas dtypes
- retry.py: Test exponential backoff, rate limiting

## Code Style

- Use **ruff** for linting: `uv run ruff check src/ tests/`
- Follow PEP 8 and Python best practices
- Use type hints for all function signatures
- Use Google-style docstrings

## Exception Hierarchy

```
PandasAirtableError (base)
├── AirtableAuthenticationError   # Invalid/missing API key
├── AirtablePermissionError       # Insufficient permissions
├── AirtableRateLimitError        # Rate limit after retries
├── AirtableSchemaError           # Schema mismatch
├── AirtableValidationError       # Input validation failure
├── AirtableBatchError            # Partial batch failure
├── AirtableDuplicateKeyError     # Duplicate keys in upsert
├── AirtableTableNotFoundError    # Table doesn't exist
└── AirtableFieldCreationError    # Field creation failed
```

## Type Mapping (pandas → Airtable)

| pandas dtype | Airtable type | Notes |
|--------------|---------------|-------|
| `object`/`str` | `singleLineText` | Default for strings |
| `int64`, `int32` | `number` | Integer precision |
| `float64` | `number` | With precision option |
| `bool` | `checkbox` | True/False |
| `datetime64[ns]` | `date` | ISO 8601 format |
| `list[str]` | `multipleSelects` | Array of strings |
| `dict` with url | `multipleAttachments` | Attachment objects |

## Key Implementation Details

### Metadata Columns
- `_airtable_id`: Record ID preserved from reads, used for updates
- `_airtable_created_time`: Record creation timestamp

### Batch Processing
Records are automatically chunked into batches of 10 (max allowed by Airtable API).
Each batch is processed with retry logic for rate limiting.

### Schema Sync Behavior
- `allow_new_columns=False`: Raises error if DataFrame has columns not in Airtable
- `allow_new_columns=True`: Creates missing fields (requires metadata API permissions)
- Existing fields are never dropped

### Rate Limiting
- pyairtable handles 429 responses with built-in retry
- Additional outer retry loop for long-running jobs
- `RateLimitedExecutor` tracks request times in sliding window

## Common Development Tasks

### Adding a New Feature
1. Add implementation in appropriate module
2. Add tests in corresponding test file
3. Update `__init__.py` if exposing new public API
4. Run `uv run pytest tests/ -v` and `uv run ruff check src/`

### Debugging API Issues
Enable debug logging:
```python
import logging
logging.getLogger("pandas_airtable").setLevel(logging.DEBUG)
```

### Testing Against Real Airtable
Set environment variables for integration tests:
```bash
export AIRTABLE_API_KEY="patXXX"
export AIRTABLE_TEST_BASE_ID="appXXX"
```

## Dependencies

- `pandas>=2.3.3` - DataFrame operations
- `pyairtable>=3.3.0` - Airtable API client

Dev dependencies:
- `pytest>=9.0.2` - Testing
- `pytest-cov>=7.0.0` - Coverage
- `ruff>=0.14.10` - Linting
