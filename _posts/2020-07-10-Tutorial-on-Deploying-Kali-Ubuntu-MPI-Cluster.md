---
layout: post
title: "Tutorial on Deploying Kali-Ubuntu MPI Cluster"
excerpt: "How to deploy MPI cluster using Kali Linux and Ubuntu?"
date: June 01, 2020
toc:
  sidebar: left

tags:
  - MPI
  - Cluster
  - Linux
---

### 1. Introduction

In this project, we aim to develop a cluster consisting of Kali Linux and Ubuntu. This brief tutorial is using Kali and Ubuntu, but any variant of Linux can be used with minimal change. It is also worth mentioning that the cluster implemented in this project consists of a base machine and a virtual machine. In the base machine, Kali Linux was installed as a base operating system. In the virtual machine, Ubuntu was installed. I opted for this environment for exhibiting an efficient way of dealing with the cluster computing paradigm while being on a single machine.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/p1_mpi_1.pdf" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

The steps mentioned in this tutorial can be followed for deploying the cluster in LAN environment also. It depends on the requirement of the user. For creating a virtual machine, Qemu/KVM was used in this tutorial but readers can also use VirtualBox or VMware.

---

### 2. Installation of Core Dependencies

For developing a cluster computing environment, MPICH is to be installed on machines. MPICH is an implementation of Message Passing Interface (MPI) that is a portable message-passing standard designed for dealing with different parallel computing architectures. The following steps should be followed for installing MPICH on both machines (i.e. Base Machine and Virtual Machine):

Download the latest version of MPICH from [here](http://www.mpich.org/downloads/).

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/p1_mpi_2.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

Un-compress and navigate to downloaded directory.

```sh
$ tar –xzf mpich-3.3.2.tar.gz
$ cd mpich-3.3.2
```

Configure the installation.

```sh
$ ./configure --disable-fortran
```

<i class="far fa-sticky-note"></i> **Troubleshooting Error:** The `disable-fortran` option disables Fortran-related packages because mpich comes in C, C++, and Fortran implementation.
{: .notice--info}
{: .text-justify}

After configuration, build and install mpich.

```sh
$ make; sudo make install
```

Check the installation.

```sh
$ mpiexec --version
```

---

### 3. Configuring IP-Hostname Mapping

The machines existing on the network can be addressed using their hostnames. Run `ifconfig` or `ip addr` command on both machines for determining their respective IP addresses. In order to map their IP addresses to the hostnames, some entries must be appended in `/etc/hosts` file of server and client machine.

On server,

```sh
$ cat /etc/hosts
127.0.0.1	localhost

192.168.56.102 master
192.168.56.104 slave

```

On client,

```sh
$ cat /etc/hosts
127.0.0.1	localhost

192.168.56.102 master
192.168.56.104 slave

```

---

### 4. Creating Dedicated User Accounts

For maintaining the simplicity and clear workflow, create separate user accounts for cluster paradigm on both machines:

On Server Machine

```sh
$ sudo adduser fserver
```

On Client Machine

```sh
$ sudo adduser fclient
```

Separate user accounts different from the conventional user accounts will help in keeping the environment different and safe.

---

### 5. SSH Configuration

Install openssh-server on both machines:

```sh
$ sudo apt install openssh-server
```

The SSH service must be active for creating the operational SSH tunnel between two machines. The status of SSH service can be checked by entering the following command:

```sh
$ sudo systemctl status ssh
```

If the status is inactive then SSH service can be enabled by using the command:

```sh
$ sudo systemctl restart ssh
```

Once the SSH service becomes active on both machines, an ssh session can be initiated from the server machine for creating an ssh tunnel to the client machine.

```sh
$ ssh fclient@ubuntu
$ ssh fclient@192.168.56.104
```

192.168.56.104 is the IP address of the client machine (slave).

---

### 6. NFS Configuration

Network File System (NFS) is a client-server application, which provides the feature of a shared directory that can be accessed by different network users. NFS uses the Virtual File System (VFS) interface based on TCP/IP.

In this project, NFS is used for creating a shared directory. This shared directory is to be used by clients for storing and accessing data. The server machine, base machine in our case, is used for installing NFS-server, while the NFS-client application is installed in every client machine. The installation and configuration of NFS are further elaborated be in the following paragraphs:

#### 6.1. Server Machine (Base Machine – Kali Linux)

Install the NFS-server

```sh
$ sudo apt-get install nfs-kernel-server
```

Create the shared directory (The name used by the directory in server machine will be used again for creating similar directory in client machine).

```sh
$ mkdir cloud
```

For exporting the shared directory, append the line `/home/fserver/cloud *(rw,sync,no_root_squash,no_subtree_check` in `/etc/exports`.

```sh
$  cat /etc/exports
/home/mpiuser/cloud *(rw,sync,no_root_squash,no_subtree_check)
```

After making any change in /etc/exports, following command is run:

```sh
$ exportfs –a
$ sudo systemctl restart nfs-kernel-server
```

The status of nfs-server can be checked by the following command:

```sh
$ sudo systemctl status nfs-kernel-server
```

#### 6.2 Client Machine (Virtual Machine – Ubuntu)

Install the NFS-client

```sh
$ sudo apt install nfs-common
```

Create the same directory in client machine which is to be shared via NFS:

```sh
$ mkdir cloud
```

Mount the shared directory

```sh
$ sudo mount -t nfs master:/home/mpiuser/cloud ~/cloud
```

The status of mounted directory can be checked by using following command:

```sh
$ df -h
```

Every time system reboots, the shared directory is to be mounted. To avoid the hassle of mounting directory again and again, this task of mounting can be automated and the shared directory will be automatically mounted on every system reboot operation.

Append the line “master:/home/mpiuser/cloud /home/mpiuser/cloud nfs“ in /etc/fstab file.

```sh
$ cat /etc/fstab
#MPI CLUSTER SETUP
master:/home/mpiuser/cloud /home/mpiuser/cloud nfs
```

The status of nfs-client can be checked by the following command:

```sh
$ sudo systemctl status nfs-common
```

---

### 7. Testing the cluster

<i class="far fa-sticky-note"></i> Download the program by cloning the repository `https://github.com/smfarjad/Kali-Ubuntu-MPI-Cluster` and follow the instructions mentioned in README.
{: .notice--info}
{: .text-justify}

To run the program only on the same master node

```sh
$ mpirun -np 10 --hosts master ./hello_MPI
```

To run the program on master and slave nodes

```sh
$ mpirun -np 10 --hosts master,slave ./hello_MPI
```

---

### 8. Conclusion

The tutorial summarizes all the necessary details for implementing a cluster using Kali Linux and Ubuntu. The readers can follow the same instructions for setting up a cluster on the other distributions of Linux. In case of any discrepancy regarding the tutorial, feel free to comment below or approach me via [email](mailto:smfarjad@outlook.com).

---

### 9. References

- [Message Passing Interface](https://en.wikipedia.org/wiki/Message_Passing_Interface)
- [MPICH](https://www.mpich.org/)
