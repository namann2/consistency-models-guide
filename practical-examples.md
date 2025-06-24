# Practical Examples of Consistency Models

## Banking System Example

Let's explore how different consistency models would handle a simple banking scenario:

**Scenario**: Alice has $100 in account A, Bob has $50 in account B. Alice transfers $30 to Bob.

### Strict Serializable
```
Initial: A=$100, B=$50

Transaction T1: Transfer $30 from A to B
- Read A: $100
- Write A: $70  
- Read B: $50
- Write B: $80

Transaction T2: Read both accounts (concurrent with T1)

Strict Serializable guarantees:
- T2 sees either (A=$100, B=$50) OR (A=$70, B=$80)
- Never sees partial state like (A=$70, B=$50)
- If T1 completes before T2 starts, T2 MUST see T1's effects
```

### Snapshot Isolation
```
Initial: A=$100, B=$50

T1 starts: sees snapshot (A=$100, B=$50)
T2 starts: sees same snapshot (A=$100, B=$50)

T1: Write A=$70, Write B=$80, Commit
T2: Read A=$100, Read B=$50 (from its snapshot)

Result: T2 sees the old snapshot, not T1's changes
```

### Read Committed
```
Initial: A=$100, B=$50

T1: Write A=$70 (uncommitted)
T2: Read A → returns $100 (can't see uncommitted data)
T1: Write B=$80, Commit
T2: Read A → returns $70 (now committed)
T2: Read B → returns $80 (committed)

Result: T2 might see inconsistent state during T1's execution
```

## Social Media Feed Example

**Scenario**: User posts a message, then comments on it. How do different users see this?

### Causal Consistency
```
User Alice: Post "Hello World!" → Comment "This is my first post!"

Causal relationship: Comment depends on Post

All users will see:
✅ Post before Comment (causally related)
❌ Comment before Post (violates causality)

But concurrent posts from different users can appear in any order.
```

### Sequential Consistency
```
Timeline:
- Alice posts "Hello"
- Bob posts "Hi there"  
- Alice comments "My first post"

Sequential Consistency guarantees:
All users see the same order, e.g.:
1. Alice: "Hello"
2. Bob: "Hi there"  
3. Alice: "My first post"

But the order might not match real-time if operations were concurrent.
```

### Eventual Consistency
```
Alice posts → Propagates to servers worldwide
Bob (in US) sees post immediately
Charlie (in Europe) sees post after 100ms
David (in Asia) sees post after 200ms

Eventually: All users see the same content, but timing varies
```

## E-commerce Shopping Cart

### Read Your Writes
```
User adds item to cart:
1. POST /cart/add {item: "laptop", price: 999}
2. GET /cart → MUST show laptop in cart

Without Read Your Writes:
User might not see their own addition immediately!
```

### Monotonic Reads
```
User views cart multiple times:
1. GET /cart → {laptop: $999, total: $999}
2. GET /cart → {laptop: $999, mouse: $25, total: $1024}
3. GET /cart → Never shows less than step 2

Monotonic Reads prevents:
❌ Seeing total go backwards
❌ Items disappearing then reappearing
```

## Distributed Counter Example

### Linearizable
```
Counter starts at 0

Process A: increment() → returns 1
Process B: increment() → returns 2  
Process C: read() → returns 2

All operations appear to happen atomically in real-time order.
```

### PRAM Consistency
```
Process A: increment(), increment(), increment() → sees 1, 2, 3
Process B: increment(), increment() → sees 1, 2
Process C: read() → might see any value, but A's operations stay ordered

A always sees its own operations in order: 1, 2, 3
B always sees its own operations in order: 1, 2
But global view can vary between processes
```

## Database Transaction Examples

### Write Skew in Snapshot Isolation
```
Hospital shift scheduling:
Rule: At least one doctor must be on call

Initial: Dr. Alice ON_CALL, Dr. Bob ON_CALL

Transaction T1 (Alice):
- Read: Alice=ON_CALL, Bob=ON_CALL ✓ (constraint satisfied)
- Write: Alice=OFF_CALL

Transaction T2 (Bob):  
- Read: Alice=ON_CALL, Bob=ON_CALL ✓ (constraint satisfied)
- Write: Bob=OFF_CALL

Both commit successfully!
Final: Alice=OFF_CALL, Bob=OFF_CALL ❌ (constraint violated)

Snapshot Isolation allows this write skew!
```

