# Mappings / Joins

**GOD PAGE COVERING MOST OF THI: [https://docs.oracle.com/cd/E19798-01/821-1841/6nmq2cpak/index.html](https://docs.oracle.com/cd/E19798-01/821-1841/6nmq2cpak/index.html)**

**What are ORM Entities?**

[https://docs.oracle.com/cd/E19798-01/821-1841/6nmq2cpak/index.html](https://docs.oracle.com/cd/E19798-01/821-1841/6nmq2cpak/index.html) A "persistence domain object". Typically, an entity represents a table in a relational database, and each entity instance corresponds to a row in that table.  The state of an entity is represented through fields or properties; which use object/relational mapping annotations (@OneToMany, @JoinColumn etc.) to map the entities and entity relationships to the relational data in the underlying data store.

**Field vs Property Access**

Field: Directly accesses attributes. If you annotate the attribute itself, Hibernate uses field-based access.

Property: Accesses attribute through the getter/setter function that you write for it. If you annotate the getter/setter, Hibernate uses property-based access. Look at examples here: [https://thorben-janssen.com/access-strategies-in-jpa-and-hibernate/](https://thorben-janssen.com/access-strategies-in-jpa-and-hibernate/)

This page also lists 5 good reasons to prefer field based access. All fields are persistent by default, I.e. they will be saved to storage; unless annotated with @Transient

**Can fields be collections?**

The following Java types can be used as valid field types

- java.util.Collection
- java.util.Set
- java.util.List
- java.util.Map

If a field consists of a collection of basic types or embeddable classes, use the @ElementCollection annotation on the field or property.

**Lazy vs Eager fetch** Can be used to specify loading mechanism for all entities that have relationships. Suppose you have

public class University {

private String id;

private String name;

private String address;

private List<Student> students;

}

Now when you load a University from the database, JPA loads its id, name, and address fields for you. But you have two options for how students should be loaded:

- To load it together with the rest of the fields (i.e. eagerly), or
- To load it on-demand (i.e. lazily) when you call the university's getStudents() method.

Annotate with @OneToMany(fetch = FetchType.EAGER) or @OneToMany(fetch = FetchType.EAGER) for either functionality.

[https://stackoverflow.com/questions/2990799/difference-between-fetchtype-lazy-and-eager-in-java-persistence-api](https://stackoverflow.com/questions/2990799/difference-between-fetchtype-lazy-and-eager-in-java-persistence-api)

**Fetching Strategies** [https://mkyong.com/hibernate/hibernate-fetching-strategies-examples/](https://mkyong.com/hibernate/hibernate-fetching-strategies-examples/)

**What is a relationship here?** If entity1 has a relationship field or property that refers to entity2, then entity1 one "knows" about entitiy2. Through the relationship field or property, an entity class's code can access its related object. 

**Multiplicity in Entity Relationships**

Multiplicities are of the following types: one-to-one, one-to-many, many-to-one, and many-to-many:

- **One-to-one**: Each entity instance is related to a single instance of another entity.

For example, to model a physical warehouse in which each storage bin contains a single widget, StorageBin and Widget would have a one-to-one relationship. One-to-one relationships use @OneToOne

- **One-to-many**: An entity instance can be related to multiple instances of the other entities.

A sales order, for example, can have multiple line items. In the order application, Order would have a one-to-many relationship with LineItem. One-to-many relationships use @OneToMany

- **Many-to-one**: Multiple instances of an entity can be related to a single instance of the other entity. This multiplicity is the opposite of a one-to-many relationship.

In the example just mentioned, the relationship to Order from the perspective of LineItem is many-to-one. Many-to-one relationships use @ManyToOne

- **Many-to-many**: The entity instances can be related to multiple instances of each other.

For example, each college course has many students, and every student may take several courses. Therefore, in an enrollment application, Course and Student would have a many-to-many relationship. Many-to-many relationships use @ManyToMany

**What is a Unidirectional Relationship?** What was shared above was an example of a unidirectional relationship. Entity1 knew about entity2, but entity2 did not know about entiity1.

Will only have an owning side.

We will use this, show our example. The best way to do unidirectional mappings is [https://vladmihalcea.com/the-best-way-to-map-a-onetomany-association-with-jpa-and-hibernate/](https://vladmihalcea.com/the-best-way-to-map-a-onetomany-association-with-jpa-and-hibernate/)

In this case we make foreign key column at the many side, this is the join column, and it's always at the many side. The name we supply in the join column will be the name of the column inserted at the many side.

**What is a Bidirectional Relationship?**

In a bidirectional relationship, each entity has a relationship field or property that refers to the other entity. Both entities know about each other.

Will have owning and inverse side. Kontekst uses this, show their example.

More details about directions in entity relationships can be found here [https://docs.oracle.com/cd/E19798-01/821-1841/6nmq2cpav/index.html](https://docs.oracle.com/cd/E19798-01/821-1841/6nmq2cpav/index.html)

Owning vs Inverse Side 

**Rules for Defining Bidirectional Relationships in JPA**

- The many side is always the owning side of the relationship in one-to-many relationships.
- The inverse side of a bidirectional relationship must refer to its owning side by using the mappedBy element of the @OneToOne, @OneToMany, or @ManyToMany annotation. The mappedBy element designates the property or field in the entity that is the owner of the relationship.
- The many side of many-to-one bidirectional relationships must not define the mappedBy element.
- For one-to-one bidirectional relationships, the owning side corresponds to the side that contains the corresponding foreign key.
- For many-to-many bidirectional relationships, either side may be the owning side.

Show Kontekst wala example

**Why is the notion of a owning side necessary:**

The idea of a owning side of a bidirectional relation comes from the fact that in relational databases there are no bidirectional relations like in the case of objects. In databases we only have unidirectional relations - foreign keys.

So what happens when we create such a bidirectional mapping is that Hibernate has to make a third table in the middle of these two tables, and have unidirectional mappings form it going both ways.

T1 <--> T2 

T1<--T'-->T2 

Another way to solve this, without introducing a third tables is to specify an owner and an inverse side.

**THIS IS SUPER IMPORTANT: THROWBACK TO THE PROBLEM YOU FACED WHENCE WE WERE GETTING EXTRA TABLES FOR EACH MAPPING**: [https://stackoverflow.com/questions/2749689/what-is-the-owning-side-in-an-orm-mapping](https://stackoverflow.com/questions/2749689/what-is-the-owning-side-in-an-orm-mapping)

**What is the reason for the name 'owning side'?**

The owning side of the relation tracked by Hibernate is the side of the relation that *owns* the foreign key in the database. Both tables that that have a relationship will have the foreign key, right? But which one owns this? The owning side of a relationship determines how the Persistence runtime makes updates to the relationship in the database.

Define the join table on the many side, but don't define it once again on the one side. This creates two unidirectional associations mapped with the same table instead of one bidirectional association.

A bidirectional association always has an owner side (where you specify the join column or join table to use, and an inverse side which says that it's the inverse of the other side by using the mappedBy attribute:

```java
public class Activity {
	// owner side: it doesn't have mappedBy, and can decide how the association is mapped: with a join table	
	@ManyToOne 	
	@JoinTable(name="category_activity",
		joinColumns={@JoinColumn(name="activities_id")},
		inverseJoinColumns={@JoinColumn(name="Category_id")})
	private Category category;

}

public class Category {
	// inverse side: it has a mappedBy attribute, and can't decide how the association is mapped, since the other side already decided it.
	@OneToMany(mappedBy = "category") 
	@Fetch(FetchMode.JOIN)
	@JsonIgnore
	private Collection<Activity> activities;
}
```

References [https://docs.oracle.com/cd/E19798-01/821-1841/bnbqi/index.html#:~:text=A%20bidirectional%20relationship%20has%20both,the%20relationship%20in%20the%20database](https://docs.oracle.com/cd/E19798-01/821-1841/bnbqi/index.html#:~:text=A%20bidirectional%20relationship%20has%20both,the%20relationship%20in%20the%20database). ULTIMATE READABLE GUIDE FOR JPA HIBERNATE: [https://medium.com/@rajibrath20/the-best-way-to-map-a-onetomany-relationship-with-jpa-and-hibernate-dbbf6dba00d3](https://medium.com/@rajibrath20/the-best-way-to-map-a-onetomany-relationship-with-jpa-and-hibernate-dbbf6dba00d3) BEST REFERENCE GUIDE TO JPA: [https://en.wikibooks.org/wiki/Java_Persistence](https://en.wikibooks.org/wiki/Java_Persistence)