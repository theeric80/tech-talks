# Dynamic GPU Model Management

## Outline

1. Problem & Solution
2. System Architecture
3. Implementation Details

---

## Problem: L40s Shortage Causes Complete Outages

**Current State:**
- Autoscaling group min size = 0 (scale-to-zero)
- Cold start every task – 4-5min cold start + 2min execution
- Complete outages during L40s shortage

**Critical Impact:**
- Zero availability during hardware shortage
- 67% overhead (5min startup / 7min total)
- 2.5x performance penalty

---

## Solution: Persistent Instances + Dynamic Engine Loading

**Architecture Shift:**
- Warm instances (min size > 0)
- Multiple engines per autoscaling group
- Dynamic engine loading/unloading

**Key Benefits:**
- Hardware shortage resilience
- 67% latency reduction
- Better resource utilization

```mermaid
graph LR
    subgraph "Before: Scale-to-Zero"
        A1[Task] --> B1[New Instance] --> C1[Boot 4-5min] --> D1[Execute 2min]
    end
    
    subgraph "After: Dynamic Loading"
        A2[Task] --> B2[Warm Instance] --> C2[Execute 2min]
    end
    
    style B1 fill:#FFB6C1
    style B2 fill:#90EE90
```

---

## System Architecture

```mermaid
graph TD
    A[Queue] --> B[Launcher]
    
    B --> F[GPU 1]
    B --> G[GPU 2]
    B --> H[GPU N]
    
    F --> F1[Engine A]
    F --> F2[Engine B]
    
    G --> G1[Engine A]
    G --> G2[Engine C]
    
    H --> H1[Engine D]
    
    style A fill:#e1f5fe
    style B fill:#e1f5fe,stroke:#333,stroke-width:3px
    style F fill:#fff3e0
    style G fill:#fff3e0
    style H fill:#fff3e0
```

## Three Core Components:**

**Task Scheduler**
- Routes tasks to optimal GPU/engine pairs

**GPU Memory Manager**
- Admission control
- Engine eviction when memory insufficient

**Engine Loader**
- Handles engine lifecycle (load/unload/execute)
- Manages per-GPU engine inventory
- Coordinates with memory manager for safe operations

```mermaid
graph TD
    A[Task Scheduler] --> B[GPU Memory Manager]
    A --> C[Engine Loader]
    B --> C
    
    style A fill:#e1f5fe
    style B fill:#f3e5f5
    style C fill:#e8f5e8
```

---

## Scheduling Algorithm

**Prerequisite:** Engine and GPU both available

**Selection Priority:**
1. **Resident engines** → 0ms execution
2. **Non-resident engines** → ~30s loading overhead  

**Optimization:** Prefer loaded engines to avoid 30s penalty

```mermaid
flowchart TD
    A[New Task] --> B{Engine & GPU Available?}
    B -->|No| C[Queue Task]
    B -->|Yes| D{Engine Loaded?}
    D -->|Yes| E[Execute Immediately]
    D -->|No| F[Load Engine] --> G[Execute Task]
    
    style E fill:#90EE90
    style F fill:#FFE4B5
    style C fill:#FFB6C1
```

---

## Memory Management: Admission Control

**Safety Rule:**
```
Can Load Engine? 
= New Engine Limit + Sum(Loaded Engines Request) ≤ GPU Memory
```

**Example:**
```
8GB + (4GB + 3GB) = 15GB ≤ 16GB ✅ Safe
10GB + (4GB + 3GB) = 17GB > 16GB ❌ Unsafe
```

**Design Rationale:**
- Use `request` for loaded engines (guaranteed baseline)
- Use `limit` for incoming engine (worst-case scenario)
- Prevents OOM while maximizing utilization

```mermaid
graph TB
    subgraph "GPU Memory (16GB)"
        A[Engine A:<br>request: 4GB <br>limit: 6GB]
        B[Engine B:<br>request: 3GB <br>limit: 5GB]
        C[Available: 9GB]
        D[New Engine:<br>limit: 8GB]
    end
    
    subgraph "Safe Zone ✅"
        E["8GB + (4GB + 3GB) = 15GB ≤ 16GB"]
    end
    
    subgraph "Unsafe Zone ❌"
        F["10GB + (4GB + 3GB) = 17GB > 16GB"]
    end
    
    style A fill:#90EE90
    style B fill:#90EE90
    style C fill:#E0E0E0
    style D fill:#FFE4B5
    style E fill:#90EE90
    style F fill:#FFB6C1
```

---

## Memory Management: Eviction Strategy

**LRU Eviction Process:**
1. **Find** least recently used idle engine
2. **Unload** engine from GPU memory  
3. **Repeat** until new engine can be loaded

```mermaid
flowchart TD
    A[Load New Engine] --> B{Memory Available?}
    B -->|Yes| C[Load]
    B -->|No| D[Find LRU Idle Engine]
    D --> E{Found?}
    E -->|Yes| F[Evict]
    E -->|No| G[Queue Task]
    F --> B
    
    style C fill:#90EE90
    style F fill:#FFE4B5
    style G fill:#FFB6C1
```

**Why LRU?**
- Only evicts idle engines (safety first)
- Assumes older = less likely to be needed
- Keeps hot engines loaded for fast access

**Trade-offs:**
- ✅ Efficient memory utilization
- ⚠️ Reload penalty for evicted engines (~30s)
- 🎯 Optimizes for common access patterns

---

## Engine Load/Unload Implementation

**Container-based Engine Lifecycle:**
- **Engine Load** → Container start
- **Engine Unload** → Container stop

**Benefits:**
- Clean memory isolation per engine
- Reliable resource cleanup
- Leverages existing container orchestration

---

## Autoscaling Policy Optimization

**Current:** Scale out when queue length ≥ 1
**Proposed:** Scale out when queue length ≥ 2

**Why Change?**
- Let warm instances handle short bursts
- Reduce unnecessary instance provisioning
- Lower cost while maintaining SLA

**Impact:** Better resource efficiency, reduced scaling churn

---

## Monitoring & Success Metrics

**Core Algorithm Metrics:**
- **Cache Hit Rate** → >80% (scheduler effectiveness)
- **Engine Reload Rate** → <20% (load/unload efficiency)
- **GPU Memory Utilization** → 70-85% (admission control balance)

**Business Impact:**
- **Service Availability** → 99.9%+ (eliminates outages)
- **P95 Latency** → <3min (vs 7min baseline)
- **Cost per Task** → 40% reduction (better utilization)

**Alert Thresholds:**
- Cache hit <60% → Review scheduling logic
- Memory >90% → Tune admission control
- Reload rate >40% → Adjust eviction policy
