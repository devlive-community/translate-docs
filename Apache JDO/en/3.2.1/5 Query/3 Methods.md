[TOC]

When writing the "filter" for a JDOQL Query you can make use of some methods on the various Java types. The range of methods included as standard in JDOQL is not as flexible as with the true Java types, but the ones that are available are typically of much use.

### String Methods

| Method | Description |
| --- | --- |
|startsWith(String)|Returns if the string starts with the passed string|
|startsWith(String, int)|Returns if the string starts with the passed string, after the specified position|
|endsWith(String)|Returns if the string ends with the passed string|
|indexOf(String)|Returns the first position of the passed string|
|indexOf(String, int)|Returns the position of the passed string, after the passed position|
|substring(int)|Returns the substring starting from the passed position|
|substring(int, int)|Returns the substring between the passed positions|
|toLowerCase()|Returns the string in lowercase|
|toUpperCase()|Returns the string in UPPERCASE|
|matches(String pattern)|Returns whether string matches the passed expression. The pattern argument follows the rules of java.lang.String.matches method.|
|trim()|Returns the string with leading/trailing spaces trimmed|
|length()|Returns the length of the string|
|charAt(int)|Returns the character at the specified position of the string|

Here’s an example using a Product class, looking for objects whose abbreviation is the beginning of a trade name. The trade name is provided as parameter.

```java
Declarative JDOQL :
Query query = pm.newQuery(mydomain.Product.class);
query.setFilter(":tradeName.startsWith(this.abbreviation)");
List results = (List)query.execute("Workbook Advanced");

Single-String JDOQL :
Query query = pm.newQuery(
    "SELECT FROM mydomain.Product " +
    "WHERE :tradeName.startsWith(this.abbreviation)");
List results = (List)query.execute("Workbook Advanced");
```

Here’s another example, demonstrating the "matches" method. Consult the javadocs for Java regular expressions for the syntax of the matches input.

```java
Declarative JDOQL :
Query query = pm.newQuery(mydomain.Product.class);
query.setFilter("this.abbreviation.matches(\\"a\*b\\")");
List results = (List)query.execute();

Single-String JDOQL :
Query query = pm.newQuery(
    "SELECT FROM mydomain.Product " +
    "WHERE this.abbreviation.matches(\\"a\*b\\")");
List results = (List)query.execute();
```

### Collection Methods

| Method | Description |
| --- | --- |
|isEmpty()|Returns whether the collection is empty|
|contains(value)|Returns whether the collection contains the passed element|
|size()|Returns the number of elements in the collection|

Here’s an example demonstrating use of contains(). We have an Inventory class that has a Collection of Product objects, and we want to find the Inventory objects with 2 particular Products in it. Here we make use of a variable (*prd* to represent the Product being contained

```java
Declarative JDOQL :
Query query = pm.newQuery(mydomain.Inventory.class);
query.setFilter("products.contains(prd) && (prd.name==\\"product 1\\" || prd.name==\\"product 2\\")");
List results = (List)query.execute();

Single-String JDOQL:
Query query = pm.newQuery(
    "SELECT FROM mydomain.Inventory EXCLUDE SUBCLASSES " +
    "WHERE products.contains(prd) && (prd.name==\\"product 1\\" || prd.name==\\"product 2\\")");
List results = (List)query.execute();
```

### List Methods

| Method | Description |
| --- | --- |
|get(position)|Returns the element at that position in the List (JDO3.1)|

### Map Methods

| Method | Description |
| --- | --- |
|isEmpty()| Returns whether the map is empty |
| containsKey(key)| Returns whether the map contains the passed key |
| containsValue(value)| Returns whether the map contains the passed value |
| get(key)| Returns the value from the map with the passed key |
| size()| Returns the number of entries in the map |

Here’s an example using a Product class as a value in a Map. Our example represents an organisation that has several Inventories of products. Each Inventory of products is stored using a Map, keyed by the Product name. The query searches for all Inventories that contain a product with the name "product 1".

```java
Declarative JDOQL :
Query query = pm.newQuery(mydomain.Inventory.class, "products.containsKey(\\"product 1\\")");
List results = (List)query.execute();

Single-String JDOQL :
Query query = pm.newQuery(
    "SELECT FROM mydomain.Inventory EXCLUDE SUBCLASSES " +
    "WHERE products.containsKey(\\"product 1\\")");
List results = (List)query.execute();
```

### Temporal Methods

The following methods apply to fields of temporal types.

| Class                   | Method | Description |
|-------------------------| --- | --- |
| java.util.Date          | getDate()| Returns the day of the month |
| java.util.Date          | getMonth()| Returns the month of the year |
| java.util.Date          | getYear()| Returns the year |
| java.util.Date          | getHour()| Returns the hour |
| java.util.Date          | getMinute()| Returns the minute |
| java.util.Date          | getSecond()| Returns the second |
| java.time.LocalDate     | getDayOfMonth()| Returns the day (of the month) for the date (1-31) in the timezone it was stored |
| java.time.LocalDate     | getMonthValue()| Returns the month for the date (1-12) in the timezone it was stored |
| java.time.LocalDate     | getYear()| Returns the year for the date in the timezone it was stored |
| java.time.LocalDateTime | getDayOfMonth()| Returns the day (of the month) for the date in the timezone it was stored |
| java.time.LocalDateTime | getMonthValue()| Returns the month for the date (1-12) in the timezone it was stored |
| java.time.LocalDateTime | getYear()| Returns the year for the date in the timezone it was stored |
| java.time.LocalDateTime | getHour()| Returns the hour for the time in the timezone it was stored |
| java.time.LocalDateTime | getMinute()| Returns the minute for the time in the timezone it was stored |
| java.time.LocalDateTime | getSecond()| Returns the second for the time in the timezone it was stored |
| java.time.LocalTime     | getHour()| Returns the hour for the time in the timezone it was stored |
| java.time.LocalTime     | getMinute()| Returns the minute for the time in the timezone it was stored |
| java.time.LocalTime     | getSecond()|Returns the second for the time in the timezone it was stored |

### Enum Methods

The following methods apply to fields of type *Enum*

| Method | Description |
| --- | --- |
|ordinal()| Returns the ordinal of the enum |
| toString()| Returns the string form of the enum |

### Optional Methods

The following methods apply to fields of type *Optional*

| Method | Description |
| --- | --- |
|get()| Returns the value of the field |
| isPresent()| Returns TRUE if the field is not null, and FALSE otherwise |
| orElse(subst)| Returns the value of the object if present, otherwise the "subst"|

### Other Methods

| Method | Description |
| --- | --- |
|Math.abs(number)| Returns the absolute value of the passed number |
| Math.sqrt(number)| Returns the square root of the passed number |
| Math.cos(number)| Returns the cosine of the passed number |
| Math.sin(number)| Returns the sine of the passed number |
| Math.tan(number)| Returns the tangent of the passed number |
| Math.acos(number)| Returns the arc cosine of the passed number |
| Math.asin(number)| Returns the arc sine of the passed number |
| Math.atan(number)| Returns the arc tangent of the passed number |
| Math.ceil(number)| Returns the ceiling of the passed number |
| Math.floor(number)| Returns the floor of the passed number |
| Math.log(number)| Returns the natural log of the passed number |
| Math.exp(number)| Returns the exponent of the passed number |
| JDOHelper.getObjectId(object)| Returns the object identity of the passed persistent object |
| JDOHelper.getVersion(object)| Returns the version of the passed persistent object |
