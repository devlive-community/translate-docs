[TOC]

All variants of JDO support specification of persistence using XML MetaData. JDO 2.1+ adds on the ability to specify persistence using Java annotations.

### XML MetaData

JDO expects any XML MetaData to be specified in a file or files in particular positions in the file system. For example, if you have a class _com.mycompany.sample.MyExample_, JDO will look for any of the following files until it finds one (in the order stated) :

```xml
META-INF/package.jdo
WEB-INF/package.jdo
package.jdo
com/package.jdo
com/mycompany/package.jdo
com/mycompany/sample/package.jdo
com/mycompany/sample/MyExample.jdo
```

In addition to specifying XML MetaData in a **jdo** file, if defining O/R mapping information you can also split this out into an ORM file. The locations for ORM files are similar in nature to those for JDO files.

```xml
META-INF/package-{mapping}.orm
WEB-INF/package-{mapping}.orm
package-{mapping}.orm
com/package-{mapping}.orm
com/mycompany/package-{mapping}.orm
com/mycompany/sample/package-{mapping}.orm
com/mycompany/sample/MyExample-{mapping}.orm
```

where _{mapping}_ is a property specified by the user and may be "mysql" for ORM information for MySQL datastores, and "oracle" for ORM information for Oracle datastores, and so on.

### Annotations

JDO 2.1+ added support for annotations. Classes and fields/properties can be annotated defining the persistence and, optionally, any ORM information.