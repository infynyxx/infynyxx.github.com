---
layout: post
title: Garbage Collection in Java
tags: jvm
---

h1. {{ page.title }}


JVM provides number of Garbage Collector modes based on nature of the appications.

* Serial GC for running applications with small memory footprints; enabled via `-XX:+UseSerialGC`
* Parallel GC for running applications with high throughput requirements but can tolerate complete
[Stop-The-World](https://en.wikipedia.org/wiki/Tracing_garbage_collection#Stop-the-world_vs._incremental_vs._concurrent)
for given time. Batch / offline processing applications like Hadoop Map Reduce uses some form of Parallel GC. Enabled via
`-XX:+UseParallelOldGC` (>= JDK5) or `-XX:+UseParallelGC` for older JDKs
* Concurrent Mark and Sweep (CMS) for low latency, high throughput applications; Most JVM based servers run in CMS mode;
In this mode, GC is carried out without doing full pause (STW) even while other threads are running. Pause time still
exists with CMS but they are shorter to have minimal impact in application latency and throughput. Enabled via
`-XX:+UseConcMarkSweepGC`.
* Garbage First or G1 is the newest GC mode available since JDK 7u4. Like other GC modes, G1 is also an incremental GC with
support for parallel compacting and predictable pause times compared to CMS GC and Parallel GC. The basic idea with G1
GC is to set your heap ranges (using `-Xms` for min heap size and `-Xmx` for the max size) and a realistic (soft real time)
pause time goal (using `-XX:MaxGCPauseMillis`) and then let the GC do its job. `-XX:MaxGCPauseMillis` is a soft threshold
that means, JVM will try its best to have pause time of less than specified value. Enabled via `-XX:+UseG1GC`

In all Java GC modes except G1, heap is divided into 3 parts: Young Generation, Old Generation and Permanent Generation.
Young generation is furthur divided into 3 layers:

* Young Generation: Most newly allocated objects are allocated in the young generation, which, relatively to the
Java heap, is typically small and collected frequently. Since most objects in it are expected to become unreachable
quickly, the number of objects that survive a young generation collection (also referred to as a minor garbage collection)
is expected to be low. In general, minor garbage collections are efficient because they concentrate on a space
that is usually small and is likely to contain a lot of garbage objects. Young Generation can be furthur divided into 2 parts;
The Eden and The Two Survivor Space.
* Old Generation: Objects that are longer-lived are eventually promoted, or tenured, to the old generation. This generation
is typically larger than the young generation, and its occupancy grows more slowly. So, garbage collection for old generation
is infrequent and it can be expensive / major.
* Permanent Generation: It's not technically part of generation hierarchy. Instead, it is only used by the JVM
itself to hold metadata, such as class data structures, interned strings, and so on.

G1 GC uses less contiguous approach or more logical generations for laying out heap structure resulting the VM to provide dynamic sizing of youg and old
generations. G1 GC still uses basic layout of conventional JVM GC like Young Generation being divided into Eden and Survivor
regions. Most allocations happen in eden except for “humongous” allocations. G1 GC is designed to be One GC to rule them all from
latency sensitive applications (HTTP servers) to throughput sensitive applications (Indexing, Batch Processing). From JDk9,
G1 is going to be the default GC mode.
