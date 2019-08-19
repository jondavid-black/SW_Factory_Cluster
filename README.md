# Kube-Cluster

This project builds up the infrastructure for our SW Factory.  We are using 3 Atomic Pi single board computers.  

## Manual Setup
We'll be using as little manual setup and configuration as possible.  Here are the manual steps performed on each Atomic Pi.

1. Install CentOS 7 Minimal from a USB stick.
2. During the installation process, enable ethernet (DHCP), allocate all disk space to the installation, set the language to English, set the keyboard layout to US, set the root password, and create a non-root user as an administrator (in the wheel group to allow sudo).
3. Set the hostname by editing /etc/hostname and rebooting.  I'm using api1, api2, and api3.
4. Run yum update and reboot.
5. Setup ssh keys from my laptop for my user and root using ssh-copy-id user@hostname on each machine.  Ensure the ssh host keys are in the local keystore.
6. Install ansible on my laptop.

## Automated Setup
The rest of the setup is automated using Ansible Infrastructure-as-Code.  Most of the content was taken from [this Digital Ocean post](https://www.digitalocean.com/community/tutorials/how-to-create-a-kubernetes-cluster-using-kubeadm-on-centos-7).

1. Create a hosts file with 1 master and 2 nodes.  In this setup, I'm using api1 as the master with api2 and api3 as nodes.  Because I'm using DHCP within my home network, I used the hostnames in the hosts file rather than IP addresses.
2. Install kubernetes dependencies using the kube-dependencies.yml playbook taken from Digital Ocean's post.  I then rebooted all the Atomic Pis.
3. Setup a centos user on each Atomic Pi using the centos-user.yml playbook.
4. Setup the master node using the master.yml playbook taken from Digital Ocean's post.

Note:  The master.yml playbook failed for me.  I learned that kubernetes does not work with an active swap file.  I have updated the kube-dependencies.yml playbook to disable the swap file before installing kubernetes.  I continued to have issues runnign the master.yml playbook.  I ssh'ed into api1 as root and ran kubeadm reset.  I than ran kubeadm init manually...which worked.  I was then able to run the master.yml playbook successfully.  The key seemed to be running kubeadm reset.

5. Setup the worker nodes using the worker.yml playbook taken from Digital Ocean's post.

## Verify the Cluster
Now that we have installed docker and kubernetes, setup our nodes, initialized our cluster, and joined workers, we need to verify the cluster status.

1. ssh into api1 as root
2. Switch to the centos user (su - centos)
3. Run kubectl get nodes

We should now see a table showing the master and both workers with a status of Ready.
