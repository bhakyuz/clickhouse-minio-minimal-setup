## Cannot read from S3 path containing white space

Notice that very same file is duplicated in s3 bucket:
- `test/sample.csv`
- `test/folder with whitespace/sample.csv`

I'm expecting to be able to read the data correctly independently of its path. But unfortunately, clickhouse always returns an error when there is a white space within the path. 

```sql
SELECT *
FROM s3('http://minio:9001/test/sample.csv', 'minio', 'minio123', 'CSV')

Query id: a6575024-8932-4e3d-a677-48bd06ea1841

    ┌───────Date─┬────Open─┬────High─┬─────Low─┬───Close─┬───Volume─┬─OpenInt─┐
 1. │ 1984-09-07 │ 0.42388 │ 0.42902 │ 0.41874 │ 0.42388 │ 23220030 │       0 │
 2. │ 1984-09-10 │ 0.42388 │ 0.42516 │ 0.41366 │ 0.42134 │ 18022532 │       0 │
 3. │ 1984-09-11 │ 0.42516 │ 0.43668 │ 0.42516 │ 0.42902 │ 42498199 │       0 │
 4. │ 1984-09-12 │ 0.42902 │ 0.43157 │ 0.41618 │ 0.41618 │ 37125801 │       0 │
 5. │ 1984-09-13 │ 0.43927 │ 0.44052 │ 0.43927 │ 0.43927 │ 57822062 │       0 │
 6. │ 1984-09-14 │ 0.44052 │ 0.45589 │ 0.44052 │ 0.44566 │ 68847968 │       0 │
 7. │ 1984-09-17 │ 0.45718 │ 0.46357 │ 0.45718 │ 0.45718 │ 53755262 │       0 │
 8. │ 1984-09-18 │ 0.45718 │ 0.46103 │ 0.44052 │ 0.44052 │ 27136886 │       0 │
 9. │ 1984-09-19 │ 0.44052 │ 0.44566 │ 0.43157 │ 0.43157 │ 29641922 │       0 │
10. │ 1984-09-20 │ 0.43286 │ 0.43668 │ 0.43286 │ 0.43286 │ 18453585 │       0 │
11. │ 1984-09-21 │ 0.43286 │ 0.44566 │ 0.42388 │ 0.42902 │ 27842780 │       0 │
    └────────────┴─────────┴─────────┴─────────┴─────────┴──────────┴─────────┘

11 rows in set. Elapsed: 0.006 sec. 
```

Now reading the very same file from path containing whitespaces fires an error:

```sql
SELECT *
FROM s3('http://minio:9001/test/folder with whitespace/sample.csv', 'minio', 'minio123', 'CSV')

Query id: f468d93a-aa59-49db-9a0c-e8c20be11f54


Elapsed: 0.004 sec. 

Received exception from server (version 25.8.10):
Code: 1000. DB::Exception: Received from localhost:9000. DB::Exception: Bad URI syntax: URI contains invalid characters. (POCO_EXCEPTION)
```

## Reading from a CSV without header within an s3 path containing names entities (e.g. `foo=bar`)

We've previously included a sample CSV file with NO header in s3. It's duplicated in two different paths:
- test/sample-no-header.csv
- test/sample/date=2025-10-12/sample-no-header.csv

I'm expecting to be able to read both of them fine. But the second fails. 

