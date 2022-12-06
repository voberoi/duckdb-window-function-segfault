# duckdb-window-function-segfault

**Update: solved in https://github.com/duckdb/duckdb/issues/5523**

---

A segfault occurs running the `list()` aggregate function as a window function on the Parquet file in this repo.

It seems to be isolated to `list()` being used as an aggregate function in a windowing context. `list()` works in a GROUP BY. Other window functions work on this data as well.

Version: This occurs in 0.6.0 and 0.5.1. 
Platform: M1 Mac OS

Code:

```
❯ duckdb                                                     
v0.6.0 2213f9c946
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
D install parquet;
D load parquet;
D SELECT user_id, list(event_name) OVER (PARTITION BY user_id ORDER BY event_timestamp ASC) FROM 'github-sample.parquet';
duckdb(80879,0x16f5db000) malloc: Incorrect checksum for freed object 0x131819200: probably modified after being freed.
Corrupt value: 0x6873755000000009
duckdb(80879,0x16f5db000) malloc: *** set a breakpoint in malloc_error_break to debug
zsh: abort      duckdb
```

I haven't tried other window functions, but list() does work as an aggregate function in a GROUP BY:

```
v0.6.0 2213f9c946
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
D install parquet;
D load parquet;
D SELECT user_id, list(event_name) FROM 'github-sample.parquet' GROUP BY 1;
D < results omitted >
```

^^ That query works.

These are Parquet files created using DuckDB after loading a bunch of JSON data from gharchive.com and imposing a schema on a subset of that data.
