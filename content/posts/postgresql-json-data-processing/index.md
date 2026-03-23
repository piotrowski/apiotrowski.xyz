---
title: "Unleashing the Power of PostgreSQL: Processing JSON Data Like a Pro!"
date: 2021-10-31
categories: ["Database"]
---

![PostgreSQL JSON logo](logo.jpg)

Learn how to load, process, and insert data into a PostgreSQL database using JSON data.

In this article, I'll walk you through preparing and processing JSON data with PostgreSQL.

<!--more-->

## Inserting Data from JSON to PostgreSQL

I've compiled a file with one object per line:

```json
{"name": "Scrambled tofu-egg", "categories": ["BREAKFAST"], "ingredients": ["carrot", "potato", "avocado"]}
{"name": "Pancakes", "categories": ["SUPPER"], "ingredients": ["carrot", "potato"]}
{"name": "Meatballs", "categories": ["DINNER"], "ingredients": ["avocado"]}
```

With three simple commands in a transaction, you can insert this data:

```sql
BEGIN;

CREATE TABLE recipes (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    user_id bigint,
    name TEXT NOT NULL,
    categories TEXT[],
    ingredients TEXT[]
);

-- Remove the table on COMMIT
CREATE TEMPORARY TABLE temp_import (doc JSON) ON COMMIT DROP;

-- Copy the JSON file to our temporary table
COPY temp_import
FROM
    '/migrations/000003_bootstrap_recipes.json';

-- Copy data from the temporary table to the final one
INSERT INTO
    recipes (name, categories, ingredients)
SELECT
    p.name,
    p.categories,
    p.ingredients
FROM
    temp_import l
    CROSS JOIN LATERAL json_populate_record(NULL :: temp_recipes, doc) AS p;

COMMIT;
```

As you can see, the process is straightforward. However, challenges arise when processing data. In my case I needed to map categories and ingredients from user friendly string versions to IDs.

## Utilizing PostgreSQL Functions

I have two tables with `ingredient - ID` and `category - ID` pairs:

```sql
CREATE TABLE categories (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    category TEXT NOT NULL
);

INSERT INTO
    categories (category)
VALUES
    ('UNKNOWN'),
    ('BREAKFAST'),
    ('LUNCH'),
    ('BRANCH'),
    ('DINNER'),
    ('SUPPER'),
    ('DESERT');

-----------------------------

CREATE TABLE ingredients (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    ingredient TEXT NOT NULL
);

INSERT INTO
    ingredients (ingredient)
VALUES
    ('carrot'),
    ('potato'),
    ('avocado');
```

To address this, we use PostgreSQL functions to map categories and ingredients from user-friendly strings to IDs.

```sql
CREATE FUNCTION map_ingredient_names_to_ids(ingredient_names TEXT[]) RETURNS BIGINT[] AS $$
    declare
        ingredient_ids BIGINT [];
        ingredient_name TEXT;
        ingredient_id BIGINT;
    BEGIN
        FOREACH ingredient_name IN array ingredient_names LOOP
            ingredient_id = (SELECT id from ingredients WHERE ingredient = ingredient_name);
            -- Check for typos in ingredients. If name do not match to any from db, raise exception
            IF ingredient_id IS NULL THEN
                RAISE EXCEPTION 'Nonexistent ingredient %', ingredient_name;
            END IF;
            -- || is a postgres syntax for appending data to an array
            ingredient_ids = ingredient_ids || ingredient_id;
        END LOOP;
        RETURN ingredient_ids;
    END;
$$ LANGUAGE plpgsql;
```

We follow a similar process for mapping categories.

## Everything together

```sql
BEGIN;

-- Define the table structure for our recipes
-- Notice that I am using BIGINT, no TEXT anymore
CREATE TABLE recipes (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    user_id bigint,
    name TEXT NOT NULL,
    categories bigint [],
    ingredients bigint []
);

-- Import the recipes from JSON to the database
CREATE FUNCTION map_category_names_to_ids(category_names TEXT[]) RETURNS BIGINT[] AS $$
    declare
        category_ids BIGINT [];
        category_name TEXT;
        category_id BIGINT;
    BEGIN
        FOREACH category_name IN array category_names LOOP
            category_id = (SELECT id from categories WHERE category = category_name);
            IF category_id IS NULL THEN
                RAISE EXCEPTION 'Nonexistent category %', category_name;
            END IF;
            category_ids = category_ids || category_id;
        END LOOP;
        RETURN category_ids;
    END;
$$ LANGUAGE plpgsql;

CREATE FUNCTION map_ingredient_names_to_ids(ingredient_names TEXT[]) RETURNS BIGINT[] AS $$
    declare
        ingredient_ids BIGINT [];
        ingredient_name TEXT;
        ingredient_id BIGINT;
    BEGIN
        FOREACH ingredient_name IN array ingredient_names LOOP
            ingredient_id = (SELECT id from ingredients WHERE ingredient = ingredient_name);
            IF ingredient_id IS NULL THEN
                RAISE EXCEPTION 'Nonexistent ingredient %', ingredient_name;
            END IF;
            ingredient_ids = ingredient_ids || ingredient_id;
        END LOOP;
        RETURN ingredient_ids;
    END;
$$ LANGUAGE plpgsql;

-- Temporary tables for importing and processing data
CREATE TEMPORARY TABLE temp_import (doc JSON) ON COMMIT DROP;

COPY temp_import
FROM
    '/migrations/000003_bootstrap_recipes.json';

CREATE TEMPORARY TABLE temp_recipes (
    name TEXT NOT NULL,
    categories TEXT [],
    ingredients TEXT []
) ON COMMIT DROP;

-- Insert data into the final recipes table, mapping categories and ingredients to their IDs
INSERT INTO
    recipes (name, categories, ingredients)
SELECT
    p.name,
    map_category_names_to_ids(p.categories),
    map_ingredient_names_to_ids(p.ingredients)
FROM
    temp_import l
    CROSS JOIN lateral json_populate_record(NULL :: temp_recipes, doc) AS p;

COMMIT;
```

## Conclusion

While loading JSON data into PostgreSQL is straightforward, processing it presents challenges. Nevertheless, with the right approach and PostgreSQL functions, you can navigate through these challenges and achieve seamless data integration.

### Further Reading

- [How to import data from JSON and use postgres ARRAY with Golang](https://tech.ingrid.com/postgres-til-datatypes-array-json/)
- [https://www.postgresql.org/docs/9.5/sql-createfunction.html](https://www.postgresql.org/docs/9.5/sql-createfunction.html)
