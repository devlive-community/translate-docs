[TOC]

JDO defines a byte-code enhancement process that provides for dirty detection of fields. Before a class is used at runtime it is compiled and then "enhanced" to implement the interface *PersistenceCapable*, and optionally also *Detachable*.

We can demonstrate this by taking a sample class, and seeing it before and after enhancement. We start with the following class

```auto
package org.apache.jdo.test;

public class A
{
    long id;
    String name;
    B b;

    public A(String name)
    {
        this.name = name;
    }

    public void setId(long id)
    {
        this.id = id;
    }

    public void setB(B b)
    {
        this.b = b;
    }

    public String getName()
    {
        return name;
    }

    public B getB()
    {
        return b;
    }

    public long getId()
    {
        return id;
    }

    public String toString()
    {
        return "A : id=" + id + " [" + name + "] b=\"" + b + "\"";
    }
}
```

and require it to be *PersistenceCapable* and *Detachable*. The enhancement process needs to intercept all updates of the fields of the class (id, name, b) as well as add on the necessary *PersistenceCapable*, *Detachable* methods. After "enhancement" it becomes

```auto
package org.apache.jdo.test;
import java.util.BitSet;

import javax.jdo.JDODetachedFieldAccessException;
import javax.jdo.JDOFatalInternalException;
import javax.jdo.PersistenceManager;
import javax.jdo.identity.LongIdentity;
import javax.jdo.spi.Detachable;
import javax.jdo.spi.JDOImplHelper;
import javax.jdo.spi.JDOPermission;
import javax.jdo.spi.PersistenceCapable;
import javax.jdo.spi.StateManager;

public class A implements PersistenceCapable, Detachable
{
    long id;
    String name;
    B b;
    protected transient StateManager jdoStateManager;
    protected transient byte jdoFlags;
    protected Object[] jdoDetachedState;
    private static final byte[] jdoFieldFlags;
    private static final Class jdoPersistenceCapableSuperclass;
    private static final Class[] jdoFieldTypes;
    private static final String[] jdoFieldNames = __jdoFieldNamesInit();
    private static final int jdoInheritedFieldCount;

    static
    {
        jdoFieldTypes = __jdoFieldTypesInit();
        jdoFieldFlags = __jdoFieldFlagsInit();
        jdoInheritedFieldCount = __jdoGetInheritedFieldCount();
        jdoPersistenceCapableSuperclass = __jdoPersistenceCapableSuperclassInit();
        JDOImplHelper.registerClass(___jdo$loadClass("org.apache.jdo.test.A"),
                    jdoFieldNames, jdoFieldTypes,
                    jdoFieldFlags,
                    jdoPersistenceCapableSuperclass, new A());
    }

    public void setId(long id)
    {
        jdoSetid(this, id);
    }

    public void setB(B b)
    {
        jdoSetb(this, b);
    }

    public String getName()
    {
        return jdoGetname(this);
    }

    public B getB()
    {
        return jdoGetb(this);
    }

    public long getId()
    {
        return jdoGetid(this);
    }

    public String toString()
    {
        return new StringBuilder().append("A : id=").append(jdoGetid(this))
           .append(" [").append(jdoGetname(this))
           .append("] b=\"").append(jdoGetb(this))
           .append("\"").toString();
    }

    public void jdoCopyKeyFieldsFromObjectId(PersistenceCapable.ObjectIdFieldConsumer fc, Object oid)
    {
        if (fc == null)
            throw new IllegalArgumentException
              ("ObjectIdFieldConsumer is null");
        if (!(oid instanceof LongIdentity))
            throw new ClassCastException
              ("oid is not instanceof javax.jdo.identity.LongIdentity");
        LongIdentity o = (LongIdentity) oid;
        fc.storeLongField(1, o.getKey());
    }

    protected void jdoCopyKeyFieldsFromObjectId(Object oid)
    {
        if (!(oid instanceof LongIdentity))
            throw new ClassCastException
              ("key class is not javax.jdo.identity.LongIdentity or null");
        LongIdentity o = (LongIdentity) oid;
        id = o.getKey();
    }

    public final void jdoCopyKeyFieldsToObjectId(Object oid)
    {
        throw new JDOFatalInternalException
          ("It's illegal to call jdoCopyKeyFieldsToObjectId for a class with Single Field Identity.");
    }

    public final void jdoCopyKeyFieldsToObjectId
    (PersistenceCapable.ObjectIdFieldSupplier fs, Object oid) {
    throw new JDOFatalInternalException
          ("It's illegal to call jdoCopyKeyFieldsToObjectId for a class with Single Field Identity.");
    }

    public final Object jdoGetObjectId()
    {
        if (jdoStateManager != null)
            return jdoStateManager.getObjectId(this);
        if (this.jdoIsDetached() != true)
            return null;
        return jdoDetachedState[0];
    }

    public final Object jdoGetVersion()
    {
        if (jdoStateManager != null)
            return jdoStateManager.getVersion(this);
        if (this.jdoIsDetached() != true)
            return null;
        return jdoDetachedState[1];
    }

    protected final void jdoPreSerialize()
    {
        if (jdoStateManager != null)
            jdoStateManager.preSerialize(this);
    }

    public final PersistenceManager jdoGetPersistenceManager()
    {
        return (jdoStateManager != null
            ? jdoStateManager.getPersistenceManager(this) : null);
    }

    public final Object jdoGetTransactionalObjectId()
    {
        return (jdoStateManager != null
           ? jdoStateManager.getTransactionalObjectId(this) : null);
    }

    public final boolean jdoIsDeleted()
    {
        return (jdoStateManager != null ? jdoStateManager.isDeleted(this): false);
    }

    public final boolean jdoIsDirty()
    {
        if (jdoStateManager != null)
            return jdoStateManager.isDirty(this);
        if (this.jdoIsDetached() != true)
            return false;
        if (((BitSet) jdoDetachedState[3]).length() <= 0)
            return false;
        return true;
    }

    public final boolean jdoIsNew()
    {
        return jdoStateManager != null ? jdoStateManager.isNew(this) : false;
    }

    public final boolean jdoIsPersistent()
    {
        return (jdoStateManager != null ? jdoStateManager.isPersistent(this): false);
    }

    public final boolean jdoIsTransactional()
    {
        return (jdoStateManager != null ? jdoStateManager.isTransactional(this): false);
    }

    public final boolean jdoIsDetached()
    {
        if (jdoStateManager == null) {
            if (jdoDetachedState == null)
                return false;
            return true;
        }
        return false;
    }

    public final void jdoMakeDirty(String fieldName)
    {
        if (jdoStateManager != null)
            jdoStateManager.makeDirty(this, fieldName);
    }

    public final Object jdoNewObjectIdInstance()
    {
        return new LongIdentity(getClass(), id);
    }

    public final Object jdoNewObjectIdInstance(Object key)
    {
        if (key == null)
            throw new IllegalArgumentException("key is null");
        if (key instanceof String != true)
            return new LongIdentity(this.getClass(), (Long) key);
        return new LongIdentity(this.getClass(), (String) key);
    }

    public final void jdoProvideFields(int[] fieldId)
    {
        if (fieldId == null)
            throw new IllegalArgumentException("argment is null");
        int i = fieldId.length - 1;
        if (i >= 0)
        {
            do
                jdoProvideField(fieldId[i]);
            while (--i >= 0);
        }
    }

    public final void jdoReplaceFields(int[] fieldId)
    {
        if (fieldId == null)
            throw new IllegalArgumentException("argument is null");
        int i = fieldId.length;
        if (i > 0)
        {
            int i_0_ = 0;
            do
                jdoReplaceField(fieldId[i_0_]);
            while (++i_0_ < i);
        }
    }

    public final void jdoReplaceFlags()
    {
        if (jdoStateManager != null)
        {
            A a = this;
            a.jdoFlags = a.jdoStateManager.replacingFlags(this);
        }
    }

    public final synchronized void jdoReplaceStateManager(StateManager stateManager)
    {
        if (jdoStateManager != null)
        {
            A a = this;
            a.jdoStateManager = a.jdoStateManager.replacingStateManager(this, stateManager);
        }
        else
        {
            JDOImplHelper.checkAuthorizedStateManager(sm);
            jdoStateManager = stateManager;
            jdoFlags = (byte) 1;
        }
    }

    public final synchronized void jdoReplaceDetachedState()
    {
        if (jdoStateManager == null)
            throw new IllegalStateException("state manager is null");
        A a = this;
        a.jdoDetachedState = a.jdoStateManager.replacingDetachedState(this, jdoDetachedState);
    }

    public PersistenceCapable jdoNewInstance(StateManager sm)
    {
        A result = new A();
        A a = result;
        a.jdoFlags = (byte) 1;
        a.jdoStateManager = sm;
        return a;
    }

    public PersistenceCapable jdoNewInstance(StateManager sm, Object o)
    {
        A result = new A();
        A a = result;
        a.jdoFlags = (byte) 1;
        a.jdoStateManager = sm;
        result.jdoCopyKeyFieldsFromObjectId(o);
        return a;
    }

    public void jdoReplaceField(int fieldIndex)
    {
        if (jdoStateManager == null)
            throw new IllegalStateException("state manager is null");
        switch (fieldIndex)
        {
            case 0:
            {
                A a = this;
                a.b = (B) a.jdoStateManager.replacingObjectField(this, fieldIndex);
                break;
            }
            case 1:
            {
                A a = this;
                a.id = a.jdoStateManager.replacingLongField(this, fieldIndex);
                break;
            }
            case 2:
            {
                A a = this;
                a.name = a.jdoStateManager.replacingStringField(this, fieldIndex);
                break;
            }
            default:
                throw new IllegalArgumentException("out of field index :" + fieldIndex);
        }
    }

    public void jdoProvideField(int fieldIndex)
    {
        if (jdoStateManager == null)
            throw new IllegalStateException("state manager is null");
        switch (fieldIndex)
        {
            case 0:
                jdoStateManager.providedObjectField(this, fieldIndex, b);
                break;
            case 1:
                jdoStateManager.providedLongField(this, fieldIndex, id);
                break;
            case 2:
                jdoStateManager.providedStringField(this, fieldIndex, name);
                break;
            default:
                throw new IllegalArgumentException("out of field index :" + fieldIndex);
         }
    }

    protected final void jdoCopyField(A obj, int index)
    {
        switch (index)
        {
            case 0:
                b = obj.b;
                break;
            case 1:
                id = obj.id;
                break;
            case 2:
                name = obj.name;
                break;
            default:
                throw new IllegalArgumentException("out of field index :" + index);
        }
    }

    public void jdoCopyFields(Object obj, int[] fieldNumbers)
    {
        if (jdoStateManager == null)
            throw new IllegalStateException("state manager is null");
        if (fieldNumbers == null)
            throw new IllegalStateException("fieldNumbers is null");
        if (obj instanceof A != true)
            throw new IllegalArgumentException("object is not org.apache.jdo.test.A");
        A me = (A) obj;
        if (jdoStateManager != me.jdoStateManager)
            throw new IllegalArgumentException("state manager unmatch");
        int i = fieldNumbers.length - 1;
        if (i >= 0)
        {
            do
                jdoCopyField(me, fieldNumbers[i]);
            while (--i >= 0);
        }
    }

    private static final String[] __jdoFieldNamesInit()
    {
        return new String[] { "b", "id", "name" };
    }

    private static final Class[] __jdoFieldTypesInit()
    {
        return new Class[] { ___jdo$loadClass("org.apache.jdo.test.B"), Long.TYPE,
                 ___jdo$loadClass("java.lang.String") };
    }

    private static final byte[] __jdoFieldFlagsInit()
    {
        return new byte[] { 10, 24, 21 };
    }

    protected static int __jdoGetInheritedFieldCount()
    {
        return 0;
    }

    protected static int jdoGetManagedFieldCount()
    {
        return 3;
    }

    private static Class __jdoPersistenceCapableSuperclassInit()
    {
        return null;
    }

    public static Class ___jdo$loadClass(String className)
    {
        try
        {
            return Class.forName(className);
        }
        catch (ClassNotFoundException e)
        {
            throw new NoClassDefFoundError(e.getMessage());
        }
    }

    private Object jdoSuperClone()
    throws CloneNotSupportedException
    {
        A o = (A) super.clone();
        o.jdoFlags = (byte) 0;
        o.jdoStateManager = null;
        return o;
    }

    public A()
    {
        /* empty */
    }

    static void jdoSetb(A objPC, B b_m)
    {
        if (objPC.jdoStateManager == null)
            objPC.b = b_m;
        else
            objPC.jdoStateManager.setObjectField(objPC, 0, objPC.b, b_m);
        if (objPC.jdoIsDetached() == true)
            ((BitSet) objPC.jdoDetachedState[3]).set(0);
    }

    static B jdoGetb(A objPC)
    {
        if (objPC.jdoStateManager != null
        && !objPC.jdoStateManager.isLoaded(objPC, 0))
            return (B) objPC.jdoStateManager.getObjectField(objPC, 0, objPC.b);
        if (objPC.jdoIsDetached() != false
        && ((BitSet) objPC.jdoDetachedState[2]).get(0) != true
        && ((BitSet) objPC.jdoDetachedState[3]).get(0) != true)
            throw new JDODetachedFieldAccessException
              ("You have just attempted to access field \"b\" yet this field was not detached when you detached the object. " +
               "Either dont access this field, or detach the field when detaching the object.");
        return objPC.b;
    }

    static void jdoSetid(A objPC, long id_n)
    {
        objPC.id = id_n;
    }

    static long jdoGetid(A objPC)
    {
        return objPC.id;
    }

    static void jdoSetname(A objPC, String name_c)
    {
        if (objPC.jdoFlags != 0 && objPC.jdoStateManager != null)
            objPC.jdoStateManager.setStringField(objPC, 2, objPC.name, name_c);
        else
        {
            objPC.name = name_c;
            if (objPC.jdoIsDetached() == true)
                ((BitSet) objPC.jdoDetachedState[3]).set(2);
        }
    }

    static String jdoGetname(A objPC)
    {
        if (objPC.jdoFlags > 0 && objPC.jdoStateManager != null && !objPC.jdoStateManager.isLoaded(objPC, 2))
            return objPC.jdoStateManager.getStringField(objPC, 2, objPC.name);
        if (objPC.jdoIsDetached() != false && ((BitSet) objPC.jdoDetachedState[2]).get(2) != true)
            throw new JDODetachedFieldAccessException
              ("You have just attempted to access field \"name\" yet this field was not detached when you detached the object." +
               "Either dont access this field, or detach the field when detaching the object.");
        return objPC.name;
    }

    public A(String name)
    {
        jdoSetname(this, name);
    }
}
```
