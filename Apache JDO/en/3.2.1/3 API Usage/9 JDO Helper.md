[TOC]

JDO provides a standard utility that gives access to useful parts of the JDO persistence process. This is known as **JDOHelper** (javax.jdo.JDOHelper)[![Image 1: image](https://db.apache.org/jdo/images/javadoc.png)](https://db.apache.org/jdo/api32/apidocs/javax/jdo/JDOHelper.html)

### PersistenceManagerFactory methods

The methods in JDOHelper can be split into categories. Here we start with the methods for creating the starting point for persistence, the PersistenceManagerFactory (PMF)

*   **getPersistenceManagerFactory(Map props)** - creates a PMF given a Map of the properties
*   **getPersistenceManagerFactory(Map props, ClassLoader cl)** - creates a PMF given a Map of the properties, and a ClassLoader to use
*   **getPersistenceManagerFactory(String resource)** - creates a PMF given a resource defining the properties. This can be used to create a named PMF in JDO2.1
*   **getPersistenceManagerFactory(String props, ClassLoader cl)** - creates a PMF given a resource defining the properties, and a ClassLoader to use
*   **getPersistenceManagerFactory(File props)** - creates a PMF given a file containing the properties
*   **getPersistenceManagerFactory(File props, ClassLoader cl)** - creates a PMF given a file containing the properties and a ClassLoader to use
*   **getPersistenceManagerFactory(String jndi, Context ctx)** - creates a PMF given a JNDI resource
*   **getPersistenceManagerFactory(String jndi, Context ctx, ClassLoader cl)** - creates a PMF given a JNDI resource and a ClassLoader to use
*   **getPersistenceManagerFactory(InputStream strm)** - creates a PMF given an InputStream
*   **getPersistenceManagerFactory(InputStream strm, ClassLoader cl)** - creates a PMF given an InputStream and a ClassLoader to use

### Persistence methods

Now we move onto the operations for persistence.

*   **getPersistenceManager(Object pc)** - returns the PersistenceManager associated with the passed object (if any)
*   **makeDirty(Object pc, String fieldName)** - marks the field of the passed object as dirty (meaning that it needs updating in its datastore)
*   **getObjectId(Object pc)** - returns the object identity for the passed object (if persistent)
*   **getObjectIds(Collection pc)** - returns the object identities for the passed objects (if persistent)
*   **getObjectIds(Object\[\] pc)** - returns the object identities for the passed objects (if persistent)
*   **getVersion(Object pc)** - returns the version for the passed object (if persistent)

### Lifecycle methods

Now we move onto lifecycle operations

*   **getObjectState(Object pc)** - returns the object state
*   **isDirty(Object pc)** - returns whether the passed object is dirty
*   **isTransactional(Object pc)** - returns whether the passed object is transactional
*   **isPersistent(Object pc)** - returns whether the passed object is persistent
*   **isNew(Object pc)** - returns whether the passed object is new
*   **isDeleted(Object pc)** - returns whether the passed object is deleted
*   **isDetached(Object pc)** - returns whether the passed object is detached
