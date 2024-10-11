# Compaction

Compaction in LSM storage engine usually causes pauses in writes.
OLTP -> deterministic, hard-bounded latency is critical.
Predictability of compaction.

Then how to make LSM-tree `compaction latency preditable, deterministic` and not impacting transaction processing latency.

when slow writing(consensus -> log)
- primary can still work

compaction is usually async in LSM storage engine
- spin up x background threads
- not so good in case background threads take most of `disk bandwidth` of foregrounds `transaction processing`
- in other systems, there will be `stop-the-world GC` if compaction does not keep up with ingest.
- in OLTP, ingestion does not stop and also stop-the-world GC is not acceptable
- TigerBeetle
  - makes sure to do just enough work when ingestion come in
  - has congestion control to spread the GC out like in V8 javascript engine instead of stop-the-world
- how?
  - not dynamic mem allocation after startup
  - follows NASA power of 10 rules of critical code
  - knows the memory limits of all db scenarios to make sure data goes in ~ data goes out in compaction. TODO how in details?

read amplication problem?
- shape of tree is what impacts -> hard limit on tree shape
 
LSM optimization choices
1. optimizing for read amplification
2. optimizing for space amplification
3. optimizing for writes

generally easier to solve for read amp with cache. write amp is harder to solve. -> focus for write in TigerBeetle.

TigerBeetle designed for 
- 1M TPS with primary indexes and Change Data Capture (CDC) indexes.
- 500K TPS with ~20 secondary indexes enabled.
- not distributed database, but `deterministic distributed database` -> LSM of each replica is deterministic, data files on different machines have the same data

`tiger style`: it's like having component limits of physical engineering for software engineering. 
-> static mem allocation -> must think about db physics, how data flows

References
- RocksDB
- Paper Silk
- LMAX Martin Thompson
