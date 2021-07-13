---
layout: post
category: MLSQL
---
# MLSQL-task3 python 环境设置
## 1. 检查本机python版本
```shell
~ python
Python 3.6.8 (v3.6.8:3c6b436a57, Dec 24 2018, 02:04:31)
```
## 2. 安装miniconda
```shell
~ brew install miniconda
```
## 3. 设置conda环境变量
```shell
~ vim ~/.bash_profile
[index]
export CONDA_HOME=/usr/local/Caskroom/miniconda/base
export PATH=$PATH:$CONDA_HOME/bin
```
## 4.创建 python 虚拟环境并启用
```shell
~ conda create -n dev python=3.6.8
```
## 5.安装依赖
```shell
~ source activate dev
(dev) ~ pip install Cython
(dev) ~ pip install pyarrow==0.10.0
(dev) ~ pip install ray==0.8.0
(dev) ~ pip install aiohttp psutil setproctitle grpcio pandas xlsxwriter
(dev) ~ pip install watchdog requests click uuid sfcli  pyjava
```
## 6.本机启动 ray
```shell
(dev) ~ ray start --head --include-webui
2021-07-07 17:13:30,597	INFO scripts.py:319 -- Using IP address 10.3.1.62 for this node.
2021-07-07 17:13:30,599	INFO resource_spec.py:216 -- Starting Ray with 3.32 GiB memory available for workers and up to 1.68 GiB for objects. You can adjust these settings with ray.init(memory=<bytes>, object_store_memory=<bytes>).
2021-07-07 17:13:31,712	INFO services.py:506 -- Failed to connect to the redis server, retrying.

======================================================================
View the dashboard at http://127.0.0.1:8080.
Note: If Ray is running on a remote node, you will need to set up an
SSH tunnel with local port forwarding in order to access the dashboard
in your browser, e.g. by running 'ssh -L 8080:127.0.0.1:8080
<username>@<host>'. Alternatively, you can set webui_host="0.0.0.0" in
the call to ray.init() to allow direct access from external machines.
======================================================================

2021-07-07 17:13:35,110	INFO scripts.py:350 -- 
Started Ray on this node. You can add additional nodes to the cluster by calling

    ray start --redis-address 10.3.1.62:57863

from the node you wish to add. You can connect a driver to the cluster from Python by running

    import ray
    ray.init(redis_address="10.3.1.62:57863")

If you have trouble connecting from a different machine, check that your firewall is configured properly. If you wish to terminate the processes that have been started, run

    ray stop

```
在本机8080端口启动
![task3-1](/public/img/task3-1.png)