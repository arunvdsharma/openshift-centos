# Install Openshift 3.11 on CentOS
Provision 'n' number of nodes. In this example, I created 3 CentOS nodes on AWS. You need to make necessary changes to be able to login on CentOS with 'root' user account. 

**Note: This is not recommended in production environment

Enable root acccess in /etc/ssh/sshd_config file. Uncomment below setting in the file. Login with *centos* user and run these commands.
```
$ sudo vi /etc/ssh/sshd_config
	PermitRootLogin yes    	#Uncomment this line in /etc/ssh/sshd_config and save it.
	 
$ sudo systemctl restart sshd
```

By default IAM doesn't allow to login with *root* user on EC2 instances but you can use the authroized_keys of *centos* user to access it.

```
$ sudo -s
$ cp /home/centos/.ssh/authorized_keys /root/.ssh/authorized_keys
```

Now you are ready to login with *root* user with the same private keys which you have used for *centos* user.

### Perform above steps on each nodes which you are planning to add in Openshit cluster.

Once root user is accessible on nodes, run install-tools.sh file.

```
$ ./install-tools.sh
```

### Note: In case there is error related to bad characters or similar, run below command before running the script. If not, you can skip this step.

```
$ sed -i -e 's/\r$//' install-tools.sh
```

Don't forget to give executable permission first, if it's already not given. 
*install-tools.sh* file installs all the pre-requisites required to setup Openshift 3.11 on CentOS. Ensure that you are logged in with root user.
```
$ chmod +x install-tools.sh
$ ./install-tools.sh
```

Copy *install-tools.sh* from one node to another one so you can run install pre-requisites on every nodes. Replace *host_ip* with you node's ip address or hostname.

```
#To ssh from one node to another using AWS .pem file
$ scp -i keypair.pem install-tools.sh  root@host_ip:~/
$ ssh -i Openshift-keypair.pem root@host_ip
$ chmod +x install-tools.sh
$ ./install-tools.sh
```

### Once all the tools are installed, you need open some of the ports on AWS network. These ports are required Openshift cluster to communicate with each node.

22 			TCP 
53 or 8053		TCP/UDP
80 or 443		TCP
1936			TCP
4001			TCP
2379 and 2380		TCP
4789			UDP
8443			TCP
10250			TCP

To get more details understanding on ports for Openshift, refer this link https://docs.openshift.com/container-platform/3.11/install/prerequisites.html

### Now you are all set to install Openshift. Run the following commands on master node:
```
$ chmod +x install-openshift.sh
$ ./install-openshift.sh
```





# How To Use NFS Persistent Volumes
The purpose of this guide is to create Persistent Volumes with NFS. It is part of [OpenShift persistent storage guide](../README.md), which explains how to use these Persistent Volumes as data storage for applications.

## NFS Provisioning

We'll be creating NFS exports on the local machine.  The instructions below are for Fedora.  The provisioning process may be slightly different based on linux distribution or the type of NFS server being used.

Create two NFS exports, each of which will become a Persistent Volume in the cluster.

```
# the directories in this example can grow unbounded
# use disk partitions of specific sizes to enforce storage quotas
mkdir -p /home/data/pv0001
mkdir -p /home/data/pv0002

# security needs to be permissive currently, but the export will soon be restricted 
# to the same UID/GID that wrote the data
chmod -R 777 /home/data/

# Add to /etc/exports
/home/data/pv0001 *(rw,sync)
/home/data/pv0002 *(rw,sync)

# Enable the new exports without bouncing the NFS service
exportfs -a

```

## Security

### SELinux

By default, SELinux does not allow writing from a pod to a remote NFS server. The NFS volume mounts correctly, but is read-only.

To enable writing in SELinux on each node:

```
# -P makes the bool persistent between reboots.
$ setsebool -P virt_use_nfs 1
```

## NFS Persistent Volumes

Each NFS export becomes its own Persistent Volume in the cluster.

```
# Create the persistent volumes for NFS.
$ oc create -f examples/wordpress/nfs/pv-1.yaml
$ oc create -f examples/wordpress/nfs/pv-2.yaml
$ oc get pv

NAME      LABELS    CAPACITY     ACCESSMODES   STATUS      CLAIM     REASON
pv0001    <none>    1073741824   RWO,RWX       Available             
pv0002    <none>    5368709120   RWO           Available             

```

Now the volumes are ready to be used by applications in the cluster.

#Reference: https://github.com/openshift/origin/tree/v3.6.0/examples/wordpress 
