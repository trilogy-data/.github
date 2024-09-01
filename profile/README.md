# Welcome to Trilogy, an experiment on open-source, better SQL

The Trilogy ecosystem is built on Trilogy, a SQL syntax refresh for the modern data stack, and supporting integrations to support scheduling, analytics, and interactive development.

- pytrilogy - the core Trilogy implementation, a python package + CLI
- pytrilogy-nlp - a trilogy/natural language interface
- trilogy-studio - an electron IDE for directly running trilogy queries, including NLP
- pytrilogy-t[ransform] - a DBT integration
- trilogy-vscode - a vscode extension to support seamless development. 

## Why isn't SQL good enough?

SQL is fantastic.

SQL has been the de-facto language for working with data for decades. Data professionals can use a common, declarative syntax to interact with anything from local file based databases to global distributed compute clusters.

But SQL solves the wrong problem for modern analytics. 

SQL is a declarative language for reading and manipulating data in tables in SQL databases. This is a perfect fit for an application interacting with a datastore doing CRUD operations against tables. 

But in data warehouses, a table is a leaky abstraction. Users don't care about tables; tables are a means to an end. They want the data, and the table is a detail. Tables will be replicated; aggregated; cached; put behind views - and the user spends all their time on the container, not the product.

> [!TIP]
> SQL is a language for interacting with tables in a database. Trilogy is a language for interacting with data in a [warehouse/lake/mart], with all the abstraction, evolution, deprecation, and change that implies. The
> tables will change, but your query doesn't need to.


## Trilogy puts the answers first

An intuitive query for data should be oriented around the outputs, not where it happens to be.

Seeing revenue by product line is a goal; the table that contains the products and the table that contains revenue are implementation details. This is a philosophically different approach from 'better' sql that orients around defining the tables first; SQL/Trilogy have you write what you want [and in the case of SQL, then how to get it - in Trilogy, that happens for you].

This is what you're thinking about when you write SQL:

```sql

# I want to see revenue by city and product line
select
    product_line,
    revenue,
    city_name
;
```

But this is what you might have to write
```sql
# fact revenue _latest is them ost up to date; 
SELECT
    product_line,
    sum(revenue),
    city_name
FROM fact_revenue_latest
    INNER JOIN dim_city on fact_revenue_latest.city_id = dim_city.city_id
    INNER JOIN dim_product on fact_revenue_latest.product_id = dim_product.product_Id
    INNER JOIN dim_product_line on dim_product.line_id = dim_product_line.line_id
GROUP BY 
    product_line,
    city_name
```

Worse, that query might work today and not tomorrow, when the revenue team decides to denormalize for performance.

Over time, growing data teams tend towards SQL sprawl - duplicative,
hard to follow, brittle adhoc scripts and pipelines - which are critical to the company. Fortune 500 companies spend millions of dollars trying to reverse engineer the original intent of SQL to document dataflow or lineage, or to refactor business logic when moving to a new database.

## How Trilogy solves this

Trilogy separates declared conceptual manipulation (ex: [Profit] = [Revenue] - [Cost]) from the database that stores columns and runs queries. This `semantic layer` isn't a new concept, but Trilogy puts at close as possible
to the SQL itself, in a familiar form - you define the semantic layer with the same language you use to query it, meaning adhoc extension and iteration is easy.

These concepts and their derivation are strongly typed and can be statically analyzed and tested against a given set of datasources to prove the correctness for a given expression of business logic. These two definitions - the business logic and the access layer - can then be independently evolved and validated over time.

#### SQL
```sql
USE AdventureWorks;

SELECT 
    t.Name, 
    SUM(s.SubTotal) AS [Sub Total],
    STR(Sum([TaxAmt])) AS [Total Taxes],
    STR(Sum([TotalDue])) AS [Total Sales]
FROM Sales.SalesOrderHeader AS s
    INNER JOIN Sales.SalesTerritory as t ON s.TerritoryID = t.TerritoryID
GROUP BY 
    t.Name
ORDER BY 
    t.Name
```

#### Trilogy
```sql
import concepts.sales as sales;

select
    sales.territory_name,
    sales.sub_total,
    sales.total_taxes,
    sales.total_sales,
order by
    sales.territory_name desc;
```

## How does it work?

The example above cheats a little - the statement `import concepts.sales as sales;` is bringing in a model definition.

As a semantic layer, Trilogy requires some up-front binding to the database before the first query can be run.  The cost to model the data is incurred infrequently, and then the savings are amortized over every single user and query.

> [!TIP]
> Models can be defined, extended and bound to a table in-line; you don't need a separate file/definition to get started.
Unlike other semantic layers, Trilogy supports - and encourages - adhoc extension and iteration. 

Read me in the [concepts and references](/concepts) section to learn how Trilogy works under the hood, and the nuances of query design and setup.

## Usage

Trilogy is designed to be easy to learn and use, and to be able to be incrementally adopted. It can be run directly as a CLI, in a [GUI](/studio), or compiled to SQL and run in standard SQL tooling. 

Head over to the [demo](/demo) to see how this semantic layer is defined and run some example queries.
