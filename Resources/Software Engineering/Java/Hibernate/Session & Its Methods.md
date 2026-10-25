# Session & Its Methods

Session has multiple methods:

- persist
- save
- update
- merge
- saveOrUpdate

# Session / Persistence Context

We can think of persistence context as a container or first-level cache for all the objects that we loaded or saved to a database during a session. For Hibernate, this context is Session object.

It's important to understand from the beginning that all of the methods (*persist*, *save*, *update*, *merge*, *saveOrUpdate*) don't immediately result in the corresponding SQL *UPDATE* or *INSERT* statements. The actual saving of data to the database occurs upon committing the transaction or flushing the *Session*.

The mentioned methods basically manage the state of entity instances by transitioning them between different states along the lifecycle.

# ***Persist***

The *persist* method is intended to add a new entity instance to the persistence context, i.e. transitioning an instance from a *transient* to *persistent* state.

We usually call it when we want to add a record to the database (persist an entity instance):

```java
Person person = new Person();
person.setName("John");
session.persist(person);
```

What happens after we call the *persist* method? The *person* object has transitioned from a *transient* to *persistent* state. The object is in the persistence context now, but not yet saved to the database. The generation of *INSERT* statements will occur only upon committing the transaction, or flushing or closing the session.

Notice that the *persist* method has a *void* return type. It operates on the passed object “in place,” changing its state. The *person* variable references the actual persisted object.

This method is a later addition to the *Session* interface. The main differentiating feature of this method is that it conforms to the JSR-220 specification (EJB persistence). We strictly define the semantics of this method in the specification, which basically states that a *transient* instance becomes *persistent* (and the operation cascades to all of its relations with *cascade=PERSIST* or *cascade=ALL*):

- if an instance is already *persistent*, then this call has no effect for this particular instance (but it still cascades to its relations with *cascade=PERSIST* or *cascade=ALL*).
- if an instance is *detached*, we'll get an exception, either upon calling this method, or upon committing or flushing the session.

Notice that there's nothing here that concerns the identifier of an instance. The spec doesn't state that the id will generate right away, regardless of the id generation strategy. The specification for the *persist* method allows the implementation to issue statements to generate the id on commit or flush. The id won't necessarily be non-null after we call this method, so we shouldn't rely upon it.

We may call this method on an already *persistent* instance, and nothing happens. But if we try to persist a *detached* instance, the implementation will throw an exception. In the following example, we'll *persist* the entity, *evict* it from the context so it becomes *detached*, and then try to *persist* again. The second call to *session.persist()* causes an exception, so the following code won't work:

```java
Person person = new Person();
person.setName("John");
session.persist(person);
session.evict(person);
session.persist(person); // PersistenceException!
```

# ***Merge***

The main intention of the *merge* method is to update a *persistent* entity instance with new field values from a *detached* entity instance.

For instance, suppose we have a RESTful interface with a method for retrieving a JSON-serialized object by its id to the caller, and a method that receives an updated version of this object from the caller. An entity that passed through such serialization/deserialization will appear in a *detached* state.

After deserializing this entity instance, we need to get a *persistent* entity instance from a persistence context and update its fields with new values from this *detached* instance. So the *merge* method does exactly that:

- finds an entity instance by id taken from the passed object (either an existing entity instance from the persistence context is retrieved, or a new instance loaded from the database)
- copies fields from the passed object to this instance
- returns a newly updated instance

In the following example, we *evict* (detach) the saved entity from the context, change the *name* field, and then *merge* the *detached* entity:

```java
Person person = new Person(); 
person.setName("John"); 
session.save(person);

session.evict(person);
person.setName("Mary");

Person mergedPerson = (Person) session.merge(person);
```

Note that the *merge* method returns an object. It's the *mergedPerson* object we loaded into the persistence context and updated, not the *person* object that we passed as an argument. They're two different objects, and we usually need to discard the *person* object.

As with the *persist* method, the *merge* method is specified by JSR-220 to have certain semantics that we can rely upon:

- if the entity is *detached*, it copies upon an existing *persistent* entity.
- if the entity is *transient*, it copies upon a newly created *persistent* entity.
- this operation cascades for all relations with *cascade=MERGE* or *cascade=ALL* mapping.
- if the entity is *persistent*, then this method call doesn't have an effect on it (but the cascading still takes place).

# ***SaveOrUpdate***

This method appears only in the Hibernate API and doesn't have its standardized counterpart. Similar to *update*, we can also use it for reattaching instances.

Actually, the internal *DefaultUpdateEventListener* class that processes the *update* method is a subclass of *DefaultSaveOrUpdateListener*, just overriding some functionality. The main difference of the *saveOrUpdate* method is that it doesn't throw an exception when applied to a *transient* instance, instead it makes this *transient* instance *persistent*. The following code will persist a newly created instance of *Person*:

```java
Person person = new Person();
person.setName("John");
session.saveOrUpdate(person);
```

We can think of this method as a universal tool for making an object *persistent* regardless of its state, whether it's *transient* or *detached*.

# **What to Use?**

If we don't have any special requirements, we should stick to the *persist* and *merge* methods because they're standardized and will conform to the JPA specification.

They're also portable in case we decide to switch to another persistence provider; however, they may sometimes seem not as useful as the “original” Hibernate methods, *save*, *update,* and *saveOrUpdate*.

# *Sesion Handling*

SessionFactory.getCurrentSession() - internally creates and assigns the session to a thread local. Each request / thread will have its own session. Once the web request processing is complete the session will be closed. From within your code you just need to use SessionFactory.getCurrentSession() and don't have to close it.