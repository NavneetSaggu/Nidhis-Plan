# Hibernate

[_States Of An Object](_States%20Of%20An%20Object.md)

[Mappings / Joins](Hibernate/Mappings%20Joins.md)

[Session & Its Methods](Hibernate/Session%20%26%20Its%20Methods.md)

# Introduction

- Hibernate has layered architecture; each layer has it’s responsibilities.

![Untitled](Hibernate/Untitled.png)

- Enables change in one layer with little impact to other layers
    - **Application Layer:**
        - manages code that accesses persistent data
    - **Framework Layer:**
        - manages Hibernate configs, sessions, and transactions
        - allows communication of Application and Persistance Layer
    - **Persistance Layer:**
        - consists of database

# Configuration Object

- First Hibernate object that’s created.
- Created only once during app initialization.
- Provides
    - **Database Connection:** 
    Handled through config files. hibernate.properties and hibernate.cfg.xml
    - **Class Mapping Setup:**
    Creates class to table mapping

# SessionFactory Object

- Created only once during app initialization; heavyweight object.
- Configuration object is used to create SessionFactory object.
- Used to make Session object, with supplied configs.
- SessionFactory is threadsafe, used by all threads.
- One SessionFactory object needed per database; since each db will have different configs.

# Session Object

- One Session is one physical connection with a db.
- Main tool used to communicate with Hibernate.
- Provides an API enabling us to create, read, update, and delete persistent objects.
- Instantiated each time we interact with db; lightweight object.
- Not thread safe; should not be kept open for long; create and destroy when needed.
- Life of a session:
    1. Open Session
    2. Perform Operations on objects. These objects are attached to the session.
    3. Close Session
    4. Changes made to attached objects are detected and saved upon closing
    5. After closing, Hibernate breaks the connections between the objects and the session.

# Transaction Object

- A transaction is a unit of work in db.
- Optional object; app may choose to manage transactions themselves.
- Really good doc on txns [https://docs.jboss.org/hibernate/stable/core.old/reference/en/html/transactions.html](https://docs.jboss.org/hibernate/stable/core.old/reference/en/html/transactions.html)

# Query Object

- Used to specify query params and execute query.