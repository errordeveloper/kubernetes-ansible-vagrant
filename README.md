# Kubernetes Playground
This project contains a `Vagrantfile` and associated `Ansible` playbook scripts
to provisioning a 3 nodes Kubernetes cluster using `VirtualBox` and `Ubuntu
16.04`.

### Prerequisites
You need the following installed to use this playground.
- `Vagrant`, version 1.8.6 or better. Earlier versions of vagrant may not work
with the Vagrant Ubuntu 16.04 box and network configuration.
- `VirtualBox`, tested with Version 5.0.26 r108824
- Internet access, this playground pulls Vagrant boxes from the Internet as well
as installs Ubuntu application packages from the Internet.

### Bringing Up The cluster
To bring up the cluster, clone this repository to a working directory.

```
git clone http://github.com/davidkbainbridge/k8s-playground
```

Change into the working directory and `vagrant up`

```
cd k8s-playground
vagrant up
```

Vagrant will start three machines. Each machine will have a NAT-ed network
interface, through which it can access the Internet, and a `private-network`
interface in the subnet 172.42.42.0/24. The private network is used for
intra-cluster communication.

The machines created are:

| NAME | IP ADDRESS | ROLE |
| --- | --- | --- |
| k8s1 | 172.42.42.1 | Cluster Master |
| k8s2 | 172.42.42.2 | Cluster Worker |
| k8s3 | 172.42.42.3 | Cluster Worker |

As the cluster brought up the cluster master (**k8s1**) will perform a `kubeadm
init` and the cluster workers will perform a `kubeadmin join`. This cluster is
using a static Kubernetes cluster token.

After the `vagrant up` is complete, the following command and output should be
visible on the cluster master (**k8s1**).

```
vagrant ssh k8s1
kubectl -n kube-system get po -o wide

NAME                             READY     STATUS              RESTARTS   AGE       IP          NODE
etcd-k8s1                        1/1       Running             0          7m        10.0.2.15   k8s1
kube-apiserver-k8s1              1/1       Running             2          8m        10.0.2.15   k8s1
kube-controller-manager-k8s1     1/1       Running             0          8m        10.0.2.15   k8s1
kube-discovery-982812725-rmkh3   1/1       Running             0          8m        10.0.2.15   k8s1
kube-dns-2247936740-26md1        0/3       ContainerCreating   0          7m        <none>      k8s1
kube-proxy-amd64-466ta           1/1       Running             0          7m        10.0.2.15   k8s1
kube-proxy-amd64-rsgfr           1/1       Running             0          2m        10.0.2.15   k8s3
kube-proxy-amd64-u333v           1/1       Running             0          4m        10.0.2.15   k8s2
kube-scheduler-k8s1              1/1       Running             0          8m        10.0.2.15   k8s1
```

***NOTE: I am a little concerned that all the IPs are showing up as 10.0.2.15.***

### Starting Networking
Stating the clustering networking is **NOT** automated and must be completed
after the `vagrant up` is complete. A script to start the networking is
installed on the cluster master (**k8s1**) as `/usr/local/bin/start-weave`.

```
vagrant ssh k8s1
sudo start-weave

daemonset "weave-net" created
```

### Clean Up
On each vagrant machine is installed a utility as `/usr/local/bin/clean-k8s`.
executing this script as `sudo` will reset the servers back to a point where
you can execute vagrant provisioning.
