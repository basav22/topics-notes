# Java Collections Framework

---

## Collection Hierarchy

```
Iterable
  └── Collection
        ├── List        → ordered, allows duplicates
        │     ├── ArrayList
        │     ├── LinkedList
        │     └── Vector (legacy)
        │           └── Stack (legacy)
        ├── Set         → no duplicates
        │     ├── HashSet
        │     ├── LinkedHashSet
        │     └── TreeSet (sorted)
        └── Queue       → FIFO ordering
              ├── PriorityQueue
              ├── ArrayDeque
              └── Deque (interface)

Map (separate hierarchy — not part of Collection)
  ├── HashMap
  ├── LinkedHashMap
  ├── TreeMap (sorted)
  ├── Hashtable (legacy)
  └── ConcurrentHashMap
```

---

## List Implementations

### ArrayList

- Backed by **dynamic array**
- Fast random access — `get(i)` is **O(1)**
- Slow insert/delete in middle — **O(n)** (shifts elements)
- Default capacity: **10**, grows by **50%**

```java
List<String> list = new ArrayList<>();
list.add("Java");
list.add("Python");
list.add(1, "C++");         // insert at index 1
list.get(0);                 // "Java"
list.set(1, "Go");           // replace at index 1
list.remove("Python");       // remove by value
list.remove(0);              // remove by index
list.contains("Go");         // true
list.size();                 // 1
list.isEmpty();              // false

// iterate
for (String s : list) { System.out.println(s); }
list.forEach(System.out::println);
```

### LinkedList

- Backed by **doubly linked list**
- Fast insert/delete at head/tail — **O(1)**
- Slow random access — `get(i)` is **O(n)**
- Also implements `Deque` (can be used as stack/queue)

```java
LinkedList<String> list = new LinkedList<>();
list.addFirst("A");
list.addLast("B");
list.getFirst();             // "A"
list.getLast();              // "B"
list.removeFirst();
list.removeLast();

// as Queue
list.offer("C");             // add to tail
list.poll();                 // remove from head

// as Stack
list.push("D");              // add to head
list.pop();                  // remove from head
```

### ArrayList vs LinkedList — When to Use

| Operation | ArrayList | LinkedList |
|-----------|-----------|------------|
| `get(index)` | **O(1)** | O(n) |
| `add(end)` | **O(1)** amortized | **O(1)** |
| `add(middle)` | O(n) | **O(1)** if at iterator pos |
| `remove(middle)` | O(n) | **O(1)** if at iterator pos |
| Memory | Less (contiguous) | More (node pointers) |

**Rule of thumb:** Use `ArrayList` by default. Use `LinkedList` only when you do frequent insert/delete at head or need deque behavior.

---

## Set Implementations

### HashSet

- Backed by **HashMap** internally
- No ordering, no duplicates
- **O(1)** for add, remove, contains
- Allows **one null**

```java
Set<String> set = new HashSet<>();
set.add("Java");
set.add("Python");
set.add("Java");             // ignored — duplicate
set.size();                  // 2
set.contains("Java");        // true
set.remove("Python");

for (String s : set) { System.out.println(s); } // order not guaranteed
```

### LinkedHashSet

- **Maintains insertion order**
- Slightly slower than HashSet (maintains a linked list)

```java
Set<String> set = new LinkedHashSet<>();
set.add("B");
set.add("A");
set.add("C");
// iteration order: B, A, C (insertion order preserved)
```

### TreeSet

- Backed by **TreeMap** (Red-Black Tree)
- Elements are **sorted** (natural order or custom Comparator)
- **O(log n)** for add, remove, contains
- Does **not** allow null

```java
Set<Integer> set = new TreeSet<>();
set.add(30);
set.add(10);
set.add(20);
// iteration order: 10, 20, 30 (sorted)

set.first();                 // 10
set.last();                  // 30

// custom sorting
Set<String> descSet = new TreeSet<>(Comparator.reverseOrder());
descSet.addAll(List.of("A", "C", "B"));
// iteration: C, B, A
```

### HashSet vs LinkedHashSet vs TreeSet

| Feature | HashSet | LinkedHashSet | TreeSet |
|---------|---------|---------------|---------|
| Order | None | Insertion order | Sorted |
| Performance | **O(1)** | **O(1)** | O(log n) |
| Null | 1 null | 1 null | No null |
| Use case | General | Need order preserved | Need sorting |

---

## Map Implementations

### HashMap

- Key-value pairs, **no ordering**
- **O(1)** for get, put, remove
- Allows **one null key**, multiple null values
- **Not thread-safe**

```java
Map<String, Integer> map = new HashMap<>();
map.put("Java", 1);
map.put("Python", 2);
map.put("Java", 10);        // overwrites — key already exists

map.get("Java");             // 10
map.getOrDefault("C++", 0); // 0 (key not found)
map.containsKey("Java");    // true
map.containsValue(10);      // true
map.remove("Python");
map.size();                  // 1

// iterate
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " = " + entry.getValue());
}

map.forEach((k, v) -> System.out.println(k + " = " + v));

// useful methods
map.putIfAbsent("Go", 3);   // only puts if key doesn't exist
map.compute("Java", (k, v) -> v + 1);  // 11
map.merge("Java", 5, Integer::sum);    // 16
```

