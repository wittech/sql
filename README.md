
[![Test and Build Workflow](https://github.com/opendistro-for-elasticsearch/sql/workflows/Java%20CI/badge.svg)](https://github.com/opendistro-for-elasticsearch/sql/actions)
[![codecov](https://codecov.io/gh/opendistro-for-elasticsearch/sql/branch/develop/graph/badge.svg)](https://codecov.io/gh/opendistro-for-elasticsearch/sql)
[![Documentation](https://img.shields.io/badge/api-reference-blue.svg)](https://opendistro.github.io/for-elasticsearch-docs/docs/sql/endpoints/)
[![Chat](https://img.shields.io/badge/chat-on%20forums-blue)](https://discuss.opendistrocommunity.dev/c/sql/)
![PRs welcome!](https://img.shields.io/badge/PRs-welcome!-success)

# Open Distro for Elasticsearch SQL


Open Distro for Elasticsearch enables you to extract insights out of Elasticsearch using the familiar SQL query syntax. Use aggregations, group by, and where clauses to investigate your data. Read your data as JSON documents or CSV tables so you have the flexibility to use the format that works best for you.


## SQL Related Projects

The following projects have been merged into this repository as separate folders as of July 9, 2020. Please refer to links below for details. This document will focus on the SQL plugin for Elasticsearch.

* [SQL CLI](https://github.com/opendistro-for-elasticsearch/sql/tree/master/sql-cli)
* [SQL JDBC](https://github.com/opendistro-for-elasticsearch/sql/tree/master/sql-jdbc)
* [SQL ODBC](https://github.com/opendistro-for-elasticsearch/sql/tree/master/sql-odbc)
* [SQL Workbench](https://github.com/opendistro-for-elasticsearch/sql/tree/master/workbench)


## Documentation

Please refer to the [SQL Language Reference Manual](./docs/user/index.rst), [Piped Processing Language (PPL) Reference Manual](./docs/experiment/ppl/index.rst) and [Technical Documentation](https://opendistro.github.io/for-elasticsearch-docs) for detailed information on installing and configuring opendistro-elasticsearch-sql plugin. Looking to contribute? Read the instructions on [Development Guide](./docs/developing.rst) and then submit a patch!


## Experimental

Recently we have been actively improving our query engine primarily for better correctness and extensibility. The new enhanced query engine has been already supporting the new released Piped Processing Language query processing behind the scene. Meanwhile, the integration with SQL language is also under way. To try out the power of the new query engine with SQL, simply run the command to enable it by [plugin setting](https://github.com/opendistro-for-elasticsearch/sql/blob/develop/docs/user/admin/settings.rst#opendistro-sql-engine-new-enabled). In future release, this will be enabled by default and nothing required to do from your side. Please stay tuned for updates on our progress and its new exciting features.

Here is a documentation list with features only available in this improved SQL query engine. Please follow the instruction above to enable it before trying out example queries in these docs:

* [Identifiers](./docs/user/general/identifiers.rst): support for identifier names with special characters
* [Data types](./docs/user/general/datatypes.rst): new data types such as date time and interval
* [Expressions](./docs/user/dql/expressions.rst): new expression system that can represent and evaluate complex expressions
* [SQL functions](./docs/user/dql/functions.rst): many more string and date functions added
* [Basic queries](./docs/user/dql/basics.rst)
    * Ordering by Aggregate Functions section
    * NULLS FIRST/LAST in section Specifying Order for Null
* [Aggregations](./docs/user/dql/aggregations.rst): aggregation over expression and more other features
* [Complex queries](./docs/user/dql/complex.rst)
    * Improvement on Subqueries in FROM clause
* [Window functions](./docs/user/dql/window.rst): ranking and aggregate window function support

To avoid impact on your side, normally you won't see any difference in query response. If you want to check if and why your query falls back to be handled by old SQL engine, please explain your query and check Elasticsearch log for "Request is falling back to old SQL engine due to ...".


## Setup

Install as plugin: build plugin from source code by following the instruction in Build section and install it to your Elasticsearch.

After doing this, you need to restart the Elasticsearch server. Otherwise you may get errors like `Invalid index name [sql], must not start with '']; ","status":400}`.


## Build

The package uses the [Gradle](https://docs.gradle.org/4.10.2/userguide/userguide.html) build system.

1. Checkout this package from version control.
2. To build from command line set `JAVA_HOME` to point to a JDK >=14
3. Run `./gradlew build`


## Basic Usage

To use the feature, send requests to the `_opendistro/_sql` URI. You can use a request parameter or the request body (recommended).

* Simple query

```
POST https://<host>:<port>/_opendistro/_sql
{
  "query": "SELECT * FROM my-index LIMIT 50"
}
```

* Explain SQL to elasticsearch query DSL
```
POST _opendistro/_sql/_explain
{
  "query": "SELECT * FROM my-index LIMIT 50"
}
```

* For a sample curl command with the Open Distro for Elasticsearch Security plugin, try:
```
curl -XPOST https://localhost:9200/_opendistro/_sql -u admin:admin -k -d '{"query": "SELECT * FROM my-index LIMIT 10"}' -H 'Content-Type: application/json'
```


## SQL Usage

* Query

        SELECT * FROM bank WHERE age >30 AND gender = 'm'

* Aggregation

        SELECT COUNT(*),SUM(age),MIN(age) as m, MAX(age),AVG(age)
        FROM bank
        GROUP BY gender
        HAVING m >= 20
        ORDER BY SUM(age), m DESC

* Join

        SELECT b1.firstname, b1.lastname, b2.age
        FROM bank b1
        LEFT JOIN bank b2
        ON b1.age = b2.age AND b1.state = b2.state

* Show

        SHOW TABLES LIKE ban%
        DESCRIBE TABLES LIKE bank

* Delete

        DELETE FROM bank WHERE age >30 AND gender = 'm'


## Beyond SQL

* Search

        SELECT address FROM bank WHERE address = matchQuery('880 Holmes Lane') ORDER BY _score DESC LIMIT 3

* Nested Field

	+ 
        
			SELECT address FROM bank b, b.nestedField e WHERE b.state = 'WA' and e.name = 'test'
	 
	+ 
			SELECT address, nested(nestedField.name)
			FROM bank
			WHERE nested(nestedField, nestedField.state = 'WA' AND nestedField.name = 'test')
			   OR nested(nestedField.state) = 'CA'

* Aggregations

	+ range age group 20-25,25-30,30-35,35-40

			SELECT COUNT(age) FROM bank GROUP BY range(age, 20,25,30,35,40)

	+ range date group by day

			SELECT online FROM online GROUP BY date_histogram(field='insert_time','interval'='1d')

	+ range date group by your config

			SELECT online FROM online GROUP BY date_range(field='insert_time','format'='yyyy-MM-dd' ,'2014-08-18','2014-08-17','now-8d','now-7d','now-6d','now')

* ES Geographic
		
		SELECT * FROM locations WHERE GEO_BOUNDING_BOX(fieldname,100.0,1.0,101,0.0)

* Select type or pattern

        SELECT * FROM indexName/type
        SELECT * FROM index*


## SQL Features

*  SQL Select
*  SQL Delete
*  SQL Where
*  SQL Order By
*  SQL Group By
*  SQL Having
*  SQL Inner Join
*  SQL Left Join
*  SQL Show
*  SQL Describe
*  SQL AND & OR
*  SQL Like
*  SQL COUNT distinct
*  SQL In
*  SQL Between
*  SQL Aliases
*  SQL Not Null
*  SQL(ES) Date
*  SQL avg()
*  SQL count()
*  SQL max()
*  SQL min()
*  SQL sum()
*  SQL Nulls
*  SQL isnull()
*  SQL floor
*  SQL trim
*  SQL log
*  SQL log10
*  SQL substring
*  SQL round
*  SQL sqrt
*  SQL concat_ws
*  SQL union and minus

## JDBC Support

Please check out JDBC driver repository for more details.

## Beyond sql features

*  ES TopHits
*  ES MISSING
*  ES STATS
*  ES GEO_INTERSECTS
*  ES GEO_BOUNDING_BOX
*  ES GEO_DISTANCE
*  ES GEOHASH_GRID aggregation

## Attribution

This project is based on the Apache 2.0-licensed [elasticsearch-sql](https://github.com/NLPchina/elasticsearch-sql) project. Thank you [eliranmoyal](https://github.com/eliranmoyal), [shi-yuan](https://github.com/shi-yuan), [ansjsun](https://github.com/ansjsun) and everyone else who contributed great code to that project. Read this for more details [Attributions](./docs/attributions.md).

## Code of Conduct

This project has adopted an [Open Source Code of Conduct](https://opendistro.github.io/for-elasticsearch/codeofconduct.html).


## Security issue notifications

If you discover a potential security issue in this project we ask that you notify AWS/Amazon Security via our [vulnerability reporting page](http://aws.amazon.com/security/vulnerability-reporting/). Please do **not** create a public GitHub issue.


## Licensing

See the [LICENSE](./LICENSE.txt) file for our project's licensing. We will ask you to confirm the licensing of your contribution.


## Copyright

Copyright 2019 Amazon.com, Inc. or its affiliates. All Rights Reserved.
