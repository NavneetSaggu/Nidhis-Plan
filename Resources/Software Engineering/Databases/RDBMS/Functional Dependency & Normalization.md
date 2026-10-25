# Functional Dependency & Normalization

Read from [here](https://drive.google.com/file/d/1WUhcbGtjOQ57M-kh377ABNhCLzJrfnad/view?usp=sharing), we only have a small refresher here

## **Guidelines for designing a DB**

1. Design a relation schema so that it is easy to explain its meaning. Do not combine attributes from multiple entity types and relationship types into a single relation.
2. Design the base relation schemas so that no insertion, deletion, or Updation anomalies are present in the relations.
3. As far as possible, avoid placing attributes in a base relation whose values may frequently be NULL. If NULLs are unavoidable, make sure that they apply in exceptional cases only and do not apply to a majority of tuples in the relation.
4. Design relation schemas so that they can be joined with equality conditions on attributes that are (primary key, foreign key) pairs in a way that guarantees that no spurious tuples are generated. Avoid relations that contain matching attributes that are not (primary key, foreign key) combinations because joining on such attributes may produce spurious tuples.

## Reasons for Normalization

Normalisation is done to

1. Eliminate Redundant Data
2. Eliminate Insertion, deletion and updation anamolies. (Anamoly examples are available in the drive doc attached)
3. Ensure that the relations make sense 4. Reducing the NULL values in tuples
4. Disallowing the possibility of generating spurious tuples

## **Functional Dependency**

When a set of attributes in a table can be determined only by a certain other set of attributes.

I.e. For two sets of attributes X and Y in relation state (explained niche) r of R, for any two tuples t1 and t2 in r that have t1[X] = t2[X], they must also have t1[Y] = t2[Y]. For same value of Determinant, you get same value of Dependant.

**Trivial**  
Where Dependant is subset of Determinant  (AB -> B)

**Non-Trivial** 
Where Dependant is not subset of Determinant  (A->b)

### **Closure Set of Attributes**

Determinant Set of all attributes that can be determined by a given attribute in table

### **Full Functional Dependency**

A functional dependency X → Y is a full functional dependency if removal of any attribute A from X means that the dependency does not hold any more; that is, for any attribute A ∈ X, (X – {A}) does not functionally determine Y.

### **Partial Functional Dependency**

When a set of attributes in a table depends on only a subset of a key and not on the whole key. A functional dependency X → Y is a partial functional dependency if some attribute A ∈ X can be removed from X and the dependency still holds; that is, for some A ∈ X, (X – {A}) → Y.

## Normalization

**1. First Normal Form (1NF)**

- First Normal Form was defined to disallow multivalued attributes, composite attributes and their combinations.
- It states that the domain of an attribute must include only atomic (simple, indivisible) values.

**2. Second Normal Form (2NF)**

- Second Normal Form is based on the concept of full functional dependency.
- The relation should be in 1NF.
- A relation schema R is in 2NF if every nonprime attribute A in R is fully Functionally dependent on the primary key of R.

**3. Third Normal Form (3NF)**

- Third Normal Form is based on the concept of transitive dependency.
- The relation should be in 2NF.
- According to Codd, a relation schema R is in 3NF if it satisfies 2NF and no nonprime attribute of R is transitively dependent of the primary key.

## **Glossary**

Primary Key

That set of attributes that uniquely identifies a tuple in your table. The closure of this set is all your attributes.

In SQL, none of the columns that are part of a primary key can have NULL values. They must be unique. Foreign keys, however, are nullable. Foreign key values, however, must come from the speficied range in the Primary Key.

Relation Schema:

R(A1,A2,…An),a relation name an list of attributes(each of which has a name and a domain)

**Domain**: A list of possible values an attribute can take (int, varchar etc.)

Attribute:

the name of a role played by a domain in that relation.

Relation state:

r(R),a set of tuples. Each tuple is an order list of values,corresponding with the domain of their attribute,and representing a fact. A relation state is a subset of the Cartesian product of the domains defining the relation schema. (Basically a relation with valud values)

Notation:

- Q,R,S: relation namesre
- R(A1..):relation schema(intension of the relation)
- q,r,s:relation states(extension of the relation)
- t,u,v:tuples
- a relation name by itself refers to the current relation state,not the relation schema.

Charcterstics of relations:

1. a relation is a set-tuples are not in any order,and have no duplicates.
2. Flat relational model-values are atomic,not structures or lists.
3. NULL- not a member of any domain.
4. Semantics of relation-ecah tuple is an assertion of true fact.

Retrieval:

a function taking a relational algebra expression,returning a result relation.

Insert:

Adds a new tuple to a relation.Can potentially violate any constraint.

Delete:

Remove a tuple from a relation.Canviolate referential integrity.

Update:

Modify a tuple in a relation. Depending on whats modified, can violate whatever the previous two can.