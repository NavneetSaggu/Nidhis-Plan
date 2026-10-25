# States Of An Object

![Untitled](States%20Of%20An%20Object/Untitled.png)

### Transient

This instance isn't, and never was, attached to a *Session.* This instance has no corresponding rows in the database; it's usually just a new object that we created to save to the database. Because no *session* is aware of it, it won't be saved automatically.

Let's create a user object with the constructor and confirm that it isn't managed by the session:

```java
Session session = openSession(); 
UserEntity userEntity = new UserEntity("John"); 
assertThat(session.contains(userEntity)).isFalse();
```

### Persistent

An object that we've associated with a *session* is in the persistent state. Upon flushing the *Session* to the database, this entity is guaranteed to have a corresponding consistent record in the database.

Let's create an object and then use the *persist* method to make it persistent:

```java
Session session = openSession(); 
UserEntity userEntity = new UserEntity("John"); 
session.persist(userEntity); 
assertThat(session.contains(userEntity)).isTrue();
```

Alternatively, we may use the *save* method. The difference is that the *persist* method will just save an object, and the *save* method will additionally generate its identifier if that's needed.

### Detached

This instance was once attached to a *Session* (in a *persistent* state), but now it’s not. An instance enters this state if we evict it from the context, clear or close the Session, or put the instance through serialization/deserialization process.

```java
session.persist(userEntity); 
session.close(); 
assertThat(session.isOpen()).isFalse(); 
assertThatThrownBy(() -> session.contains(userEntity));
```

# Using Object States

When the entity instance is in the *persistent* state, all the changes that we make to the mapped fields of this instance will be applied to the corresponding database records and fields upon flushing the *Session*. The *persistent* instance is “online,” whereas the *detached* instance is “offline” and not monitored for changes.

This means that when we change the fields of a *persistent* object, we don't have to call *save*, *update,* or any of those methods to get these changes to the database. All we need to do is commit the transaction, flush the session, or close the session.

# **Saving a Transient Entity**

Let's create a new entity and save it to the database. **When we first construct the object, it'll be in the transient state.**

To *persist* our new entity, we'll use the *persist* method:

```
UserEntity userEntity =newUserEntity("John");
session.persist(userEntity);
```

Now, we'll create another object with the same identifier as the first one. This second object is transient because it's not yet managed by any *session*, but we can't make it persistent using the *persist* method. It's already represented in the database, so it's not really new in the context of the persistence layer.

Instead, **we'll use the *merge* method to update the database and make the object persistent**:

```
UserEntity onceAgainJohn =newUserEntity("John");
session.merge(onceAgainJohn);
```