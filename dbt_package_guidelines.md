# Fivetran dbt Package Style Guidelines
- [YML Guidelines](#yml-style-guidelines)
- [dbt project Variables](#variables)
- [Jinja Formatting](#jinja-formatting)

## YML Guidelines

### Formatting
- Indents should be 2 spaces
- Add newline between models
- Include column tests within their respective sections
- Use identifiers

### YMLs
- **Rule of thumb for freshness**<br>
For tables that generally do not refresh often, no freshness specification is required but specifications can definitely be adjusted to longer periods such as monthly.<br><br>
For tables with frequent refreshes (daily or multiple times a day) we recommend using the below:
```yml
    freshness:
      warn_after: {count: 24, period: hour}
      error_after: {count: 48, period: hour}
```
- **Use identifiers for table names, schema and database**
```yml
# source.yml
version: 2

sources:
  - name: google_play
    schema: "{{ var('google_play_schema', 'google_play') }}"
    database: "{% if target.type != 'spark'%}{{ var('google_play_database', target.database) }}{% endif %}"
    loader: Fivetran
    loaded_at_field: _fivetran_synced

    tables:
      - name: stats_installs_app_version
        identifier: "{{ var('stats_installs_app_version_identifier', 'stats_installs_app_version') }}"
```

**Example YML**
```yml
# stg.yml & final.yml
models:
  - name: orders
    description: This table tracks all orders by day
    tests: # include all tests that span multiple columns here for example...
      - dbt_utils.unique_combination_of_columns:
          combination_of_columns:
            - column1
            - column2
    columns:
      - name: id
        description: ID of order
      - name: reported_at
        description: Date of order report
        tests: # Include any tests for specific columns within respective sections
          - not_null
    
```
 
## Variables
- **Declaring variables that can be enabled/disabled in `dbt_project.yml`**
    - Variable naming convention: <package_name>_\_using\_<feature_name><br> 
    (e.g. `apple_search_ads__using_search_terms`)
    - These variables should also be included in `.circleci/config.yml` dbt run tests as well
    - Add below config to corresponding models
```yml
{{ 
    config(
        enabled=var('apple_search_ads__using_search_terms', True)
        ) 
}}
```

## Jinja Formatting
- include a space
    - after opening `{{` and before closing `}}` for `ref` and `var`
    - after opening `{%` and before closing `%}` for `if`/`for` statements
- Try to align `if`/`for` statements so that the resulting jinja will be seamlessly added during compile
    - Note: There will be exceptions for `for` loops inside of `if` statements
- Use leading commas for `if`/`for` column selections
- Ensure you do not a include whitespace buffer when using jinja operations (`for`,`if`, `set`, `execute`, etc.)
  - However, there is an exception if a comment is used to introduce the jinja. This shows the logic is important/complex enough that we want to draw attention to it.
```sql
/* Best Practice */
/* Example 1 */
select *
from {{ ref('orders') }}

/* Example 2 */
select 
    *

    -- We are iterating through the data columns in order for this to be a more dynamic operation.
    {% for col in data_columns if col.name|lower not in ['col1', 'col2'] %}
    , {{ col.name }}
    {% endfor %}

from orders

/* Example 3 */
select *
from orders
{% if 'order_id' in var('app_store__using_subscriptions') %}
left join subscriptions
    on orders.subscription_id = subscriptions.id
{% endif %}

/* Anti-pattern */
/* Example 1 */
select 
    *
    -- We are iterating through the data columns in order for this to be a more dynamic operation.
    {% for col in data_columns if col.name|lower not in ['col1', 'col2'] %}
        , {{ col.name }}
    {% endfor %}
from orders

/* Example 2 */
select *
from orders
{% if 'order_id' in var('app_store__using_subscriptions') %}
    left join subscriptions
        on orders.subscription_id = subscriptions.id
{% endif %}
```

## References
- [dbt-labs coding conventions](https://github.com/dbt-labs/corp/blob/b5c6f55b9e7594e1a1e562edf2378b6dd78a1119/dbt_coding_conventions.md)