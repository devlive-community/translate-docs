[TOC]

There are times when you need to replicate data between datastores. In many cases datastores themselves provide a means of doing this, however if you want to avoid using datastore-specific functionality you can utilise JDO to perform this task. JDO2 allows replication by use of detach/attach functionality. We demonstrate this with an example

```java
public class ElementHolder
{
long id;
private Set elements = new HashSet();

    ...
}

public class Element
{
String name;

    ...
}

public class SubElement extends Element
{
double value;

    ...
}
```

so we have a 1-N unidirectional (Set) relation, and we define the metadata like this

```xml
<jdo>
    <package name="org.apache.jdo.test">
        <class name="ElementHolder" identity-type="application" detachable="true">
            <inheritance strategy="new-table"/>
            <field name="id" primary-key="true"/>
            <field name="elements" persistence-modifier="persistent">
                <collection element-type="Element"/>
                <join/>
            </field>
        </class>

        <class name="Element" identity-type="application" detachable="true">
            <inheritance strategy="new-table"/>
            <field name="name" primary-key="true"/>
        </class>

        <class name="SubElement">
            <inheritance strategy="new-table"/>
            <field name="value"/>
        </class>
    </package>
</jdo>
```

and so in our application we create some objects in _datastore1_, like this

```java
PersistenceManagerFactory pmf1 = JDOHelper.getPersistenceManagerFactory("jdo.1.properties");
PersistenceManager pm1 = pmf1.getPersistenceManager();
Transaction tx1 = pm1.currentTransaction();
Object holderId = null;
try
        {
        tx1.begin();

ElementHolder holder = new ElementHolder(101);
    holder.addElement(new Element("First Element"));
        holder.addElement(new Element("Second Element"));
        holder.addElement(new SubElement("First Inherited Element"));
        holder.addElement(new SubElement("Second Inherited Element"));
        pm1.makePersistent(holder);

    tx1.commit();
holderId = JDOHelper.getObjectId(holder);
}
        finally
        {
        if (tx1.isActive())
        {
        tx1.rollback();
    }
            pm1.close();
}
```

and now we want to replicate these objects into _datastore2_, so we detach them from _datastore1_ and attach them to _datastore2_, like this

```java
// Detach the objects from "datastore1"
ElementHolder detachedHolder = null;
pm1 = pmf1.getPersistenceManager();
tx1 = pm1.currentTransaction();
try
{
    pm1.getFetchPlan().setGroups(new String[] {FetchPlan.DEFAULT, FetchPlan.ALL});
    pm1.getFetchPlan().setMaxFetchDepth(-1);

    tx1.begin();

    ElementHolder holder = (ElementHolder) pm1.getObjectById(holderID);
    detachedHolder = (ElementHolder) pm1.detachCopy(holder);

    tx1.commit();
}
finally
{
    if (tx1.isActive())
    {
        tx1.rollback();
    }
    pm1.close();
}

// Attach the objects to datastore2
PersistenceManagerFactory pmf2 = JDOHelper.getPersistenceManagerFactory("jdo.2.properties");
PersistenceManager pm2 = pmf2.getPersistenceManager();
Transaction tx2 = pm2.currentTransaction();
try
{
    tx2.begin();

    pm2.makePersistent(detachedHolder);

    tx2.commit();
}
finally
{
    if (tx2.isActive())
    {
        tx2.rollback();
    }
    pm2.close();
}
```

These objects are now replicated into _datastore2_. Clearly you can extend this basic idea and replicate large amounts of data.

***
