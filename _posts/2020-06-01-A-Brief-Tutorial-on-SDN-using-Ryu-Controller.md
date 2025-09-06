---
layout: post
title: "A Brief Tutorial on SDN using Ryu Controller"
header:
  teaser: /assets/images/thumbnails/joel-filipe-thumb-800.jpg
excerpt: "A descriptive tutorial about the development and deployment of the SDN paradigm in Linux environment using Mininet and Ryu controller."
date: June 01, 2020
toc:
  sidebar: left

tags:
  - Software-Defined Networking
  - Ryu Controller
  - Open vSwitch
---

### 1. Introduction

The conventional networking paradigm consists of end devices, switches, and routers. The end devices connected in the network communicate with each other through packet transmission. The task of transmitting packets to respective destination hosts is primarily performed by the routers and switches. The routers and switches are termed as networking devices. A networking device consists of two components: a data plane for forwarding packets and a control plane for routing packets. The control plane performs a more intensive task when compared with the data plane because it has to maintain and manage routing tables, routing algorithms, etc. This is why the control plane can be considered as the brain of a networking device.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/p2_sdn1.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

With the expansion of the network, the number of networking devices also increases which directly renders an impact on the performance of the network. The degradation in performance is caused by the burden of managing routing information at every networking device. The innovation of software-defined networking aims to solve this degradation problem which often arises in large networks.

---

### 2. Software-Defined Networking

Software-Defined Networking (SDN) is an architecture that centralizes the brains of all networking devices into a single component. All the networking devices are operated through this central component. The management of packet transmission is performed by the same central entity and the networking devices behave as ‘forwarding devices’, which receive and forward packets. In technical terms, SDN dissects the control planes from networking devices and centralizes them to a single component, and the networking devices are left out with only data planes that are responsible for forwarding packets.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/p2_sdn2.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

---

### 3. SDN Architecture

SDN architecture consists of three major layers:

- **Application Layer:** It is responsible for developing and managing networking applications.
- **Control Layer:** It is responsible for routing packets to their respective destination while applying different rules and policies.
- **Infrastructure Layer:** It is responsible for dealing with forwarding devices or SDN switches.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/p2_sdn_arch.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

---

### 4. Working

The networking applications are built and deployed on the top of the application layer and these applications communicate with Network Operating System (NOS) through Application Programming Interfaces (APIs) which are called “Northbound APIs”. NOS communicates and implements the routing rules and policies on forwarding devices via “Southbound APIs”. The end devices are connected to these forwarding devices for executing the task of communication through packets.

---

### 5. SDN Terminologies

<ul style="list-style-type:disc">
 <li><b>Network Operating System:</b> Network Operating System (NOS) is used for routing packets. In the SDN paradigm, NOS is called the controller because it consists of a control plane. An SDN controller is the central component where dissected intelligence of conventional networking devices is merged. There are different SDN controllers built on the stack of different programming languages.
</li>
  
<li><b>OpenFlow:</b> OpenFlow is the core protocol of SDN architecture. The controllers use OpenFlow protocol for communicating with the switches (forwarding devices). OpenFlow is not a vendor-specific protocol which infers that the controller can communicate with every switch, irrespective of the vendors.
</li>
  
<li><b>SDN Switches:</b> The SDN switches are different from conventional switches and this is why it is better to refer SDN switches as forwarding devices because SDN switches are equipped with only data planes. SDN switches can either be in hardware form or software. Open vSwitch (OVS) is the more popular virtual switch that is used in the SDN paradigm for connecting end devices.
</li>
</ul>

---

### 6. Implementation

#### 6.1. Setting Up Environment

Install Ubuntu on the base machine or in a virtual environment. For creating and managing virtual machines, I prefer to use Qemu/KVM and virt-manager application. Other virtualization alternatives like VirtualBox and VMware can also be used for the same purpose. I am using Ubuntu 18.04 at the time of writing this tutorial.

