[TOC]

## Meta-Data - JDO

JDO defines XML MetaData in **jdo** files as well as **orm** files. As always with XML, the metadata must match the defined DTD/XSD for that file type. This section describes the content of the **jdo** files. The content of **orm** files can be found [here]($orm-DTD-XSD). All **jdo** files must contain a valid DTD/DOCTYPE specification. You can use PUBLIC or SYSTEM versions of these.

Here are a few examples valid for **jdo** files with DTD specifications

```xml
<!DOCTYPE jdo PUBLIC
    "-//The Apache Software Foundation//DTD Java Data Objects Metadata 3.2//EN"
    "https://db.apache.org/jdo/xmlns/jdo\_3\_2.dtd">
```

or

```xml
<!DOCTYPE jdo SYSTEM "file:/javax/jdo/jdo.dtd">
```

Here is an example valid for **jdo** files with XSD specification

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<jdo xmlns="https://db.apache.org/jdo/xmlns/jdo"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="https://db.apache.org/jdo/xmlns/jdo https://db.apache.org/jdo/xmlns/jdo\_3\_2.xsd" version="3.2">
    ...
</jdo>
```

Your MetaData should match either the [DTD](http://db.apache.org/jdo/xmlns/jdo_3_2.dtd) or the [XSD](http://db.apache.org/jdo/xmlns/jdo_3_2.xsd) specification.
