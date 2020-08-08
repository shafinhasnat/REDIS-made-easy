---


---

<h1 id="redis-made-easy">REDIS made easy</h1>
<p><strong>REDIS</strong> is a NoSQL in memory database which is faster than many other SQL and NoSql data stores. It stores data in <strong>key-value</strong> pair just like Python dictionary or Javascript object. It provides persistency of data alongside storing immediately in RAM as well. Redis gives the freedom of storing variety of data structures. It supports most of the modern programming languages which. We will be using <code>Python</code> in this case.</p>
<h2 id="installation">Installation</h2>
<p>Redis is available in <code>apt</code>  in Linux distros. Bash command for installing it from apt is:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> <span class="token function">apt-get</span> update
</code></pre>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> <span class="token function">apt-get</span> upgrade
</code></pre>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> <span class="token function">apt-get</span> redis
</code></pre>
<p>or it can be installed manually by downloading the <code>.tar.gz</code> file from <a href="http://download.redis.io/releases/">here</a> and <code>make</code> command.<br>
After installation check the version with <code>--version</code> command:</p>
<pre class=" language-bash"><code class="prism  language-bash">redis-server ---version
</code></pre>
<p>It will return something like this:<br>
<img src="https://i.ibb.co/R04kqWH/001-ver.png" alt=""><br>
In  my case I am using version <code>5.0.7</code></p>

