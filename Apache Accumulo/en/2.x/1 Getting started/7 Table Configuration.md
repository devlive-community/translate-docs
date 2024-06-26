[TOC]

Accumulo tables have a few options that can be configured to alter the default behavior of Accumulo as well as improve performance based on the data stored. These include locality groups, constraints, bloom filters, iterators, and block cache. See the [server properties](https://accumulo.apache.org/docs/2.x/configuration/server-properties) documentation for a complete list of available configuration options.

Locality Groups
-----------------------------------------------------------------------------------------------------------

Accumulo supports storing sets of column families separately on disk to allow clients to efficiently scan over columns that are frequently used together and to avoid scanning over column families that are not requested. After a locality group is set, [Scanner](https://static.javadoc.io/org.apache.accumulo/accumulo-core/2.1.2/org/apache/accumulo/core/client/Scanner.html) and [BatchScanner](https://static.javadoc.io/org.apache.accumulo/accumulo-core/2.1.2/org/apache/accumulo/core/client/BatchScanner.html) operations will automatically take advantage of them whenever the fetchColumnFamilies() method is used.

By default, tables place all column families into the same `default` locality group. Additional locality groups can be configured at any time via the shell or programmatically as follows:

### Managing Locality Groups via the Shell

```
usage: setgroups <group>=<col fam>{,<col fam>}{ <group>=<col fam>{,<col fam>}}
    [-?] -t <table>

user@myinstance mytable> setgroups group_one=colf1,colf2 -t mytable

user@myinstance mytable> getgroups -t mytable
```

### Managing Locality Groups via the Client API

```
AccumuloClient client = Accumulo.newClient()
                          .from("/path/to/accumulo-client.properties").build();

HashMap<String,Set<Text>> localityGroups = new HashMap<String, Set<Text>>();

HashSet<Text> metadataColumns = new HashSet<Text>();
metadataColumns.add(new Text("domain"));
metadataColumns.add(new Text("link"));

HashSet<Text> contentColumns = new HashSet<Text>();
contentColumns.add(new Text("body"));
contentColumns.add(new Text("images"));

localityGroups.put("metadata", metadataColumns);
localityGroups.put("content", contentColumns);

client.tableOperations().setLocalityGroups("mytable", localityGroups);

// existing locality groups can be obtained as follows
Map<String, Set<Text>> groups = client.tableOperations().getLocalityGroups("mytable");
```

The assignment of Column Families to Locality Groups can be changed at any time. The physical movement of column families into their new locality groups takes place via the periodic [major compaction](https://accumulo.apache.org/docs/2.x/getting-started/table_configuration#compaction) process that takes place continuously in the background or manually using the `compact` command in the shell.

Constraints
---------------------------------------------------------------------------------------------------

Accumulo supports constraints applied on mutations at insert time. This can be used to disallow certain inserts according to a user defined policy. Any mutation that fails to meet the requirements of the constraint is rejected and sent back to the client.

Constraints can be enabled by setting a table property as follows:

```
user@myinstance mytable> constraint -t mytable -a com.test.ExampleConstraint com.test.AnotherConstraint

user@myinstance mytable> constraint -l
com.test.ExampleConstraint=1
com.test.AnotherConstraint=2
```

Currently, there are no general-purpose constraints provided with the Accumulo distribution. New constraints can be created by writing a Java class that implements the [Constraint](https://static.javadoc.io/org.apache.accumulo/accumulo-core/2.1.2/org/apache/accumulo/core/constraints/Constraint.html) interface.

To deploy a new constraint, create a jar file containing a class implementing [Constraint](https://static.javadoc.io/org.apache.accumulo/accumulo-core/2.1.2/org/apache/accumulo/core/constraints/Constraint.html) and place it in the `lib/` directory of the Accumulo installation. New constraint jars can be added to Accumulo and enabled without restarting but any change to an existing constraint class requires Accumulo to be restarted.

See the [constraints examples](https://github.com/apache/accumulo-examples/blob/main/docs/constraints.md) for example code.

Bloom Filters
-------------------------------------------------------------------------------------------------------

As mutations are applied to an Accumulo table, several files are created per tablet. If bloom filters are enabled, Accumulo will create and load a small data structure into memory to determine whether a file contains a given key before opening the file. This can speed up lookups considerably. Bloom filters can be enabled on a table by setting the [table.bloom.enabled](https://accumulo.apache.org/docs/2.x/configuration/server-properties#table_bloom_enabled) property to `true` in the shell:

```
user@myinstance> config -t mytable -s table.bloom.enabled=true
```

The [bloom filter examples](https://github.com/apache/accumulo-examples/blob/main/docs/bloom.md) contains an extensive example of using Bloom Filters.

Iterators
-----------------------------------------------------------------------------------------------

Iterators provide a modular mechanism for adding functionality to be executed by TabletServers when scanning or compacting data. This allows users to efficiently summarize, filter, and aggregate data. In fact, the built-in features of cell-level security and column fetching are implemented using Iterators. Some useful Iterators are provided with Accumulo and can be found in the [org.apache.accumulo.core.iterators.user](https://static.javadoc.io/org.apache.accumulo/accumulo-core/2.1.2/org/apache/accumulo/core/iterators/user/package-summary.html) package. In each case, any custom Iterators must be included in Accumulo’s classpath, typically by including a jar in `lib/` or `lib/ext/`, although the VFS classloader allows for classpath manipulation using a variety of schemes including URLs and HDFS URIs.

### Setting Iterators via the Shell

Iterators can be configured on a table at scan, minor compaction and/or major compaction scopes. If the Iterator implements the [OptionDescriber](https://static.javadoc.io/org.apache.accumulo/accumulo-core/2.1.2/org/apache/accumulo/core/iterators/OptionDescriber.html) interface, the setiter command can be used which will interactively prompt the user to provide values for the given necessary options.

```
usage: setiter [-?] -ageoff | -agg | -class <name> | -regex |
    -reqvis | -vers   [-majc] [-minc] [-n <itername>] -p <pri>
    [-scan] [-t <table>]

user@myinstance mytable> setiter -t mytable -scan -p 15 -n myiter -class com.company.MyIterator
```

The config command can always be used to manually configure iterators which is useful in cases where the Iterator does not implement the [OptionDescriber](https://static.javadoc.io/org.apache.accumulo/accumulo-core/2.1.2/org/apache/accumulo/core/iterators/OptionDescriber.html) interface.

```
config -t mytable -s table.iterator.scan.myiter=15,com.company.MyIterator
config -t mytable -s table.iterator.minc.myiter=15,com.company.MyIterator
config -t mytable -s table.iterator.majc.myiter=15,com.company.MyIterator
config -t mytable -s table.iterator.scan.myiter.opt.myoptionname=myoptionvalue
config -t mytable -s table.iterator.minc.myiter.opt.myoptionname=myoptionvalue
config -t mytable -s table.iterator.majc.myiter.opt.myoptionname=myoptionvalue
```

Typically, a table will have multiple iterators. Accumulo configures a set of system level iterators for each table. These iterators provide core functionality like visibility label filtering and may not be removed by users. User level iterators are applied in the order of their priority. Priority is a user configured integer; iterators with lower numbers go first, passing the results of their iteration on to the other iterators up the stack.

### Setting Iterators Programmatically

```
scanner.addIterator(new IteratorSetting(
    15, // priority
    "myiter", // name this iterator
    "com.company.MyIterator" // class name
));
```

Some iterators take additional parameters from client code, as in the following example:

```
IteratorSetting iter = new IteratorSetting(...);
iter.addOption("myoptionname", "myoptionvalue");
scanner.addIterator(iter)
```

Tables support separate Iterator settings to be applied at scan time, upon minor compaction and upon major compaction. For most uses, tables will have identical iterator settings for all three to avoid inconsistent results.

### Versioning Iterators and Timestamps

Accumulo provides the capability to manage versioned data through the use of timestamps within the Key. If a timestamp is not specified in the key created by the client then the system will set the timestamp to the current time. Two keys with identical rowIDs and columns but different timestamps are considered two versions of the same key. If two inserts are made into Accumulo with the same rowID, column, and timestamp, then the behavior is non-deterministic.

Timestamps are sorted in descending order, so the most recent data comes first. Accumulo can be configured to return the top k versions, or versions later than a given date. The default is to return the one most recent version.

The version policy can be changed by changing the VersioningIterator options for a table as follows:

```
user@myinstance mytable> config -t mytable -s table.iterator.scan.vers.opt.maxVersions=3

user@myinstance mytable> config -t mytable -s table.iterator.minc.vers.opt.maxVersions=3

user@myinstance mytable> config -t mytable -s table.iterator.majc.vers.opt.maxVersions=3
```

When a table is created, by default it’s configured to use the VersioningIterator and keep one version. A table can be created without the VersioningIterator with the -ndi option in the shell. Also, the Java API has the following method

```
client.tableOperations.create(String tableName, boolean limitVersion);
```

#### Logical Time

Accumulo 1.2 introduces the concept of logical time. This ensures that timestamps set by Accumulo always move forward. This helps avoid problems caused by TabletServers that have different time settings. The per tablet counter gives unique one up time stamps on a per-mutation basis. When using time in milliseconds, if two things arrive within the same millisecond then both receive the same timestamp. When using time in milliseconds, Accumulo set times will still always move forward and never backwards.

A table can be configured to use logical timestamps at creation time as follows:

```
user@myinstance> createtable -tl logical
```

#### Deletes

Deletes are special keys in Accumulo that get sorted along will all the other data. When a delete key is inserted, Accumulo will not show anything that has a timestamp less than or equal to the delete key. During major compaction, any keys older than a delete key are omitted from the new file created, and the omitted keys are removed from disk as part of the regular garbage collection process.

### Filters

When scanning over a set of key-value pairs it is possible to apply an arbitrary filtering policy through the use of a [Filter](https://static.javadoc.io/org.apache.accumulo/accumulo-core/2.1.2/org/apache/accumulo/core/iterators/Filter.html). Filters are types of iterators that return only key-value pairs that satisfy the filter logic. Accumulo has a few built-in filters that can be configured on any table: AgeOff, ColumnAgeOff, Timestamp, NoVis, and RegEx. More can be added by writing a Java class that extends the [Filter](https://static.javadoc.io/org.apache.accumulo/accumulo-core/2.1.2/org/apache/accumulo/core/iterators/Filter.html) class.

The AgeOff filter can be configured to remove data older than a certain date or a fixed amount of time from the present. The following example sets a table to delete everything inserted over 30 seconds ago:

```
user@myinstance> createtable filtertest

user@myinstance filtertest> setiter -t filtertest -scan -minc -majc -p 10 -n myfilter -ageoff
AgeOffFilter removes entries with timestamps more than <ttl> milliseconds old
----------> set org.apache.accumulo.core.iterators.user.AgeOffFilter parameter negate, default false
                keeps k/v that pass accept method, true rejects k/v that pass accept method:
----------> set org.apache.accumulo.core.iterators.user.AgeOffFilter parameter ttl, time to
                live (milliseconds): 30000
----------> set org.apache.accumulo.core.iterators.user.AgeOffFilter parameter currentTime, if set,
                use the given value as the absolute time in milliseconds as the current time of day:

user@myinstance filtertest>

user@myinstance filtertest> scan

user@myinstance filtertest> insert foo a b c

user@myinstance filtertest> scan
foo a:b [] c

user@myinstance filtertest> sleep 4

user@myinstance filtertest> scan

user@myinstance filtertest>
```

To see the iterator settings for a table, use:

```
user@example filtertest> config -t filtertest -f iterator
---------+---------------------------------------------+------------------
SCOPE    | NAME                                        | VALUE
---------+---------------------------------------------+------------------
table    | table.iterator.majc.myfilter .............. | 10,org.apache.accumulo.core.iterators.user.AgeOffFilter
table    | table.iterator.majc.myfilter.opt.ttl ...... | 30000
table    | table.iterator.majc.vers .................. | 20,org.apache.accumulo.core.iterators.VersioningIterator
table    | table.iterator.majc.vers.opt.maxVersions .. | 1
table    | table.iterator.minc.myfilter .............. | 10,org.apache.accumulo.core.iterators.user.AgeOffFilter
table    | table.iterator.minc.myfilter.opt.ttl ...... | 30000
table    | table.iterator.minc.vers .................. | 20,org.apache.accumulo.core.iterators.VersioningIterator
table    | table.iterator.minc.vers.opt.maxVersions .. | 1
table    | table.iterator.scan.myfilter .............. | 10,org.apache.accumulo.core.iterators.user.AgeOffFilter
table    | table.iterator.scan.myfilter.opt.ttl ...... | 30000
table    | table.iterator.scan.vers .................. | 20,org.apache.accumulo.core.iterators.VersioningIterator
table    | table.iterator.scan.vers.opt.maxVersions .. | 1
---------+---------------------------------------------+------------------
```

### Combiners

Accumulo supports on the fly lazy aggregation of data using Combiners. Aggregation is done at compaction and scan time. No lookup is done at insert time, which\` greatly speeds up ingest.

Accumulo allows Combiners to be configured on tables and column families. When a Combiner is set it is applied across the values associated with any keys that share rowID, column family, and column qualifier. This is similar to the reduce step in MapReduce, which applied some function to all the values associated with a particular key.

For example, if a summing combiner were configured on a table and the following mutations were inserted:

```
Row     Family Qualifier Timestamp  Value
rowID1  colfA  colqA     20100101   1
rowID1  colfA  colqA     20100102   1
```

The table would reflect only one aggregate value:

Combiners can be enabled for a table using the setiter command in the shell. Below is an example.

```
root@a14 perDayCounts> setiter -t perDayCounts -p 10 -scan -minc -majc -n daycount
                       -class org.apache.accumulo.core.iterators.user.SummingCombiner
TypedValueCombiner can interpret Values as a variety of number encodings
  (VLong, Long, or String) before combining
----------> set SummingCombiner parameter columns,
            <col fam>[:<col qual>]{,<col fam>[:<col qual>]} : day
----------> set SummingCombiner parameter type, <VARNUM|LONG|STRING>: STRING

root@a14 perDayCounts> insert foo day 20080101 1
root@a14 perDayCounts> insert foo day 20080101 1
root@a14 perDayCounts> insert foo day 20080103 1
root@a14 perDayCounts> insert bar day 20080101 1
root@a14 perDayCounts> insert bar day 20080101 1

root@a14 perDayCounts> scan
bar day:20080101 []    2
foo day:20080101 []    2
foo day:20080103 []    1
```

Accumulo includes some useful Combiners out of the box. To find these look in the [org.apache.accumulo.core.iterators.user](https://static.javadoc.io/org.apache.accumulo/accumulo-core/2.1.2/org/apache/accumulo/core/iterators/user/package-summary.html) package.

Additional Combiners can be added by creating a Java class that extends [Combiner](https://static.javadoc.io/org.apache.accumulo/accumulo-core/2.1.2/org/apache/accumulo/core/iterators/Combiner.html) and adding a jar containing that class to Accumulo’s `lib/ext` directory.

See the [combiner example](https://github.com/apache/accumulo-examples/blob/main/docs/combiner.md) for example code.

Block Cache
---------------------------------------------------------------------------------------------------

A Block Cache can be enabled on tables to limit reads from disk which can result in reduced read latency. Read the [Caching]($Caching) documentation to learn more.

Compaction
-------------------------------------------------------------------------------------------------

See [compaction]($Compactions)

Pre-splitting tables
---------------------------------------------------------------------------------------------------------------------

Accumulo will balance and distribute tables across servers. Before a table gets large, it will be maintained as a single tablet on a single server. This limits the speed at which data can be added or queried to the speed of a single node. To improve performance when a table is new, or small, you can add split points and generate new tablets.

In the shell:

```
root@myinstance> createtable newTable
root@myinstance> addsplits -t newTable g n t
```

This will create a new table with 4 tablets. The table will be split on the letters `g`, `n`, and `t` which will work nicely if the row data start with lower-case alphabetic characters. If your row data includes binary information or numeric information, or if the distribution of the row information is not flat, then you would pick different split points. Now ingest and query can proceed on 4 nodes which can improve performance.

Merging tablets
-----------------------------------------------------------------------------------------------------------

Over time, a table can get very large, so large that it has hundreds of thousands of split points. Once there are enough tablets to spread a table across the entire cluster, additional splits may not improve performance, and may create unnecessary bookkeeping. The distribution of data may change over time. For example, if row data contains date information, and data is continually added and removed to maintain a window of current information, tablets for older rows may be empty.

Accumulo supports tablet merging, which can be used to reduce the number of split points. The following command will merge all rows from `A` to `Z` into a single tablet:

```
root@myinstance> merge -t myTable -s A -e Z
```

If the result of a merge produces a tablet that is larger than the configured split size, the tablet may be split by the tablet server. Be sure to increase your tablet size prior to any merges if the goal is to have larger tablets:

```
root@myinstance> config -t myTable -s table.split.threshold=2G
```

In order to merge small tablets, you can ask Accumulo to merge sections of a table smaller than a given size.

```
root@myinstance> merge -t myTable -s 100M
```

By default, small tablets will not be merged into tablets that are already larger than the given size. This can leave isolated small tablets. To force small tablets to be merged into larger tablets use the `--force` option:

```
root@myinstance> merge -t myTable -s 100M --force
```

Merging away small tablets works on one section at a time. If your table contains many sections of small split points, or you are attempting to change the split size of the entire table, it will be faster to set the split point and merge the entire table:

```
root@myinstance> config -t myTable -s table.split.threshold=256M
root@myinstance> merge -t myTable
```

Delete Range
-----------------------------------------------------------------------------------------------------

Consider an indexing scheme that uses date information in each row. For example `20110823-15:20:25.013` might be a row that specifies a date and time. In some cases, we might like to delete rows based on this date, say to remove all the data older than the current year. Accumulo supports a delete range operation which efficiently removes data between two rows. For example:

```
root@myinstance> deleterange -t myTable -s 2010 -e 2011
```

This will delete all rows starting with `2010` and it will stop at any row starting `2011`. You can delete any data prior to 2011 with:

```
root@myinstance> deleterange -t myTable -e 2011 --force
```

The shell will not allow you to delete an unbounded range (no start) unless you provide the `--force` option.

Range deletion is implemented using splits at the given start/end positions, and will affect the number of splits in the table.

Cloning Tables
---------------------------------------------------------------------------------------------------------

A new table can be created that points to an existing table’s data. This is a very quick metadata operation, no data is actually copied. The cloned table and the source table can change independently after the clone operation. One use case for this feature is testing. For example to test a new filtering iterator, clone the table, add the filter to the clone, and force a major compaction. To perform a test on less data, clone a table and then use delete range to efficiently remove a lot of data from the clone. Another use case is generating a snapshot to guard against human error. To create a snapshot, clone a table and then disable write permissions on the clone.

The clone operation will point to the source table’s files. This is why the flush option is present and is enabled by default in the shell. If the flush option is not enabled, then any data the source table currently has in memory will not exist in the clone.

A cloned table copies the configuration of the source table. However, the permissions of the source table are not copied to the clone. After a clone is created, only the user that created the clone can read and write to it.

In the following example we see that data inserted after the clone operation is not visible in the clone.

```
root@a14> createtable people

root@a14 people> insert 890435 name last Doe
root@a14 people> insert 890435 name first John

root@a14 people> clonetable people test

root@a14 people> insert 890436 name first Jane
root@a14 people> insert 890436 name last Doe

root@a14 people> scan
890435 name:first []    John
890435 name:last []    Doe
890436 name:first []    Jane
890436 name:last []    Doe

root@a14 people> table test

root@a14 test> scan
890435 name:first []    John
890435 name:last []    Doe

root@a14 test>
```

The du command in the shell shows how much space a table is using in HDFS. This command can also show how much overlapping space two cloned tables have in HDFS. In the example below du shows table ci is using 428M. Then ci is cloned to cic and du shows that both tables share 428M. After three entries are inserted into cic and its flushed, du shows the two tables still share 428M but cic has 226 bytes to itself. Finally, table cic is compacted and then du shows that each table uses 428M.

```
root@a14> du ci
             428,482,573 [ci]

root@a14> clonetable ci cic

root@a14> du ci cic
             428,482,573 [ci, cic]

root@a14> table cic

root@a14 cic> insert r1 cf1 cq1 v1
root@a14 cic> insert r1 cf1 cq2 v2
root@a14 cic> insert r1 cf1 cq3 v3

root@a14 cic> flush -t cic -w
27 15:00:13,908 [shell.Shell] INFO : Flush of table cic completed.

root@a14 cic> du ci cic
             428,482,573 [ci, cic]
                     226 [cic]

root@a14 cic> compact -t cic -w
27 15:00:35,871 [shell.Shell] INFO : Compacting table ...
27 15:03:03,303 [shell.Shell] INFO : Compaction of table cic completed for given range

root@a14 cic> du ci cic
             428,482,573 [ci]
             428,482,612 [cic]

root@a14 cic>
```

Exporting Tables
-------------------------------------------------------------------------------------------------------------

Accumulo supports exporting tables for the purpose of copying tables to another cluster. Exporting and importing tables preserves the tables’ configuration, splits, and logical time. Tables are exported and then copied via the hadoop `distcp` command. To export a table, it must be offline and stay offline while `distcp` runs. Staying offline prevents files from being deleted during the process. An easy way to take a table offline without interrupting access is to clone it and take the clone offline.

### Table Import/Export Example

The following example demonstrates Accumulo’s mechanism for exporting and importing tables.

The shell session below illustrates creating a table, inserting data, and exporting the table.

```
root@test15> createtable table1
root@test15 table1> insert a cf1 cq1 v1
root@test15 table1> insert h cf1 cq1 v2
root@test15 table1> insert z cf1 cq1 v3
root@test15 table1> insert z cf1 cq2 v4
root@test15 table1> addsplits -t table1 b r
root@test15 table1> scan
a cf1:cq1 []    v1
h cf1:cq1 []    v2
z cf1:cq1 []    v3
z cf1:cq2 []    v4
root@test15> config -t table1 -s table.split.threshold=100M
root@test15 table1> clonetable table1 table1_exp
root@test15 table1> offline table1_exp
root@test15 table1> exporttable -t table1_exp /tmp/table1_export
root@test15 table1> quit
```

After executing the export command, a few files are created in the hdfs dir. One of the files is a list of files to distcp as shown below.

```
$ hadoop fs -ls /tmp/table1_export
Found 2 items
-rw-r--r--   3 user supergroup        162 2012-07-25 09:56 /tmp/table1_export/distcp.txt
-rw-r--r--   3 user supergroup        821 2012-07-25 09:56 /tmp/table1_export/exportMetadata.zip
$ hadoop fs -cat /tmp/table1_export/distcp.txt
hdfs://n1.example.com:6093/accumulo/tables/3/default_tablet/F0000000.rf
hdfs://n1.example.com:6093/tmp/table1_export/exportMetadata.zip
```

Before the table can be imported, it must be copied using `distcp`. After the `distcp` completes, the cloned table may be deleted.

```
$ hadoop distcp -f /tmp/table1_export/distcp.txt /tmp/table1_export_dest
```

The Accumulo shell session below shows importing the table and inspecting it. The data, splits, config, and logical time information for the table were preserved.

```
root@test15> importtable table1_copy /tmp/table1_export_dest
root@test15> table table1_copy
root@test15 table1_copy> scan
a cf1:cq1 []    v1
h cf1:cq1 []    v2
z cf1:cq1 []    v3
z cf1:cq2 []    v4
root@test15 table1_copy> getsplits -t table1_copy
b
r
root@test15> config -t table1_copy -f split
---------+--------------------------+-------------------------------------------
SCOPE    | NAME                     | VALUE
---------+--------------------------+-------------------------------------------
default  | table.split.threshold .. | 1G
table    |    @override ........... | 100M
---------+--------------------------+-------------------------------------------
root@test15> tables -l
accumulo.metadata    =>        !0
accumulo.root        =>        +r
table1_copy          =>         5
trace                =>         1
root@test15 table1_copy> scan -t accumulo.metadata -b 5 -c srv:time
5;b srv:time []    M1343224500467
5;r srv:time []    M1343224500467
5< srv:time []    M1343224500467
```
