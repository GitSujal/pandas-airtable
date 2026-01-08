**Build a small, well‑tested pandas wrapper that exposes `pd.read_airtable(...)` and a `DataFrame.airtable.to_airtable(...)` accessor; implement robust pagination, batching, schema mapping, attachment/linked‑record handling, retries with exponential backoff, and clear permission/plan error messages.** 

### Scope and goals
- **Primary features:** `pd.read_airtable(base_id, table_name, api_key, view=None, formula=None, page_size=100)` and `df.airtable.to_airtable(base_id, table_name, api_key, if_exists='append'|'replace'|'upsert', schema=None, batch_size=10, create_table=True)`.  
- **Non‑goals:** UI, webhooks, or background sync; do not assume elevated Airtable plan permissions unless explicitly provided.


### Key Airtable / pyairtable constraints to encode
- **Rate limit:** Airtable enforces **5 QPS per base**; pyairtable will retry 429s by default (configurable) — design to respect this and back off on 429s. Where possible use the retry mechanism built into pyairtable.  
- **Batch limits:** **Max records per batch = 10** (pyairtable constant `MAX_RECORDS_PER_REQUEST`) — enforce when creating/updating/deleting.  
- **API surface:** Use `pyairtable.Api`, `Base`, or `Table` objects; supported methods include `all()`, `get()`, `create()`, `batch_create()`, `batch_update()`, `batch_upsert()`, `delete()`.


### Data model & schema mapping (detailed)
- **Airtable record shape:** `{ "id": "...", "fields": { ... }, "createdTime": "..." }`. Map `fields` → DataFrame columns; preserve `id` as `_airtable_id` column. Normalize nested fields (attachments, arrays) into JSON/dedicated columns. Use `createdTime` → `_airtable_created_time`. Use consistent column naming rules (snake_case). Cite record shape docs for implementer reference.  
- **Type mapping rules (default inference):**
  - `object`/`str` → **Single line text**  
  - `int`/`float` → **Number**  
  - `bool` → **Checkbox**  
  - `datetime64[ns]` → **Date** (ISO 8601)  
  - `list[str]` → **Multiple select** or **Linked records** (user must choose)  
  - `dict` with `url`/`filename` → **Attachment** (upload by URL)  
  Allow explicit `schema` override: `{ "Field Name": {"type":"singleLineText"/"number"/"date"/"attachment"/"linkToAnotherRecord", "options": {...}} }`.
- in to_airtable provide an option to create_table_if_not_exists (default True). If table does not exist, attempt to create it using metadata endpoints if available; otherwise raise informative error.
- provide another option to allow_new_columns (default False). If True, create missing fields in Airtable table as needed; otherwise raise error if schema mismatch. Even when its true it won't drop existing fields, it will only add new fields. 
- If the table exists the default mode will be batch update unless otherwise stated using arguments if_exists. if_exists can take values 'append', 'replace', 'upsert'. 'append' will add new records, 'replace' will delete all existing records and add new ones, 'upsert' will update existing records based on a key field and add new ones.

### Read implementation (step‑by‑step)
1. **API init:** `api = Api(api_key)`; `table = api.table(base_id, table_name)`.  
2. **Pagination:** call `table.all(view=view, formula=formula, page_size=page_size)`; iterate generator, collect `record['fields']` and `id` into list of dicts; normalize attachments into list of `{url,name}`; convert to DataFrame and coerce dtypes. Respect pyairtable retry behavior for 429s.  
3. **Return:** DataFrame with `_airtable_id` and `_airtable_created_time` columns.
4. Use can either specify a formula or view to filter records returned. If both are not specifed all records are returned.


### Write implementation (step‑by‑step)
1. **Preflight:** validate `api_key`, `base_id`. If `create_table=True`, attempt to create table via metadata endpoints if available (use `api.urls` meta endpoints to check permissions) and fall back to informative error if not permitted.  
2. **Schema sync:** compare DataFrame columns to Airtable fields; if missing fields and API allows, create fields (call metadata endpoints or use `Table` helper if pyairtable exposes field creation). If not allowed, either **fail** or **add columns as JSON** depending on `if_exists`.  
Use the behavior specified in the previous section regarding allow_new_columns.

3. **Prepare records:** map DataFrame rows to Airtable `fields` dicts. For **attachments**, convert to `{"url": ...}` objects; for **linked records**, accept either record IDs or objects with `create_if_missing` flag.  

4. **Batching & upsert:** chunk into `batch_size` ≤ `MAX_RECORDS_PER_REQUEST` (10). Use `table.batch_create()` for new records, `table.batch_upsert()` for upserts keyed by a user‑specified key field. Implement **exponential backoff** on 429/5xx with jitter; rely on pyairtable retry but add outer retry loop for long jobs.  While doing upsert if there are duplicates on the key field within the dataframe, raise an error before making any API calls. This behavior can be overridden by an argument allow_duplicate_keys (default False). If set to True, the last occurrence of the duplicate key in the dataframe will be used for upsert and the other occurrences will be ignored with a warning.

5. **Idempotency & conflict handling:** return mapping of DataFrame index → Airtable record id; on partial failures, provide a structured error report with failed rows and API error messages.


### Edge cases & operational concerns 
- **Linked records:** if values are not record IDs, optionally create referenced records first and map to IDs.  
- **Schema drift:** log and surface schema changes; provide `dry_run=True` to preview changes.  
- **Testing:** unit tests should mock `pyairtable.Api.table()` and simulate pagination, 429s, and batch responses; integration tests against a sandbox base recommended.  
- **Observability:** emit structured logs for batches, retries, and API responses.


### Deliverables for an LLM
- **Function signatures** (as above).  
- **Concrete code templates**: accessor registration, `read_airtable` implementation, `to_airtable` implementation with batching loop, retry/backoff helper, schema inference module, and test stubs.  
- **Test vectors:** small dataset with attachments, linked records, numeric/date types, and a 25‑row dataset to validate batching.  
- **Error messages and user‑facing docs** for permission/plan failures and rate‑limit guidance.