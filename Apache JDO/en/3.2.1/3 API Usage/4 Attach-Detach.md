[TOC]

JDO provides an interface to the persistence of objects. JDO 1.0 didn’t provide a way of taking an object that was just persisted and just work on it and update the persisted object later; the user had to copy the fields manually and copy them back to the persisted object later. JDO 2.0 introduced a new way of handling this situation, by **detaching** an object from the persistence graph, allowing it to be worked on in the users application. It could then be **attached** to the persistence graph later. Please refer to [Object Lifecycle](https://db.apache.org/jdo/state_transition.html) for where this fits in. The first thing to do to use a class with this facility is to tag it as "detachable". This is done by adding the attribute

    <class name="MyClass" detachable="true">

This acts as an instruction to the [enhancement process](https://db.apache.org/jdo/enhancement.html) to add methods necessary to utilise the attach/detach process.

The following code fragment highlights how to use the attach/detach mechanism

    Product working_product=null;
    Transaction tx=pm.currentTransaction();
    try
    {
        tx.begin();
    
        Product prod=new Product(name,description,price);
        pm.makePersistent(prod);
    
        // Detach the product for use
        working_product = (Product)pm.detachCopy(prod);
    
        tx.commit();
    }
    catch (Exception e)
    {
        // Handle the exception
    }
    finally
    {
        if (tx.isActive())
        {
            tx.rollback();
        }
    }
    
    // Work on the detached object in our application
    working_product.setPrice(new_price);
    
    ...
    
    // Reattach the updated object
    tx = pm.currentTransaction();
    try
    {
        tx.begin();
    
        Product attached_product = pm.makePersistent(working_product);
    
        tx.commit();
    }
    catch (Exception e)
    {
        // Handle the exception
    }
    finally
    {
        if (tx.isActive())
        {
            tx.rollback();
        }
    }

So we now don’t need to do any manual copying of object fields just using a simple call to detach the object, and then attach it again later. Here are a few things to note with _attach/detach_ :

*   Calling _detachCopy_ on an object that is not detachable will return a **transient** instance that is a COPY of the original, so use the COPY thereafter.

*   Calling _detachCopy_ on an object that is detachable will return a **detached** instance that is a COPY of the original, so use this COPY thereafter

*   A _detached_ object retain the id of its datastore entity. Detached objects should be used where you want to update the objects and attach them later (updating the associated object in the datastore. If you want to create copies of the objects in the datastore with their own identities you should use _makeTransient_ instead of _detachCopy_.

*   Calling _detachCopy_ will detach all fields of that object that are in the current [Fetch Group](https://db.apache.org/jdo/fetchgroups.html) for that class for that _PersistenceManager_.

*   By default the fields of the object that will be detached are those in the _Default Fetch Group_.

*   You should choose your [Fetch Group](https://db.apache.org/jdo/fetchgroups.html) carefully, bearing in mind which object(s) you want to access whilst detached. Detaching a relation field will detach the related object as well.

*   If you don’t detach a field of an object, you cannot access the value for that field while the object is detached.

*   If you don’t detach a field of an object, you can update the value for that field while detached, and thereafter you can access the value for that field.

*   Calling _makePersistent_ will return an (attached) copy of the detached object. It will attach all fields that were originally detached, and will also attach any other fields that were modified whilst detached.


### Detach All On Commit

JDO also provides a mechanism whereby all objects that were enlisted in a transaction are automatically detached when the transaction is committed. You can enable this in one of 3 ways. If you want to use this mode globally for all _PersistenceManager_s (PMs) from a _PersistenceManagerFactory_ (PMF) you could either set the PMF property "datanucleus.DetachAllOnCommit", or you could create your PMF and call the PMF method **setDetachAllOnCommit(true)**. If instead you wanted to use this mode only for a particular PM, or only for a particular transaction for a particular PM, then you can call the PM method **setDetachAllOnCommit(true)** before the commit of the transaction, and it will apply for all transaction commits thereafter, until turned off (**setDetachAllOnCommit(false)**. Here’s an example

    // Create a PMF
    ...
    
    // Create an object
    MyObject my = new MyObject();
    
    PersistenceManager pm = pmf.getPersistenceManager();
    Transaction tx = pm.currentTransaction();
    try
    {
        tx.begin();
    
        // We want our object to be detached when it's been persisted
        pm.setDetachAllOnCommit(true);
    
        // Persist the object that we created earlier
        pm.makePersistent(my);
    
        tx.commit();
        // The object "my" is now in detached state and can be used further
    }
    finally
    {
        if (tx.isActive)
        {
            tx.rollback();
        }
    }

### Copy On Attach

By default when you are attaching a detached object it will return an attached copy of the detached object. JDO2.1 provides a new feature that allows this attachment to just migrate the existing detached object into attached state.

You enable this by setting the _PersistenceManagerFactory_ (PMF) property **datanucleus.CopyOnAttach** to false. Alternatively you can use the methods _PersistenceManagerFactory.setCopyOnAttach(boolean flag)_ or _PersistenceManager.setCopyOnAttach(boolean flag)_. If we return to the example at the start of this page, this now becomes

    // Reattach the updated object
    pm.setCopyOnAttach(false);
    tx = pm.currentTransaction();
    try
    {
        tx.begin();
    
        // working product is currently in detached state
    
        pm.makePersistent(working_product);
        // working_product is now in persistent (attached) state
    
        tx.commit();
    }
    catch (Exception e)
    {
        // Handle the exception
    }
    finally
    {
        if (tx.isActive())
        {
            tx.rollback();
        }
    }

Please note that if you try to attach two detached objects representing the same underlying persistent object within the same transaction (i.e a persistent object with the same identity already exists in the level 1 cache), then a JDOUserException will be thrown.

### Serialization of Detachable classes

During enhancement of Detachable classes, a field called _jdoDetachedState_ is added to the class definition. This field allows reading and changing tracking of detached objects while they are not managed by a PersistenceManager.

When serialization occurs on a Detachable object, the _jdoDetachedState_ field is written to the serialized object stream. On deserialize, this field is written back to the new deserialized instance. This process occurs transparently to the application. However, if deserialization occurs with an un-enhanced version of the class, the detached state is lost.

Serialization and deserialization of Detachable classes and un-enhanced versions of the same class is only possible if the field _serialVersionUID_ is added. It’s recommended during development of the class, to define the _serialVersionUID_ and make the class to implement the _java.io.Serializable_ interface, as the following example:

    class MyClass implements java.io.Serializable
    {
        private static final long serialVersionUID = 2765740961462495537L; // any random value here
    
        //.... other fields
    }