```sql
SELECT *
FROM s3('http://minio:9001/test/sample-no-header.csv', 'minio', 'minio123', 'CSV')

Query id: ec707ea1-2d54-461d-ac0e-95bed5fa86e3

    ┌─────────c1─┬──────c2─┬──────c3─┬──────c4─┬──────c5─┬───────c6─┬─c7─┐
 1. │ 1984-09-07 │ 0.42388 │ 0.42902 │ 0.41874 │ 0.42388 │ 23220030 │  0 │
 2. │ 1984-09-10 │ 0.42388 │ 0.42516 │ 0.41366 │ 0.42134 │ 18022532 │  0 │
 3. │ 1984-09-11 │ 0.42516 │ 0.43668 │ 0.42516 │ 0.42902 │ 42498199 │  0 │
 4. │ 1984-09-12 │ 0.42902 │ 0.43157 │ 0.41618 │ 0.41618 │ 37125801 │  0 │
 5. │ 1984-09-13 │ 0.43927 │ 0.44052 │ 0.43927 │ 0.43927 │ 57822062 │  0 │
 6. │ 1984-09-14 │ 0.44052 │ 0.45589 │ 0.44052 │ 0.44566 │ 68847968 │  0 │
 7. │ 1984-09-17 │ 0.45718 │ 0.46357 │ 0.45718 │ 0.45718 │ 53755262 │  0 │
 8. │ 1984-09-18 │ 0.45718 │ 0.46103 │ 0.44052 │ 0.44052 │ 27136886 │  0 │
 9. │ 1984-09-19 │ 0.44052 │ 0.44566 │ 0.43157 │ 0.43157 │ 29641922 │  0 │
10. │ 1984-09-20 │ 0.43286 │ 0.43668 │ 0.43286 │ 0.43286 │ 18453585 │  0 │
11. │ 1984-09-21 │ 0.43286 │ 0.44566 │ 0.42388 │ 0.42902 │ 27842780 │  0 │
    └────────────┴─────────┴─────────┴─────────┴─────────┴──────────┴────┘

11 rows in set. Elapsed: 0.006 sec. 
```

The second one fails indicating that the file might have missing columns. 

```sql
SELECT *
FROM s3('http://minio:9001/test/sample/date=2025-10-12/sample-no-header.csv', 'minio', 'minio123', 'CSV')

Query id: 712d8d31-203c-4a9a-91ed-e6df668ea78b


Elapsed: 0.008 sec. 

Received exception from server (version 25.8.10):
Code: 27. DB::Exception: Received from localhost:9000. DB::Exception: Cannot parse input: expected ',' before: '\n1984-09-10,0.42388,0.42516,0.41366,0.42134,18022532,0\n1984-09-11,0.42516,0.43668,0.42516,0.42902,42498199,0\n1984-09-12,0.42902,0.43157,0.41618,0.41618,37125801': (at row 1)
: 
Row 1:
Column 0,   name: c1,   type: Nullable(Date),       parsed text: "1984-09-07"
Column 1,   name: c2,   type: Nullable(Float64),    parsed text: "0.42388"
Column 2,   name: c3,   type: Nullable(Float64),    parsed text: "0.42902"
Column 3,   name: c4,   type: Nullable(Float64),    parsed text: "0.41874"
Column 4,   name: c5,   type: Nullable(Float64),    parsed text: "0.42388"
Column 5,   name: c6,   type: Nullable(Int64),      parsed text: "23220030"
Column 6,   name: c7,   type: Nullable(Int64),      parsed text: "0"
ERROR: Line feed found where delimiter (,) is expected. It's like your file has less columns than expected.
And if your file has the right number of columns, maybe it has unescaped quotes in values.

: (in file/uri test/sample/date=2025-10-12/sample-no-header.csv): While executing ParallelParsingBlockInputFormat: While executing ReadFromObjectStorage. (CANNOT_PARSE_INPUT_ASSERTION_FAILED)
```

Strange enought `DESCRIBE TABLE` function is able to detect columns just fine. 
```sql
DESCRIBE TABLE s3('http://minio:9001/test/sample/date=2025-10-12/sample-no-header.csv', 'minio', 'minio123', 'CSV')

Query id: ec5d6649-1c39-40ca-902c-9048353382ba

   ┌─name─┬─type──────────────┬─default_type─┬─default_expression─┬─comment─┬─codec_expression─┬─ttl_expression─┐
1. │ c1   │ Nullable(Date)    │              │                    │         │                  │                │
2. │ c2   │ Nullable(Float64) │              │                    │         │                  │                │
3. │ c3   │ Nullable(Float64) │              │                    │         │                  │                │
4. │ c4   │ Nullable(Float64) │              │                    │         │                  │                │
5. │ c5   │ Nullable(Float64) │              │                    │         │                  │                │
6. │ c6   │ Nullable(Int64)   │              │                    │         │                  │                │
7. │ c7   │ Nullable(Int64)   │              │                    │         │                  │                │
   └──────┴───────────────────┴──────────────┴────────────────────┴─────────┴──────────────────┴────────────────┘

7 rows in set. Elapsed: 0.004 sec. 
```

