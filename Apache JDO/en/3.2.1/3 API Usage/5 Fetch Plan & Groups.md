[TOC]

When an object is retrieved from the datastore by JDO typically not all fields are retrieved immediately. This is because for efficiency purposes only particular field types are retrieved in the initial access of the object, and then any other objects are retrieved when accessed (lazy loading). The group of fields that are loaded is defined by a **FetchPlan** and is comprised of multiple \*fetch group\*s. There are 3 types of "fetch groups" to consider

*   [Default Fetch Group](#dfg) : defined in the JDO spec, containing the fields of a class that will be retrieved by default (with no user specification).

*   [Named Fetch Groups](#static) : defined in MetaData (XML/annotations) with the fields of a class that are part of that fetch group. The definition here is _static_.

*   [Dynamic Fetch Groups](#dynamic) : Programmatic definition of fetch groups at runtime via an API


The **fetch group**(s) in use for a class is controlled via the _FetchPlan_ [![Image 1: image](https://db.apache.org/jdo/images/javadoc.png)](http://db.apache.org/jdo/api32/apidocs/javax/jdo/FetchPlan.html) interface. To get a handle on the current _FetchPlan_ we do

FetchPlan fp = pm.getFetchPlan();

### Default Fetch Group

JDO provides an initial fetch group, comprising the fields that will be retrieved when an object is retrieved if the user does nothing to define the required behaviour. By default the _default fetch group_ comprises all fields of the following types :

*   primitives : boolean, byte, char, double, float, int, long, short

*   Object wrappers of primitives : Boolean, Byte, Character, Double, Float, Integer, Long, Short

*   java.lang.String, java.lang.Number, java.lang.Enum

*   java.math.BigDecimal, java.math.BigInteger

*   java.util.Date


If you wish to change the **Default Fetch Group** for a class you can update the Meta-Data for the class as follows (using XML)

    <class name="MyClass">
        ...
        <field name="fieldX" default-fetch-group="true"/>
    </class>

or using annotations

    @Persistent(defaultFetchGroup="true")
    SomeType fieldX;

When a _PersistenceManager_ is created it starts with a FetchPlan of the "default" fetch group. That is, if we call

    Collection fetchGroups = fp.getGroups();

this will have one group, called "default". At runtime, if you have been using other fetch groups and want to revert back to the default fetch group at any time you simply do

    fp.setGroup(FetchPlan.DEFAULT);

### Named Fetch Groups

As mentioned above, JDO allows specification of users own fetch groups. These are specified in the MetaData of the class. For example, if we have the following class

    class MyClass
    {
        String name;
        HashSet coll;
        MyOtherClass other;
    }

and we want to have the other field loaded whenever we load objects of this class, we define our MetaData as

    <package name="mydomain">
        <class name="MyClass">
            <field name="name">
                <column length="100" jdbc-type="VARCHAR"/>
            </field>
            <field name="coll" persistence-modifier="persistent">
                <collection element-type="mydomain.Address"/>
                <join/>
            </field>
            <field name="other" persistence-modifier="persistent"/>
            <fetch-group name="otherfield">
                <field name="other"/>
            </fetch-group>
        </class>
    </package>

or using annotations

    @PersistenceCapable
    @FetchGroup(name="otherfield", members={@Persistent(name="other")})
    public class MyClass
    {
        ...
    }

So we have defined a fetch group called "otherfield" that just includes the field with name _other_. We can then use this at runtime in our persistence code.

    PersistenceManager pm = pmf.getPersistenceManager();
    pm.getFetchPlan().addGroup("otherfield");
    
    ... (load MyClass object)

By default the _FetchPlan_ will include the default fetch group. We have changed this above by adding the fetch group "otherfield", so when we retrieve an object using this _PersistenceManager_ we will be retrieving the fields _name_ AND _other_ since they are both in the current _FetchPlan_. We can take the above much further than what is shown by defining nested fetch groups in the MetaData. In addition we can change the _FetchPlan_ just before any _PersistenceManager_ operation to control what is fetched during that operation. The user has full flexibility to add many groups to the current **Fetch Plan**. This gives much power and control over what will be loaded and when.

The _FetchPlan_ applies not just to calls to _PersistenceManager.getObjectById()_, but also to _PersistenceManager.newQuery()_, _PersistenceManager.getExtent()_, _PersistenceManager.detachCopy_ and much more besides.

You can read more about **named fetch-groups** and how to use it with [**attach/detach**](https://db.apache.org/jdo/attach_detach.html)

### Dynamic Fetch Groups

The mechanism above provides static fetch groups defined in XML or annotations. That is great when you know in advance what fields you want to fetch. In some situations you may want to define your fields to fetch at run time. It operates as follows

    import org.datanucleus.FetchGroup;
    
    // Create a FetchGroup on the PMF called "TestGroup" for MyClass
    FetchGroup grp = myPMF.getFetchGroup("TestGroup", MyClass.class);
    grp.addMember("field1").addMember("field2");
    
    // Add this group to the fetch plan (using its name)
    fp.addGroup("TestGroup");

So we use the PMF as a way of creating a FetchGroup, and then register that FetchGroup with the PMF for use by all PMs. We then enable our FetchGroup for use in the FetchPlan by using its group name (as we do for a static group). The FetchGroup allows you to add/remove the fields necessary so you have full API control over the fields to be fetched.

### Fetch Depth

The basic fetch group defines which fields are to be fetched. It doesn’t explicitly define how far down an object graph is to be fetched. JDO provides two ways of controlling this.

The first is to set the **maxFetchDepth** for the _FetchPlan_. This value specifies how far out from the root object the related objects will be fetched. A positive value means that this number of relationships will be traversed from the root object. A value of -1 means that no limit will be placed on the fetching traversal. The default is 1. Let’s take an example

    public class MyClass1
    {
        MyClass2 field1;
        ...
    }
    
    public class MyClass2
    {
        MyClass3 field2;
        ...
    }
    
    public class MyClass3
    {
        MyClass4 field3;
        ...
    }

and we want to detach _field1_ of instances of _MyClass1_, down 2 levels - so detaching the initial "field1" _MyClass2_ object, and its "field2" _MyClass3_ instance. So we define our fetch-groups like this

    <class name="MyClass1">
        ...
        <fetch-group name="includingField1">
            <field name="field1"/>
        </fetch-group>
    </class>
    <class name="MyClass2">
        ...
        <fetch-group name="includingField2">
            <field name="field2"/>
        </fetch-group>
    </class>

and we then define the **maxFetchDepth** as 2, like this

    pm.getFetchPlan().setMaxFetchDepth(2);

A further refinement to this global fetch depth setting is to control the fetching of recursive fields. This is performed via a MetaData setting "recursion-depth". A value of 1 means that only 1 level of objects will be fetched. A value of -1 means there is no limit on the amount of recursion. The default is 1. Let’s take an example

    public class Directory
    {
        Collection children;
        ...
    }

    <class name="Directory">
        <field name="children">
            <collection element-type="Directory"/>
        </field>
    
        <fetch-group name="grandchildren">
            <field name="children" recursion-depth="2"/>
        </fetch-group>
        ...
    </class>

So when we fetch a Directory, it will fetch 2 levels of the _children_ field, hence fetching the children and the grandchildren.

### Fetch Size

A FetchPlan can also be used for defining the fetching policy when using queries. This can be set using

    pm.getFetchPlan().setFetchSize(value);

The default is _FetchPlan.FETCH\_SIZE\_OPTIMAL_ which leaves it to the JDO provider to optimise the fetching of instances. A positive value defines the number of instances to be fetched. Using _FetchPlan.FETCH\_SIZE\_GREEDY_ means that all instances will be fetched immediately.
