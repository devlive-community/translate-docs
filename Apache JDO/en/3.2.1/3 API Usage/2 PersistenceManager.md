[TOC]

Any JDO-enabled application will require at least one _PersistenceManager_ (PM). This is obtained from the [PersistenceManagerFactory](https://db.apache.org/jdo/pmf.html) for the datastore.

The simplest way of creating a _PersistenceManager_ [![Image 1: image](https://db.apache.org/jdo/images/javadoc.png)](https://db.apache.org/jdo/api32/apidocs/javax/jdo/PersistenceManager.html) is as follows

    PersistenceManagerFactory pmf = JDOHelper.getPersistenceManagerFactory(props);
    PersistenceManager pm = pmf.getPersistenceManager();

A _PersistenceManager_ is the key to all persistence operations in JDO. With it you can persist, update, delete, and retrieve objects from the datastore. A _PersistenceManager_ has a single transaction.

### Persist Objects

To persist an object, the object must first be marked as persistable using [MetaData (XML/Annotations)](https://db.apache.org/jdo/metadata.html). Then you would start the PM transaction, and use _makePersistent_ as follows

    PersistenceManager pm = pmf.getPersistenceManager();
    Transaction tx = pm.currentTransaction();
    try
    {
        // Start the transaction
        tx.begin();
    
        // Create the object to persist
        MyClass obj = new MyClass();
    
        // Persist it to the datastore
        pm.makePersistent(obj);
    
        // Commit the transaction, flushing the object to the datastore
        tx.commit();
    }
    catch (Exception e)
    {
        ... handle exceptions
    }
    finally
    {
        if (tx.isActive())
        {
            // Error occurred so rollback the transaction
            tx.rollback();
        }
        pm.close();
    }

The _makePersistent_ method of **PersistenceManager** makes the object persistent in the datastore, and updates the 'state' of the object from _Transient_ (at the start) to _Hollow_ (after commit() of the transaction).

When an object is persisted, if it has any other objects referenced from that object they also will be made persistent. This is referred to as **persistence-by-reachability**. The main benefit of this is that if you have an object graph to persist, then you don’t need to call _makePersistent()_ on all objects, instead just using one that can be used to find all of the others. **persistence-by-reachability** is also run at the time of calling _commit()_ on the transaction. This has the effect that if you had called _makePersistent()_ on an object and that had persisted another object, and before commit you had removed the relation to this other object, then at _commit()_ the reachability algorithm will find that this other object is no longer reachable and will remove it from persistence.

### Retrieve Objects

So we’ve made some of our objects persistent, and now we want to retrieve them in our application. Here’s one way of retrieving objects of a particular type.

    tx = pm.currentTransaction();
    try
    {
        tx.begin();
    
        Extent e = pm.getExtent(mydomain.MyClass.class, true);
        Iterator iter=e.iterator();
        while (iter.hasNext())
        {
            MyClass my_obj=(MyClass)iter.next();
            ...
        }
    
        tx.commit();
    }
    catch (Exception e)
    {
        if (tx.isActive())
        {
            tx.rollback();
        }
    }

The **Extent** interface is one of the ways to retrieve your objects. The others use the **Query** interface, allowing more precise filtering over the objects returned.

### Update Objects

To update an object we firstly retrieve it, as above, and then we call any of its mutator methods. For example

    tx = pm.currentTransaction();
    try
    {
        tx.begin();
    
        Extent e = pm.getExtent(mydomain.MyClass.class, true);
        Iterator iter=e.iterator();
        while (iter.hasNext())
        {
            MyClass my_obj=(MyClass)iter.next();
            my_obj.setValue(25.0); // Change the value
            ...
        }
    
        tx.commit();
    }
    catch (Exception e)
    {
        if (tx.isActive())
        {
            tx.rollback();
        }
    }

When _setValue()_ is called on the persistent object this change is intercepted by JDO and the value change will be automatically sent to the datastore …​ transparently!

### Delete Objects

So we can persist objects, and retrieve them. Now we want to remove one from persistence.

    try
    {
        tx = pm.currentTransaction();
        tx.begin();
    
        ... (code to retrieve object in question) ...
    
        pm.deletePersistent(my_obj);
    
        tx.commit();
    }
    catch (Exception e)
    {
        if (tx.isActive())
        {
            tx.rollback();
        }
    }

### Making an object transient

As we have seen in the [JDO States guide](https://db.apache.org/jdo/state_transition.html), an object can have many possible states. When we want to take an object and work on it, but removing its identity we can make it **transient**. This means that it will retain the values of its fields, yet will no longer be associated with the object in the datastore. We do this as follows

    try
    {
        tx = pm.currentTransaction();
        tx.begin();
    
        ... (code to retrieve object in question) ...
    
        pm.makeTransient(my_obj);
    
        tx.commit();
    }
    catch (Exception e)
    {
        if (tx.isActive())
        {
            tx.rollback();
        }
    }
    
    ... (code to work on "my_obj")