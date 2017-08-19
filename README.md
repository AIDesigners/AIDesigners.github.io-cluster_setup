## Spark Cluster Setup

<p align="justify">
In 2010, Google introduced MapReduce, a batch-processing framework for running computations involving large data volume over a commodity cluster. MapReduce was later adopted by several companies including Google, Amazon, IBM etc. It solved the scalability issues to great extent when dealing with large datasets and showed good performance, provided that the recipes for data processing do not involve any repeatative processing of a single step. The reason for this was that MapReduce reads data at the begining of a step and the scope of the objects holding this data are upto the end of this step. So the data objects are destoyed once a step ends. As a result, repeating a step incurs significant parallelism overhead. 
</p>
<p align="justify">
Spark was introduced to mainly overcome this limitation of MapReduce.Spark is a fast and memory-efficent clustering computing framework. It support both batch-style as well as computations over data streams. It provides high-level APIs in Scala, Java, Python, and R. It also provides an optimized engine that supports general computation graphs for data analysis and a machine learning library. Support for running on YARN (Hadoop NextGen) was added to Spark in version 0.6.0. You can learn more about this <a href="https://spark.apache.org/docs/latest/index.html">here</a> and <a href="https://en.wikipedia.org/wiki/Apache_Spark">here</a> 
</p>
<p align="justify">
In this tutorial, we are going to show you how to setup a spark cluster in distributed mode using a cluster of machine running Linux. For simplicity, we are going to setup a cluster of two nodes only. 
</p>
<p align="justify">
Here we describe a simple guide of how to make a heterogenous Spark cluster for custom built Python3.6 on SuSE Leap 42 linux. For cluster we will use two computers: 4-cores ‘quad’ with 4Gb RAM and 2-cores ‘duo’, also with 4Gb memory. The ‘quad’ is Alex’s workstation (${USER}=alex) and ‘duo’ is Neelam’s workstation (have ${USER}=neelam). In our setup, duo will be master because it has the same memory but less cores and quad will be the slave.
</p>

### Stage I. Build custom python3.6

<p>Python compilation and installation (into user’s home directory)</p> 
<p>I.1. Download and unpack python3.6</p> 
<p>I.2. Compile and install python</p>
<pre><code><i>CXX="/usr/bin/g++" ./configure --prefix=/home/${USER}/local/ --enable-shared --with-system-expat --with-system-ffi --with-ensurepip=install --enable-optimizations --enable-loadable-sqlite-extensions=yes 
make -j 2 && make test && make install
</i></code></pre>

<p>I.3. Register the libraries.</p>
<p>Add /home/${USER}/local/lib and /home/${USER}/local/lib64 into /etc/ld.so.cache and then <i>sudo ldconfig</i></p>

<p>I.4. Create python environment file ~/python.bashrc :</p>
<pre><code><i>unset PYTHONSTARTUP
export PATH="/home/${USER}/local/bin/:${PATH}"
export PYTHONHOME="/home/${USER}/local/"
export PYTHONPATH="/home/${USER}/local/lib/python3.6/site-packages/:/home/${USER}/local/lib64/python3.6/lib-dynload/"
</i></code></pre>

### Stage II. Create a dummy ‘<b>hduser</b>’ on both machines to run the cluster on his behalf (this require a reboot)

<p>II.1. Add user</p> 
<p><i>sudo useradd -d hduser</i></p>

<p>II.2. Add password</p>
<p><i>sudo passwd hduser</i></p>

<p>II.3. Create identification key for password-less ssh</p>
<p><i>ssh-keygen -t rsa</i></p>
<p>and put the same .ssh/id_rsa.pub into .ssh/id_rsa.pub at each machine</p>

<p>II.4. Allow authentification at both computers</p>
<p><i>cat .ssh/id_rsa.pub >> .ssh/authorized_keys</i></p> 

<p>II.5. Open firewall parts</p>
<p>II.5.a) at duo</p>
<p>edit file /etc/sysconfig/SuSEfirewall2</p>
<p><i>FW_CONFIGURATIONS_EXT="nfs-client sshd Spark"</i></p>
<p>create file /etc/sysconfig/SuSEfirewall2.d/services/Spark</p>
<pre><code><i>## Name: Spark
## Description: Open ports for Spark
# space separated list of allowed TCP ports
TCP="4040 7077 7078 8080 8081 18080 22221 22222"</i></code></pre>

