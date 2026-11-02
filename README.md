# murmur2-partitioner-go

> [!WARNING]
> **Deprecated — no longer maintained.** Sarama now ships a Java-compatible
> murmur2 partitioner built in: [`sarama.NewMurmur2Partitioner`][godoc],
> available since [Sarama v1.50.0][release] (May 2026, [PR #3567][pr]).
> Please migrate. Switching is behavior-preserving: both partitioners route
> keyed messages to identical partitions, so no data movement occurs.

This package provides a [Sarama](https://github.com/IBM/sarama) partitioner
that uses the same murmur2-based hashing as the Apache Kafka Java client
(`org.apache.kafka.common.utils.Utils#murmur2`), so keyed messages are
routed to the same partitions as JVM (and other murmur2-compatible) clients.
The hash implementation is verified against test vectors generated with the
JVM Kafka client 1.0.0.

## Migration

**Before** (this library):

```go
import (
	"github.com/IBM/sarama"
	partitioner "github.com/lucky/murmur2-partitioner-go/partitioner"
)

config := sarama.NewConfig()
config.Producer.Partitioner = partitioner.NewJVMCompatiblePartitioner
```

**After** (Sarama ≥ v1.50.0):

```go
config := sarama.NewConfig()
config.Producer.Partitioner = sarama.NewMurmur2Partitioner
```

Both partitioners compute `(murmur2(keyBytes) & 0x7fffffff) % numPartitions`
and fall back to a random partition for messages without a key, so the
switch changes nothing about where messages are routed.

Note that keyless messages never had cross-client affinity anyway: Java
clients have used sticky partitioning for null-key messages since 2.4,
while Sarama assigns them randomly. Partition affinity between Java and Go
producers only holds for keyed messages.

## If you can't migrate

Consumers pinned to Sarama versions older than v1.50.0 can keep using this
code by vendoring the [`partitioner`](partitioner/) package (two files, no
dependencies beyond Sarama itself) into their project.

[godoc]: https://pkg.go.dev/github.com/IBM/sarama#NewMurmur2Partitioner
[release]: https://github.com/IBM/sarama/releases/tag/v1.50.0
[pr]: https://github.com/IBM/sarama/pull/3567
