# Consistency Models Comparison Table

## Overview Table

| Model | Strength | Real-time Order | Global Order | Causality | Availability | Use Cases |
|-------|----------|----------------|--------------|-----------|--------------|-----------|
| **Strict Serializable** | Strongest | ✅ Required | ✅ Required | ✅ Preserved | ❌ Not Available | Banking, Financial |
| **Linearizable** | Very Strong | ✅ Required | ✅ Required | ✅ Preserved | ❌ Not Available | Configuration, Coordination |
| **Sequential** | Strong | ❌ Not Required | ✅ Required | ✅ Preserved | ❌ Not Available | Distributed Caches |
| **Serializable** | Strong | ❌ Not Required | ✅ Required | ✅ Preserved | ❌ Not Available | ACID Databases |
| **Snapshot Isolation** | Medium-Strong | ❌ Not Required | ❌ Not Required | ⚠️ Partial | ⚠️ Limited | MVCC Databases |
| **Repeatable Read** | Medium-Strong | ❌ Not Required | ❌ Not Required | ⚠️ Partial | ⚠️ Limited | SQL Databases |
| **Causal** | Medium | ❌ Not Required | ❌ Not Required | ✅ Preserved | ⚠️ Limited | Social Media, Chat |
| **PRAM** | Medium-Weak | ❌ Not Required | ❌ Not Required | ⚠️ Per-Process | ✅ Available | Distributed Systems |
| **Read Committed** | Weak | ❌ Not Required | ❌ Not Required | ❌ Not Preserved | ✅ Available | Web Applications |
| **Monotonic Reads** | Weak | ❌ Not Required | ❌ Not Required | ❌ Not Preserved | ✅ Sticky Available | Session-based Apps |
| **Monotonic Writes** | Weak | ❌ Not Required | ❌ Not Required | ❌ Not Preserved | ✅ Sticky Available | Logging Systems |
| **Read Your Writes** | Weak | ❌ Not Required | ❌ Not Required | ❌ Not Preserved | ✅ Sticky Available | User Profiles |
| **Read Uncommitted** | Very Weak | ❌ Not Required | ❌ Not Required | ❌ Not Preserved | ✅ Available | Analytics, Reporting |
| **Eventual** | Weakest | ❌ Not Required | ❌ Not Required | ❌ Not Preserved | ✅ Totally Available | DNS, CDN |

## Detailed Properties

### Transaction Isolation Levels

| Model | Dirty Read | Non-repeatable Read | Phantom Read | Write Skew |
|-------|------------|-------------------|--------------|------------|
| **Strict Serializable** | ❌ Prevented | ❌ Prevented | ❌ Prevented | ❌ Prevented |
| **Serializable** | ❌ Prevented | ❌ Prevented | ❌ Prevented | ❌ Prevented |
| **Repeatable Read** | ❌ Prevented | ❌ Prevented | ✅ Possible | ✅ Possible |
| **Snapshot Isolation** | ❌ Prevented | ❌ Prevented | ❌ Prevented | ✅ Possible |
| **Read Committed** | ❌ Prevented | ✅ Possible | ✅ Possible | ✅ Possible |
| **Read Uncommitted** | ✅ Possible | ✅ Possible | ✅ Possible | ✅ Possible |

### Performance vs. Consistency Trade-offs

| Model | Latency | Throughput | Network Overhead | Implementation Complexity |
|-------|---------|------------|------------------|--------------------------|
| **Strict Serializable** | Very High | Very Low | Very High | Very Complex |
| **Linearizable** | High | Low | High | Complex |
| **Sequential** | High | Low | High | Complex |
| **Serializable** | High | Low | Medium | Complex |
| **Snapshot Isolation** | Medium | Medium | Medium | Medium |
| **Causal** | Medium | Medium | Medium | Medium |
| **PRAM** | Low | High | Low | Simple |
| **Read Committed** | Low | High | Low | Simple |
| **Session Guarantees** | Low | High | Low | Simple |
| **Eventual** | Very Low | Very High | Very Low | Very Simple |

## Anomaly Prevention

### Read Anomalies
- **Dirty Read**: Reading uncommitted data that might be rolled back
- **Non-repeatable Read**: Getting different values when reading the same data twice
- **Phantom Read**: New rows appearing between reads in a range query

### Write Anomalies
- **Lost Update**: Two concurrent updates, one overwrites the other
- **Write Skew**: Two transactions read overlapping data and make disjoint updates
- **Read Skew**: Reading inconsistent data across multiple reads

| Model | Lost Update | Write Skew | Read Skew |
|-------|-------------|------------|-----------|
| **Strict Serializable** | ❌ Prevented | ❌ Prevented | ❌ Prevented |
| **Linearizable** | ❌ Prevented | N/A (Single Object) | ❌ Prevented |
| **Serializable** | ❌ Prevented | ❌ Prevented | ❌ Prevented |
| **Snapshot Isolation** | ❌ Prevented | ✅ Possible | ❌ Prevented |
| **Repeatable Read** | ❌ Prevented | ✅ Possible | ❌ Prevented |
| **Read Committed** | ✅ Possible | ✅ Possible | ✅ Possible |

## Real-World System Examples

### Strong Consistency Systems
- **Google Spanner**: Strict Serializable (globally)
- **FaunaDB**: Strict Serializable
- **FoundationDB**: Strict Serializable
- **etcd**: Linearizable
- **Consul**: Linearizable
- **Zookeeper**: Sequential Consistency

### Medium Consistency Systems
- **PostgreSQL**: Serializable, Repeatable Read, Read Committed
- **MySQL**: Repeatable Read, Read Committed
- **MongoDB**: Causal Consistency (with causal reads)
- **Cassandra**: Eventual + Tunable Consistency

### Weak Consistency Systems
- **Redis**: Eventual Consistency
- **DynamoDB**: Eventual + Strong Consistency options
- **Riak**: Eventual Consistency
- **DNS**: Eventual Consistency
- **CDNs**: Eventual Consistency

## Choosing the Right Model

### Decision Matrix

**Use Strict Serializable when:**
- Financial transactions
- Inventory management
- Any system where correctness is critical
- You can tolerate high latency

**Use Linearizable when:**
- Configuration management
- Leader election
- Coordination services
- Single-object operations with strong consistency needs

**Use Sequential when:**
- Distributed state machines
- Replicated logs
- Systems needing global order without real-time constraints

**Use Causal when:**
- Social media feeds
- Collaborative editing
- Chat applications
- Comment threads

**Use Session Guarantees when:**
- User profile systems
- Shopping carts
- Personal dashboards
- Single-user workflows

**Use Eventual when:**
- Content delivery
- DNS resolution
- Analytics data
- Systems prioritizing availability over consistency

## Summary

The choice of consistency model involves fundamental trade-offs:

1. **Consistency vs. Availability**: Stronger models provide better guarantees but limit availability
2. **Consistency vs. Performance**: Stronger models typically have higher latency and lower throughput
3. **Consistency vs. Partition Tolerance**: Stronger models are more sensitive to network partitions

The key is to choose the weakest consistency model that still meets your application's correctness requirements, as this will give you the best performance and availability characteristics.