#### 6.2. Installation of Dependencies

Run the following two commands before proceeding to other commands:

```sh
$ sudo apt update
$ sudo apt upgrade
```

Installation of prerequisites.

```sh
$ sudo apt-get install git gcc python-dev libffi-dev libssl-dev libxml2-dev libxslt1-dev zlib1g-dev python-pip
```

Installation of Ryu.

```sh
$ sudo pip install ryu
$ ryu-manager --version
```

<i class="far fa-sticky-note"></i> **Troubleshooting Error:** It is highly probable that `ryu-manager` command can render an error message: `SyntaxError: invalid syntax`. This error is caused by the upgraded version of tinyrpc that is not compatible with Python 2. In order to resolve this error, uninstall the current version of tinyrpc using the command `pip uninstall tinyrpc` and install the lower version using `pip install tinyrpc==0.8`.
{: .notice--info}
{: .text-justify}

Installation of Open vSwitch.

```sh
$ sudo apt-get install openvswitch-switch
$ ovs-vsctl --version
```

Installation of Wireshark.

```sh
$ sudo apt-get install wireshark
$ wireshark --version
```

Installation of Mininet.

```sh
$ sudo apt-get install mininet
$ mn --version
```

<i class="far fa-sticky-note"></i> **Tip:** Mininet can also be installed by cloning its [Github Repository](https://github.com/mininet/mininet), but using `apt-get` package for installation is more convenient.
{: .notice--info}
{: .text-justify}

#### 6.3. Running Simple SDN Application

Open Wireshark utility and start capturing packets. Open three terminals and run respective commands in these terminals as mentioned below.

##### Terminal 1:

```sh
$ sudo mn --topo=single,3 --controller=remote,ip=127.0.0.1 --mac --switch=ovsk,protocols=OpenFlow13
```

The command in Terminal 1 creates a simple topology in mininet. The topology consists of a single switch and three hosts that are connected to the switch. The topology can be tested by using `pingall` command on the same terminal. At this point, the hosts are unreachable. For making the hosts communicate with each other, there must be some rules which are implemented by the controller.

##### Terminal 2:

```sh
$ ryu-manager ryu.app.simple_switch_13
```

The command in Terminal 2 runs a simple application of switch. The program file for `simple_switch_13` locates in `/usr/lib/python2.7/ryu/ryu/app/`directory. The `simple_switch_13` program sets rules and policies in ryu controller, which are necessary for routing packets in a fashion similar to that of L2 Switch of conventional networking. Switch to Terminal 1 and run `pingall` command again. The hosts will be able to communicate with each other this time. A simple SDN infrastructure has been implemented at this point!

##### Terminal 3:

```sh
$ sudo ovs-vsctl show
$ sudo ovs-ofctl -O OpenFlow13 dump-flows s1
```

The commands in Terminal 3 show the brief configuration and flow table of Open vSwitch. These commands are used for inspecting the behavior and statistics of SDN switches upon receiving packets from the hosts.

Stop capturing packets on Wireshark utility. Type `openflow_v4` in Wireshark Filter for filtering out OpenFlow (version 4) packets.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/p2_wireshark.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

---

### 7. Conclusion

In this tutorial, I tried my best to provide readers with the necessary knowledge that is required for deploying SDN infrastructure using mininet and ryu controller. The tutorial was written to be brief and this is why the excessive details are not mentioned in the tutorial. [Mininet Walkthrough](http://mininet.org/walkthrough/) already provides rich resources for developing topologies and performing other relevant operations in mininet. The explanation of working modules and ryu applications is covered in my next blog on this site. In case of any issue or providing your feedback, please drop a comment here or connect with me via [email](mailto:smfarjad@outlook.com).

---

### 8. References

- [Mininet](http://mininet.org/)
- [Ryu SDN Framework](https://ryu-sdn.org/)
- [Open vSwitch](https://www.openvswitch.org/)
- [Wireshark](https://www.wireshark.org/)
