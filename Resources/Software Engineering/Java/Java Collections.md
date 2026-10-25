# Java Collections

### `priority_queue` (C++)

Java Equivalent: `PriorityQueue<E>`

- `push(x)` → `pq.add(x)` or `pq.offer(x)` , both work the same in unbounded queues.
- `pop()` → `pq.poll()`
- `top()` → `pq.peek()`

**Note:** Java `PriorityQueue` is a min-heap by default (unlike C++). To create a max-heap, use a custom comparator:

```java
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
```

In Java, a `Comparator<T>` must implement the method:

```java
int compare(T a, T b);
```

The result must be:

- **negative** if `a < b`
- **zero** if `a == b`
- **positive** if `a > b`

---

### `queue` (C++)

Java Equivalent: `Queue<E>` interface (e.g., `LinkedList<E>`)

- `push(x)` → `queue.add(x)` or `queue.offer(x)`
- `pop()` → `queue.poll()`
- `front()` → `queue.peek()`
- `back()` → No direct equivalent. Use `Deque` (like `LinkedList`) and `getLast()`.
- `empty()` → `queue.isEmpty()`
- `size()` → `queue.size()`

---

### `set` (C++)

Java Equivalent: `Set<E>` (e.g., `HashSet<E>`, `TreeSet<E>`)

- `find(x)` → `set.contains(x)`
- `insert(x)` → `set.add(x)`
- `erase(x)` → `set.remove(x)`

---

### `map` (C++)

Java Equivalent: `Map<K, V>` (e.g., `HashMap<K, V>`)

- `m[key]` → `map.get(key)` or `map.put(key, value)`
- `find(key)` → `map.containsKey(key)`
- `insert(key, val)` → `map.put(key, val)`
- `begin(), end()` → `map.entrySet().iterator()` or enhanced `for` loop
- `count(key)` → `map.containsKey(key)`
- `erase(key)` → `map.remove(key)`
- `clear()` → `map.clear()`

**NOTES:**

- Like C++, you can operate directly on empty maps:
    
    ```java
    java
    CopyEdit
    Map<Integer, List<Integer>> m = new HashMap<>();
    m.computeIfAbsent(1, k -> new ArrayList<>()).add(2);
    
    ```
    

---

### `unordered_map` (C++)

Java Equivalent: `HashMap<K, V>`

Same API as `map`.

---

### `vector` (C++)

Java Equivalent: `ArrayList<E>`

- `[]` → `list.get(i)` and `list.set(i, val)`
- `push_back()` → `list.add(val)`
- `pop_back()` → `list.remove(list.size() - 1)`
- `erase(pos)` → `list.remove(index)`
- `erase(start, end)` → `list.subList(start, end).clear()`
- `resize(n)` → Use loops or `ensureCapacity()` + manual adds
- `insert(pos, val)` → `list.add(index, val)`

**NOTES:**

```java
java
CopyEdit
List<Pair<Integer, Integer>> list = Arrays.asList(new Pair<>(Integer.MAX_VALUE, 0), new Pair<>(Integer.MAX_VALUE, 0));

```

(Or use a custom `Pair` class or `AbstractMap.SimpleEntry`.)

- `distance(it1, it2)` → `it2 - it1` using index

---

### `string` (C++)

Java Equivalent: `String`

- `[]` → `str.charAt(i)`
- `length()` → `str.length()`
- `rfind(substr)` → `str.lastIndexOf(substr)`
- `find(substr)` → `str.indexOf(substr)`
- To find all occurrences:
    
    ```java
    java
    CopyEdit
    int pos = str.indexOf(substr);
    while (pos != -1) {
        System.out.println("Found at: " + pos);
        pos = str.indexOf(substr, pos + 1);
    }
    
    ```
    

---

### `pair` (C++)

Java Equivalent: `AbstractMap.SimpleEntry<K, V>` or custom `Pair` class

- Initialization:
    
    ```java
    java
    CopyEdit
    Map.Entry<Integer, Integer> p = new AbstractMap.SimpleEntry<>(2, 3);
    
    ```
    
- Destructuring (not directly supported before Java 14; use getters)
- Inline insert:
    
    ```java
    java
    CopyEdit
    Queue<Map.Entry<String, Double>> q = new LinkedList<>();
    q.offer(new AbstractMap.SimpleEntry<>("Hello", 0.1));
    
    ```
    

---

### `for (int i : {1,2,3})`

Java Equivalent:

```java
java
CopyEdit
for (int i : new int[]{1, 2, 3}) {
    // do something
}

```

Let me know if you need examples for any specific data structure conversions or usage patterns.