### Phantom Read in Repeatable Read
```
Banking: Calculate total balance across all accounts

Transaction T1:
- SELECT SUM(balance) FROM accounts WHERE user_id=123 → $1000
- (Another transaction inserts new account with $500)
- SELECT SUM(balance) FROM accounts WHERE user_id=123 → $1500

Same query, different results = Phantom Read
```

## Real-World System Behaviors

### DNS (Eventual Consistency)
```
1. Update DNS record: example.com → 1.2.3.4
2. Propagation begins to DNS servers worldwide
3. Users in different locations see different IPs temporarily
4. Eventually: All DNS servers have the new record

Timeline:
T0: Update made
T1: 30% of servers updated  
T2: 70% of servers updated
T3: 100% of servers updated (eventual consistency achieved)
```

### CDN (Eventual Consistency)
```
1. Upload new version of website to origin server
2. CDN edge servers gradually update their caches
3. Users see different versions based on which edge server they hit

User in NYC: Sees new version (edge server updated)
User in London: Sees old version (edge server not updated yet)
User in Tokyo: Sees new version (edge server updated)

Eventually: All edge servers serve the new version
```

### Distributed Cache (Sequential Consistency)
```
Cache cluster with 3 nodes

Operation sequence:
1. SET key1=value1
2. SET key2=value2  
3. GET key1
4. GET key2

Sequential Consistency guarantees:
- All nodes see operations in same order
- GET operations return consistent values
- But order might not match real-time if SETs were concurrent
```

## Code Examples

### Implementing Read Your Writes
```python
class UserSession:
    def __init__(self, user_id):
        self.user_id = user_id
        self.last_write_timestamp = 0
    
    def write(self, data):
        # Write to database
        timestamp = database.write(self.user_id, data)
        self.last_write_timestamp = timestamp
        return timestamp
    
    def read(self):
        # Ensure we read from a replica that has our writes
        return database.read_after_timestamp(
            self.user_id, 
            self.last_write_timestamp
        )
```

### Implementing Monotonic Reads
```python
class MonotonicReader:
    def __init__(self):
        self.last_read_timestamp = 0
    
    def read(self, key):
        # Only read from replicas with timestamp >= last read
        data, timestamp = database.read_after(key, self.last_read_timestamp)
        self.last_read_timestamp = max(self.last_read_timestamp, timestamp)
        return data
```

### Causal Consistency Example
```python
class CausalStore:
    def __init__(self):
        self.vector_clock = {}
    
    def write(self, key, value, depends_on=None):
        # Update vector clock based on dependencies
        if depends_on:
            self.vector_clock = merge_clocks(self.vector_clock, depends_on)
        
        self.vector_clock[self.node_id] += 1
        
        return database.write(key, value, self.vector_clock.copy())
    
    def read(self, key):
        # Only return values that respect causal ordering
        return database.read_causally_consistent(key, self.vector_clock)
```

## Performance Implications

### Latency Comparison
```
Operation: Read from distributed system

Eventual Consistency:    ~1ms   (local read)
Read Your Writes:        ~5ms   (check local writes)
Monotonic Reads:         ~10ms  (find consistent replica)
Causal Consistency:      ~50ms  (check causal dependencies)
Sequential Consistency:  ~100ms (coordinate global order)
Linearizable:           ~200ms (strong coordination)
Strict Serializable:    ~500ms (global coordination + transactions)
```

### Throughput Comparison
```
Requests per second for a distributed system:

Eventual Consistency:    100,000 RPS
Session Guarantees:      50,000 RPS
Causal Consistency:      10,000 RPS
Sequential Consistency:  1,000 RPS
Linearizable:           500 RPS
Strict Serializable:    100 RPS
```

## Choosing Based on Use Case

### Financial System
```
Account Balance: Linearizable (critical accuracy)
Transaction History: Sequential (ordered view)
User Preferences: Read Your Writes (personal settings)
Analytics Data: Eventual (can tolerate delays)
```

### Social Media Platform
```
User Posts: Causal (replies after original posts)
Friend Lists: Read Your Writes (see your own connections)
Trending Topics: Eventual (aggregated data)
Direct Messages: Sequential (conversation order)
```

### E-commerce Site
```
Inventory Count: Linearizable (prevent overselling)
Shopping Cart: Read Your Writes (see your additions)
Product Catalog: Eventual (can cache aggressively)
Order History: Monotonic Reads (never lose orders)
```

These examples show how different consistency models solve different problems, and how the same application might use multiple consistency models for different data types based on their specific requirements.
