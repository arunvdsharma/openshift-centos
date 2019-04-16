# Ports required for Openshift Cluster

Open ports on the network which will be used to setup Openshift Cluster.
```
22 			TCP
53 or 8053		TCP/UDP
80 or 443		TCP
1936			TCP
4001			TCP
2379 and 2380		TCP
4789			UDP
8443			TCP
10250			TCP
```
To get more details understanding on ports for Openshift, refer this link https://docs.openshift.com/container-platform/3.11/install/prerequisites.html


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
#### Enable root configuration on each node which will be part of Openshift Cluster.

# Install git and checkout arunvdsharma/openshift-centos project on the *Master* node

```
$ yum install -y git
$ git clone https://github.com/arunvdsharma/openshift-centos.git
$ cd openshift-centos
```

### Note: If there is error of "bad characters" while running any script command, execute sed command to remove bad characters, for example:

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
#To ssh from one node to another using .pem file
$ scp -i keypair.pem install-tools.sh  root@host_ip:~/
$ ssh -i Openshift-keypair.pem root@host_ip
$ ./install-tools.sh
```


## Update DOMAIN_NAME in the inventory file

You may want to configure new **domain name**, you can replace it. In my case, domain name I used is **aruntechhub.xyz** which I have procured on a DNS provider. 

#### There are multiple inventory files as **inventory-*.ini** files in the repository. You can choose one, customize it according to your needs and rename it with **inventory.ini**. The same inventory file will be used to install Openshift cluster.

Replace your domain name with the following properties in inventory file
```
openshift_public_hostname=console.aruntechhub.xyz
openshift_master_default_subdomain=apps.aruntechhub.xyz
```

Once all the changes are done, you are all set to install Openshift. Run the following commands on master node:

```
$ chmod +x install-openshift.sh
$ ./install-openshift.sh
```

If all the ansible jobs are successful, you can access Openshift dashboard on https://console.your_domain_name
