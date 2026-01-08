# pandas-airtable

A pandas wrapper for Airtable that provides seamless integration between pandas DataFrames and Airtable tables.

## Features

- **`pd.read_airtable()`** - Read Airtable tables directly into pandas DataFrames
- **`df.airtable.to_airtable()`** - Write DataFrames to Airtable with append, replace, or upsert modes
- **Automatic schema inference** - Maps pandas dtypes to Airtable field types
- **Robust batching** - Handles Airtable's 10-record batch limit automatically
- **Rate limiting** - Built-in respect for Airtable's 5 QPS limit with exponential backoff
- **Table/field creation** - Optionally create tables and fields on the fly
- **Dry run mode** - Preview changes before executing

## Installation

```bash
# Using uv
uv add pandas-airtable

# Using pip
pip install pandas-airtable
```

## Quick Start

```python
import pandas as pd
import pandas_airtable  # Registers the accessor and pd.read_airtable

# Read from Airtable
df = pd.read_airtable(
    base_id="appXXXXXXXXXXXXXX",
    table_name="Contacts",
    api_key="patXXXXXXXXXXXXXX"
)

# Write to Airtable
df.airtable.to_airtable(
    base_id="appXXXXXXXXXXXXXX",
    table_name="Contacts",
    api_key="patXXXXXXXXXXXXXX"
)
```

## Reading from Airtable

```python
# Basic read
df = pd.read_airtable("appXXX", "TableName", api_key="patXXX")

# Read with view filter
df = pd.read_airtable("appXXX", "TableName", api_key="patXXX", view="Active Records")

# Read with formula filter
df = pd.read_airtable("appXXX", "TableName", api_key="patXXX", formula="{Status} = 'Active'")

# Custom page size (max 100)
df = pd.read_airtable("appXXX", "TableName", api_key="patXXX", page_size=50)
```

### Returned DataFrame Structure

The returned DataFrame includes metadata columns:
- `_airtable_id` - The Airtable record ID
- `_airtable_created_time` - Record creation timestamp

```python
>>> df.columns
Index(['_airtable_id', '_airtable_created_time', 'Name', 'Email', 'Status'], dtype='object')
```

## Writing to Airtable

### Append Mode (Default)

Add new records to an existing table:

```python
df.airtable.to_airtable(
    base_id="appXXX",
    table_name="Contacts",
    api_key="patXXX",
    if_exists="append"  # Default
)
```

### Replace Mode

Delete all existing records and insert new ones:

```python
df.airtable.to_airtable(
    base_id="appXXX",
    table_name="Contacts",
    api_key="patXXX",
    if_exists="replace"
)
```

### Upsert Mode

Update existing records by key field, insert new ones:

```python
df.airtable.to_airtable(
    base_id="appXXX",
    table_name="Contacts",
    api_key="patXXX",
    if_exists="upsert",
    key_field="email"  # Required for upsert
)
```

### Additional Options

```python
result = df.airtable.to_airtable(
    base_id="appXXX",
    table_name="Contacts",
    api_key="patXXX",
    if_exists="append",

    # Table creation
    create_table=True,        # Create table if it doesn't exist (default: True)
    allow_new_columns=False,  # Create missing fields in Airtable (default: False)

    # Batching
    batch_size=10,            # Records per API call (max: 10)

    # Schema override
    schema={
        "Description": {"type": "multilineText"},
        "Tags": {"type": "multipleSelects"}
    },

    # Upsert options
    key_field="email",              # Field for matching records
    allow_duplicate_keys=False,     # Allow duplicate keys (uses last occurrence)

    # Preview mode
    dry_run=False             # If True, returns preview without making changes
)

# Check results
print(f"Created: {result.created_count}")
print(f"Updated: {result.updated_count}")
print(f"Deleted: {result.deleted_count}")
print(f"Success: {result.success}")
```

## Type Mapping

| pandas dtype | Airtable Field Type |
|--------------|---------------------|
| `object`/`str` | Single line text |
| `int64`, `int32` | Number |
| `float64`, `float32` | Number |
| `bool` | Checkbox |
| `datetime64[ns]` | Date |
| `list[str]` | Multiple select |
| `dict` with `url`/`filename` | Attachment |

### Custom Schema Override

```python
df.airtable.to_airtable(
    ...,
    schema={
        "notes": {"type": "multilineText"},
        "priority": {"type": "singleSelect", "options": {"choices": [{"name": "High"}, {"name": "Low"}]}},
        "website": {"type": "url"},
        "contact_email": {"type": "email"}
    }
)
```

## Error Handling

The package provides specific exceptions for different error conditions:

```python
from pandas_airtable import (
    PandasAirtableError,          # Base exception
    AirtableAuthenticationError,   # Invalid API key
    AirtablePermissionError,       # Insufficient permissions
    AirtableRateLimitError,        # Rate limit exceeded after retries
    AirtableSchemaError,           # Schema mismatch
    AirtableValidationError,       # Input validation failure
    AirtableDuplicateKeyError,     # Duplicate keys in upsert mode
    AirtableTableNotFoundError,    # Table doesn't exist
    AirtableFieldCreationError,    # Field creation failed
    AirtableBatchError,            # Partial batch failure
)

try:
    df.airtable.to_airtable("appXXX", "Table", api_key="patXXX")
except AirtableSchemaError as e:
    print(f"Missing columns: {e.missing_columns}")
except AirtableDuplicateKeyError as e:
    print(f"Duplicate values: {e.duplicate_values}")
except AirtableRateLimitError as e:
    print(f"Rate limited after {e.retries} retries")
```

## Dry Run Mode

Preview changes before executing:

```python
preview = df.airtable.to_airtable(
    base_id="appXXX",
    table_name="Contacts",
    api_key="patXXX",
    dry_run=True
)

print(f"Operation: {preview.operation}")
print(f"Records to process: {preview.record_count}")
print(f"Schema: {preview.schema}")
print(f"Sample records: {preview.sample_records[:3]}")
```

## Logging

Enable structured logging for debugging:

```python
import logging

logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger("pandas_airtable")
logger.setLevel(logging.DEBUG)
```

## API Reference

### `pd.read_airtable()`

```python
pd.read_airtable(
    base_id: str,           # Airtable base ID (e.g., "appXXX")
    table_name: str,        # Table name
    api_key: str,           # API key or personal access token
    view: str = None,       # Optional view name
    formula: str = None,    # Optional Airtable formula
    page_size: int = 100,   # Records per page (max 100)
) -> pd.DataFrame
```

### `df.airtable.to_airtable()`

```python
df.airtable.to_airtable(
    base_id: str,                              # Airtable base ID
    table_name: str,                           # Table name
    api_key: str,                              # API key or personal access token
    if_exists: str = "append",                 # "append", "replace", or "upsert"
    key_field: str = None,                     # Key field for upsert
    schema: dict = None,                       # Schema override
    batch_size: int = 10,                      # Records per batch (max 10)
    create_table: bool = True,                 # Create table if missing
    allow_new_columns: bool = False,           # Create missing fields
    allow_duplicate_keys: bool = False,        # Allow duplicate keys in upsert
    dry_run: bool = False,                     # Preview without executing
) -> WriteResult | DryRunResult
```

## Requirements

- Python >= 3.13
- pandas >= 2.3.3
- pyairtable >= 3.3.0

## License

MIT License

## Contributing

Contributions are welcome! Please ensure all tests pass before submitting a PR:

```bash
uv run pytest tests/ -v
uv run ruff check src/ tests/
```
