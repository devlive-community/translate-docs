[TOC]

When managing the persistence of objects using a [PersistenceManager]($Persistence-Manager) it is common to handle all datastore operations in a transaction. For this reason each _PersistenceManager_ has its own transaction. Consequently a typical JDO persistence method will look something like this

    PersistenceManager pm = pmf.getPersistenceManager();
    Transaction tx = pm.currentTransaction();
    try
    {
        tx.begin(); // Start the PM transaction
    
        ... perform some persistence operations
    
        tx.commit(); // Commit the PM transaction
    }
    finally
    {
        if (tx.isActive())
        {
            tx.rollback(); // Error occurred so rollback the PM transaction
        }
    }

### Locking

JDO supports the two main forms of transaction locking

*   Pessimistic (or datastore) locking is where you lock all records in a datastore affected by a transaction and keep them locked until you are ready to commit their changes. This is suitable for short lived operations where no user interaction is taking place.
*   Optimistic locking is where you can simply assume that things in the datastore will not change until they are ready to commit, not lock any records and then just before committing make a check for changes. This is suitable for longer lived operations maybe where user interaction is taking place and where it would be undesirable to block access to datastore entities for the duration of the transaction

You select the type of locking used by the transaction of a _PersistenceManager_ (PM) either by setting the persistence property **javax.jdo.option.Optimistic**, or on the transaction you call

    pm.currentTransaction().setOptimistic(true);

### Non-Transactional

The JDO API also allows the ability to operate without transactions. You have 2 persistence properties **javax.jdo.option.NontransactionalRead** and **javax.jdo.option.NontransactionalWrite** to enable this mode of operation assuming your JDO provider supports it. Use of these two properties mean that you can then read objects and make updates outside of transactions. This is effectively an "auto-commit" mode.

    PersistenceManager pm = pmf.getPersistenceManager();
    
    {users code to persist objects}
    
    pm.close();