# Clickhouse Docker Compose

## Running

In this placeholder, the nodes for 1, 2, and 3 have their IPs listed as `NODE1HOST`, `NODE2HOST`, and `NODE3HOST` respectively.

You must also replace the `MYNODE` with some node-unique name (e.g. hostname) to identify the node and replica in the cluster.

There is a configuration file for each host, it is up to you to mount the right one when launching the docker compose.

On every node, run this (replacing your hosts):

```
find . -type f -name "*.xml" -print0 | xargs -0 sed -i'' -e 's/NODE1HOST/1.1.1.1/g'
find . -type f -name "*.xml" -print0 | xargs -0 sed -i'' -e 's/NODE2HOST/2.2.2.2/g'
find . -type f -name "*.xml" -print0 | xargs -0 sed -i'' -e 's/NODE3HOST/3.3.3.3/g'
```

Also run this, but the value of `myhostname` should change per node.
```
find . -type f -name "*.xml" -print0 | xargs -0 sed -i'' -e 's/MYNODE/myhostname/g'
```

You might also want to replace the default password:

```
find . -type f -name "*.xml" -print0 | xargs -0 sed -i'' -e 's/MYPASSWORD/yourpassword/g'
```

## Verify setup

On each clickhouse node, run:

```
clickhouse-client -q "SELECT * FROM system.clusters WHERE cluster='mycluster' FORMAT Vertical;"
```

that you node is aware of replicas, and:

```
clickhouse-client -q "select * from system.zookeeper where path='/clickhouse/task_queue/'"
```

to verify that your node is connected to keeper.

Then, you can create replicated tables on cluster:

```sql
CREATE TABLE testtable ON CLUSTER '{cluster}'
(
    timestamp DateTime,
    contractid UInt32,
    userid UInt32
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{cluster}/{shard}/default/testtable', '{replica}')
PARTITION BY toYYYYMM(timestamp)
ORDER BY (contractid, toDate(timestamp), userid)
SAMPLE BY userid;
```

## Running keeper on dedicated nodes

It's simple to move keeper to its own nodes, just clone the docker file and drop the clickhouse service (and drop the keeper on the original).


## Logging and metrics

Vector is setup on each node, you'll need to replace the sink and configure as needed. See `config/vector/vector.yaml`
