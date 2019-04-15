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

Once all the tools are installed, you need open some of the ports on AWS network. These ports are required Openshift cluster to communicate with each node.
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

I have created multiple **inventory-*.ini** files in my git repository. You can choose one, customize it and rename it with **inventory.ini**. The same inventory file will be used to install Openshift cluster.

If you have different **domain name**, you can replace it. Make sure that your domain name is a valid domain which could be used to access Openshift Dashboard.

Once all the changes are done, you are all set to install Openshift. Run the following commands on master node:

```
$ chmod +x install-openshift.sh
$ ./install-openshift.sh
```

If all the ansible jobs are run successfully, you can access Openshift dashboard on https://console.<<your_domain_name>> 