### HashMap Internals (Interview Favorite)

```
Bucket Array (default size 16)
[0] -> null
[1] -> Node("Java", 10) -> Node("Go", 3) -> null   (linked list / tree)
[2] -> null
...
[15] -> Node("Python", 2) -> null
```

- **Hash** → `key.hashCode()` → compressed to bucket index
- **Collision** → chaining (LinkedList, converts to **Red-Black Tree** if bucket size >= 8, Java 8+)
- **Load factor** → default **0.75** → rehash (double size) when 75% full
- **Rehashing** → all entries re-distributed to new buckets

### LinkedHashMap

- Maintains **insertion order** (or access order)

```java
Map<String, Integer> map = new LinkedHashMap<>();
map.put("B", 2);
map.put("A", 1);
map.put("C", 3);
// iteration: B=2, A=1, C=3

// LRU Cache using access order
Map<String, Integer> lru = new LinkedHashMap<>(16, 0.75f, true) {
    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > 3; // max 3 entries
    }
};
```

### TreeMap

- Sorted by keys (Red-Black Tree)
- **O(log n)** operations

```java
TreeMap<String, Integer> map = new TreeMap<>();
map.put("Banana", 2);
map.put("Apple", 1);
map.put("Cherry", 3);
// iteration: Apple=1, Banana=2, Cherry=3

map.firstKey();              // "Apple"
map.lastKey();               // "Cherry"
map.lowerKey("Banana");      // "Apple" (strictly less)
map.higherKey("Banana");     // "Cherry" (strictly greater)
map.subMap("Apple", "Cherry"); // Apple=1, Banana=2
```

### HashMap vs LinkedHashMap vs TreeMap

| Feature | HashMap | LinkedHashMap | TreeMap |
|---------|---------|---------------|---------|
| Order | None | Insertion/Access | Sorted by key |
| Performance | **O(1)** | **O(1)** | O(log n) |
| Null key | Yes (1) | Yes (1) | No |
| Use case | General | LRU cache, order | Range queries, sorted |

---

## Queue & Deque

```java
// Queue (FIFO)
Queue<String> queue = new LinkedList<>();
queue.offer("A");            // add to tail
queue.offer("B");
queue.peek();                // "A" (doesn't remove)
queue.poll();                // "A" (removes)

// PriorityQueue (min-heap by default)
Queue<Integer> pq = new PriorityQueue<>();
pq.offer(30);
pq.offer(10);
pq.offer(20);
pq.poll();                   // 10 (smallest first)

// Max-heap
Queue<Integer> maxPq = new PriorityQueue<>(Comparator.reverseOrder());

// Deque (double-ended queue) — prefer ArrayDeque over Stack
Deque<String> deque = new ArrayDeque<>();
deque.push("A");             // stack push
deque.push("B");
deque.pop();                 // "B" (stack pop — LIFO)
deque.offerFirst("X");      // add to front
deque.offerLast("Y");       // add to back
```

---

## Comparable vs Comparator

```java
// Comparable — natural ordering (inside the class)
class Student implements Comparable<Student> {
    String name;
    int marks;

    public int compareTo(Student other) {
        return this.marks - other.marks; // ascending by marks
    }
}

Collections.sort(studentList); // uses compareTo

// Comparator — custom/external ordering
Comparator<Student> byName = (a, b) -> a.name.compareTo(b.name);
Comparator<Student> byMarksDesc = (a, b) -> b.marks - a.marks;

Collections.sort(studentList, byName);
studentList.sort(Comparator.comparing(Student::getName)
                           .thenComparing(Student::getMarks));
```

---

## Common Utility Methods

```java
// Collections utility class
Collections.sort(list);
Collections.reverse(list);
Collections.shuffle(list);
Collections.frequency(list, "Java");  // count occurrences
Collections.min(list);
Collections.max(list);
Collections.unmodifiableList(list);   // read-only view
Collections.synchronizedList(list);   // thread-safe wrapper

// Arrays utility
Arrays.sort(arr);
Arrays.binarySearch(arr, key);
Arrays.asList(1, 2, 3);              // fixed-size list
List.of(1, 2, 3);                    // immutable list (Java 9+)
Map.of("a", 1, "b", 2);             // immutable map (Java 9+)
```

---

## Interview Quick Reference

| Question | Answer |
|----------|--------|
| HashMap vs Hashtable | HashMap is not synchronized, allows null; Hashtable is synchronized, no null |
| fail-fast vs fail-safe | fail-fast throws `ConcurrentModificationException`; fail-safe works on copy (`CopyOnWriteArrayList`) |
| How HashMap handles collision | Chaining → LinkedList → TreeMap (8+ nodes) |
| When to use TreeMap | Need sorted keys or range queries |
| ArrayList default capacity | 10, grows by 50% |
| HashMap default capacity | 16, load factor 0.75 |
