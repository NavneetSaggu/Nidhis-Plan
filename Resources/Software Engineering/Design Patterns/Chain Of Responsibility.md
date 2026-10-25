# Chain Of Responsibility

- Used when there’s an order of priority in which things need to be called
- Think of cache tiers; tier 1 is called first and then calls go to 2, 3 and so on

```java
interface Cache {
	void put(int key, int value);
	int get(int key);
}

class L1Cache implements Cache {
	Cache nextTier;
	
	void setNextTier(Cache cache) {
		this.nextTier = cache;
	}

	void put(int key, int value) {
		// implementation
	}
	
	int get(int key) {
		// implementation
	}
}

class L2Cache implements Cache {
	Cache nextTier;

	void setNextTier(Cache cache) {
		this.nextTier = cache;
	}

	void put(int key, int value) {
		// implementation
	}
	
	int get(int key) {
		// implementation
	}
}

class L3Cache implements Cache {
	Cache nextTier;
	
	void setNextTier(Cache cache) {
		this.nextTier = cache;
	}

	void put(int key, int value) {
		// implementation
	}
	
	int get(int key) {
		// implementation
	}
}

class CacheManager {
	public CacheManager() {
		L1Cache l1Cache = new L1Cache();
		L2Cache l2Cache = new L2Cache();
		L3Cache l3Cache = new L3Cache();
		l1Cache.setNextTier(l2Cache);
		l2Cache.setNextTie(l3Cache);
	}
}
```