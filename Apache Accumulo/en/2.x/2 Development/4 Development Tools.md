[TOC]

Accumulo has several tools that can help developers test their code.

MiniAccumuloCluster
-------------------------------------------------------------------------------------------------------------

[MiniAccumuloCluster](https://static.javadoc.io/org.apache.accumulo/accumulo-minicluster/2.1.2/org/apache/accumulo/minicluster/MiniAccumuloCluster.html) is a standalone instance of Apache Accumulo for testing. It will create Zookeeper and Accumulo processes that write all of their data to a single local directory. [MiniAccumuloCluster](https://static.javadoc.io/org.apache.accumulo/accumulo-minicluster/2.1.2/org/apache/accumulo/minicluster/MiniAccumuloCluster.html) makes it easy to code against a real Accumulo instance. Developers can write realistic-to-end integration tests that mimic the use of a normal Accumulo instance.

[MiniAccumuloCluster](https://static.javadoc.io/org.apache.accumulo/accumulo-minicluster/2.1.2/org/apache/accumulo/minicluster/MiniAccumuloCluster.html) is published in a separate jar that should be added to your pom.xml as a test dependency:

```
<dependency>
  <groupId>org.apache.accumulo</groupId>
  <artifactId>accumulo-minicluster</artifactId>
  <version>${accumulo.version}</version>
  <scope>test</scope>
</dependency>
```

To start it up, you will need to supply an empty directory and a root password as arguments:

```
File tempDirectory = // JUnit and Guava supply mechanisms for creating temp directories
MiniAccumuloCluster mac = new MiniAccumuloCluster(tempDirectory, "password");
mac.start();
```

Once we have our mini cluster running, we will want to interact with the Accumulo client API:

```
AccumuloClient client = mac.getAccumuloClient("root", new PasswordToken("password"));
```

Upon completion of our development code, we will want to shutdown our MiniAccumuloCluster:

```
mac.stop();
// delete your temporary folder
```

Iterator Test Harness
-----------------------------------------------------------------------------------------------------------------

Iterators, while extremely powerful, are notoriously difficult to test. While the API defines the methods an Iterator must implement and each method’s functionality, the actual invocation of these methods by Accumulo TabletServers can be surprisingly difficult to mimic in unit tests.

The Apache Accumulo “Iterator Test Harness” is designed to provide a generalized testing framework for all Accumulo Iterators to leverage to identify common pitfalls in user-created Iterators.

### Framework Use

The Iterator Test Harness is published in a separate jar that should be added to your pom.xml as a test dependency:

```
<dependency>
  <groupId>org.apache.accumulo</groupId>
  <artifactId>accumulo-iterator-test-harness</artifactId>
  <version>${accumulo.version}</version>
  <scope>test</scope>
</dependency>
```

To use the Iterator test harness, create a class that extends the [IteratorTestBase](https://static.javadoc.io/org.apache.accumulo/accumulo-iterator-test-harness/2.1.2/org/apache/accumulo/iteratortest/IteratorTestBase.html) class and defines the following:

*   A `SortedMap` of input data (`Key`\-`Value` pairs)
*   A [Range](https://static.javadoc.io/org.apache.accumulo/accumulo-core/2.1.2/org/apache/accumulo/core/data/Range.html) to use in tests
*   A `Map` of options (`String` to `String` pairs)
*   A `SortedMap` of output data (`Key`\-`Value` pairs)
*   A list of [IteratorTestCase](https://static.javadoc.io/org.apache.accumulo/accumulo-iterator-test-harness/2.1.2/org/apache/accumulo/iteratortest/IteratorTestCase.html)s (these can be automatically discovered)

The majority of effort a user must make is in creating the input dataset and the expected output dataset for the iterator being tested.

### Normal Test Outline

Most iterator tests will follow the given outline:

```
import java.util.List;
import java.util.SortedMap;

import org.apache.accumulo.core.data.Key;
import org.apache.accumulo.core.data.Range;
import org.apache.accumulo.core.data.Value;
import org.apache.accumulo.iteratortest.IteratorTestBase;
import org.apache.accumulo.iteratortest.IteratorTestInput;
import org.apache.accumulo.iteratortest.IteratorTestOutput;
import org.apache.accumulo.iteratortest.IteratorTestParameters;

public class MyIteratorTest extends IteratorTestBase {

  @Override
  protected Stream<IteratorTestParameters> parameters() {
      var input = new IteratorTestInput(MyIterator.class, Map.of(), createRange(), INPUT_DATA);
      var expectedOutput = new IteratorTestOutput(OUTPUT_DATA);
      return builtinTestCases().map(test -> test.toParameters(input, expectedOutput));
  }

  private static SortedMap<Key,Value> INPUT_DATA = createInputData();
  private static SortedMap<Key,Value> OUTPUT_DATA = createOutputData();

  private static SortedMap<Key,Value> createInputData() {
    // TODO -- implement this method
  }

  private static SortedMap<Key,Value> createOutputData() {
    // TODO -- implement this method
  }

  private static Map<String,String> createOpts() {
    IteratorSetting setting = new IteratorSetting(50, MyIterator.class);
    // TODO -- add iterator specific options
    return setting.getOptions();
  }

  private static Range createRange() {
    // TODO -- implement this method
  }
}
```

### Limitations

While the classes that implement [IteratorTestCase](https://static.javadoc.io/org.apache.accumulo/accumulo-iterator-test-harness/2.1.2/org/apache/accumulo/iteratortest/IteratorTestCase.html)s should exercise common edge-cases in user iterators, there are still many limitations to the existing test harness. Some of them are:

*   Can only specify a single iterator, not many (a “stack”)
*   No control over provided IteratorEnvironment for tests
*   Exercising delete keys (especially with major compactions that do not include all files)

These are left as future improvements to the harness.
