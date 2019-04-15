172.31.30.54
172.31.30.195
scp -i Openshift-keypair.pem install-tools.sh  centos@172.31.30.195:~/
ssh -i Openshift-keypair.pem centos@172.31.30.195

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
- Enable root acccess in /etc/ssh/sshd_config file. Uncomment below setting.
	#PermitRootLogin yes
	sudo vi /etc/ssh/sshd_config
	
- Restart sshd service.
	sudo systemctl restart sshd
	sudo -s
	cp -r /home/centos/.ssh/authorized_keys /root/.ssh/authorized_keys
	
- Swith to root user with "sudo -s" command
- Copy authorization_key of centos user to root's user.
	$ cp /home/centos/.ssh/authorized_keys /root/.ssh/authorized_keys
- Now you can login with root user.
- Once logged on terminal, run install-tools.sh file. In case there is error related to bad characters or similar, run below command before running the script. Don't forget to give executable permission first, if it's already not given.
	chmod +x install install-tools.sh
	sed -i -e 's/\r$//' install-tools.sh
- Now copy install-tools.sh from one master node to another one.
	scp -i Openshift-keypair.pem install-tools.sh  centos@172.31.17.55:~/

[ ! -d openshift-ansible ] && git clone https://github.com/openshift/openshift-ansible.git -b release-3.11 --depth=1

Once you have install all the tools successfully, now you can start Openshift installation with the following commands:
	ansible-playbook -i inventory.ini openshift-ansible/playbooks/prerequisites.yml
	ansible-playbook -i inventory.ini openshift-ansible/playbooks/deploy_cluster.yml

htpasswd -b /etc/origin/master/htpasswd admin admin123
oc adm policy add-cluster-role-to-user cluster-admin admin