# Databanken 2: Samenvatting

Een samenvatting van het vak Databanken 2 geschreven door Quentin Hacourt.

Hallo, ik ben Quentin Hacourt en ik schrijf deze samenvatting met als doel om minstens 16/20 te halen op het vak Databanken 2 (bertels BTFO).

## Week 1: herhaling TDM/DB1

### Syntax

```SQL
    SELECT
    FROM
    WHERE
    GROUP BY
    HAVING
    ORDER BY
```

### OoE

```SQL
    FROM
    WHERE
    GROUP BY
    HAVING
    SELECT
    ORDER BY
```

### Joins

- Inner Join
    - Implicit (! Join condition in WHERE)
    - Explicit, Join condition
        - ON
        - USING
- OUTER JOIN
    - RIGHT
    - LEFT
    - FULL

### "Exotic joins"

- NATURAL
- CROSS
- LATERAL

### Combining output

- Q1 UNION [ALL] Q2
    - appends the result of Q2 to the result of Q1, no duplicates allowed, unless "all" is used
- Q1 INTERSECT [ALL] Q2
    - returns all rows that are both in the result of Q1 and in the result of Q2. Duplicate rows are eliminated unless INTERSECT ALL is used
- Q1 EXCEPT [ALL] Q2
    - returns all rows that are in the result of Q1 but not in the result of Q2. Duplicates are eliminated unless EXCEPT ALL is used

## Week 2: oefeningen

Geen theorie

## Week 3: CTEs

### What are CTEs?

Common Table Expressions

### Use?

- A temporary table ~ view
- To lighten complicated queries by defining auxiliary tables
- ~ subqueries but tightened in a looser way to the main query
- To parse recursive tables in a recursive way

Keyword to be used: WITH

### When?

- With SELECT, DELETE, UPDATE, INSERT main queries
- Can be by itself a SELECT, DELETE, UPDATE or INSERT auxiliary table

### Example:

update fines by 10% of all competition players that have never been in the board:

```SQL
WITH comp_spelers AS (
SELECT spelersnr, naam, voornaam
FROM spelers
WHERE bondsnr IS NOT NULL
), non_bestuur AS (
SELECT *
FROM bestuursleden b RIGHT OUTER JOIN spelers s
ON (b.spelersnr = s.spelersnr)
WHERE b.spelersnr IS NULL
)
UPDATE boetes
SET bedrag = bedrag * 1.1
FROM comp_spelers c INNER JOIN non_bestuur n ON (c.spelersnr = n.spelersnr)
WHERE c.spelersnr = b.spelersnr
```

### Recursive Example:


```SQL
WITH RECURSIVE lijst (naam, zoon, nr) AS (
SELECT naam, zoon, 0
FROM kings
WHERE naam = 'Louis V'
UNION ALL
SELECT naam, zoon, nr + 1
FROM lijst L INNER JOIN kings K
ON (L.naam = K.zoon)
WHERE nr < 5
)
SELECT naam
FROM lijst;
```

## Week 4: Optimalisations

### Algorithmic / Understanding

- Location of subqueries
    - Preferably
        - In FROM
        - Uncorrelated
    - Avoid
        - In SELECT
        - In WHERE
        - Correlated to SELECT, WHERE,... (Correlated to FROM is not that bad)
- Order of "tables" when JOINING
    - Start with those having the smallest *footprint*
        - Tables with few records
        - Subqueries delivering few records
    - End with the largest tables/subqueries returning the largest amount of records

### Indexes

**What?**

Indexes help to reduce computational time:
- Records are stored in =/= files
- Each file exists of =/= pages
- If one needs a record:
    1. The right page is selected using hash algorithms
    2. The right record is extracted from that page

**How?**

Indexes can be searched by using two methods:

- Sequential: row by row
    - Which completely defeats the point of having an index in the first place lmao don't do this
- indexed
    - Tree structure
    - 2 ways:
        - Search records based on their values
        - Parse a table using a sorted, auxiliary column

**Different algo's**

- B-tree (default)
- Hash
- GiST
- SP-Gist
- GIN
- BRIN

**Index Types**

- CREATE INDEX spelers_postcode_idx ON spelers (postcode
asc);
- CREATE UNIQUE INDEX spelers_naam_vl_idx ON spelers (naam, voorletters);
- CREATE INDEX spelers_naam_vl_partial_idx ON spelers (naam, voorletters) WHERE spelersnr < 100;
- REINDEX INDEX een_index;
REINDEX TABLE een_tabel;
REINDEX DATABASE een_database;

