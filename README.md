# Clickhouse Docker Compose

## Setup

In this placeholder, the nodes for 1, 2, and 3 have their IPs listed as `NODE1HOST`, `NODE2HOST`, and `NODE3HOST` respectively.

You must also replace the `MYNODE` with some node-unique name (e.g. hostname) to identify the node and replica in the cluster.


On every node, run this (replacing your hosts):

```
find . -type f -name "*.xml" -print0 | xargs -0 sed -i'' -e 's/NODE1HOST/1.1.1.1/g'
find . -type f -name "*.xml" -print0 | xargs -0 sed -i'' -e 's/NODE2HOST/2.2.2.2/g'
find . -type f -name "*.xml" -print0 | xargs -0 sed -i'' -e 's/NODE3HOST/3.3.3.3/g'
```

Also run this, but the value of `myhostname` should change per node (for naming).
```
find . -type f -name "*.xml" -print0 | xargs -0 sed -i'' -e 's/MYNODEHOST/1/g'
find . -type f -name "*.xml" -print0 | xargs -0 sed -i'' -e 's/MYNODEREPLICA/01/g'
```

You might also want to replace the default password:

```
export PASSWORD=$(echo -n 'mysecretpassword' | sha256sum | tr -d '-' | tr -d ' ')
find . -type f -name "/etc/clickhouse-server/users.d/users.xml" -print0 | xargs -0 sed -i'' -e "s/MYPASSWORDSHA/$PASSWORD/g"
```

Copy the `config.d` and `users.d` files into the `/etc/clickhouse-server/{config,users}.d` directories respectively.

```
cp config.d/config.xml /etc/clickhouse-server/config.d/config.xml
cp users.d/users.xml /etc/clickhouse-server/users.d/users.xml
```

Then start them all at the same time with:

```
service clickhouse-server restart
service clickhouse-server enable
```

## Verify setup

On each clickhouse node, run:

```sql
SELECT * FROM system.clusters WHERE cluster='mycluster' FORMAT Vertical
```

that you node is aware of replicas, and:

```sql
select * from system.zookeeper where path='/clickhouse/task_queue/'
```

to verify that your node is connected to keeper.

Then, you can create replicated tables on cluster:

```sql
CREATE TABLE bb ON CLUSTER '{cluster}'
(
    timestamp DateTime,
    contractid UInt32,
    userid UInt32
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{cluster}/{shard}/default/bb', '{replica}')
PARTITION BY toYYYYMM(timestamp)
ORDER BY (contractid, toDate(timestamp), userid)
SAMPLE BY userid;
```

## Running keeper on dedicated nodes

It's simple to move keeper to its own nodes, just clone the config file and drop the clickhouse configs.


## Logging and metrics

Vector is setup on each node, you'll need to replace the sink and configure as needed. See `vector/vector.yaml`
