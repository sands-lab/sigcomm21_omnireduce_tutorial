# OmniReduce Tutorial Hands-On

This repository is part of the [ACM SIGCOMM 2021 TUTORIAL: Network-Accelerated Distributed Deep Learning](https://conferences.sigcomm.org/sigcomm/2021/NADDL-tutorial.html).
More specifically, it contains the hands-on labs enabling participants to run OmniReduce benchmarks.

## Prerequisites and Setup
### 1. Software install
To benefit from the hands-on, you need recent versions of the following software installed on your local computer:

* [Vagrant](https://www.vagrantup.com/docs/installation)
* [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

### 2. VM creation
To setup the vagrant box, simply `cd` to this folder and run the following commands on your host
```bash
# The first `vagrant up` invocation fetches the vagrant box.
# It is likely that this takes some time, so launch this command ASAP!
$ vagrant up
```
Once done, two virtual machines, `node1` and `node2`, are created and started.
### 3. Add RDMA device
In this tutorial, we use [Soft-RoCE](https://www.roceinitiative.org/wp-content/uploads/2016/11/SoftRoCE_Paper_FINAL.pdf) to support RDMA. We have configured Soft-RoCE in VMs, what you  need to do is to add an rdma link for the specified type (rxe) to the network device on both `node1` and `node2`. 
Firstly, open two terminals on your host and run the following commands to SSH `node1` and `node2`

```bash
# In terminal 1, SSH node1
$ vagrant ssh node1
```
```bash
# In terminal 2, SSH node2
$ vagrant ssh node2
```
> Starting from now, we assume that otherwise stated, all commands are run inside the vagrant box. 

After that, you need to run the following commands inside both `node1` and `node2`
```bash
# Check the network interface name, it should be eth1
$ ifconfig
# Add an rdma link for rxe to eth1
$ sudo rdma link add roce type rxe netdev eth1
# Check the status of RDMA configuration, make sure that device was added under RDEV (rxe device)
# If there is no error, you will see the following outputs.
$ ibv_devices
#    device                 node GUID
#    ------              ----------------
#    rocep0s8            0a0027fffedd1943
```
### 4. Other setups
We have configured an SSH login without password between `node1` and `node2` for `mpirun`. You need to check it with the following command 
```bash
# On node1
$ ssh 192.168.59.102
```
```bash
# On node2
$ ssh 192.168.59.101
```
By default, vagrant will share your project directory (the directory with the Vagrantfile) to `/vagrant`. Check whether `node1` and `node2` can access to this folder. If it is accessible, copy omnireduce project folder to `vagrant` with following command
```bash
# On node1
$ cp -r /home/vagrant/omnireduce /vagrant
```


## Experiment
Once setup is done,  we can run the experiment. We run OmniReduce on `node1` and `node2` in colocated mode, which means we run both `aggregator` and `worker` on each VM.  In order to reduce the hardware requirements, we only run AllReduce on CPU tensors in this tutorial.
The steps are as follows.

**1. Open three terminals on the host and `cd` to this folder. Use two of them to SSH `node1` and one to SSH `node2`.**
```bash
# In terminal 1, SSH node1
$ vagrant ssh node1
```
```bash
# In terminal 2, SSH node1
$ vagrant ssh node1
```
```bash
# In terminal 3, SSH node2
$ vagrant ssh node2
```
**2. Check the `omnireduce.cfg` in `/vagrant/omnireduce/omnireduce-RDMA/example`.**

In most cases, you do not need to change anything in this file. One thing you need to confirm is that `ib_hca` is the same to the device output of `ibv_devices` command.

**3. Use terminal 2 and terminal 3 to run `aggregator` program.**
```bash
# In terminal 2, inside node1
$ cd /vagrant/omnireduce/omnireduce-RDMA/example
$ ./aggregator
```
```bash
# In terminal 3, inside node2
$ cd /vagrant/omnireduce/omnireduce-RDMA/example
$ ./aggregator
```
**4. Use terminal 1 to run `worker` program.**
```bash
# In terminal 1, inside node1
$ cd /vagrant/omnireduce/omnireduce-RDMA/example
$ mpirun -n 2 -hosts 192.168.59.101,192.168.59.102 ./worker
```
The benchmark code we run is `worker_test.cpp`, the float tensor size is 262144 and block density is 1.0 in default setting. You can modify these in the `worker_test.cpp` (`tensor_size` in line 16 and `density` in line 25) and run `make` in this folder. Note that you need to restart `aggregator` before each run.
