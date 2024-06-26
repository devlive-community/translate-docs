[TOC]

Using JDO requires several components that work together:

*   a database that stores your data persistently
*   an application that operates through the JDO layers on the database
*   a JDO implementation that provides the implementation of the JDO APIs
*   Java classes (object model) that provide the application view of the data

A good place to start is to define your object model. This is the collection of Java classes that you use to represent the persistent data in your database. Some JDO implementations allow you to create your database schema from your object model. Or you might already have a database and database schema that you can use to derive the object model.

You will also need to define Java classes that implement the application that operates on the object model. The application is responsible for using the JDO implementation classes to define the application view of PersistenceManagerFactory and PersistenceManager.

Next, choose a JDO implementation. You can get information on JDO implementations [here](https://db.apache.org/jdo/impls.html). Some factors to consider when choosing a JDO implementation is what databases they support, what Java version they support, and whether they provide tools to create database schema from an object model or create an object model from the database schema.

Some JDO implementations also package the JDO interfaces as a separate jar file, so there may be no need to download the JDO interfaces from this site.

More information on how to proceed can be found on your JDO implementation site.
