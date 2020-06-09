# KubeKey

Since v3.0, KubeSphere changes the ansible-based installer to the new installer called KubeKey that is developed in Go language. With KubeKey, you can install Kubernetes and KubeSphere separately or as a whole easily, efficiently and flexibly.

There are three scenarios to use KubeKey.

* Install Kubernetes only
* Install Kubernetes and KubeSphere together in one command
* Install Kubernetes first, then deploy KubeSphere on it using [ks-installer](https://github.com/kubesphere/ks-installer)

## Motivation

* Ansible-based installer has a bunch of software dependency such as Python. KubeKey is developed in Go language to get rid of the problem in a variety of environment so that increasing the success rate of installation.
* KubeKey uses Kubeadm to install K8s cluster on nodes in parallel as much as possible in order to reduce installation complexity and improve efficiency. It will greatly save installation time compared to the older installer.
* KubeKey supports for scaling cluster from allinone to multi-node cluster.
* KubeKey aims to install cluster as an object, i.e., CaaO.

## Supported 
### Linux Distributions

* **Ubuntu**  *16.04, 18.04*
* **Debian**  *Buster, Stretch*
* **CentOS/RHEL**  *7*

### Kubernetes Versions

* **v1.15**: &ensp; *v1.15.12*
* **v1.16**: &ensp; *v1.16.10*
* **v1.17**: &ensp; *v1.17.6* (default)
* **v1.18**: &ensp; *v1.18.3*

## Getting Started

### Requirements and Recommendations

* Minimum resource requirements (For Minimal Installation of KubeSphere only)：
  * 2 vCPUs
  * 4 GB RAM
  * 20 GB Storage
                                                                                                                       
  > /var/lib/docker is mainly used to store the container data, and will gradually increase in size during use and operation. In the case of a production environment, it is recommended that /var/lib/docker mounts a drive separately.

* OS requirements:
  * `SSH` can access to all nodes.
  * Time synchronization for all nodes.
  *  `sudo`/`curl`/`openssl` can be used in all nodes.
  * `ebtables`/`socat`/`ipset`/`conntrack` should be installed in all nodes.
  * The [relevant client](./docs/storage-client.md) should be installed in all nodes, if NFS / Ceph / GlusterFS is used as persistent storage.
  * `docker` can be installed by yourself or by KubeKey. 
  
  > It's recommended that Your OS is clean (without any other software installed), otherwise there may be conflicts.  
  > A container image mirror (accelerator) is recommended to be prepared, if you have trouble downloading images from dockerhub.io.

* Networking and DNS requirements:
  * Make sure the DNS address in /etc/resolv.conf is available. Otherwise, it may cause some issues of DNS in cluster.
  * If your network configuration uses Firewall or Security Group，you must ensure infrastructure components can communicate with each other through specific ports. It's recommended that you turn off the firewall or follow the link configuriation: [NetworkAccess](./docs/network-access.md).


### Usage

#### Get the Installer Excutable File

* Download Binary

```shell script
curl -O -k https://kubernetes.pek3b.qingstor.com/tools/kubekey/kk
chmod +x kk
```

or

* Build Binary from Source Code

```shell script
git clone https://github.com/kubesphere/kubekey.git
cd kubekey
./build.sh
```

> Note:
>
> * Docker needs to be installed before building.
> * If you have problem to access `https://proxy.golang.org/` in China mainland, excute `build.sh -p` instead.

#### Create a Cluster

##### allinone

* `./kk create cluster` will use the default file `config.yaml` under the current folder.
* If the file `config.yaml` does not exist, the command will create an allinone environemt on the machine itself. In addition, `./kk create cluster` will create Kubernetes only, while `./kk create cluster --all` will create Kubernetes and KubeSphere. Please using `./kk version` to check the built-in versions of them.
* You also can specify the file that could be a different filename, or in different folder, or even from a remote repo (This feature currently is not supported).
  * ./kk create cluster -f ~/myfolder/abc.yaml
  * ./kk create cluster -f <https://github.com/kubesphere/kubekey/docs/config-example.md> # Will supported in next version

```shell script
$ ./kk create cluster --all
```

##### multi-node

* Create an example configuration file by the following command. A file named `config-sample.yaml` with default storage class local volume will be created. You also can specify the file name by adding `-f filename.yaml`
* You also can specify the storage class (localVolume, nfsClient, rbd, glusterfs) by adding --add as follows.
  * ./kk create config --add localVolume
* The default storage class is the first one you add via command line into the config file which you can modify through editor.
* KubeSphere's configuration can be generated by adding `--add` as follows.
  * ./kk create config --add localVolume,kubesphere
  > Notice: If you want to deploy KubeSphere, you must specify a storage class, and make sure the storage server is available. 
                                                                                                                       
```shell script
$ ./kk create config

# Now modify the file config-sample.yaml according to your environment.

# create cluster. If the KubeSphere info is in the config file, the following command will create a cluster with Kubernetes and KubeSphere.
$ ./kk create cluster -f config-sample.yaml
```

#### Add Nodes

Add new node's information to the cluster config file, then apply the changes.

```shell script
$ ./kk scale -f config-sample.yaml
```

#### Delete Cluster

You can delete the cluster by the following command

```shell script
$ ./kk delete [-f config-sample.yaml]
```
#### Enable kubectl autocompletion

KubeKey doesn't enable kubectl autocompletion. Refer to the guide below and turn it on:

**Prerequisite**: make sure bash-autocompletion is installed and works.
In case that the OS is Linux based:

```shell script
# Install bash-completion
$ apt-get install bash-completion

# Source the completion script in your ~/.bashrc file
$ echo 'source <(kubectl completion bash)' >>~/.bashrc

# Add the completion script to the /etc/bash_completion.d directory
$ kubectl completion bash >/etc/bash_completion.d/kubectl
```

More detail reference could be found [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/#enabling-shell-autocompletion)

## Documents
* [Configuration example](docs/config-example.md)
* [Network access](docs/network-access.md)
* [Storage clients](docs/storage-client.md)
* [Roadmap](docs/roadmap.md)
