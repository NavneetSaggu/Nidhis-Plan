# Adapter

1. Used when functionality of an incompatible type has to be provided to an incompatible type. Think of a dog who needs to behave like a lion, think of putting him in a lion costume.
2. Wrap the adapting type in an adapter, that extends your main type.

```java
// Target Interface
interface Lion {
	String roar();
}

class AsiaticLion implements Lion {
	String roar() {
		return "GrrrrrKemcho";
	}
}

class SaharanLion implements Lion {
	String roar() {
		return "GRRRRRILLFUCKYOUUP";
	}
}

// Adaptee
class Dog {
	Sring bark() {
		return "bowow";
	}
}

// Adapter
class DogLionAdapter implements Lion {
	Dog dog;
	String roar() {
		return dog.bark();
	}
}
```

# Where to use this?

- Suppose you have two buddies, one of them speaks French exclusively and the other English exclusively. The language barrier prevents them from communicating the way you want them to.