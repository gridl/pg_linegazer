## [WIP] pg_linegazer
Transparent code coverage for PL/pgSQL

This project aims to provide a code coverage toolkit for PL/pgSQL, a procedural programming language supported by PostgreSQL. At the moment, pg_linegazer is in the alpha stage of development. It might be buggy, so bear with it :)

## Features

* No need for code instrumentation;
* Nested calls are taken into account;
* Compatible with PostgreSQL 9.5+;
* Well, that's it for now :|

## Roadmap

* Persistent storage for stats;
* Show branch hits (`if` / `case`);
* Build reports for `lcov`;

## Setup

Execute the following command in your favorite shell: 

```bash
make USE_PGXS=1 install
```

Don't forget to set `PG_CONFIG` if you'd like to install pg_linegazer into a custom build of PostgreSQL:

```bash
make USE_PGXS=1 PG_CONFIG=/path/to/pg_config install
```

After that, execute this query:

```plpgsql
CREATE EXTENSION pg_linegazer;
```

Done!

## Usage

Just execute your function a few times, then request a report:

```plpgsql
/* Step #1: define a function */
CREATE OR REPLACE FUNCTION test_func()
RETURNS VOID AS
$$
declare
	a int4 := 0;
	i int4;

begin
	for i in 1..100 loop
		a = a + i;
	end loop;

	if i < 10 then
		raise exception 'wtf!';
	end if;
end;
$$ LANGUAGE plpgsql;

/* Step #2: execute it 2 times */
select test_func();
select test_func();

/* Step #3: examine the report */
select * from linegazer_simple_report('test_func'::regproc);
 line | hits |                  code
------+------+-----------------------------------------
    1 |      |
    2 |      | declare
    3 |      |         a int4 := 0;
    4 |      |         i int4;
    5 |      |
    6 |      | begin
    7 |    2 |         for i in 1..100 loop
    8 |  200 |                 a = a + i;
    9 |      |         end loop;
   10 |      |
   11 |    2 |         if i < 10 then
   12 |    0 |                 raise exception 'wtf!';
   13 |      |         end if;
   14 |      | end;
(14 rows)
```

Moreover, we can estimate code coverage in function `test_func`:

```plpgsql
select linegazer_simple_coverage('test_func'::regproc);
 linegazer_simple_coverage
---------------------------
                      0.75
(1 row)
```

To reset **all** coverage stats, execute the following function:

```plpgsql
/* Clear stats cache */
select linegazer_clear();
 linegazer_clear
-----------------

(1 row)

/* Coverage is 0% */
select linegazer_simple_coverage('test_func'::regproc);
 linegazer_simple_coverage
---------------------------
                         0
(1 row)
```