I've figured that I can force reading the file from this kind of path by enabling `input_format_csv_allow_variable_number_of_columns` setting. 

```sql
SELECT *
FROM s3('http://minio:9001/test/sample/date=2025-10-12/sample-no-header.csv', 'minio', 'minio123', 'CSV')
SETTINGS input_format_csv_allow_variable_number_of_columns = 1

Query id: b42f5872-8e40-4711-ac47-2e1a577a572e

    ┌─────────c1─┬──────c2─┬──────c3─┬──────c4─┬──────c5─┬───────c6─┬─c7─┬───────date─┐
 1. │ 1984-09-07 │ 0.42388 │ 0.42902 │ 0.41874 │ 0.42388 │ 23220030 │  0 │ 2025-10-12 │
 2. │ 1984-09-10 │ 0.42388 │ 0.42516 │ 0.41366 │ 0.42134 │ 18022532 │  0 │ 2025-10-12 │
 3. │ 1984-09-11 │ 0.42516 │ 0.43668 │ 0.42516 │ 0.42902 │ 42498199 │  0 │ 2025-10-12 │
 4. │ 1984-09-12 │ 0.42902 │ 0.43157 │ 0.41618 │ 0.41618 │ 37125801 │  0 │ 2025-10-12 │
 5. │ 1984-09-13 │ 0.43927 │ 0.44052 │ 0.43927 │ 0.43927 │ 57822062 │  0 │ 2025-10-12 │
 6. │ 1984-09-14 │ 0.44052 │ 0.45589 │ 0.44052 │ 0.44566 │ 68847968 │  0 │ 2025-10-12 │
 7. │ 1984-09-17 │ 0.45718 │ 0.46357 │ 0.45718 │ 0.45718 │ 53755262 │  0 │ 2025-10-12 │
 8. │ 1984-09-18 │ 0.45718 │ 0.46103 │ 0.44052 │ 0.44052 │ 27136886 │  0 │ 2025-10-12 │
 9. │ 1984-09-19 │ 0.44052 │ 0.44566 │ 0.43157 │ 0.43157 │ 29641922 │  0 │ 2025-10-12 │
10. │ 1984-09-20 │ 0.43286 │ 0.43668 │ 0.43286 │ 0.43286 │ 18453585 │  0 │ 2025-10-12 │
11. │ 1984-09-21 │ 0.43286 │ 0.44566 │ 0.42388 │ 0.42902 │ 27842780 │  0 │ 2025-10-12 │
    └────────────┴─────────┴─────────┴─────────┴─────────┴──────────┴────┴────────────┘

11 rows in set. Elapsed: 0.007 sec. 
```
The query above runs without an error and adds an additional column coming from the s3 path. 

I've tested the same thing with a csv having a header row. And in that case, everything works fine. 

```sql
SELECT *
FROM s3('http://minio:9001/test/sample.csv', 'minio', 'minio123', 'CSV')
LIMIT 1

Query id: dd464d01-89bf-4d0d-9d00-fd443bdf2bae

   ┌───────Date─┬────Open─┬────High─┬─────Low─┬───Close─┬───Volume─┬─OpenInt─┐
1. │ 1984-09-07 │ 0.42388 │ 0.42902 │ 0.41874 │ 0.42388 │ 23220030 │       0 │
   └────────────┴─────────┴─────────┴─────────┴─────────┴──────────┴─────────┘

1 row in set. Elapsed: 0.005 sec. 
```

And from similar path:

```sql
SELECT *
FROM s3('http://minio:9001/test/sample/date=2025-10-12/sample.csv', 'minio', 'minio123', 'CSV')
LIMIT 1

Query id: c1f09583-0184-4c4c-8631-2311b8e6b21a

   ┌───────Date─┬────Open─┬────High─┬─────Low─┬───Close─┬───Volume─┬─OpenInt─┬───────date─┐
1. │ 1984-09-07 │ 0.42388 │ 0.42902 │ 0.41874 │ 0.42388 │ 23220030 │       0 │ 2025-10-12 │
   └────────────┴─────────┴─────────┴─────────┴─────────┴──────────┴─────────┴────────────┘

1 row in set. Elapsed: 0.008 sec. 

```