This file contains AI generated explanations

```
import javax.servlet.http.*;
import java.util.concurrent.atomic.*;
import java.util.concurrent.locks.*;

/* ============================================================================
   HOW JAVA ENFORCES ATOMICITY
   ============================================================================
   
   1. SYNCHRONIZED: Uses monitor locks (intrinsic locks)
      - JVM guarantees mutual exclusion via object monitor
      - Memory barriers ensure visibility across threads
      - Heavyweight - OS-level context switching
   
   2. ATOMIC CLASSES: Use CPU-level Compare-And-Swap (CAS) instructions
      - Hardware instruction: CMPXCHG on x86
      - Lock-free algorithm: "optimistic locking"
      - Volatile field + CAS loop ensures atomicity
   
   3. VOLATILE: Memory visibility only, NOT atomicity
      - Prevents CPU caching, forces read from main memory
      - Write barrier: flush to main memory immediately
      - Read barrier: always read from main memory
      - Does NOT make compound operations atomic (like i++)
   
   4. LOCKS: Similar to synchronized but more flexible
      - Uses AbstractQueuedSynchronizer (AQS) internally
      - Can be fair/unfair, tryLock, timed locks
*/

// ============================================================================
// APPROACH 1: LOCAL VARIABLES
// ============================================================================
class LocalVariableServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
        int counter = 0;
        counter++; // Thread-safe
    }
}
/* CONSEQUENCES:
   ✅ Pros:
      - Fastest - no synchronization overhead
      - Zero contention between threads
      - Each thread gets own stack frame
   
   ❌ Cons:
      - Can't share state across requests
      - Useless for counters, caching, statistics
      - No persistence between invocations
   
   USE WHEN: You don't need to share data across threads
*/

// ============================================================================
// APPROACH 2: SYNCHRONIZED (Monitor Locks)
// ============================================================================
class SynchronizedServlet extends HttpServlet {
    private int counter = 0;
    private final Object lock = new Object();
    
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
        synchronized(lock) { // JVM acquires monitor lock
            counter++; // Guaranteed atomic + visible to all threads
            // Multiple operations are atomic together
            if (counter > 100) {
                counter = 0;
            }
        }
    }
}
/* HOW JVM ENFORCES:
   - monitorenter/monitorexit bytecode instructions
   - Only one thread can hold the monitor at a time
   - Memory barriers ensure changes visible to next thread
   
   CONSEQUENCES:
   ✅ Pros:
      - Simple to understand and implement
      - Can make multiple operations atomic together
      - Guarantees both atomicity AND visibility
      - Built into language
   
   ❌ Cons:
      - BLOCKING: Threads wait in queue (context switching overhead)
      - Can cause thread starvation
      - Deadlock risk if locking multiple objects
      - Performance bottleneck under high contention
      - No timeout or try-lock capability
   
   SCALABILITY: Terrible under high concurrency
   - 100 threads = 99 waiting, 1 working
   - Throughput degrades linearly
*/

// ============================================================================
// APPROACH 3: ATOMIC CLASSES (Lock-Free CAS)
// ============================================================================
class AtomicServlet extends HttpServlet {
    private AtomicInteger counter = new AtomicInteger(0);
    
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
        counter.incrementAndGet(); // Lock-free CAS operation
    }
}
/* HOW JVM ENFORCES (under the hood):
   
   public final int incrementAndGet() {
       for (;;) {  // Infinite loop until success
           int current = get();  // Read current value
           int next = current + 1;
           if (compareAndSet(current, next))  // CAS instruction
               return next;
           // If CAS failed, retry (another thread changed it)
       }
   }
   
   compareAndSet uses CPU instruction:
   - x86: LOCK CMPXCHG (atomic compare-and-swap)
   - ARM: LDREX/STREX (load/store exclusive)
   
   CONSEQUENCES:
   ✅ Pros:
      - NON-BLOCKING: No thread waiting
      - Much faster than synchronized (10-100x in high contention)
      - No deadlock possible
      - Better scalability - all threads make progress
      - Wait-free for readers
   
   ❌ Cons:
      - Only works for single variable operations
      - CAS loops can spin under extreme contention (CPU waste)
      - Can't make multiple operations atomic together
      - ABA problem (value changes A→B→A, CAS succeeds incorrectly)
      - More complex to understand
   
   SCALABILITY: Excellent under contention
   - All threads compete, but all make progress
   - No context switching
*/

// ============================================================================
// APPROACH 4: VOLATILE (Visibility Only, NOT Atomic)
// ============================================================================
class VolatileServlet extends HttpServlet {
    private volatile int counter = 0; // Visible but NOT atomic!
    
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
        counter++; // ❌ RACE CONDITION! (read-modify-write not atomic)
    }
}
/* HOW JVM ENFORCES:
   - Prevents CPU from caching the variable
   - Write: immediate flush to main memory
   - Read: always fetch from main memory
   - Memory barriers prevent reordering
   
   CONSEQUENCES:
   ✅ Pros:
      - Faster than synchronized for simple reads/writes
      - Ensures visibility across threads
      - No locking overhead
   
   ❌ Cons:
      - Does NOT make compound operations atomic
      - counter++ is THREE operations: read, add, write
      - Race condition between read and write
      - Only useful for simple flags or references
   
   USE WHEN:
   - One thread writes, others only read
   - Single write operations (not read-modify-write)
   - Example: volatile boolean shutdown = false;
*/

// ============================================================================
// APPROACH 5: REENTRANT LOCKS
// ============================================================================
class ReentrantLockServlet extends HttpServlet {
    private int counter = 0;
    private final ReentrantLock lock = new ReentrantLock();
    
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
        lock.lock();
        try {
            counter++;
        } finally {
            lock.unlock(); // Must unlock in finally!
        }
    }
    
    // Advanced usage
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) {
        if (lock.tryLock()) { // Non-blocking attempt
            try {
                counter++;
            } finally {
                lock.unlock();
            }
        } else {
            resp.getWriter().println("Too busy, try later");
        }
    }
}
/* CONSEQUENCES:
   ✅ Pros:
      - More flexible than synchronized
      - tryLock() - non-blocking attempt
      - lockInterruptibly() - can be interrupted
      - Fair locks possible (FIFO queue)
      - Can check if locked: isLocked()
   
   ❌ Cons:
      - Must manually unlock (error-prone)
      - Same blocking issues as synchronized
      - Slightly more overhead than synchronized
      - Forget finally block = deadlock
   
   USE WHEN: You need lock flexibility (timeout, tryLock, fairness)
*/

// ============================================================================
// APPROACH 6: ATOMIC REFERENCE (Complex Objects)
// ============================================================================
class AtomicReferenceServlet extends HttpServlet {
    static class Stats {
        final int requests;
        final long totalTime;
        Stats(int r, long t) { requests = r; totalTime = t; }
    }
    
    private AtomicReference<Stats> stats = 
        new AtomicReference<>(new Stats(0, 0));
    
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
        long start = System.currentTimeMillis();
        
        // Process request...
        
        long time = System.currentTimeMillis() - start;
        
        // CAS loop to update immutable object atomically
        Stats current, updated;
        do {
            current = stats.get();
            updated = new Stats(current.requests + 1, 
                              current.totalTime + time);
        } while (!stats.compareAndSet(current, updated));
        // If CAS fails, another thread updated it, retry
    }
}
/* CONSEQUENCES:
   ✅ Pros:
      - Lock-free for complex objects
      - Immutable objects prevent partial updates
      - Multiple fields updated atomically together
   
   ❌ Cons:
      - Creates garbage (new object each update)
      - CAS loop can spin under contention
      - Must use immutable objects (or risk seeing partial updates)
      - More complex than simple atomic integers
   
   USE WHEN: Need to update multiple related fields atomically
*/

// ============================================================================
// PERFORMANCE COMPARISON (10 threads, 1 million operations)
// ============================================================================
/*
   NO SYNC (wrong):       ~10ms    (race conditions!)
   LOCAL VARIABLES:       ~10ms    (but can't share state)
   ATOMIC:               ~100ms    (lock-free, scales well)
   SYNCHRONIZED:        ~500ms     (blocking, context switching)
   REENTRANT LOCK:      ~550ms     (similar to synchronized)
   VOLATILE:             ~10ms     (but broken for counter++)
   
   Under HIGH CONTENTION (100 threads):
   ATOMIC:              ~500ms     (CAS spins but makes progress)
   SYNCHRONIZED:      ~5000ms      (severe bottleneck, 99 waiting)
*/

// ============================================================================
// MEMORY CONSISTENCY ERRORS (Why we need these mechanisms)
// ============================================================================
class BrokenServlet extends HttpServlet {
    private int counter = 0; // NOT volatile, NOT synchronized
    
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
        counter++; // THREE PROBLEMS:
        // 1. Read and write not atomic (race condition)
        // 2. Value cached in CPU registers (invisibility)
        // 3. Compiler/CPU can reorder operations
    }
}
/* WHAT GOES WRONG:
   Thread 1: reads counter=5, computes 6
   Thread 2: reads counter=5, computes 6  (both read same value!)
   Thread 1: writes 6
   Thread 2: writes 6  (lost update! should be 7)
   
   Also: Thread 2 might see stale cached value even after Thread 1 updates
*/
```
