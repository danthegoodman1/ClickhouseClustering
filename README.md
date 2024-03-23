# Clickhouse Docker Compose

Run this docker compose on each distinct node, replacing the IPs for each node.

In this placeholder, the nodes for 1, 2, and 3 have their IPs listed as `NODE1HOST`, `NODE2HOST`, and `NODE3HOST` respectively.

There is a configuration file for each host, it is up to you to mount the right one when launching the docker compose.

Find and replace command:

```
find . -type f -name "*.xml" -print0 | xargs -0 sed -i '' -e 's/NODE1HOST/1.1.1.1/g'
```
