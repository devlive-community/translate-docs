[TOC]

As we have seen, a JDOQL query is made up of different parts. In this section we look at the *result* part of the query. The result is what we want returning. By default (when not specifying the result) the objects returned will be of the candidate class type, where they match the query filter. Firstly let’s look at what you can include in the *result* clause.

+   *this* - the candidate instance
+   A field name
+   A variable
+   A parameter (though why you would want a parameter returning is hard to see since you input the value in the first place)
+   An aggregate (count(), avg(), sum(), min(), max())
+   An expression involving a field (e.g "field1 + 1")
+   A navigational expression (navigating from one field to another …​ e.g "field1.field4")

The result is specified in JDOQL like this

```java
query.setResult("count(field1), field2");
```

In **Single-String JDOQL** you would specify it directly.

### Result type

What you specify in the *result* of a JDOQL query defines what form of result you get back.

+   **Object** - this is returned if you have only a single row in the results and a single column. This is achieved when you specified either UNIQUE, or just an aggregate (e.g "max(field2)")
+   **Object\[\]** - this is returned if you have only a single row in the results, but more than 1 column (e.g "max(field1), avg(field2)")
+   **List\<Object\>** - this is returned if you have only a single column in the result, and you don’t have only aggregates in the result (e.g "field2")
+   **List\<Object\[\]\>** - this is returned if you have more than 1 column in the result, and you don’t have only aggregates in the result (e.g "field2, avg(field3)")

### Aggregates

There are situations when you want to return a single number for a column, representing an aggregate of the values of all records. There are 5 standard JDOQL aggregate functions available. These are

+   **avg(val)** - returns the average of "val". "val" can be a field, numeric field expression or "distinct field".
+   **sum(val)** - returns the sum of "val". "val" can be a field, numeric field expression, or "distinct field".
+   **count(val)** - returns the count of records of "val". "val" can be a field, or can be "this", or "distinct field".
+   **min(val)** - returns the minimum of "val". "val" can be a field
+   **max(val)** - returns the maximum of "val". "val" can be a field

So to utilise these you could specify something like

```java
Query q = pm.newQuery("SELECT max(price), min(price) FROM mydomain.Product WHERE status == 1");
```

This will return a single row of results with 2 values, the maximum price and the minimum price of all products that have status code of 1.

### Example - Use of aggregates

JDOQL has the ability to use aggregates in queries. Here’s another example using the same Product class as above, but this time looking for the maximum price of products that are CD Players. Note that the result for this particular query will be of type Double since there is a single double precision value being returned via the "result".

Declarative JDOQL :

```java
Query query = pm.newQuery(mydomain.Product.class);
query.setFilter("name == \"CD Player\"");
query.setResult("max(this.price)");
List results = (List)query.execute();
Iterator iter = c.iterator();
Double max_price = (Double)iter.next();
```

Single-String JDOQL :

```java
Query query = pm.newQuery("SELECT max(price) FROM mydomain.Product WHERE name == \"CD Player\"");
List results = (List)query.execute();
Iterator iter = c.iterator();
Double max_price = (Double)iter.next();
```