**Index Commands**

- CREATE
- ALTER
- DROP

**Indexes: Not Only Pros**

- Cons
    - Every write to an indexed table requires index adaptations
    - One table can have multiple indexes (multi-col or multiple single-col)
    - Index itself needs (physical space as well)
- Column choice:
    - Unique index on candidate keys
    - Index on (combinations of) cols being used in comparisons, selections
    - Index on (combinations of) cols being used in ORDER BY

**Different kinds of indexes:**

- Multi-table:
    - Index on cols across tables
- virtual:
    - Index on expression
- Selective:
    - Index on part of table
- Hash:
    - Index based on page-address
- Bitmap:
    - Interesting when a lot of redundant data exists

### Operators

- Operators to avoid:
    - OR
    - AND
    - UNION
    - Expressions with indexed cols
    - LIKE if pattern starts with _ or %
    - no smart joins (use redundant conditions in a join to force SQL into a certain processing order)
    - HAVING
    - Too many cols in SELECT clause
    - DISTINCT
    - not using keywords ALL option with SET operators like UNION (ALL does shorten the execution time)
    - DATATYPE conversions
    - ANY & ALL operators
        - use min/max instead if possible

### Tools

- EXPLAIN
    - Shows execution plan of statement
- EXPLAIN ANALYZE
    - Carry out the command and show the actual run times

## Week 5: Security, Embedded en Views

### Security

**Different layers:**

- Hardware:
    - Secured server
    - Redundant Hardware
    - Backups
- Software:
    - Update platform
    - DBMS
        - Roles
        - Users
        - Stored Procedures
        - ...

**SQL Injection: Solutions**

- Remove Escape characters from input
- Data type check
- Prepared statements
- Stored Procedures

***BUT***

- Combinations of above can be necessary$
- Easier to restrict to desired input, than check for input to refuse

### Views

**What?**

- Table, based on other table(s)
    - Part of existing table
    - Output of (complex) query
- Content varies according to underlying tables
    - Update of base tables => update of view
    - Update of view does not imply update of base tables ...
- Content of view is given/calculated when SELECT view is issued

**Views - Syntax example:**


```SQL
CREATE VIEW leeftijden (spelersnr, leeftijd) AS
    SELECT spelersnr, 2008 – year(geb_datum)
    FROM spelers
;
```

**Views - Main Properties**

- Security wise: access to view, not to raw material
- Requires no storage space
- Allows to simplify complex queries that need to be executed a lot
- Views cannot always be mutated
- In case of reorganisation of database:
    - A view can let old applications still be usable although base database structura has been altered

### Embedded

**Embedded: SQL is pre-compiled**

= SQL within a programming language, eg PhP:


```PhP
<?php
$dbconn = pg_connect("dbname=publisher") or die("Could not
connect");
if (!pg_connection_busy($dbconn)) {
pg_send_query($dbconn, "select * from authors;");
}
$res = pg_get_result($dbconn);
echo "Call to pg_get_result(): $res\n";
?>
```

## Week 6: Stored Procedures, Functions, Triggers

### Stored Procedures - Functions

Both are a set of SQL and procedural statements

- Set of SQL and procedural statements:
    - Declarations
    - Assignments
    - Loops
    - Flow control
    - ...
- Stored on DBMS
- Created using "CREATE FUNCTION" syntax
- Overloading is possible (different input params, same name)

Differences:

|                                 | Stored Procedure | Function                  |
|---------------------------------|-----------------:|---------------------------|
| Use in an expression            | x                | yes                       |
| Return a value                  | x                | Yes                       |
| Return values as OUT parameters | Yes              | x                         |
| Return a single result set      | Yes              | Yes (as a table function) |
| Return multiple result sets     | Yes              | x                         |

So in most cases, the purpose of a stored procedure is to:

- Perform actions without returning any result (INSERT, UPDATE operations i.e.)
- Return one or more scalar values as OUT parameters
- Return one or more result sets

Usualy the purpose of a user-defined function is to process the input parameters and return a new value.

**PostgreSQL: 4 kinds of functions**

- Query language functions
- Procedural language functions (functions written in, for example, PL/pgSQL or PL/Tcl)
- Internal functions
- C-language functions

**Example of function in PL/pgSQL

```SQL
CREATE OR REPLACE FUNCTION increment(i INT) RETURNS INT AS $$
BEGIN
RETURN i + 1;
END;
$$ LANGUAGE plpgsql;
-- An example how to use the function:
SELECT increment(10);
```







TODO: Hoofdstuk 6-9:

```SQL

```
