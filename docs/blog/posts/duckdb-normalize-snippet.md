---
date:
  created: 2025-01-14
tags:
  - DuckDB
categories:
  - Tools
links:
- External links:
    - Docs: https://duckdb.org/docs/sql/functions/char.html
---

# A neat duckdb snipped for string normalization

A recent project of mine involved determining duplicate CRM objects across Salesforce and Hubspot. I utilized duckdb for my data processing <!-- more --> and found this neat little text function duckdb provides: `strip_accents(string)`.

It does exactly what it says: Strip accents from a string. Thus `Mühleisen` becomes `Muheisen`.

This feature saved me from manually defining a map of umlaut characters and replacing them in a bunch of places.


```SQL
SELECT
    strip_accents(first_name) as first_name_normalized,
    ...
FROM salesforce.contacts
```

Neat!

[Link to docs](https://duckdb.org/docs/sql/functions/char.html)
