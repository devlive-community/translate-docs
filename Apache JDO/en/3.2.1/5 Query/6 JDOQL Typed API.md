[TOC]

## JDOQL Typed API

JDO 3.2 introduces a way of performing queries using a JDOQLTypedQuery API, that copes with refactoring of classes/fields. The API follows the same [JDOQL]($JDOQL) syntax that we have seen earlier in terms of the components of the query etc. It produces queries that are much more elegant and simpler than the equivalent "Criteria" API in JPA.

### Preparation

To set up your environment to use this JDOQLTypedQuery API you need to enable annotation processing, place your JDO 3.2 provider jars in your build path, and specify a `@PersistenceCapable` annotation on your classes to be used in queries (you can still provide the remaining information in XML metadata if you wish to). This annotation processor will (just before compile of your persistable classes), create a query metamodel "Q" class for each persistable class. This is similar step to what QueryDSL requires, or indeed the JPA Criteria static metamodel.

#### Using Maven

For example, using the JDO 3.2 RI, with Maven you need to have the following in your POM

```xml
<dependencies>
    <dependency>
        <groupId>org.datanucleus</groupId>
        <artifactId>datanucleus-jdo-query</artifactId>
        <version>5.0.9</version>
    </dependency>
    <dependency>
        <groupId>javax.jdo</groupId>
        <artifactId>jdo-api</artifactId>
        <version>3.2</version>
    </dependency>
    ...
 </dependencies>
```

