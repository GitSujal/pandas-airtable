# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

```bash
# Install dependencies
uv sync --all-extras

# Run unit tests (no credentials needed)
uv run pytest tests/ --ignore=tests/integration/ -v

# Run single test file
uv run pytest tests/test_batch.py -v

# Run single test
uv run pytest tests/test_batch.py::TestChunkRecords::test_chunk_records_exact_batch -v

# Run with coverage
uv run pytest tests/ --ignore=tests/integration/ --cov=pandas_airtable --cov-report=term-missing

# Run integration tests (requires AIRTABLE_API_KEY and either AIRTABLE_BASE_ID or AIRTABLE_BASE_NAME)
uv run pytest tests/integration/ -v

# Lint
uv run ruff check src/ tests/

# Auto-fix lint errors
uv run ruff check src/ tests/ --fix

# Build package
uv build
```

## Architecture

### Public API

The package exposes two main interfaces:
- `pd.read_airtable(base_id, table_name, api_key, ...)` - Attached to pandas namespace in `__init__.py`
- `df.airtable.to_airtable(base_id, table_name, api_key, ...)` - DataFrame accessor registered in `accessor.py`

Both functions accept either `base_id` or `base_name` (not both). If multiple bases share the same name, the first one found is used.

### Module Responsibilities

| Module | Purpose |
|--------|---------|
| `accessor.py` | `AirtableAccessor` class - orchestrates writes, validates inputs, manages schema sync |
| `reader.py` | `read_airtable()` - handles pagination, type coercion, metadata columns |
| `batch.py` | Batch operations (create/update/upsert/delete) with 10-record chunking |
| `schema.py` | Infers Airtable field types from pandas dtypes, converts values |
| `retry.py` | `retry_with_backoff()` and `RateLimitedExecutor` for 5 QPS limit |
| `types.py` | Constants (`RATE_LIMIT_QPS=5`, `MAX_RECORDS_PER_REQUEST=10`), dataclasses |
| `utils.py` | `get_base_id_from_name()` - resolves base name to base ID |
| `exceptions.py` | 9 exception classes inheriting from `PandasAirtableError` |

### Data Flow

**Read**: `read_airtable()` → pyairtable `table.all()` → normalize attachments/links → coerce dtypes → add metadata columns (`_airtable_id`, `_airtable_created_time`)

**Write**: `to_airtable()` → validate inputs → check duplicates (upsert) → infer schema → ensure table exists → sync schema → prepare records → execute batches with retry

### Key Constraints (Airtable API)

- **Batch size**: 10 records max per API call
- **Rate limit**: 5 requests per second per base
- **Page size**: 100 records max for reads

### Integration Tests

Integration tests run against real Airtable API and:
- Create tables with `test_` prefix
- Clean up all `test_*` tables before and after test session
- Skip automatically if credentials not set

Set credentials:
```bash
export AIRTABLE_API_KEY="patXXX"
# Use either base_id OR base_name
export AIRTABLE_BASE_ID="appXXX"
# OR
export AIRTABLE_BASE_NAME="My Test Base"
```
