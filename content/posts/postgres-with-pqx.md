---
title: "PGX Error: Cannot scan unknown type"
date: 2024-05-11
categories: ["Go", "Database"]
---

You probably end up here because you saw this error returned from `jackc/pgx`
```
cannot scan unknown type (OID 16458) in text format into *[]recipe.CategoryType
```

This error is connected with custom database types, so if this is what you did, this article is for you.

<!--more-->

I have this type in my db:

```sql
CREATE TYPE category_type AS ENUM (
  'easy',
  'medium',
  'hard'
);
```

## Fix for pgx.Conn

When you are only using connection try below code after you connect

```go
t, err := conn.LoadType(ctx, "category_type")
if err != nil {
	return fmt.Errorf("could not load type: %w", err)
}
conn.TypeMap().RegisterType(t)

t, err = conn.LoadType(ctx, "_category_type")
if err != nil {
	return fmt.Errorf("could not load type: %w", err)
}
conn.TypeMap().RegisterType(t)
```

## Fix for pgxpool.Pool

When you are using connection pool fix is similar. It is impossible to load type into connection pool. You need to use custom config for your pgxpool.

```go
config, err := pgxpool.ParseConfig(uri)
config.AfterConnect = func(ctx context.Context, conn *pgx.Conn) error {
	t, err := conn.LoadType(ctx, "category_type")
	if err != nil {
		return fmt.Errorf("could not load type: %w", err)
	}
	conn.TypeMap().RegisterType(t)

	t, err = conn.LoadType(ctx, "_category_type")
	if err != nil {
		return fmt.Errorf("could not load type: %w", err)
	}
	conn.TypeMap().RegisterType(t)

	return nil
}
if err != nil {
	return nil, fmt.Errorf("could not parse config: %w", err)
}

pool, err := pgxpool.NewWithConfig(context.Background(), config)
if err != nil {
	return nil, fmt.Errorf("could not create pool: %w", err)
}
```

---

Hope you found this article before your 30 min google about this issue :D
