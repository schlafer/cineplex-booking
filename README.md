# Cineplex booking

A ticket booking system to solve for the double booking system and 
provide a good UX for clients when booking a ticket and 
avoid conflicts in a high contention environment.

## The Plan
- Start simple and gradually increase justified complexity.
- Build the solution in Repository pattern so that it is clean, reusable and compose-able. It can be easily changed.
- Implement a robust production solution in Redis.

## Functional Requirements

- Synchronous approach 
  1 ticket counter for serialized users. Slow big queue. Not scalable.
- Async approach
  2 or more counter. For same movie queue is halved. But each counter needs to keep in check with each other for every ticket sold to avoid double booking.
- Movie-specific queue
  For different movies, if one movie is more popular than others that queue is longer.
- Online booking
  What we will be working on. Similar problems as above approaches.

### The Problem - Double booking

Two users click "Book" on seat A1 at the same instant. Only one should win.

```bash
User A ──► read seat A1 → "free" ──► write booking ──► success
User B ──► read seat A1 → "free" ──► write booking ──► ???
 ```

Without any protection, both succeed. Now two people show up for the same seat.

#### Strat 1: Locking
Lock the resource before read. Hold the lock through entire read-check-write cycle. Everyone else waits in line.

#### Strat 2: Concurrency
No locking. Read freely. Do the work. Then attempt to write. If someone changed the data between your read and write, your write fails, retry or abandon.

### Solution
Ticket booking in Cineplex has high contention by design - popular showtimes, limited seats, opening night rushes. So Concurrency in Go and Redis as persistent storage is a good fit here.

Naive solution is implemented in memory_store.go.
> It fails in service_test.go due to race condition.

Concurrency is implemented in concurrent_store.go
> It succeeds in service_test.go.

Redis DB is implemented in redis_store.go
> It succeeds in service_test.go
