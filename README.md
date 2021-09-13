# Kubernetes Cluster using Ansible

- Clone repository.

- Create multiple centos7 servers. One master and many worker to connect to cluster.

- Change the “ad_addr” in the env_variables file with the IP address of the Kubernetes master node.

Note: In this case deploy with using user "root". You can use any another user with small modify in hosts file at "ansible_user={{your user}}" but let use privilege sudo


# Prerequisites

- 2 CentOS7 servers (you can add as many servers as you want). 

- Ansible installed & configured to SSH to the remote servers without a password.

- A sudo user configured on all the Kubernetes nodes.

# Step the follow

- This is tree directory Playbook Ansible that we will deploy

root@LDCC-VN:~/kubernetes-ansible# tree
.
├── README.md
└── centos
    ├── ansible.cfg
    ├── hosts
    ├── playbooks
    │   ├── configure_master_node.yml
    │   ├── configure_worker_nodes.yml
    │   ├── env_variables
    │   ├── join_token
    │   ├── prerequisites.yml
    │   └── setting_up_nodes.yml
    ├── setup_master_node.yml
    └── setup_worker_nodes.yml
    
## Step 1: Test the connection from the Ansible node

- We will add 2 groups in the Ansible inventoty: 1 for the master node and another worker node. This way we can define which Ansible task should be executed on a certain node. For this, add the line similar ~/kubenetes-ansible/centos/hosts   

[kubernetes_master_nodes]
kubernetes-master ansible_host=10.69.20.45 ansible_user=root
[kubernetes_worker_nodes]
kubernetes-worker1 ansible_host=10.69.20.40 ansible_user=root

Run command  the below:

[root@ansible centos]# ansible -m ping all
kubernetes-worker1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
kubernetes-master | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}

## Step 2: Run the following command to setup the Kubernetes Master node.

    [root@ansible centos]# ansible-playbook setup_master_node.yml

we can see import configure in the setup_master_node.yml file.
     
     - import_playbook: playbooks/prerequisites.yml
     - import_playbook: playbooks/setting_up_nodes.yml
     - import_playbook: playbooks/configure_master_node.yml

Tasks in the playbook will:
  - `playbooks/prerequisites.yml` :
      Disbling Swap on all
      Commentting Swap entries in /etc/fstab 
      Disabling SElinux enforcement.
      Add IPs to /etc/hosts on master and worker node

  - `playbooks/setting_up_nodes.yml`  :
      Creating a repository details in Kubernetes repo file
      Installing Docker and firewalld
      Installing required packages
      Starting and Enabling the required serviecs
      Allow Network Ports in Firewalld service
      Enabling Bridge Firewall rule

  - `playbooks/configure_master_node.yml` :
     - Pulling images required for setting up a Kubernetes cluster
     - Resetting kubeadm
     - Initializing Kubernetes cluster
     - Storing Logs and Generated token for future purpose
     - Copying require files
     - Install Network Add-on Flannel
     In this case we're using Plannel. Flannel is a simple, lightweight layer 3 fabric for Kubernetes. Flannel manages an IPv4 network between multiple nodes in a cluster. It does not control how containers are networked to the host, only how the traffic is transported between hosts  

Resutl after run command success :

PLAY RECAP *****************************************************************************************************************************
kubernetes-master          : ok=21   changed=11   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
kubernetes-worker1         : ok=14   changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0


## Step 3 Once the master node is ready, run the following command to set up the worker nodes.

    [root@ansible centos]# ansible-playbook setup_worker_nodes.yml

we can see import configure in the setup_master_node.yml file.

     - import_playbook: playbooks/prerequisites.yml
     - import_playbook: playbooks/setting_up_nodes.yml
     - import_playbook: playbooks/configure_worker_nodes.yml

We also see two import playbook `playbooks/prerequisites.yml`, `playbooks/setting_up_nodes.yml` for purpose when scale Kubernetes cluster we only run setup_worker_nodes.yml to deployed join kubernetes cluster.

Task in the playbook/configure_worker_nodes.yml will :
     - Copying token to worker nodes
     - Joining woker nodes to with kubernetes cluster

Result after run command success:

PLAY RECAP *****************************************************************************************************************************
kubernetes-master          : ok=14   changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
kubernetes-worker1         : ok=17   changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

Step 4: Once the workers have joined the cluster, run the following command to check the status of the worker nodes.

     [root@kubernetes-master ~]# kubectl get nodes

Result after run command success:

[root@kubernetes-master ~]# kubectl get nodes
NAME                 STATUS   ROLES                  AGE     VERSION
kubernetes-master    Ready    control-plane,master   10m     v1.22.1
kubernetes-worker1   Ready    <none>                 3m24s   v1.22.1


## Additional information
* The IP addresses of the workers and masters added to the /etc/hosts file on all workers and masters as part of the [prerequisites.yml](centos/playbooks/prerequisites.yml) playbook,
if necessary or in case of DNS issues make sure the addresses have been added to /etc/hosts file.
  
Thank you !! 

