---
layout: post
title: Setup Spark cluster on AWS EMR
date: 2018-08-11
logo: far fa-star
---

AWS provides
[an easy way](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-gs.html)
to run a Spark cluster. Let's use it to analyze the publicly
available [IRS 990 data](https://registry.opendata.aws/irs990/) from 2011 to present.
This data is already
[available on S3](https://aws.amazon.com/blogs/publicsector/irs-990-filing-data-now-available-as-an-aws-public-data-set/)
which makes it a good candidate to learn Spark. [This medium post](https://medium.com/@open990/the-irs-990-e-file-dataset-getting-to-the-chocolatey-center-of-data-deliciousness-90f66097a600)
describes the IRS 990 dataset. [This documentation](https://docs.opendata.aws/irs-990/readme.html)
shows you how to access this dataset on AWS S3.
We will see more details of the dataset later.


### Setup a Spark cluster

#### Caveats
I do not go over the details of setting up AWS EMR cluster.
Interested readers can read the
[official AWS guide](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-gs.html)
for details. Setting up an AWS EMR cluster requires
some familiarity with AWS concepts such as EC2, ssh keys, VPC subnets, and security groups.
I do not cover these details in this post either. Using AWS will probably require you
to setup a payment method (such as a credit card) and **may cost you some money**.
I advise you to keep an eye on the cost.
If this is your first time using a cloud computing platform
(such as [AWS](https://aws.amazon.com/) or [GCP](https://cloud.google.com/)),
I recommend you first try out setting up an EC2 instance before you move on to
setup an EMR cluster. Some steps such as setting up billing and setting up an
[EC2 key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)
are common to both EC2 and EMR.

We will use Python, [Jupyter notebook](http://jupyter.org/), and
[pyspark](http://spark.apache.org/docs/2.2.0/api/python/pyspark.html) to analyze the data.
AWS provides an already configured
[JupyterHub installation](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-jupyterhub.html)
but we will **not** use the AWS-provided installation. Instead, we will install our own
jupyter package on the cluster and use that instead.
This is because the AWS-provided JupyterHub installation runs within
a docker container and deprives the end user (which is you and me) of many python and
jupyter benefits. It also adds another layer of complexity to an already complicated
Spark cluster setup. The AWS-provided JupytrHub installation does provide us
with some features that we don't really need at this time.
The increased complexity in the AWS-provided installation makes it
difficult to install and use python packages (such as pandas and numpy).
Finally, installing our own jupyter package is a good way to learn how to setup some tools for
yourself, which is always a good thing for a data scientist.

#### Launch EMR cluster
We will use advanced options to launch the EMR cluster.

##### Step 1: Software and Steps
Launch an EMR cluster with a software configuration shown below in the picture.
Ensure that Hadoop and Spark are checked. If this is your first time setting up
an EMR cluster go ahead and check Hadoop, Zepplein, Livy, JupyterHub, Pig, Hive, Hue, and
Spark. Enabling JupyterHub and Zeppelin options will provide a
[read-to-use](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-jupyterhub-connect.html)
albeit clunky jupyter notebook (as described above in Caveats) but it won't
prevent us from what we are trying to do.

![AWS EMR Setup Step 1: Software Configuration](/assets/aws-emr-step-1-software-configuration.jpg)


##### Step 2: Hardware
You can choose any hardware you'd like. More powerful instances cost more money so decide
what's reasonable for you. If this is your first time, stay with the default hardware
options.

For now, we will use one `m4.large` instance for the Master and one `m4.large` instance for Core.
We don't use any task instances just yet because we can easily add those later.
We choose only one core instance for the same reason
-- we can easily add more instances later.

One restriction is the instance type --  you may not be able to change the instance types
later, especially for the master node. Therefore, you may want to choose a more powerful
instance type for the master node to begin with.
Typically, an instance with 8 virtual cores and 16GB of memory is a good start for a master
node. For Core instances, 4 virtual cores and 32 GB of memory or higher may be a good fit.
Typically, more memory and more EBS storage is useful for worker nodes.
If we need task instances, we can add them later as spot
instances which can be [significantly cheaper](https://aws.amazon.com/ec2/spot/).

![AWS EMR Setup Step 2: Hardware](/assets/aws-emr-step-2-hardware.jpg)


##### Step 3: General Cluster Settings
Let's provide a name for our cluster. The name doesn't have to be unique (even within one AWS
account, you can have multiple EMR clusters with the same exact name).
We enable logging to AWS S3 to a bucket of our choosing (for security reasons, it's a good idea
to have this bucket be private and encrypted). We don't need to specify any bootstrap actions
at this time.

![AWS EMR Setup Step #: General Cluster Settings](/assets/aws-emr-step-3-general-cluster-settings.jpg)


##### Step 4: Security
We use an already setup EC2 key pair. We will need to be able to ssh into the EMR cluster which
means this step is necessary. You may find these instructions on how to setup an
[EC2 key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) useful.
We use the default EC2 security groups for now. Security groups are important consideration
because Spark
[requires](https://spark.apache.org/docs/latest/spark-standalone.html#cluster-launch-scripts)
that master node can talk to the worker nodes via password-less (using a private key) ssh.
We need to make sure that the instances have the necessary ports open to allow ssh traffic
between them. Ideally, you should open your EMR cluster
just enough so that you can access it but any unauthorized person/program cannot.
If you're setting this cluster up for work, please consult your DevOps engineer or someone
in-charge of security.

![AWS EMR Setup Step 4: Security](/assets/aws-emr-step-4-security.jpg)


##### Launch
It might take 5-10 minutes after you click on `Create Cluster` button for your cluster to be
ready. When the cluster is ready, it should say `Waiting`, in green color.

### Setup `ssh` into the EMR cluster

#### Create a new security group
While the cluster is launching, let's create a new security group so that we can ssh into the
master node of the EMR cluster. Note that you can use this security group for any EC2 instance
including the EMR cluster we just launched.

1. Go to [EC2 homepage](https://console.aws.amazon.com/ec2) on AWS.
2. Make sure you're in the same region as the EMR cluster.
3. Click on `NETWORK & SECURITY > Security Groups` in the left-hand-side menu.
4. Click on `Create Security Group` button.
5. Provide any `Security group name`; we will use `allow-ping-ssh`.
6. Provide any `Description`; we will use 'Allows pinging and ssh'.
7. Use the same `VPC` as the one you used for the EMR cluster. Typically, this is the default value.
8. Click on `Add Rule` three times and add rules to allow ICMP (pings) in v4, v6 and SSH from
your own IP. See the screenshot below.
9. Click on `Create` to create the security group.

![Allow Ping and SSH from my IP](/assets/aws-security-group-alllow-ping-ssh.jpg)

**Caveat:** This security group setting may not be sufficiently secure for your situation.
As mentioned before, if you're doing this for for work, please consult your DevOps engineer
or someone in-charge of security.


#### Add security group to master node
We want to be able to ssh into the master node of the EMR cluster. To ssh, we want to allow
TCP traffic on port 22 (default port for ssh) from our IP (or similar) going to the master node
of the EMR cluster. We also allow pinging the master node from our IP address (this helps
in debugging ssh failures).


1. Go to [EMR homepage](https://console.aws.amazon.com/elasticmapreduce) on AWS.
2. Find your EMR cluster and click on it. _Tip: Ensure you're on the same region as the one you
launched your cluster in. Check the top right of the webpage for the name of the region._
3. Once you're on the cluster's homepage, click on `Hardware`. You should see your master and core
instance(s) listed.
4. Click on the `ID` of the master node. Typically, this is a link that looks like
like `ig-S0MESTR1NG`.
5. Click on the `EC2 instance ID`. Typically, this is a link that looks like `i-0a1b2c3d4e5f`.
You should now only see only row showing your master EC2 instance.
6. Right-click on this row, click on `Networking > Change Security Groups`.
7. Leave the already checked security group as-is. Check the `allow-ping-ssh` security group that we
created in the previous step.

#### SSH into master node
We should be all set to ssh into our EMR cluster. By logging into the EMR cluster, we mean
logging into the master node of the EMR cluster.

1. Go to the cluster homepage.
2. Ensure that the cluster is done provisioning and that you can
see `Waiting` in a green color.
3. Click on the SSH link on the cluster's homepage. This should tell you how to ssh into
your cluster. Typically, this requires running the `ssh` command on you local machine as
shown below.

```bash
# On local machine
ssh -i ~/my-ec2-key-pair.pem hadoop@ec2-xxx-xxx-xxx-xxx.compute-1.amazonaws.com
```

<br/>
<br/>
**Troubleshooting**

This is how a successful ssh into the master node looks like.

![SSH into master node](/assets/aws-ssh-into-master-node.jpg)

If you cannot ssh into the master node, try pinging it first.

```bash
# On local machine
ping ec2-xxx-xxx-xxx-xxx.compute-1.amazonaws.com
```

If you can successfully ping the instance, then you're halfway there. Try
using the verbose mode of ssh to better understand what's preventing you from `ssh`ing into
the master node.

```bash
# On local machine
ssh -vv -i ~/my-ec2-key-pair.pem hadoop@ec2-xxx-xxx-xxx-xxx.compute-1.amazonaws.com
```

Finally, take a look at this
[AWS troubleshooting page](https://aws.amazon.com/premiumsupport/knowledge-center/ec2-linux-ssh-troubleshooting/)
which contains some really helpful videos to solve this exact issue.

You _will_ need to ssh into the master node to perform the next steps in this post.


### Get to know your EMR cluster
Let's explore the installation first.
This builds a general understanding of how things work, especially when you're using a
machine that someone else (in our case, AWS) setup for you. Note that your results might vary a
little depending on specific versions of software packages being run on your instance.

#### User
Firstly, we see that we are logged in as the user `hadoop`.

```bash
# On EMR master node
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ whoami
hadoop
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ pwd
/home/hadoop
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ echo $HOSTNAME
ip-xxx-xxx-xxx-xxx
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ echo $SHELL
/bin/bash

```

#### Python
Since we want to use python, let's see what python goodies are available and where they are
stored. It looks like `python` 2.7, `python` 3.4, and `pip` for `python` 2 are installed.

```bash
# On EMR master node
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ which python
/usr/bin/python
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ which python3
/usr/bin/python3
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ python --version
Python 2.7.14
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ python3 --version
Python 3.4.8
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ which pip
/usr/bin/pip
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ which pip3
/usr/bin/which: no pip3 in (/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin:/opt/aws/bin)
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ pip --version
pip 9.0.3 from /usr/lib/python2.7/dist-packages (python 2.7)
```

Let's see what `python` 2 packages are installed. Note that `numpy` is installed but `pandas`
and `matplotlib` are not installed.

```bash
# On EMR master node
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ pip freeze
aws-cfn-bootstrap==1.4
awscli==1.14.9
...
nltk==3.2
...
numpy==1.14.5
...
virtualenv==15.1.0
...
You are using pip version 9.0.3, however version 18.0 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
```

Let's check the user-site and system-site for the installed `python` 2. It looks like
the user-site doesn't exist yet which means all of the packages we saw above are installed
at the system site. We also get warning to upgrade `pip` but we won't do that because `pip`
seems to be installed in `/usr/lib` which will require the use of `sudo` to upgrade.
Using `sudo` can mess up the system python installation.

```bash
# On EMR master node
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ python -m site
sys.path = [
    '/home/hadoop',
    '/usr/lib/python27.zip',
    '/usr/lib64/python2.7',
    '/usr/lib64/python2.7/plat-linux2',
    '/usr/lib64/python2.7/lib-tk',
    '/usr/lib64/python2.7/lib-old',
    '/usr/lib64/python2.7/lib-dynload',
    '/usr/local/lib64/python2.7/site-packages',
    '/usr/local/lib/python2.7/site-packages',
    '/usr/lib64/python2.7/site-packages',
    '/usr/lib/python2.7/site-packages',
    '/usr/lib64/python2.7/dist-packages',
    '/usr/lib64/python2.7/dist-packages/PIL',
    '/usr/lib/python2.7/dist-packages',
]
USER_BASE: '/home/hadoop/.local' (doesn't exist)
USER_SITE: '/home/hadoop/.local/lib/python2.7/site-packages' (doesn't exist)
ENABLE_USER_SITE: True
```

```bash
# On EMR master node
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ pip show numpy
Name: numpy
Version: 1.14.5
Summary: NumPy: array processing for numbers, strings, records, and objects.
Home-page: http://www.numpy.org
Author: Travis E. Oliphant et al.
Author-email: None
License: BSD
Location: /usr/local/lib64/python2.7/site-packages
Requires:
You are using pip version 9.0.3, however version 18.0 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
```

#### Spark
Let's inspect the spark installation, starting with `pyspark`.
We see that `pyspark` is installed in `/usr/bin` and it's simply a bash script that calls another
bash script `/usr/lib/spark/bin/pyspark`.

```bash
# On EMR master node
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ which pyspark
/usr/bin/pyspark
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ more /usr/bin/pyspark
#!/bin/bash

# Autodetect JAVA_HOME if not defined
. /usr/lib/bigtop-utils/bigtop-detect-javahome

exec /usr/lib/spark/bin/pyspark "$@"
```



```bash
# On EMR master node
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ more /usr/lib/spark/bin/pyspark
#!/usr/bin/env bash
...
export PYSPARK_DRIVER_PYTHON
export PYSPARK_DRIVER_PYTHON_OPTS
exec "${SPARK_HOME}"/bin/spark-submit pyspark-shell-main --name "PySparkShell" "$@"
```

Let's check out `/usr/lib/spark`. We see that `/usr/lib/spark` is `SPARK_HOME` -- it
contains the spark installation. We can get to the spark config files
(`/usr/lib/spark/conf`) and work folder (`/usr/lib/spark/work`) from here.

```bash
# On EMR master node
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ cd /usr/lib/spark/
[hadoop@ip-xxx-xxx-xxx-xxx spark]$ ls -l
total 112
drwxr-xr-x 2 root root  4096 Aug 13 04:04 bin
lrwxrwxrwx 1 root root    15 Aug 13 04:04 conf -> /etc/spark/conf
drwxr-xr-x 5 root root  4096 Aug 13 04:04 data
drwxr-xr-x 4 root root  4096 Aug 13 04:04 examples
drwxr-xr-x 3 root root  4096 Aug 13 04:05 external
drwxr-xr-x 2 root root 20480 Aug 13 04:05 jars
-rw-r--r-- 1 root root 18045 Jun 29 18:11 LICENSE
drwxr-xr-x 2 root root  4096 Aug 13 04:04 licenses
-rw-r--r-- 1 root root 24913 Jun 29 18:11 NOTICE
drwxr-xr-x 6 root root  4096 Aug 13 04:04 python
drwxr-xr-x 3 root root  4096 Aug 13 04:04 R
-rw-r--r-- 1 root root  3809 Jun 29 18:11 README.md
-rw-r--r-- 1 root root   753 Jun 29 18:11 RELEASE
drwxr-xr-x 2 root root  4096 Aug 13 04:04 sbin
lrwxrwxrwx 1 root root    19 Aug 13 04:04 work -> /var/run/spark/work
drwxr-xr-x 3 root root  4096 Aug 13 04:05 yarn
```

Note that `pyspark` is not installed by `pip`. Instead, `pyspark` is available within our
spark installation at `/usr/lib/spark/python`.
We won't need to install `pysaprk` using `pip`; instead, we will modify
`PYTHONPATH` and use the `pyspark` installation at `/usr/lib/spark/python`.

```bash
# On EMR master node
[hadoop@ip-xxx-xxx-xxx-xxx python]$ pip show pyspark
You are using pip version 9.0.3, however version 18.0 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
```

Finally, let's see if we already have spark master daemon running on our master node.
It appears that we don't have a spark master daemon running, which is expected.

```bash
# On EMR master node
[hadoop@ip-xxx-xxx-xxx-xxx python]$ ps aux | grep "spark"
hadoop    7416  0.0  0.0 110512  2100 pts/0    S+   05:33   0:00 grep --color=auto spark
spark    16036  0.2  2.6 3614692 213244 ?      Sl   04:09   0:11 /usr/lib/jvm/java-openjdk/bin/java -cp /usr/lib/spark/conf/:/usr/lib/spark/jars/*:/etc/hadoop/conf/ -XX:OnOutOfMemoryError=kill -9 %p -Xmx1g org.apache.spark.deploy.history.HistoryServer
```


### Install Python packages
Let's install `jupyter` and any other python package on the master node.
Unlike AWS-provided
[JupyterHub installation](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-jupyterhub.html),
here, we can easily install python packages (including `jupyter`) using a familiar process;
we don't have to worry about docker instances.

Note that we never use `sudo` to install a python
package in these instructions. Using `sudo` is in general a bad idea, even on your local machines
(Linux and MacOS) because there are many system utilities that depend on system python and
using `sudo` updates your system-wide python installation. We always use
`pip install --user <pkg-name>` which installs python packages in the user-site while leaving
system python dependencies alone. This also makes for an easy cleanup if things don't work out
because we can easily delete user-site files and folders.

```bash
# On EMR master node
pip install --user jupyter
pip install --user pandas
pip install --user matplotlib
```

Note that in some cases you'll want to install a python package (typically, as a dependency)
that is already installed in the system-site. It may be useful to re-install the package in the
user-site to avoid dependency errors. To do this, you can use `-I` option like this:

```bash
# On EMR master node
pip install --user -I --upgrade python-dateutil

# Tip: You may want to try this to avoid errors associated with cached files.
# This typically happens when you install a package for both python2 and python3.
# pip install --user -I --upgrade --no-cache-dir python-dateutil
```


### Launch Jupyter Notebook
#### Setup port forwarding
To setup port forwarding, we need to make a (new) ssh connection with the port forwarding
information specified. If you still have the ssh connection that we made in the previous steps,
you can keep that running or exit that;
you'll still need to setup port forwarding using another ssh connection.

```bash
# On local machine
ssh -L 8889:localhost:8889 -i ~/my-ec2-key-pair.pem hadoop@ec2-xxx-xxx-xxx-xxx.compute-1.amazonaws.com
```

This sets up your local machine's port `8889` to listen to the master node's
port `8889`. The port `8889` is the default port at which jupyter notebook serves. You should
confirm that `8889` is indeed the port on which jupyter notebook is served (keep on reading).
If not, you can simply replace `8889` with the port that jupyter notebook serves on.
Typically, the port `8889` does not conflict with the other default
[EMR ports](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-web-interfaces.html).
The AWS-provided JupyterHub notebook is
[served](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-jupyterhub-connect.html)
on port `9443`.

Note that if you don't need to use jupyter notebook and want to use ipython or
python terminal instead, then you don't need to setup port forwarding.

#### Setup environment variables
We will use the provided `pyspark` script -- it comes with support for jupyter notebook and
even `ipython` terminal.

```bash
# On EMR master node
export PYTHONPATH="/home/hadoop/.local/lib/python2.7/site-packages:$PYTHONPATH"
export PYSPARK_DRIVER_PYTHON=/home/hadoop/.local/bin/jupyter
export PYSPARK_DRIVER_PYTHON_OPTS='notebook'

# Check
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ echo $PYSPARK_DRIVER_PYTHON
/home/hadoop/.local/bin/jupyter

[hadoop@ip-xxx-xxx-xxx-xxx ~]$ echo $PYSPARK_DRIVER_PYTHON_OPTS
notebook

[hadoop@ip-xxx-xxx-xxx-xxx ~]$ echo $PYTHONPATH
/home/hadoop/.local/lib/python2.7/site-packages:
```

<br/>
**Aside: ipython terminal instead of jupyter notebook**

If you want to use `ipython`, then you don't need to setup port forwarding.
You need to simply install `ipython` on the master node and then use different
environment variables. A benefit of using this method is that you can use `ipython` instead
of a regular `python` REPL. The `ipython` session will come with all the bells and whistles such
as syntax-highlighted output, tab-completion, and "?help".

```bash
# On EMR master node
pip install --user ipython
```

```bash
# On EMR master node
export PYTHONPATH="/home/hadoop/.local/lib/python2.7/site-packages:$PYTHONPATH"
export PYSPARK_DRIVER_PYTHON=/home/hadoop/.local/bin/ipython

# Check
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ echo $PYSPARK_DRIVER_PYTHON
/home/hadoop/.local/bin/ipython

[hadoop@ip-xxx-xxx-xxx-xxx ~]$ echo $PYTHONPATH
/home/hadoop/.local/lib/python2.7/site-packages:
```



#### Launch `pyspark`
We launch `pyspark` which will, in turn, launch the pyspark driver that we setup as the
environmental variable.

```bash
# On EMR master node
# Launch pyspark. This will use the environmental variables we setup.
pyspark
```

If you setup jupyter notebook as the driver, then you may see an interactive question in your
terminal asking you to trust the notebook. Select the "A" (Always) option -- this lasts only as
long as the jupyter notebook is running. You will have to make this selection every time you
quit and re-launch `pyspark`. Press "Q" to exit the text-view of the jupyter notebook; your
notebook will still be running after. Jupyter should now tell you which port it's serving on.
This is where you ensure that the port is `8889`. If it's not, you should setup another ssh
port forwarding connection in a separate terminal window with the correct port number.
You must leave the port forwarding ssh session running.

### Open jupyter notebook in a browser
Go to URL printed on the terminal screen; it should look something like this.
You should see a jupyter notebook homepage. The ssh terminal will print useful
messages and you must keep it open.

```bash
# Output of pyspark (on EMR master node)
...
Copy/paste this URL into your browser when you connect for the first time,
to login with a token:
    http://localhost:8889/?token=s0m3l0ngalphanumer1cstr1ng
...
```



#### Find your bearings
From the jupyter notebook homepage, click on "New" and open a terminal.
Type out these commands to check things out. Note that this terminal provides `sudo`-level
access to the master node of your EMR cluster. Be careful!

```bash
# On terminal within the jupyter browser tab
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ whoami
hadoop
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ pwd
/home/hadoop
[hadoop@ip-xxx-xxx-xxx-xxx ~]$ echo $HOSTNAME
ip-xxx-xxx-xxx-xxx
```

#### Test Spark variables
From the jupyter notebook homepage, click on "New" and open a "Python 2" notebook.
The spark-related variables should already be available.

![Jupyter notebook screenshot showing spark variables](/assets/aws-jupyter-notebook-screenshot.png)


#### Import packages
You can import _any_ package that you installed previously. You install more packages in
using the instructions provided previously.
You may need to restart your ssh or jupyter session (in some cases)
before you can import newly installed packages. As an example, you can import `pandas` and
`matplotlib`.

![Jupyter notebook: import packages like pandas](/assets/aws-jupyter-notebook-import-packages.png)


#### Test jupyter features
Jupyter notebook comes with all the (default) bells and whistles. You can make and display plots.
The dataframes are printed in a pretty table format. Tab-completion works and "?sc" help
is available.

![Jupyter notebook: bells and whistles](/assets/aws-jupyter-notebook-bells-whistles.png)


This is enough for one post. Stay tuned for another post in which we will actually
analyze the data using our newly created Spark cluster.
_To be continued ..._