# Fivetran dbt Style Guidelines
- [General Guidelines](#general-guidelines)
- [SQL Style Guidelines](#sql-style-guidelines)
    - [Formatting](#formatting)
    - [CTEs](#ctes)
    - [Joins](#joins)
    - [Naming](#naming)
- [References](#references)

# General Guidelines
- **Optimize for maintainability, robustness and computational efficiency**<br>
Think of future you and any other developer that has to maintain your code in the future, do not optimize for less lines of code if you are sacrificing the ease of understanding your code. Newlines are cheap, time is expensive.

- **Lowercase code**<br>
This is the general rule for all code including CTE names, aggregation functions, query clauses (e.g. select, from, where, group by, etc).

- **Use CTEs over subqueries**

- **Try to break down complicated select statements into digestible CTEs**

# SQL Style Guidelines

## Formatting
- **Tabbing is preferred. One tab is 4 spaces** (??)

- **Queries should be no more than 150(??) characters wide**

- **Use leading commas** (??) - since we use leading commas for jinja notation

## General SQL Syntax

- **Use cross database compatible syntax**
    - Use `coalesce` instead of `iffnull` or `nvl`
    - Use `case when` instead of `iff` or `if`
    - Use `column is null` and `column is not null` rather than `isnull` functions

- **Use the most performant approach**
    - Use `union all` instead of `union` unless de-duping is necessary
    - Use `where` instead of `having` when you have the option
    - Avoid `order by` in models

- **`select` statement**
    - Start columns list on next line after `select` statement
    - Using `case when`, line up `case` with `end` statements and allow each `when` condition a line of its own
```sql
/* Best Practice */
select 
    id as order_id,
    case 
        when order_status = 0 then 'canceled'
        when order_status = 1 then 'completed'
        else 'n/a'
    end as order_status
from orders
```

- **Operators**
    - Use `!=` instead of `<>` (??)
    - Start operators (`and`, `or`, `||`, etc) on next line 
```sql
/* Best Practice */
select *
from orders
where order_date >= '2021-01-01'
    and order_date <= '2021-03-31'

/* Anti-pattern */
select *
from orders
where order_date >= '2021-01-01' and 
    order_date <= '2021-03-31'
```

- **Use "not" for BOOLEAN statements**
```sql
/* Best Practice */
select *
from orders
where not is_deleted

/* Anti-pattern */
select * 
from orders
where is_deleted = false
```

- **Aliases**
    - Always explicitly use 'as' when defining aliases for columns, aggregates and tables
    - Never use reserved words as aliases
```sql
/* Best Practice */
/* Example 1 */
select id as order_id
from orders

/* Example 2 */
select max(date) as most_recent_date
from orders

/* Anti-pattern */
select id order_id
from orders
```

## CTEs

- **Line up closing `)` with `with` statement**
- **Newlines**
    - Add newline after closing `)` or before declaring the next CTE
    - Add newline after starting `(`
- Start `on`, and if applicable `and`, statements on next line
- All `ref`s and `var`s should be referenced at the top of the model's file
```sql
/* Best Practice */
with orders as (

    select *
    from {{ ref('int_orders') }}
), 

customers as (
    
    select * 
    from {{ vars('customers') }}
)

, joined as (
    
    select *
    from orders
    left join customers 
        on orders.customer_id = customers.id
)

select *
from joined
```
## Joins

- **No need to declare table aliases** unless joining to the same table multiple times
- **Always prefix column name with table name/alias when selecting from a query that joins additional tables**
- **Use "join on" as opposed to "join using"**
```sql
/* Best Practice */
select 
    orders.id as order_id,
    customers.id as customer_id,
    customers.created_at as customer_created_at
from orders
left join customers 
    on orders.customer_id = customers.id
    and orders.date = customers.created_at

/* Anti-pattern */
select 
    id as order_id,
    customer_id,
    created_at as customer_created_at
from orders
left join customers using(customer_id, created_at)
```

## Naming
- **Booleans should follow prefix conventions like below**
    - `is_` (e.g. `is_deleted`)
    - `has_` (e.g. `has_orders`)

- **Timestamps/Dates**
    - `date_day` for report dates
    - `<event>_at` for timestamps (e.g. `created_at`, `deleted_at`)

- **Model types**
    - `stg_*` for `source` package models
    - `int_*` for `modeling` package models that require intermediate transformation before final model
        - Please put all intermediate models in an `intermediate` directory within `modeling` package `models` directory
    - `<package>__<model_name>` for final `modeling` package model 

## References
- [Brooklyn Data SQL style guide](https://github.com/brooklyn-data/co/blob/main/sql_style_guide.md)
- [dbt-labs coding conventions](https://github.com/dbt-labs/corp/blob/b5c6f55b9e7594e1a1e562edf2378b6dd78a1119/dbt_coding_conventions.md)