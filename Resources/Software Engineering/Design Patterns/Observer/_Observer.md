# Observer

## Elements

- **Subject / Publisher :** Object that has some interesting state; it’s going to notify other objects about the changes to its state.
- **Subscriber** : All other objects that want to track changes to the publisher’s state.

## Idea

This pattern suggests that you add a subscription mechanism to the publisher class so individual objects can subscribe to or unsubscribe from a stream of events coming from that publisher. 

Thus, this mechanism consists of

1. an array field for storing a list of references to subscriber objects
2. several public methods for adding and removing subscribers from that list

![Untitled](Observer/Untitled.png)

1. on an event, the publisher calls a specific notification method on subscriber ojects.
2. to ensure all publisher can interact with subscribers, all of them must implement an interface which declares notification method and some parameters to pass data. publisher uses only this interface to interact with subscribers. 

![Untitled](Observer/Untitled%201.png)

```java
interface IObservable {
	void add(IObserver observer);
	void remove(IObserver observer);
	void notify();
	void changeState();
}

interface IObserver {
	void update();
}

class Topic implements IObservable {
	List<IObserver> observers;
	int state;
	
	public void add(IObserver observer) {
		observers.add(observer);
	}
	public void remove(IObserver observer) {
		observers.remove(observer);
	}
	// another way to implement this is to keep observable object in 
	// observer, instead of passing this in the update call
	public void notify() {
		for(IObserver observer: observers)
			observers.update(this);
	}
	// function that will be called by external code
	public void changeState(int newState) {
		this.state = newState;
	}
	// function that will be called by observer
	public int getState() {
		return this.state;
	}
}

class Queue implements IObserver {
	void update(IObservable observable) {
		// observable.getState();
	}
}
```