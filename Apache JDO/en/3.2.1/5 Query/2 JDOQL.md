[TOC]

## JDOQL

JDO defines ways of querying objects persisted into the datastore. It provides its own object-based query language (JDOQL). JDOQL is designed as the Java developers way of having the power of SQL queries, yet retaining the Java object relationship that exist in their application model. A typical JDOQL query may be set up in one of 2 ways. Here’s an example.

Declarative JDOQL :

```java
Query q = pm.newQuery(mydomain.Person.class, "lastName == \"Jones\" && age < age_limit");
q.declareParameters("double age_limit");
List results = (List)q.execute(20.0);
```

Single-String JDOQL :

```java
Query q = pm.newQuery("SELECT FROM mydomain.Person WHERE lastName == \"Jones\"" +
                      " && age < :age_limit PARAMETERS double age_limit");
List results = (List)q.execute(20.0);
```

So here in our example we select all "Person" objects with surname of "Jones" and where the persons age is below 20. The language is intuitive for Java developers, and is intended as their interface to accessing the persisted data model. As can be seen above, the query is made up of distinct parts. The class being selected (the SELECT clause in SQL), the filter (which equates to the WHERE clause in SQL), together with any sorting (the ORDER BY clause in SQL), etc.

Before giving details on JDOQL, you can download a quick reference guide [here](https://db.apache.org/jdo/jdoql_quickref.pdf)

### Single-String JDOQL

In original (declarative) JDOQL (JDO 1.0) it was necessary to specify the component parts (filter, candidate class, ordering, etc) of the query using the mutator methods on the Query. From JDO 2 onwards you can specify it all in a single string. This string has to follow a particular pattern, but provides the convenience that many people require. The pattern to use is as follows

```sql
SELECT [UNIQUE] [<result>] [INTO <result-class>]
    [FROM <candidate-class> [EXCLUDE SUBCLASSES]]
    [WHERE <filter>]
         [VARIABLES <variable declarations>]
         [PARAMETERS <parameter declarations>]
         [<import declarations>]
         [GROUP BY <grouping>]
    [ORDER BY <ordering>]
    [RANGE <start>, <end>]
```

The "keywords" in the query are shown in UPPER CASE but can be in *UPPER* or *lower* case.

Lets give an example of a query using this syntax

```sql
SELECT UNIQUE FROM org.datanucleus.samples.Employee ORDER BY departmentNumber
```

so we form the parts of the query as before, yet here we just specify it all in a single call.

### Accessing Fields

In JDOQL you access fields in the query by referring to the field name. For example, if you are querying a class called *Product* and it has a field "price", then you access it like this

```auto
Query query = pm.newQuery(mydomain.Product.class, "price < 150.0");
```

In addition to the persistent fields, you can also access "public static final" fields of any class. You can do this as follows

```auto
Query query = pm.newQuery(mydomain.Product.class, "taxPercent < mydomain.Product.TAX_BAND_A");
```

So this will find all products that include a tax percentage less than some "BAND A" level. Where you are using "public static final" fields you can either fully-qualify the class name or you can include it in the "imports" section of the query (see later).

### Data types : literals

JDOQL supports the following literals: IntegerLiteral, FloatingPointLiteral, BooleanLiteral, CharacterLiteral, StringLiteral, and NullLiteral.

### Operators precedence

The following list describes the operator precedence in JDOQL.

1.  Cast
2.  Unary ("~") ("!")
3.  Unary ("+") ("-")
4.  Multiplicative ("\*") ("/") ("%")
5.  Additive ("+") ("-")
6.  Relational (">=") (">") ("⇐") ("<") ("instanceof")
7.  Equality ("==") ("!=")
8.  Boolean logical AND ("&")
9.  Boolean logical OR ("|")
10.  Conditional AND ("&&")
11.  Conditional OR ("||")

### Concatenation Expressions

The concatenation operator(+) concatenates a String to either another String or Number. Concatenations of String or Numbers to null results in null.

### Example 1 - Use of Explicit Parameters

Here’s a simple example for finding the elements of a class with a field below a particular threshold level. Here we pass in the threshold value (*limit*), and sort the output in order of ascending price.

Declarative JDOQL :

```java
Query query = pm.newQuery(mydomain.Product.class,"price < limit");
query.declareParameters("double limit");
query.setOrdering("price ascending");
List results = (List)query.execute(150.00);
```

Single-String JDOQL :

```java
Query query = pm.newQuery("SELECT FROM mydomain.Product WHERE " +
                "price < limit PARAMETERS double limit ORDER BY price ASCENDING");
List results = (List)query.execute(150.00);
```

For completeness, the class is shown here

```java
class Product
{
    String name;
    double price;
    java.util.Date endDate;
    ...
}
```

```xml
<jdo>
    <package name="mydomain">
        <class name="Product">
            <field name="name">
                <column length="100" jdbc-type="VARCHAR"/>
            </field>
            <field name="abreviation">
                <column length="20" jdbc-type="VARCHAR"/>
            </field>
            <field name="price"/>
            <field name="endDate"/>
        </class>
    </package>
</jdo>
```

### Example 2 - Use of Implicit Parameters

Let’s repeat the previous query but this time using *implicit* parameters.

Declarative JDOQL :

```java
Query query = pm.newQuery(mydomain.Product.class,"price < :limit");
query.setOrdering("price ascending");
List results = (List)query.execute(150.00);
```

Single-String JDOQL :

```java
Query query = pm.newQuery("SELECT FROM mydomain.Product WHERE " +
                "price < :limit ORDER BY price ASCENDING");
List results = (List)query.execute(150.00);
```

So we omitted the declaration of the parameter and just prefixed it with a colon (:).

### Example 3 - Comparison against Dates

Here’s another example using the same Product class as above, but this time comparing to a Date field. Because we are using a type in our query, we need to *import* it …​ just like you would in a Java class if you were using it there.

Declarative JDOQL :

```java
Query query = pm.newQuery(domain.Product.class,
                          "endDate > best_before_limit");
query.declareImports("import java.util.Date");
query.declareParameters("Date best_before_limit");
query.setOrdering("endDate descending");
Collection results = (Collection)query.execute(my_date_limit);
```

Single-String JDOQL :

```java
Query query = pm.newQuery("SELECT FROM mydomain.Product " +
                "WHERE endDate > best_before_limit " +
                "PARAMETERS Date best_before_limit " +
                "import java.util.Date ORDER BY endDate DESC");
List results = (List)query.execute(my_date_limit);
```

### Example 4 - Instanceof

This example demonstrates use of the "instanceof" operator. We have a class A that has a field "b" of type B and B has subclasses B1, B2, B3. Clearly the field "b" of A can be of type B, B1, B2, B3 etc, and we want to find all objects of type A that have the field "b" that is of type B2. We do it like this

Declarative JDOQL :

```java
Query query = pm.newQuery(mydomain.A.class);
query.setFilter("b instanceof mydomain.B2");
List results = (List)query.execute();
```

Single-String JDOQL :

```java
Query query = pm.newQuery("SELECT FROM mydomain.A WHERE b instanceof mydomain.B2");
List results = (List)query.execute();
```
