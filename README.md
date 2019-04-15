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
```
$ chmod +x install-tools.sh
$ ./install-openshift.sh
```

You can also copy *install-tools.sh* from one node to another one.
```
scp -i keypair.pem install-tools.sh  root@ip_host:~/
```

This script file installs all the pre-requisites required to setup Openshift 3.11 on CentOS. Ensure that you are logged in with root user.

# Basic Commands
To ssh from one node to another using AWS .pem file
```
  scp -i keypair.pem install-tools.sh  centos@172.31.30.195:~/

```
sudo -s
mv install-tools.sh ~/
cd
./install-tools.sh


172.31.28.144
ssh -i Openshift-keypair.pem centos@172.31.28.144
scp -i Openshift-keypair.pem install-tools.sh  root@172.31.28.144:~/
sudo -s
mv install-tools.sh ~/
cd
./install-tools.sh


172.31.29.41

scp -i Openshift-keypair.pem install-tools.sh  centos@172.31.29.41:~/
ssh -i Openshift-keypair.pem centos@172.31.29.41
sudo -s
mv install-tools.sh ~/
cd
./install-tools.sh

172.31.25.239

scp -i Openshift-keypair.pem install-tools.sh  centos@172.31.25.239:~/
ssh -i Openshift-keypair.pem centos@172.31.25.239
sudo -s
mv install-tools.sh ~/
cd
./install-tools.sh



Master node steps
=====================
- Install nano editor









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