<p>II.5.b) at quad</p>
<p>edit file /etc/sysconfig/SuSEfirewall2</p>
<p><i>FW_CONFIGURATIONS_EXT="nfs-client sshd Spark"</i></p>
<p>create file /etc/sysconfig/SuSEfirewall2.d/services/Spark</p>
<pre><code><i>## Name: Spark
## Description: Open ports for Spark
# space separated list of allowed TCP ports
TCP="7077 7078 8080 8081"</i></code></pre>

<p>II.6. Setup hduser's .bashrc</p>
<pre><code><i>source '/home/${USER}/python.bashrc'
export PYTHONPATH=${PYTHONPATH}:/home/hduser/spark-2.2.0/python/</i></code></pre>

<p>II.7. Reboot both computers to apply the settings</p>

### Stage III. Installing Spark under <b>hduser</b>

<p>III.1. Download and unpack Spark-2.2.0</p> 
<p>III.2. Build Spark</p>
<p><i>/home/hduser/spark-2.2.0/dev/make-distribution.sh --name yspark --pip --tgz -Phive -Phive-thriftserver </i></p>

<p>III.3. Configure cluster</p>
<p>III.3.a) at duo:</p>
<p>/home/hduser/spark-2.2.0/conf/spark-defaults.conf</p>
<pre><code><i>SPARK_MASTER_HOST=duo
SPARK_MASTER_PORT=7077
SPARK_MASTER_WEBUI_PORT=8080
SPARK_WORKER_CORES=2
SPARK_WORKER_MEMORY=1792m
SPARK_WORKER_PORT=7077
SPARK_WORKER_WEBUI_PORT=8081
SPARK_WORKER_DIR=/tmp/
SPARK_DAEMON_MEMORY=256m
PYSPARK_PYTHON=/home/neelam/local/bin/python3.6</i></code></pre>

<p>/home/hduser/spark-2.2.0/conf/slaves</p>
<pre><code><i>duo
quad</i></code></pre>

<p>/home/hduser/spark-2.2.0/conf/spark-defaults.conf</p>
<pre><code><i>spark.ui.port                     4040
spark.history.ui.port             18080
spark.driver.port                 22221
spark.blockManager.port           22222</i></code></pre>

<p>III.3.a) at quad:</p>
<p>/home/hduser/spark-2.2.0/conf/spark-defaults.conf</p>
<pre><code><i>SPARK_MASTER_HOST=duo
SPARK_MASTER_PORT=7077
SPARK_MASTER_WEBUI_PORT=8080
SPARK_WORKER_CORES=4
SPARK_WORKER_MEMORY=3584m
SPARK_WORKER_PORT=7078
SPARK_WORKER_WEBUI_PORT=8081
SPARK_WORKER_DIR=/tmp/
SPARK_DAEMON_MEMORY=128m
PYSPARK_PYTHON=/home/alex/local/bin/python3.6</i></code></pre>

<p>/home/hduser/spark-2.2.0/conf/spark-defaults.conf</p>
<pre><code><i>spark.ui.port                     4040
spark.history.ui.port             18080
spark.driver.port                 22221
spark.blockManager.port           22222</i></code></pre>

<p>III.4.. Run cluster (from duo) and check it state at duo:8080</p>
<p><i>/home/neelam/spark-2.2.0/sbin/start-all.sh</i></p>

<p>III.5. Run a test job over the cluster (from duo)</p>
<p><i>/home/hduser/spark-2.2.0/bin/spark-submit --master spark://duo:7077 examples/src/main/python/pi.py 1000</i></p>

<p>III.6. (Optional) stop the cluster</p>
<p><i>/home/neelam/spark-2.2.0/sbin/stop-all.sh</i></p>
<p><br></p>
<p><br></p>












You can use the [editor on GitHub](https://github.com/AIDesigners/AIDesigners.github.io-cluster_setup/edit/master/README.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/AIDesigners/AIDesigners.github.io-cluster_setup/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
