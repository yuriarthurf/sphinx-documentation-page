# Parser

Dora's [parser](https://en.wikipedia.org/wiki/Parsing) and [transpiler](https://en.wikipedia.org/wiki/Source-to-source_compiler) tool for some big data **SQL** dialects, based on [Mozilla SQL Parser](https://github.com/mozilla/moz-sql-parser)  project.

[![PyPI](https://img.shields.io/pypi/v/dora-parser)](https://pypi.org/project/dora-parser/) ![PyPI - Python Version](https://img.shields.io/pypi/pyversions/dora-parser)

## Getting Started

An application that will translate source code from a given language (**Impala**, **Spark**, **Hive**, **Presto** and **Athena**) and produce equivalent code in another language that has a similar level of abstraction.

## Installation

To install dora-parser if you has _pandas_ and _seaborn_ on your machine, use
```bash
pip install dora-parser
```
otherwise you can use 
```bash
pip install dora-parser pandas seaborn 
```

## Usage

### Translate **Query**

I. Import Modules

```python
from dora_parser.parser import Parser
from dora_parser.transpiler import Transpiler
```

II. Generate the parse `tree` from `query`

```python
tree = Parser(query)
```

Where `query` have the following value:

```sql
WITH t1 as (select now() att_day)
SELECT DECODE(DAYOFWEEK(att_day)
            , 1 , 'Monday'
            , 2 , 'Tuesday'
            , 3 , 'Wednesday'
            , 4 , 'Thursday'
            , 5 , 'Friday'
            , 6 , 'Saturday'
            , 7 , 'Sunday'
            , 'Unknown day') "Day of week"
  , TRUNC(MONTHS_ADD(att_day,2),'SYEAR') as "Trunc SYEAR"
  , TRUNC(MONTHS_ADD(att_day,2),'YEAR') as  "Trunc YEAR"
  FROM t1;
```

creates the tree object as shown below:

```json
{
    "select": [
        {
          "value": {"decode": [
            {"dayofweek": "att_day"},1,
              {"literal": "Monday"},2,
              {"literal": "Tuesday"},3,
              {"literal": "Wednesday"},4,
              {"literal": "Thursday"},5,
              {"literal": "Friday"},6,
              {"literal": "Saturday"},7,
              {"literal": "Sunday"},
              {"literal": "Unknown day"}]},
          "name": "Day of week"},
        {
          "value": {"trunc": [{"months_add": ["att_day",2]},{"literal": "SYEAR"}]},
          "name": "Trunc SYEAR"
        },
        {
          "value": {"trunc": [{"months_add": ["att_day",2]},{"literal": "YEAR"}]},
          "name": "Trunc YEAR"
        }
    ],
    "from": "t1",
    "with": {
        "name": "t1",
        "value": {
          "select": {
            "value": {"now": {}},
            "name": "att_day"}}
    }
}
```

III. Translate the `tree` object from your original SQL Dialect (*impala*) to the new one (*spark*)

```python
transpiler = Transpiler(from_dialect='impala', to_dialect='spark')
result, errors = transpiler.translate(tree)
```

the `result` value will be like

```sql
WITH t1 AS (SELECT NOW() AS att_day) 
SELECT CASE
        WHEN DAYOFWEEK(att_day) = 1 THEN 'Monday' 
        WHEN DAYOFWEEK(att_day) = 2 THEN 'Tuesday' 
        WHEN DAYOFWEEK(att_day) = 3 THEN 'Wednesday'
        WHEN DAYOFWEEK(att_day) = 4 THEN 'Thursday' 
        WHEN DAYOFWEEK(att_day) = 5 THEN 'Friday' 
        WHEN DAYOFWEEK(att_day) = 6 THEN 'Saturday' 
        WHEN DAYOFWEEK(att_day) = 7 THEN 'Sunday' 
        ELSE 'Unknown day' 
      END AS `Day of week`
  , TRUNC(att_day + INTERVAL 2 MONTHS, 'SYEAR') AS `Trunc SYEAR`
  , TRUNC(att_day + INTERVAL 2 MONTHS, 'YEAR') AS `Trunc YEAR`
FROM t1
```

You can also have access to a list with information about any `errors`, as well as where they occur.
In this example, The `TRUNC` function in **Spark** only works with a few data formats, so you can not use it with "*SYEAR*"

```json
[
  {
    "trunc": "MEDIUM:20:[{'add': ['att_day', {'interval': [2,'MONTHS']}]}, {'literal': 'SYEAR'}]"
  }
] 
```

Information about the errors will also appear in the output log.

```log
dora_parser 2021-08-12 17:24:24,650 WARNING _TRUNC_ Spark 
data formats:['YEAR', 'YYYY', 'YY', 'QUARTER', 'MONTH', 'MM', 'MON', 'WEEK']
dora_parser 2021-08-12 17:24:24,650 WARNING resolve trunc NotImplemented: 
--TRANSPILER:MEDIUM:LEVEL20:'trunc'
```

#### Translate **Script**

Are considered an *script* any type of *string* with **multiple SQL statements**

I. Import Module

``` py
from dora_parser.reader import Reader
```

II. Translate

```python
script="""
INSERT INTO t.customer SELECT DCEIL(p_sale) FROM Customers;

COMPUTE STATS customer;

SELECT staff_id, staff_name, CHAR_LENGTH(staff_name) AS lengthofname, COUNT(*) order_count  
FROM sales.orders 
WHERE YEAR(order_date) = 2021 
GROUP BY staff_id;
"""
reader = Reader(from_dialect='impala',to_dialect='athena')
result, errors, n_queries = reader.translate_script(script)
```

Give you as `result`

```sql
INSERT INTO t.customer SELECT CEIL(p_sale) FROM Customers;

/* STATEMENT ERRORS:COMPUTE STATS*/
COMPUTE STATS customer;

SELECT staff_id, staff_name, LENGTH(staff_name) AS lengthofname, 
COUNT(*) AS order_count 
FROM sales.orders 
WHERE YEAR(order_date) = 2021 
GROUP BY staff_id;
```

III. Generate a summary list (*optional*)

```python
summary = reader.create_summary(errors, n_queries)
```

value for `summary` variable:

```json
[
  {"N_queries": 3}, 
  {"Success": 2}, 
  {"Failed": {"HARD": 1}}, 
  {"Er_types": ["compute stats"]}
]
```

#### Translate multiple **Files**

I. Import Module

``` python
from dora_parser.reader import Reader
```

II. Translate

``` py
dir_impala = 'scrpits/impala/'
dir_spark = 'scripts/spark'
reader = Reader(from_dialect='impala', to_dialect='spark', input_dir =dir_impala,output_dir=dir_spark)
reader.translate_files()
```

The translated files will be saved to folders in the output directory according to the result of the translation.
If you don't specify the output directory, the resulting folders will be in the input directory.

III. Generate a migration report (*optional*)

You can also have access to a report in HTML with an overview of the result of migration process.
To do this, set the **migration_report** argument equal to True".

``` python
reader = Reader(from_dialect='impala', to_dialect='spark', input_dir =dir_impala,output_dir=dir_spark, migration_report=True)
reader.translate_files()
```

IV. Generate a summary (optional)

If you want to access a summary dictionary of the migration process, set the **summary_dict** argument equal to True, as in the example below:

```python
reader = Reader(from_dialect='impala', to_dialect='spark', input_dir =dir_impala,output_dir=dir_spark)
reader.translate_files(summary_dict=True)
```

```json
{
    "Input_dir": "/scripts/impala",
    "From_dialect": "impala",
    "To_dialect": "spark",
    "Sucess_files": 3,
    "Failed_files": 1,
    "Files": {
        "row_benchmark.sql": [
            {"N_queries": 7},
            {"Success": 7},
            {"Failed": {}},
            {"Er_types": []}],
        "customer_facts.sql": [
            {"N_queries": 12},
            {"Success": 12},
            {"Failed": {}},
            {"Er_types": []}],
        "document.sh": [
            {"N_queries": 1},
            {"Success": 1},
            {"Failed": {}},
            {"Er_types": []}],
        "orches.sql": [
            {"N_queries": 9}, 
            {"Success": 5}, 
            {"Failed": {"HARD": 3}}, 
            {"Er_types": ["create role", "appx_median", "parser"]}]
    }
}
```

## Error Types

+ `Parser`: Errors when generating the tree structure.
+ `Transpiler`: Unimplemented functions and their levels of complexity.
+ `Statement`: Commands that are not capable of being reproduced, since there are no equivalent in the target language.

## Supported Languages

+ Apache Impala 2.12
+ Hive 3.12
+ Spark 3.1
+ Presto 0.217

## Getting Help

We use GitHub [issues](https://github.com/doraproject/parser/issues) for tracking [bugs](https://github.com/doraproject/parser/labels/bug), [questions](https://github.com/doraproject/parser/labels/question) and [feature requests](https://github.com/doraproject/parser/labels/enhancement).

## Contributing

Please read through this [contributing](.github/CONTRIBUTING.md) document to get start and before submitting any issues or pull requests to ensure we have all the necessary information to effectively respond to your contribution.

---

[Dora Project](https://github.com/doraproject) is a recent open-source project based on technology developed at [Compasso UOL](https://compassouol.com/)

