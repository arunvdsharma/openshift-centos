# MySQL container with NFS volumes deployment on Openshift

This guide will help you to deploy MySQL container with NFS volumes on Openshift 3.11. It's assumed that you have NFS installed on one of your node.

## NFS Provisioning

You can have NFS installed externally on any node or within Openshift cluster. 
Before creating MySQL cluster, you need to provision NFS Volume. 

We'll be creating NFS exports on the master node where Openshift is installed. The provisioning process may be slightly different based on linux distribution or the type of NFS server being used.

Create two NFS exports, each of which will become a Persistent Volume in the cluster.

```
# use any disk partitions of specific sizes to enforce storage. Here I will be using /home/data folder
mkdir -p /home/data/mysqlpv

# Needs to give permission for Docker containers to write their data in these volumes
chmod -R 777 /home/data/

# Add to /etc/exports
/home/data/mysqlpv *(rw,sync)

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
$ oc create -f openshift-centos/mysqlconfig/pv.yaml
$ oc get pv

NAME       LABELS    CAPACITY     ACCESSMODES   STATUS      CLAIM     REASON
mysqlpv    <none>    2303741824   RWO,RWX       Available             

```

```
# Deploy MySQL Container claiming the same pvc
$ oc create -f openshift-centos/mysqlconfig/deployment.yaml
```

Now the volumes are ready to be used by applications in the cluster.

#Refer https://github.com/openshift/origin/tree/v3.6.0/examples/wordpress 
