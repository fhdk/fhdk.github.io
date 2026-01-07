---
title: Contraints
date: '11:19 09-03-2023'
taxonomy:
    category:
        - docs
---


- Unique
- Check
- Default
- Primary key 
- Foreign key
- Not null

basic syntax
```
ALTER TABLE <table_name> ADD CONSTRAINT <constraint_name> TYPE;
```
sample unique constraint
```
ALTER TABLE <table_name> ADD CONSTRAINT <constraint_name> UNIQUE(<id_column>);
```
Validation
```
ALTER TABLE <table_name> ADD CONSTRAINT >constraint_name> CHECK(column_value > 0);
```
Drop constraint
```
ALTER TABLE <table_name> DROP CONSTRAINT <constraint_name>;
```
Drop constraint only if exists
```
ALTER TABLE <table_name> DROP CONSTRAINT IF EXISTS <constraint_name>;
```