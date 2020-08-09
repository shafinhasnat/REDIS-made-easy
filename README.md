# REDIS made easy

**_-Shafin Hasnat_**

![](https://i.ibb.co/T2n60wq/redis-logo.png)

**REDIS** is a NoSQL in memory database which is faster than many other SQL and NoSql data stores. It stores data in **key-value** pair just like Python dictionary or Javascript object. It provides persistency of data alongside storing immediately in RAM as well. Redis gives the freedom of storing variety of data structures. It supports most of the modern programming languages which. We will be using `Python` in this case.

## Installation

Redis is available in `apt` in Linux distros. Bash command for installing it from apt is:

```bash
sudo apt-get update
```

```bash
sudo apt-get upgrade
```

```bash
sudo apt-get redis-server
```

or it can be installed manually by downloading the `.tar.gz` file from [here](http://download.redis.io/releases/) and `make` command.
After installation check the version with `--version` command:

```bash
redis-server ---version
```

It will return something like this:

![](https://i.ibb.co/R04kqWH/001-ver.png)

In my case I am using version `5.0.7`

## Running and configuration

Redis server runs on default port `6379`. It can be accessed with

```bash
redis-cli
```

This command will bind directly with port 6379 of localhost. Lets apply some Redis cli command here.

![](https://i.ibb.co/gRs1Dmq/003-def-port-play.png)

So far so good!

Redis can be configured for different port also. If we want to make a custom port for redis database, we need to make a `.conf` file in `/etc/redis` folder, and initialize it.
Lets say we will launch redis in port 6000. Just navigate to `/etc/redis` and make a file naming `6000.conf` and paste the snippet inside.

```
port              6000
daemonize         yes
save              60 1
bind              127.0.0.1
tcp-keepalive     300
dbfilename        dump.rdb
dir               ./
rdbcompression    yes
```

Here `daemonize yes` command allows Redis to run on that port continuously. `save 60 1` means dump data from ram to hdd or ssd every 60 second for 1 change. `tcp-keepalive 300` means server connection time out period with the client. `dbfilename dump.rdb` and `dir ./` means the snapshot of data in the root folder naming `dump.rdb`.
To run the database in port 6000, I used the command in the following:

```bash
redis-server /etc/redis/6000.conf
```

Access the cli:

```bash
redis-cli -p 6000
```

After pushing few data I faced this problem:

![](https://i.ibb.co/d2JYy38/002-rdb-error.png)

The error message says:

```
(error) MISCONF Redis is configured to save RDB snapshots, but it is currently not able to persist on disk. Commands that may modify the data set are disabled, because this instance is configured to report errors during writes if RDB snapshotting fails (stop-writes-on-bgsave-error option). Please check the Redis logs for details about the RDB error.
```

**_Debug:_**
This problem occurs because of redis is not able to write data for the lack of permission. By default the `.rdb` file is saved into `/var/lib/redis` folder. I decided to save the dump file in this folder with a new name `dump_6000.rdb`. So, I changed the value of `dir` and `dbfilename` to following:

```
dbfilename        dump_6000.rdb
dir               /var/lib/redis
```

Force stop the database in the port with `redis-cli -p 6000 shutdown NOSAVE` and run `redis-server /etc/redis/6000.conf` command. It gave me another error regarding permission.

![](https://i.ibb.co/Y8jGBHH/004-permission-error.png)

To grant permission navigate to `/etc/systemd/system/` and comment out (disable) line 21
`# ProtectHome=yes`
Then restart all daemon with `sudo systemctl daemon-reload` and restart the redis-server with `sudo service redis-server restart` command.
Then like before run redis-server on port 6000 with `redis-server /etc/redis/6000.conf` command. Now it works properly!
Now access the cli on port 6000 with `redis-cli -p 6000`.

![](https://i.ibb.co/q5MXcT4/005-debug-6000.png)

<sup>Oops! misspelled my own country Bandladesh-->Bangladesh :'(</sup>

## Plugging Redis with Python

Redis has an official Python client. It can be downloaded easily with PyPI package archive
`pip3 install redis`
To connect the existing database localhost:6000 to Python redis client-

```python
>>> import redis
>>> r = redis.Redis(host='127.0.0.1', port=6000, db=0, decode_responses=True)
```

We can run Redis cli command in this python script

```python
>>> r.get("India")
'rupee'
>>> r.get("US")
'dollar'
>>> r.set("Russia", "ruble")
True
>>> r.get("Russia")
'ruble'
```

# Redis cluster

Redis cluster allows automatically shard data among multiple standalone nodes. It allows the system to be up and running despite some of the node(s) goes down. Nodes in the cluster are divided into master and slave. The cluster shards data according to the hash slot. A cluster provides 16384 hash slots. These slots are equally divided among the master nodes. All nodes are connected to each other in mesh via gossip protocol.

## Setup a Redis cluster

Here, we will simulate a redis cluster with total 6 nodes (3 master, 3 slave). Flow chart of the redis cluster:

![](https://i.ibb.co/4grCJkH/flow.jpg)

This cluster uses port 7000-7005 as nodes. Hash slot distribution:

- Port 7000 ---> 0-5460
- Port 7001 ---> 5461-10922
- Port 7000 ---> 10923-16383

We will simulate the cluster in a single host. For this, navigate to the working directory, and download and make redis in that folder:

```
wget http://download.redis.io/releases/redis-5.0.7.tar.gz
```

```
tar xzf redis-5.0.7.tar.gz
```

```
cd redis-5.0.7
```

```
make
```

To enable clustering, uncomment (enable) these 3 lines in `redis.conf` file:
`cluster-enabled yes`
`cluster-config0file nodes-6379.conf`
`cluster-node-timeout 15000`

Then make 6 `.conf` in the root of the working directory

```
touch 7000.conf 7001.conf 7002.conf 7003.conf 7004.conf 7005.conf
```

And paste the snippet below in each of the `.conf` file:

```bash
port 7000
cluster-enabled yes
cluster-config-file cluster-node-0.conf
cluster-node-timeout 5000
appendonly yes
appendfilename node-0.aof
dbfilename dump-0.rdb
```

This is `7000.conf` file. Change `port`, `cluster-config-file`, `appendfilename`, `dbfilename` name accordingly.
Run redis server in each created port:
`./redis-5.0.7/src/redis-server 7000.conf` (change `.conf` file name for other ports)
According to the configuration, a `.aof` and a `.conf` file is created for each. The working directory looks like this in this stage:

![](https://i.ibb.co/kmKS5YN/006-folder-ls-1.png)

The cluster is not up yet. Each of the nodes are still isolated. The `cluster-node-0.conf` looks like this in this stage:

![](https://i.ibb.co/kMBFxzn/007-cat-conf-1.png)

It's time to create the cluster. Our cluster will have one replica per master.

```
./redis-5.0.7/src/redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1
```

Accept what is asked, and out cluster in up and running! We can see the confirmation message in the logs of each running nodes. If we cat any `cluster-node.conf`,

![](https://i.ibb.co/G70mFmL/012-cat-conf-2.png)

As we can see here, connection has been established between each nodes. And port 7000, 7001, 7002 has been assigned as master and other 3 ports 7003, 7004, 7005 as slave.
In this stage, `.rdb` files are added in the root working directory.

![](https://i.ibb.co/GRKKtcx/011-folder-ls-2.png)

## Testing

If we `SET` any value in any port, they get redirected in another port according to the assigned slot.
To check with redis cli, `./redis-5.0.7/src/redis-cli -c -p 7002` (here `-c` is for cluster mode, and we are using port 7002)

![](https://i.ibb.co/VSmS1cQ/014-node-2-before-fail-assign-1.png)

<sup>Oops! misspelled again... thiland--->thailand :'(</sup>

As we can see, this key value has been redirected to port 7001 in hash slot 6369. Each time we GET this key, it will redirect from port 7001.

## Server failure simulation

If we kill server in port 7002, Other nodes will take this message, and its slave 7005 will be assigned as master.
kill 7002:

![](https://i.ibb.co/V3NDm6p/019-kill-7002.png)

new state of 7005:

![](https://i.ibb.co/Z67ccqH/020-7005-master.png)

Cat any `cluster-node.conf` file-

![](https://i.ibb.co/LpzPWwf/013-node-2-fail.png)

Here, we can see master port 7002 is in `fail` state, and 7005 is now master.
If we query the keys which was assigned to port 7002, which is now dead, it will still give the result from port 7001.

![](https://i.ibb.co/p2yrxf0/016-node-2-failure-handle.png)

Cool right? So, 7002 is dead already, now if we kill 7001, which holds our key value, what happens? 7001 was a master node, as it is now dead, 7004 becomes the new master of the cluster.

![](https://i.ibb.co/82gw9md/021-kill-7001-7002-get-thiland.png)

Our data still exists in port 7004!! INSANE!!
If any hash slot is unused due to failure of a master slave block, the server will return `CLUSTERDOWN` message:

![](https://i.ibb.co/LvWr5Wc/018-7001-7004-ms-fail.png)

---

---
