[TOC]

JDO provides a means of transparent persistence of objects of user defined classes. With JDO there are actually 3 types of classes.

*   **Persistence Capable** classes are classes whose instances can be persisted to a datastore. JDO provide the mechanism for persisting these instances, and they are core to JDO. These classes need to be _enhanced_ according to a JDO Meta-Data specification before use within a JDO environment.

*   **Persistence Aware** classes are classes that manipulate PersistenceCapable instances through direct attribute manipulation. These classes are typically enhanced with very minimal JDO Meta-Data. The enhancement process performs very little changes to these classes.

*   **Normal** classes are classes that aren’t themselves persistable, and have no knowledge of persistence either. These classes are totally unchanged in JDO, and require no JDO Meta-Data whatsoever.


### PersistenceCapable

Classes are defined as **PersistenceCapable**, either by XML MetaData, like this

    <class name="MyClass">
        ...
    </class>

or using Annotations, like this

    @PersistenceCapable
    public class MyClass
    {
        ...
    }

### PersistenceAware

Classes are defined as **PersistenceAware** either by XML MetaData, like this

    <class name="MyClass" persistence-modifier="persistence-aware"/>

or using Annotations, like this

    @PersistenceAware
    public class MyClass
    {
        ...
    }