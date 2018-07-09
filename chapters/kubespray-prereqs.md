# Prerequisites for Using Kubespray
## Hardward Requirements:
### Virtual Machines
   * Number of VMs: 4
   * Memory: 2 GB
   * CPU: 2 Cores
   * Disk: 50 GB

## Software Requirements on Control Node:
### Prerequisite 1: Install Python
Install Python Pip on your `Ansible Controller` machine
We don't need to install Python as it comes preinstalled on Ubuntu 16.04
  ```
  sudo apt-get update
  sudo apt-get -y upgrade
  sudo apt-get install -y python3-pip
  # Check the installation
  python3 -V
  pip3 -V
  ```

### Prerequisite 2: Set up Passwordless SSH
#### Generating ssh keypair on control host

Now on `Ansible Controller` host, execute the following command

```
ssh-keygen -t rsa
```

Now press enter for the passphrase and other queries.

```
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
c5:a5:6d:60:56:5a:7b:3c:60:23:b5:0f:1b:cf:f9:fd root@ansible
The key's randomart image is:
+--[ RSA 2048]----+
|          =oO    |
|         + X *   |
|          = B +  |
|         . . O o |
|        S   . =  |
|               ..|
|                o|
|                .|
|                E|
+-----------------+
```

#### Copying public key to inventory hosts

Copy public key of control node to other hosts

```
ssh-copy-id vagrant@10.10.1.3

ssh-copy-id vagrant@10.10.1.4

ssh-copy-id vagrant@10.10.1.5

ssh-copy-id vagrant@10.10.1.6
```

See this example output to verify with your output

```
The authenticity of host '10.10.1.3 (10.10.1.3)' can't be established.
RSA key fingerprint is 32:7f:ad:d7:da:63:32:b6:a9:ff:59:af:09:1e:56:22.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.10.1.3' (RSA) to the list of known hosts.
```

The password for user *vagrant* is *vagrant*

#### Validate the passwordless login

Let us check the connection of control node with other hosts

```
ssh vagrant@10.10.1.3

ssh vagrant@10.10.1.4

ssh vagrant@10.10.1.5

ssh vagrant@10.10.1.6
```


## Set up Environment for Kubespray
`On Ansible Controller`  
First clone the Kubespray repository and change directory.
  ```
  git clone https://github.com/kubernetes-incubator/kubespray.git
  cd kubespray
  ```
Install the python dependencies. **This step installs Ansible as well. You do not need to install Ansible separately**.
  ```
  sudo pip install -r requirements.txt
  ```
Copy the Sample Inventory.
  ```
  cp -rfp inventory/sample inventory/mycluster
  ```
Update Ansible inventory file with inventory builder. **Do not forget to replace these IPs with the IPs of your VMs**.
  ```
  declare -a IPS=(10.10.1.3 10.10.1.4 10.10.1.5 10.10.1.6)
  CONFIG_FILE=inventory/mycluster/hosts.ini python3 contrib/inventory_builder/inventory.py ${IPS[@]}
  ```
Review and Change Kubespray variables, *only if necessary*.
  ```
  cat inventory/mycluster/group_vars/all.yml
  cat inventory/mycluster/group_vars/k8s-cluster.yml
  ```
Deploy Kubespray with Ansible Playbook. This takes about 20-30 minutes depending on your infrastructure.
  ```
  ansible-playbook -i inventory/mycluster/hosts.ini cluster.yml
  ```