This creates the "metamodel" Q classes under *target/generated-sources/annotations/*. You can change this location using the configuration property **generatedSourcesDirectory** of the *maven-compiler-plugin*.

#### Using Eclipse

With Eclipse you need to

+   Go to *Java Compiler* and make sure the compiler compliance level is 1.8 or above
+   Go to *Java Compiler → Annotation Processing* and enable the project specific settings and enable annotation processing
+   Go to *Java Compiler → Annotation Processing → Factory Path*, enable the project specific settings and then add the following jars to the list: `datanucleus-jdo-query.jar`, `jdo-api.jar`

This creates the "metamodel" Q classes under *target/generated-sources/annotations/*. You can change this location on the *Java Compiler → Annotation Processing* page.

### Query Classes

The above preparation will mean that whenever you compile, the DataNucleus annotation processor (in `datanucleus-jdo-query.jar`) will generate a **query class** for each model class that is annotated as persistable. So what is a **query class** you ask. It is simply a mechanism for providing an intuitive API to generating queries. If we have the following model class

```java
@PersistenceCapable
public class Product
{
    @PrimaryKey
    long id;
    String name;
    double value;

    ...
}
```

then the (generated) **query class** for this will be

```java
public class QProduct extends org.datanucleus.api.jdo.query.PersistableExpressionImpl<Product>
    implements PersistableExpression<Product>
{
    public static QProduct candidate(String name) {...}
    public static QProduct candidate() {...}
    public static QProduct variable(String name) {...}
    public static QProduct parameter(String name) {...}

    public NumericExpression<Long> id;
    public StringExpression name;
    public NumericExpression<Double> value;

    ...
}
```

The generated class has the name of form *\*Q\*{className}*. Also the generated class, by default, has a public field for each persistable field/property and is of a type *XXXExpression*. These expressions allow us to give Java like syntax when defining your queries (see below). So you access your persistable members in a query as **candidate.name** for example.

As mentioned above this is the default style of query class. However you can also create it in *property* style, where you access your persistable members as **candidate.name()** for example. The benefit of this approach is that if you have 1-1, N-1 relationship fields then it only initialises the members when called, whereas in the *field* case above it has to initialise all in the constructor, so at static initialisation.

<table><tbody><tr><td class="icon"><i class="fa icon-note" title="Note"></i></td><td class="content">The JDOQL Typed query mechanism only works for classes that are annotated, and not for classes that use XML metadata. This is due to the fact that it makes use of a Java <em>annotation processor</em>.</td></tr></tbody></table>

#### Limitations

There are some corner cases where the use of expressions and this API may require casting to allow the full range of operations for JDOQL. Some examples

+   If you have a List field and call `ListExpression.get(position)` this returns an `Expression` rather than a specific `NumericExpression`, `StringExpression`, or whatever subtype. You would need to cast the result to do subsequent calls.
+   If you have a Map field and call `MapExpression.get(key)` this returns an `Expression` rather than a specific `NumericExpression`, `StringExpression`, or whatever subtype. You would need to cast the result to do subsequent calls.
+   If you have a Collection parameter and call `CollectionParameter.contains(fieldExpression)` then you may need to cast the `fieldExpression` to `Expression` since the `CollectionParameter` will not have adequate java generic information for the compiler to do it automatically
+   If you have a Map parameter and call `MapParameter.contains(fieldExpression)` then you may need to cast the `fieldExpression` to `Expression` since the `MapParameter` will not have adequate java generic information for the compiler to do it automatically

### Filtering

Let’s provide a sample usage of this query API. We want to construct a query for all products with a value below a certain level, and where the name starts with "Wal". So a typical query in a JDO-enabled application

```java
pm = pmf.getPersistenceManager();

JDOQLTypedQuery<Product> tq = pm.newJDOQLTypedQuery(Product.class);
QProduct cand = QProduct.candidate();
List<Product> results = tq.filter(cand.value.lt(40.00).and(cand.name.startsWith("Wal")))
    .executeList();
```

This equates to the single-string query

```sql
SELECT FROM mydomain.Product WHERE this.value < 40.0 && this.name.startsWith("Wal")
```

As you see, we create a parametrised query, and then make use of the **query class** to access the candidate, and from that make use of its fields, and the various Java methods present for the types of those fields. Note that the API is *fluent*, meaning you can chain calls easily.

### Ordering

We want to order the results of the previous query by the product name, putting nulls first.

```java
tq.orderBy(cand.name.asc().nullsFirst());
```

This query now equates to the single-string query

```sql
SELECT FROM mydomain.Product WHERE this.value < 40.0 && this.name.startsWith("Wal") ORDER BY this.name ASCENDING NULLS FIRST
```

If you don’t want to specify null positioning, simply omit the `nullsFirst()` call. Similarly to put nulls last then call `nullsLast()`.

### Methods

In the above example you will have seen the use of some of the normal JDOQL methods. With the JDOQLTyped API these are available on the different types of expressions. For example, *cand.name* is a `StringExpression` and consequently it has all of the normal String methods available, just like in JDOQL and just like in Java. Similarly if we had a class `Inventory` which had a Collection of `Product`, then we could use the method **contains** on the `CollectionExpression`.

> The JDOQL methods _JDOHelper.getObjectId_ and _JDOHelper.getVersion_ are available on `PersistableExpression`, for the object that they would be invoked on.

> The JDOQL methods _Math.{xxx}_ are available on `NumericExpression`, for the numeric that they would be invoked on.

### Results

Let’s take the query in the above example and return the name and value of the Products only

```java
JDOQLTypedQuery<Product> tq = pm.newJDOQLTypedQuery(Product.class);
QProduct cand = QProduct.candidate();
List<Object[]> results = tq.filter(cand.value.lt(40.00).and(cand.name.startsWith("Wal"))).orderBy(cand.name.asc())
        .result(false, cand.name, cand.value).executeResultList();
```

This equates to the single-string query

```sql
SELECT this.name,this.value FROM mydomain.Product WHERE this.value < 40.0 && this.name.startsWith("Wal") ORDER BY this.name ASCENDING
```

A further example using aggregates

```java
JDOQLTypedQuery<Product> tq = pm.newJDOQLTypedQuery(Product.class);
Object results =
    tq.result(false, QProduct.candidate().value.max(), QProduct.candidate().value.min()).executeResultUnique();
```

This equates to the single-string query

```sql
SELECT max(this.value), min(this.value) FROM mydomain.Product
```

If you wanted to assign an alias to a result component you do it like this

```java
tq.result(false, cand.name.as("THENAME"), cand.value.as("THEVALUE"));
```

### Parameters

It is important to note that JDOQLTypedQuery only accepts **named** parameters. You obtain a named parameter from the JDOQLTypedQuery, and then use it in the specification of the filter, ordering, grouping etc. Let’s take the query in the above example and specify the "Wal" in a parameter.

```java
JDOQLTypedQuery<Product> tq = pm.newJDOQLTypedQuery(Product.class);
QProduct cand = QProduct.candidate();
List<Product> results =
    tq.filter(cand.value.lt(40.00).and(cand.name.startsWith(tq.stringParameter("prefix"))))
        .orderBy(cand.name.asc())
        .setParameter("prefix", "Wal").executeList();
```

This equates to the single-string query

```sql
SELECT FROM mydomain.Product WHERE this.value < 40.0 && this.name.startsWith(:prefix) ORDER BY this.name ASCENDING
```

### Variables

Let’s try to find all Inventory objects containing a Product with a particular name. This means we need to use a variable. Just like with a parameter, we obtain a *variable* from the Q class.

```java
JDOQLTypedQuery<Inventory> tq = pm.newJDOQLTypedQuery(Inventory.class);
QProduct var = QProduct.variable("var");
QInventory cand = QInventory.candidate();
List<Inventory> results = tq.filter(cand.products.contains(var).and(var.name.startsWith("Wal"))).executeList();
```

This equates to the single-string query

```sql
SELECT FROM mydomain.Inventory WHERE this.products.contains(var) && var.name.startsWith("Wal")
```

### If-Then-Else

Let’s make use of an IF-THEN-ELSE expression to return the products based on whether they are "domestic" or "international" (in our case its just based on the "id")

```java
JDOQLTypedQuery<Product> tq = pm.newJDOQLTypedQuery(Product.class);
QProduct cand = QProduct.candidate();
IfThenElseExpression<String> ifElseExpr = tq.ifThenElse(String.class, cand.id.lt(1000), "Domestic", "International");
tq.result(false, ifElseExpr);
List<String> results = tq.executeResultList();
```

This equates to the single-string query

```sql
SELECT IF (this.id < 1000) "Domestic" ELSE "International" FROM mydomain.Product
```

### Subqueries

Let’s try to find all Products that have a value below the average of all Products. This means we need to use a subquery

```java
JDOQLTypedQuery<Product> tq = pm.newJDOQLTypedQuery(Product.class);
QProduct cand = QProduct.candidate();
TypesafeSubquery<Product> tqsub = tq.subquery(Product.class, "p");
QProduct candsub = QProduct.candidate("p");
List<Product> results = tq.filter(cand.value.lt(tqsub.selectUnique(candsub.value.avg()))).executeList();
```

Note that where we want to refer to the candidate of the subquery, we specify the alias ("p") explicitly. This equates to the single-string query

```sql
SELECT FROM mydomain.Product WHERE this.value < (SELECT AVG(p.value) FROM mydomain.Product p)
```

> When you are using a subquery and want to refer to the candidate (or field thereof) of the outer query in the subquery then you would use `cand` in the above example (or a field of it as required).

### Candidates

If you don’t want to query instances in the datastore but instead query a collection of candidate instances, you can do this by setting the candidates, like this

```java
JDOQLTypedQuery<Product> tq = pm.newJDOQLTypedQuery(Product.class);
QProduct cand = QProduct.candidate();
List<Product> results = tq.filter(cand.value.lt(40.00)).setCandidates(myCandidates).executeList();
```
