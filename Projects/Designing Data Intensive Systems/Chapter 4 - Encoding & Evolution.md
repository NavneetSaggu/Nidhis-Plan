## Why Encoding Matters

Applications change over time — schema changes, new fields, removed fields. Old and new code often coexist:
- **Rolling upgrades**: new code deployed to some nodes, old code still running on others
- **Clients**: users don't always upgrade immediately

This requires **backward compatibility** (new code can read data written by old code) and **forward compatibility** (old code can read data written by new code).

Data in memory needs to be **encoded** (serialized) to bytes for storage or transmission, and **decoded** (deserialized) on the other end.

---

## Formats for Encoding Data

### Language-Specific Formats

Built-in serialization: Java's `Serializable`, Ruby's `Marshal`, Python's `pickle`.

**Problems:**
- Encoding is **tied to a particular language** — reading in another language is difficult
- Decoding needs to **instantiate arbitrary classes** → security problems (remotely executing arbitrary code)
- **Versioning** is often an afterthought
- **Efficiency** is often an afterthought (e.g. Java's built-in serialization has poor performance and large encoded size)

### JSON, XML, and Binary Variants

Standardised encodings — JSON, XML, CSV. Problems:
- **Ambiguity around numbers**: XML/CSV can't distinguish a number from a string. JSON distinguishes strings and numbers but **doesn't distinguish integers from floats** and doesn't specify precision (problematic for numbers > 2⁵³ in JavaScript)
- **No support for binary strings** (sequences of bytes without a character encoding) — workaround is Base64-encoding the data, which **increases size by 33%**
- Optional schema support exists for XML and JSON but is complex and not widely used

Despite flaws, JSON/XML are fine for many purposes — especially where human readability matters. For internal high-performance use, binary encodings are better.

**Binary encodings for JSON** (MessagePack, BSON, BJSON): keep JSON's data model but encode more compactly. Modest space savings — e.g. MessagePack encodes an 81-byte JSON record in **66 bytes**. But with a proper schema (Thrift/Protobuf/Avro) you can do far better — same record in **32 bytes**.

![[Pasted image 20260527150310.png]]

---

### Thrift and Protocol Buffers

- **Thrift** — developed at Facebook
- **Protocol Buffers (protobuf)** — developed at Google
- Both require a **schema** defined in an IDL (interface definition language)
- Both come with a **code generation tool** that takes the schema and produces classes in various languages — application code uses these generated classes to encode/decode

Both encode without field names — instead, each field has a **field tag** (a number) and a **type annotation**. This is what makes them compact.

**Thrift BinaryProtocol** — straightforward encoding, one byte per field type + tag, then the value:

![[Pasted image 20260527150544.png]]

**Thrift CompactProtocol** — packs field tag + type into a single byte, uses **variable-length integers** (smaller numbers use fewer bytes). Same data in **34 bytes** vs 59:

![[Pasted image 20260527150631.png]]

**Protocol Buffers** — similar to Thrift CompactProtocol, fits in **33 bytes**:

![[Pasted image 20260527150648.png]]

**Note on `required`/`optional`**: in the schema these are annotations, but they make **no difference to the binary encoding** — `required` simply enables a runtime check that fails if the field is missing. This is why required fields are dangerous for schema evolution.

#### Schema Evolution with Thrift / Protocol Buffers

Each field identified by its **tag number** — this is the key to compatibility:
- You can **change a field's name** freely (encoded data never uses names)
- You **cannot change a field's tag** — that would invalidate all existing encoded data
- **Adding new fields**: old code reading new data will simply **ignore unknown tag numbers** (forward compatibility). New code reading old data gets the default value for missing fields (backward compatibility). Every field added after initial deployment **must be optional or have a default value**.
- **Removing fields**: you can only remove an **optional** field. You can **never reuse the same tag number** again.
- **Changing datatypes**: possible but risky — may lose precision or truncate values
- **Protocol Buffers lists**: no dedicated list type — uses a `repeated` marker. A single-valued (`optional`) field can be promoted to `repeated` (multi-valued) safely — old code reads the last element, new code reads all elements.

---

### Avro

Avro uses a schema to specify the structure of data. Two schema languages: **Avro IDL** (human-editable) and **JSON-based** (machine-readable).

Avro encodes **without field tags and without type annotations** — just values concatenated in schema order. The most compact of the three: same record in **32 bytes**.

![[Pasted image 20260527150709.png]]

#### Writer's Schema and Reader's Schema

The key idea: the writer encodes data using the **writer's schema** (the version of the schema the writing code was compiled with). The reader decodes using the **reader's schema** (the version the reading code knows about). **These two schemas don't have to be the same — they just have to be compatible.**

Avro resolves differences between the two schemas **field by field**, matching by **field name** (not tag number):

![[Pasted image 20260527150731.png]]

- If a field exists in the writer's schema but not the reader's → **ignored**
- If a field exists in the reader's schema but not the writer's → **filled with the default value** declared in the reader's schema (this is why Avro fields must have defaults for forward/backward compatibility)
- Fields can be **reordered** freely — matched by name, not position
- **Renaming a field** requires the reader's schema to include the old name as an **alias**

#### How Does the Reader Know the Writer's Schema?

Depends on context:
- **Large file with many records** (e.g. Hadoop): writer's schema stored once at the start of the file
- **Database with individually written records**: store a version number with each record; keep a DB of schema versions
- **Network connection**: negotiate schema version at connection setup

#### Avro's Advantage: Dynamic Schema Generation

Because Avro matches fields by name (not tag), the writer's schema can be **generated dynamically** — e.g. from a relational DB's column names. Renaming a column just means generating a new Avro schema. With Thrift/Protobuf, you'd need to manually assign and maintain field tags.

Avro is also more **friendly to dynamically typed languages** — no code generation required (though it's supported for statically typed languages).

---

### The Merits of Schemas

Compared to JSON/XML:
- **More compact** — field names omitted from encoded data
- **Schema is documentation** that is always up to date (unlike manually maintained docs)
- **Compatibility checking** — can check forward/backward compatibility before deploying
- Code generation from schema provides **compile-time type checking** in statically typed languages

Thrift, Protocol Buffers, and Avro schema languages are **much simpler** than XML Schema or JSON Schema (which support detailed validation rules like regex matching).

---

## Modes of Dataflow

Three main ways data flows between processes:

### Dataflow Through Databases

- The process writing encodes; the process reading decodes — often **the same process at a different point in time**
- Backward compatibility is required (new code must read old data)
- **Forward compatibility** is also required during rolling upgrades — old code may read records written by new code, and must not corrupt unknown fields when rewriting

![[Pasted image 20260527150754.png]]

- **Data outlives code** — a value written 5 years ago may still be in the DB. Migrating all data to a new schema upfront is expensive; most DBs avoid it (schema-on-read, or `ALTER TABLE` with a null default for new columns).

### Dataflow Through Services: REST and RPC

**Services**: processes that communicate over a network. Clients make requests to servers. Servers expose an **API** (service interface). Servers themselves may be clients to other services — **microservices architecture**.

Key design goal of microservices: make services **independently deployable and evolvable** — different services may be running different versions, so API compatibility matters.

**REST**:
- Not a protocol — a **design philosophy** built on HTTP principles
- Uses HTTP features: URLs for resources, HTTP verbs (`GET`, `POST`, `PUT`, `DELETE`), content negotiation
- APIs described with **OpenAPI (Swagger)**
- Favoured for public APIs and cross-organisational services

**SOAP**:
- XML-based protocol, commonly used over HTTP but **deliberately avoids HTTP features**
- API described using **WSDL** — enables code generation so clients access remote services using local classes/method calls
- WSDL is not human-readable; SOAP messages often too complex to construct manually → **heavily dependent on tool support and code generation**
- Has largely **fallen out of favour** outside of large enterprise environments

**RPC (Remote Procedure Call)**:
- Makes a network request **look like a local function call** (location transparency) — EJB, RMI, DCOM, CORBA, gRPC
- This abstraction is **fundamentally flawed** — network requests are very different from local calls:

| Local function call | Network request |
|---|---|
| Predictably succeeds or throws | **Unpredictable** — may time out, may be lost, may be slow |
| Returns result or throws exception | May return no response at all |
| Same execution time each call | **Wildly variable latency** |
| Can pass references (pointers) | Must **encode all parameters** — problematic with large objects |
| Single language | Client and server may use **different languages** — type translation required (e.g. JS numbers > 2⁵³) |
| Idempotency not needed | **Retries may cause action to be performed multiple times** — need idempotence / deduplication |

> REST doesn't try to hide that it's a network protocol — that's part of the appeal.

**New-generation RPC frameworks** (Finagle, gRPC, Rest.li) are **explicit** about the fact that a remote request is different from a local call — use futures/promises to encapsulate async actions that may fail, and streams for sequences.

- gRPC uses Protocol Buffers
- Thrift and Avro come with RPC support
- Rest.li uses JSON over HTTP

**RPC evolution**: since client and server are often deployed independently, the server may be newer than the client. Need both backward compatibility (new server, old client) and forward compatibility (old server, new client). With REST APIs, this is typically managed via URL versioning (`/v1/`, `/v2/`) or via request headers.

### Asynchronous Message Passing

Sits between RPC and databases:
- Like RPC: messages delivered with **low latency**
- Like databases: message goes via an intermediary **message broker** (message queue) — not directly peer-to-peer

**Advantages of a message broker over direct RPC:**
- Acts as a **buffer** if the recipient is overloaded or unavailable — improves reliability
- **Automatically redelivers** messages if the recipient crashes
- Sender doesn't need to know the **IP address or port** of the recipient
- One message can be delivered to **multiple recipients**
- **Decouples sender from recipient**

**One-way communication**: sender sends a message to a queue/topic and doesn't wait for a reply (fire-and-forget). The broker delivers to one or more consumers/subscribers.

Message brokers don't enforce a data model — a message is just a **sequence of bytes**. Any encoding format works; Avro is sometimes used with message brokers that store a schema registry.

**Actor model**: a concurrency framework where logic is encapsulated in *actors*. Each actor has local state and communicates with other actors by sending/receiving async messages. Message brokers and actor frameworks can be combined (**distributed actor frameworks** like Akka, Orleans, Erlang OTP) to scale across multiple nodes — location transparency between local and remote actors.
