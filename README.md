# Deploy K8s Cluster

An step by step guide and automation tool (with Ansible playbooks) for deploying a Kubernetes cluster.

* You can either treat this repository as a guide to learn how to setup a kubernetes cluster,
* Or you can just let Ansible do the work and run these playbooks as instructed in the Usage section. 

## Prerequisites:
* x86_64 machines
* Debian based OS
* All machines are outside of sanctioned countries (because of unnecessary proxy complexities)

## Usage

First, install ansible (instructions provided at the bottom).

Then, edit the hosts in `inventory/hosts.yml` to point to your machines. User must have `sudo` privileges.

After that, you can run the playbooks in order.

**Step 0** - If you want to create a new sudoer user with your home ssh key (for the manual spectation later), run the `0-create-non-root-user.yml` playbook. [This is not needed in the Kubernetes installtion process but it is a convenient tool]:

```shell
$ ansible-playbook -i inventory/hosts 0-create-non-root-user.yml
```

**Step 1** - Install required packages and dependencies:
```shell
$ ansible-playbook -i inventory/hosts 1-install-dependencies.yml
```

**Step 2** - Install Docker:
```shell
$ ansible-playbook -i inventory/hosts 2-install-docker.yml
```

**Step 3** - Install cri-dockerd, which is needed in k8s v1.25.0 and later to enable docker engine as a runtime container (read more: [Don't Panic: Kubernetes and Docker](https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/)):
```shell
$ ansible-playbook -i inventory/hosts 3-install-cri-dockerd.yml
```

**Step 4** - Install Kubernetes packages, including `kubelet`, `kubeadm`, and `kubectl`:
```shell
$ ansible-playbook -i inventory/hosts 4-install-k8s-components.yml
```

**Step 5** - Setup kubernetes cluster in `control_plane` and join `workers` to that: (Change `using_vagrant` variable to `true` in this playbook if you are testing on Vagrant machines)
```shell
$ ansible-playbook -i inventory/hosts 5-setup-k8s-cluster.yml
```

**Step 5.1** - Destroy and clean up the Kubernetes cluster [if needed]:
```shell
$ ansible-playbook -i inventory/hosts 5.1-destroy-k8s-cluster.yml
```

# Testing
[Vagrant](https://www.vagrantup.com/) is a virtual machine configuration tool that makes it easy to test and deploy software, including the playbooks in this repository.

Just simply run this command:
```shell
$ vagrant up
```
And 4 virtual machines will be up and running to test these playbooks.

Hosts in `inventory/hosts.yml` are preconfigured to work with these VMs and you can just follow above playbooks, from step 1 to the end, and expect it to work as expected without any further configurations.

Just remember to change `using_vagrant` variable to `true` in `5-setuo-k8s-cluster.yml` playbook before executing it.

Other useful vagrant commands are:

Get the status of vagrant machines:
```shell
$ vagrant status
```

SSH into any of the machines:
```shell
$ vagrant ssh <vm-name>
$ vagrant ssh k8s-c-1 # First and only controller
$ vagrant ssh k8s-w-1 # First worker
$ vagrant ssh k8s-w-2 # Second worker
$ vagrant ssh k8s-w-3 # Third worker
```

Shutdown the VMs:
```shell
$ vagrant halt
```

Delete the VMs:
```shell
$ vagrant destroy
```

# Installing Ansible and Collections
1. Install Ansible:
```shell
$ sudo apt update
$ sudo apt install software-properties-common python3 python3-pip
$ sudo add-apt-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible
```

2. Install the required collections:
```shell
$ ansible-galaxy collection install ansible.posix
$ ansible-galaxy collection install community.general
```

# Acknowledgment

Heavily inspired by:
- [How To Create a Kubernetes Cluster Using Kubeadm on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-create-a-kubernetes-cluster-using-kubeadm-on-ubuntu-20-04)
- [kubeadm-ansible GitHub repository](https://github.com/kairen/kubeadm-ansible)
