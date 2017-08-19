## Spark Cluster Setup

<p align="justify">
In 2010, Google introduced MapReduce, a batch-processing framework for running computations involving large data volume over a commodity cluster. MapReduce was later adopted by several companies including Google, Amazon, IBM etc. It solved the scalability issues to great extent when dealing with large datasets and showed good performance, provided that the recipes for data processing do not involve any repeatative processing of a single step. The reason for this was that MapReduce reads data at the begining of a step and the scope of the objects holding this data are upto the end of this step. So the data objects are destoyed once a step ends. As a result, repeating a step incurs significant parallelism overhead. 
</p>
<p align="justify">
Spark was introduced to mainly overcome this limitation of MapReduce.Spark is a fast and memory-efficent clustering computing framework. It support both batch-style as well as computations over data streams. It provides high-level APIs in Scala, Java, Python, and R. It also provides an optimized engine that supports general computation graphs for data analysis and a machine learning library. Support for running on YARN (Hadoop NextGen) was added to Spark in version 0.6.0. You can learn more about this <a href="https://spark.apache.org/docs/latest/index.html">here</a> and <a href="https://en.wikipedia.org/wiki/Apache_Spark">here</a> 
</p>>
<p align="justify">
In this tutorial, we are going to show you how to setup a spark cluster in distributed mode using a cluster of machine running Linux. For simplicity, we are going to setup a cluster of two nodes only. 
</p>
<p align="justify">
Here we describe a simple guide of how to make a heterogenous Spark cluster for custom built Python3.6 on SuSE Leap 42 linux. For cluster we will use two computers: 4-cores ‘quad’ with 4Gb RAM and 2-cores ‘duo’, also with 4Gb memory. The ‘quad’ is Alex’s workstation (${USER}=alex) and ‘duo’ is Neelam’s workstation (have ${USER}=neelam). In our setup, duo will be master because it has the same memory but less cores and quad will be the slave.
</p>
Stage I. Build custom python3.6
Python compilation and installation (into user’s home directory) 
I.1. Download and unpack python3.6 
I.2. Compile and install python
<body>
  <p>
CXX="/usr/bin/g++" ./configure --prefix=/home/${USER}/local/ --enable-shared \<\br>
--with-system-expat --with-system-ffi --with-ensurepip=install \ <\br>
--enable-optimizations --enable-loadable-sqlite-extensions=yes <\br>
make -j 2 && make test && make install 
  </p>
</body>
I.3. Register the libraries.
Add /home/${USER}/local/lib and /home/${USER}/local/lib64 into /etc/ld.so.cache and then sudo ldconfig

I.4. Create python environment file ~/python.bashrc :
unset PYTHONSTARTUP
export PATH="/home/${USER}/local/bin/:${PATH}"
export PYTHONHOME="/home/${USER}/local/"
export PYTHONPATH="/home/${USER}/local/lib/python3.6/site-packages/:/home/${USER}/local/lib64/python3.6/lib-dynload/"

Stage II. Create a dummy ‘hduser’ on both machines to run the cluster on his behalf (this require a reboot)

II.1. Add user 
sudo useradd -d hduser

II.2. Add password
sudo passwd hduser

II.3. Create identification key for password-less ssh
ssh-keygen -t rsa
and put the same .ssh/id_rsa.pub into .ssh/id_rsa.pub at each machine

II.4. Allow authentification at both computers
cat .ssh/id_rsa.pub >> .ssh/authorized_keys 
II.5. Open firewall parts
II.5.a) at duo
edit file /etc/sysconfig/SuSEfirewall2
FW_CONFIGURATIONS_EXT="nfs-client sshd Spark"
create file /etc/sysconfig/SuSEfirewall2.d/services/Spark
## Name: Spark
## Description: Open ports for Spark
# space separated list of allowed TCP ports
TCP="4040 7077 7078 8080 8081 18080 22221 22222"

II.5.b) at quad
edit file /etc/sysconfig/SuSEfirewall2
FW_CONFIGURATIONS_EXT="nfs-client sshd Spark"
create file /etc/sysconfig/SuSEfirewall2.d/services/Spark
## Name: Spark
## Description: Open ports for Spark
# space separated list of allowed TCP ports
TCP="7077 7078 8080 8081"

II.6. Reboot both computers to apply the settings

Stage III. Installing Spark under hduser
III.1. Download and unpack Spark-2.2.0 
III.2. Build Spark
./dev/make-distribution.sh --name yspark --pip --tgz -Phive -Phive-thriftserver

III.3. Configure cluster
III.3.a) at duo:
/home/hduser/spark-2.2.0/conf/spark-defaults.conf
SPARK_MASTER_HOST=duo
SPARK_MASTER_PORT=7077
SPARK_MASTER_WEBUI_PORT=8080
SPARK_WORKER_CORES=2
SPARK_WORKER_MEMORY=1792m
SPARK_WORKER_PORT=7077
SPARK_WORKER_WEBUI_PORT=8081
SPARK_WORKER_DIR=/tmp/
SPARK_DAEMON_MEMORY=256m
PYSPARK_PYTHON=/home/neelam/local/bin/python3.6

/home/hduser/spark-2.2.0/conf/slaves
duo
quad

/home/hduser/spark-2.2.0/conf/spark-defaults.conf
spark.ui.port                     4040
spark.history.ui.port             18080
spark.driver.port                 22221
spark.blockManager.port           22222

III.3.a) at quad:
/home/hduser/spark-2.2.0/conf/spark-defaults.conf
SPARK_MASTER_HOST=duo
SPARK_MASTER_PORT=7077
SPARK_MASTER_WEBUI_PORT=8080
SPARK_WORKER_CORES=4
SPARK_WORKER_MEMORY=3584m
SPARK_WORKER_PORT=7078
SPARK_WORKER_WEBUI_PORT=8081
SPARK_WORKER_DIR=/tmp/
SPARK_DAEMON_MEMORY=128m
PYSPARK_PYTHON=/home/alex/local/bin/python3.6

/home/hduser/spark-2.2.0/conf/spark-defaults.conf
spark.ui.port                     4040
spark.history.ui.port             18080
spark.driver.port                 22221
spark.blockManager.port           22222

III.4.. Run cluster (from duo) and check it state at duo:8080
/home/neelam/spark-2.2.0/sbin/start-all.sh

III.5. Run a test job over the cluster (from duo) 
/home/hduser/spark-2.2.0/bin/spark-submit --master spark://duo:7077 examples/src/main/python/pi.py 1000

III.6. (Optional) stop the cluster
/home/neelam/spark-2.2.0/sbin/stop-all.sh







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
