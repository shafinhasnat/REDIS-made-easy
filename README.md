---


---

<h1 id="redis-made-easy">REDIS made easy</h1>
<h3>-Shafin Hasnat</h3> 
<p><img src="https://i.ibb.co/T2n60wq/redis-logo.png" alt=""><br>
<strong>REDIS</strong> is a NoSQL in memory database which is faster than many other SQL and NoSql data stores. It stores data in <strong>key-value</strong> pair just like Python dictionary or Javascript object. It provides persistency of data alongside storing immediately in RAM as well. Redis gives the freedom of storing variety of data structures. It supports most of the modern programming languages which. We will be using <code>Python</code> in this case.</p>
<h2 id="installation">Installation</h2>
<p>Redis is available in <code>apt</code>  in Linux distros. Bash command for installing it from apt is:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> <span class="token function">apt-get</span> update
</code></pre>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> <span class="token function">apt-get</span> upgrade
</code></pre>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> <span class="token function">apt-get</span> redis-server
</code></pre>
<p>or it can be installed manually by downloading the <code>.tar.gz</code> file from <a href="http://download.redis.io/releases/">here</a> and <code>make</code> command.<br>
After installation check the version with <code>--version</code> command:</p>
<pre class=" language-bash"><code class="prism  language-bash">redis-server ---version
</code></pre>
<p>It will return something like this:<br>
<img src="https://i.ibb.co/R04kqWH/001-ver.png" alt=""><br>
In  my case I am using version <code>5.0.7</code></p>
<h2 id="running-and-configuration">Running and configuration</h2>
<p>Redis server runs on default port <code>6379</code>. It can be accessed with</p>
<pre class=" language-bash"><code class="prism  language-bash">redis-cli
</code></pre>
<p>This command will bind directly with port 6379 of localhost. Lets apply some Redis cli command here.<br>
<img src="https://i.ibb.co/gRs1Dmq/003-def-port-play.png" alt=""><br>
So far so good!</p>
<p>Redis can be configured for different port also. If we want to make a custom port for redis database, we need to make a <code>.conf</code> file in <code>/etc/redis</code> folder, and initialize it.<br>
Lets say we will launch redis in port 6000. Just navigate to <code>/etc/redis</code> and make a file naming <code>6000.conf</code> and paste the snippet inside.</p>
<pre class=" language-bash"><code class="prism  language-bash">port              6000
daemonize         <span class="token function">yes</span>
save              60 1
bind              127.0.0.1
tcp-keepalive     300
dbfilename        dump.rdb
<span class="token function">dir</span>               ./
rdbcompression    <span class="token function">yes</span>
</code></pre>
<p>Here <code>daemonize yes</code> command allows Redis to run on that port continuously. <code>save 60 1</code> means dump data from ram to hdd or ssd every 60 second for 1 change. <code>tcp-keepalive 300</code> means server connection time out period with the client. <code>dbfilename dump.rdb</code> and <code>dir ./</code> means the snapshot of data in the root folder naming <code>dump.rdb</code>.<br>
To run the database in port 6000, I used the command in the following:</p>
<pre class=" language-bash"><code class="prism  language-bash">redis-server /etc/redis/6000.conf
</code></pre>
<p>Access the cli:</p>
<pre class=" language-bash"><code class="prism  language-bash">redis-cli -p 6000
</code></pre>
<p>After pushing few data I faced this problem:<br>
<img src="https://i.ibb.co/d2JYy38/002-rdb-error.png" alt=""><br>
The error message says:</p>
<pre><code>(error) MISCONF Redis is configured to save RDB snapshots, but it is currently not able to persist on disk. Commands that may modify the data set are disabled, because this instance is configured to report errors during writes if RDB snapshotting fails (stop-writes-on-bgsave-error option). Please check the Redis logs for details about the RDB error.
</code></pre>
<p><em><strong>Debug:</strong></em><br>
This problem occurs because of redis is not able to write data for the lack of permission. By default the <code>.rdb</code> file is saved into <code>/var/lib/redis</code> folder. I decided to save the dump file in this folder with a new name <code>dump_6000.rdb</code>. So, I changed the value of <code>dir</code> and <code>dbfilename</code> to following:</p>
<pre><code>dbfilename        dump_6000.rdb
dir               /var/lib/redis
</code></pre>
<p>Force stop the database in the port with <code>redis-cli -p 6000 shutdown NOSAVE</code> and run <code>redis-server /etc/redis/6000.conf</code> command. It gave me another error regarding permission.<br>
<img src="https://i.ibb.co/Y8jGBHH/004-permission-error.png" alt=""><br>
To grant permission navigate to <code>/etc/systemd/system/</code> and comment out (disable) line 21<br>
<code># ProtectHome=yes</code><br>
Then restart all daemon with <code>sudo systemctl daemon-reload</code> and restart the redis-server with <code>sudo service redis-server restart</code> command.<br>
Then like before run redis-server on port 6000 with <code>redis-server /etc/redis/6000.conf</code> command. Now it works properly!<br>
Now access the cli on port 6000 with <code>redis-cli -p 6000</code>.<br>
<img src="https://i.ibb.co/q5MXcT4/005-debug-6000.png" alt=""><br>
<sup>Oops! misspelled my own country Bandladesh–&gt;Bangladesh :’(</sup></p>
<h2 id="plugging-redis-with-python">Plugging Redis with Python</h2>
<p>Redis has an official Python client. It can be downloaded easily with PyPI package archive<br>
<code>pip3 install redis</code><br>
To connect the existing database localhost:6000 to Python redis client-</p>
<pre class=" language-python"><code class="prism  language-python"><span class="token operator">&gt;&gt;</span><span class="token operator">&gt;</span> <span class="token keyword">import</span> redis
<span class="token operator">&gt;&gt;</span><span class="token operator">&gt;</span> r <span class="token operator">=</span> redis<span class="token punctuation">.</span>Redis<span class="token punctuation">(</span>host<span class="token operator">=</span><span class="token string">'127.0.0.1'</span><span class="token punctuation">,</span> port<span class="token operator">=</span><span class="token number">6000</span><span class="token punctuation">,</span> db<span class="token operator">=</span><span class="token number">0</span><span class="token punctuation">,</span> decode_responses<span class="token operator">=</span><span class="token boolean">True</span><span class="token punctuation">)</span>
</code></pre>
<p>We can run Redis cli command in this python script</p>
<pre class=" language-python"><code class="prism  language-python"><span class="token operator">&gt;&gt;</span><span class="token operator">&gt;</span> r<span class="token punctuation">.</span>get<span class="token punctuation">(</span><span class="token string">"India"</span><span class="token punctuation">)</span>
<span class="token string">'rupee'</span>
<span class="token operator">&gt;&gt;</span><span class="token operator">&gt;</span> r<span class="token punctuation">.</span>get<span class="token punctuation">(</span><span class="token string">"US"</span><span class="token punctuation">)</span>
<span class="token string">'dollar'</span>
<span class="token operator">&gt;&gt;</span><span class="token operator">&gt;</span> r<span class="token punctuation">.</span><span class="token builtin">set</span><span class="token punctuation">(</span><span class="token string">"Russia"</span><span class="token punctuation">,</span> <span class="token string">"ruble"</span><span class="token punctuation">)</span>
<span class="token boolean">True</span>
<span class="token operator">&gt;&gt;</span><span class="token operator">&gt;</span> r<span class="token punctuation">.</span>get<span class="token punctuation">(</span><span class="token string">"Russia"</span><span class="token punctuation">)</span>
<span class="token string">'ruble'</span>
</code></pre>
<h1 id="redis-cluster">Redis cluster</h1>
<p>Redis cluster allows automatically shard data among multiple standalone nodes. It allows the system to be up and running despite some of the node(s) goes down.  Nodes in the cluster are divided into master and slave. The cluster shards data according to the hash slot. A cluster provides 16384 hash slots. These slots are equally divided among the master nodes. All nodes are connected to each other in mesh via gossip protocol.</p>
<h2 id="setup-a-redis-cluster">Setup a Redis cluster</h2>
<p>Here, we will simulatea= redis cluster with total 6 nodes (3 master, 3 slave). Flow chart of the redis cluster:</p>
<p><img src="https://i.ibb.co/4grCJkH/flow.jpg" alt=""></p>
<p>This cluster uses port 7000-7005 as nodes. Hash slot distribution:<br>
Port 7000 —&gt; 0-5460<br>
Port 7001 —&gt; 5461-10922<br>
Port 7000 —&gt; 10923-16383<br>
We will simulate the cluster in a single host. For this, navigate to the working directory, and download and make redis in that folder:</p>
<pre><code>wget http://download.redis.io/releases/redis-5.0.7.tar.gz
</code></pre>
<pre><code>tar xzf redis-5.0.7.tar.gz
</code></pre>
<pre><code>cd redis-5.0.7
</code></pre>
<pre><code>make
</code></pre>
<p>To enable clustering, uncomment (enable) these 3 lines in <code>redis.conf</code> file:<br>
<code>cluster-enabled yes</code><br>
<code>cluster-config0file nodes-6379.conf</code><br>
<code>cluster-node-timeout 15000</code></p>
<p>Then make 6 <code>.conf</code> in the root of the working directory</p>
<pre><code>touch 7000.conf 7001.conf 7002.conf 7003.conf 7004.conf 7005.conf
</code></pre>
<p>And paste the snippet below in each of the <code>.conf</code> file:</p>
<pre class=" language-bash"><code class="prism  language-bash">port 7000
cluster-enabled <span class="token function">yes</span>
cluster-config-file cluster-node-0.conf
cluster-node-timeout 5000
appendonly <span class="token function">yes</span>
appendfilename node-0.aof
dbfilename dump-0.rdb
</code></pre>
<p>This is <code>7000.conf</code> file. Change <code>port</code>, <code>cluster-config-file</code>, <code>appendfilename</code>, <code>dbfilename</code> name accordingly.<br>
Run redis server in each created port:<br>
<code>./redis-5.0.7/src/redis-server 7000.conf</code> (change <code>.conf</code> file name for other ports)<br>
According to the configuration, a <code>.aof</code> and a <code>.conf</code> file is created for each. The working directory looks like this in this stage:<br>
<img src="https://i.ibb.co/kmKS5YN/006-folder-ls-1.png" alt=""><br>
The cluster is not up yet. Each of the nodes are still isolated. The <code>cluster-node-0.conf</code> looks like this in this stage:<br>
<img src="https://i.ibb.co/kMBFxzn/007-cat-conf-1.png" alt=""><br>
It’s time to create the cluster. Our cluster will have one replica per master.</p>
<pre><code>./redis-5.0.7/src/redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1
</code></pre>
<p>Accept what is asked, and out cluster in up and running! We can see the confirmation message in the logs of each running nodes. If we cat any <code>cluster-node.conf</code>,<br>
<img src="https://i.ibb.co/G70mFmL/012-cat-conf-2.png" alt=""><br>
As we can see here, connection has been established between each nodes. And port 7000, 7001, 7002 has been assigned as master and other 3 ports 7003, 7004, 7005 as slave.<br>
In this stage, <code>.rdb</code> files are added in the root working directory.<br>
<img src="https://i.ibb.co/GRKKtcx/011-folder-ls-2.png" alt=""></p>
<p><em><strong>Testing</strong></em><br>
If we <code>SET</code> any value in any port, they get redirected in another port according to the assigned slot.<br>
To check with redis cli, <code>./redis-5.0.7/src/redis-cli -c -p 7002</code> (here <code>-c</code> is for cluster mode, and we are using port 7002)<br>
<img src="https://i.ibb.co/VSmS1cQ/014-node-2-before-fail-assign-1.png" alt=""><br>
<sup>Oops! misspelled again… thiland—&gt;thailand :’(</sup><br>
As we can see, this key value has been redirected to port 7001 in hash slot 6369. Each time we GET this key, it will redirect from port 7001.</p>
<h2 id="server-failure-simulationif-we-kill-server-in-port-7002-other-nodes-will-take-this-message-and-its-slave-7005-will-be-assigned-as-master.kill-7002new-state-of-7005cat-any-cluster-node.conf-file-here-we-can-see-master-port-7002-is-in-fail-state-and-7005-is-now-master.if-we-query-the-keys-which-was-assigned-to-port-7002-which-is-now-dead-it-will-still-give-the-result-from-port-7001.cool-right-so-7002-is-dead-already-now-if-we-kill-7001-which-holds-our-key-value-what-happens-7001-was-a-master-node-as-it-is-now-dead-7004-becomes-the-new-master-of-the-cluster.our-data-still-exists-in-port-7004-insaneif-any-hash-slot-is-unused-due-to-failure-of-a-master-slave-block-the-server-will-return-clusterdown-message"><em><strong>Server failure simulation</strong></em><br>
If we kill server in port 7002, Other nodes will take this message, and its slave 7005 will be assigned as master.<br>
kill 7002:<br>
<img src="https://i.ibb.co/V3NDm6p/019-kill-7002.png" alt=""><br>
new state of 7005:<br>
<img src="https://i.ibb.co/Z67ccqH/020-7005-master.png" alt=""><br>
Cat any <code>cluster-node.conf</code> file-<br>
<img src="https://i.ibb.co/LpzPWwf/013-node-2-fail.png" alt=""><br>
Here, we can see master port 7002 is in <code>fail</code> state, and 7005 is now master.<br>
If we query the keys which was assigned to port 7002, which is now dead, it will still give the result from port 7001.<br>
<img src="https://i.ibb.co/p2yrxf0/016-node-2-failure-handle.png" alt=""><br>
Cool right? So, 7002 is dead already, now if we kill 7001, which holds our key value, what happens? 7001 was a master node, as it is now dead, 7004 becomes the new master of the cluster.<br>
<img src="https://i.ibb.co/82gw9md/021-kill-7001-7002-get-thiland.png" alt=""><br>
Our data still exists in port 7004!! INSANE!!<br>
If any hash slot is unused due to failure of a master slave block, the server will return <code>CLUSTERDOWN</code> message:<br>
<img src="https://i.ibb.co/LvWr5Wc/018-7001-7004-ms-fail.png" alt=""></h2>
<hr>

