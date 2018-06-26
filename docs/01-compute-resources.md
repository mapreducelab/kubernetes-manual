# Provisioning Compute Resources

### Install VirtualBox

VirtualBox will be used by Vagrant to create Virtual Machines.

- Download and install it from [here.](https://www.virtualbox.org/wiki/Downloads)

### Install Vagrant

Vagrant is a tool for building and managing virtual machine environments in a single workflow. With an easy-to-use workflow and focus on automation, Vagrant lowers development environment setup time, increases production parity, and makes the "works on my machine" excuse a relic of the past.

- Follow the instructions based on your OS [here.](https://www.vagrantup.com/docs/installation/)

### Install Vagrant Host Manager

Vagrant hostmanager is a Vagrant plugin that manages the hosts file on guest machines (and optionally the host). Its goal is to enable resolution of multi-machine environments deployed with a cloud provider where IP addresses are not known in advance.

```
vagrant plugin install vagrant-hostmanager
```

### Create your virtual machines

```
git clone https://github.com/mapreducelab/kubernetes-manual.git
cd kubernetes-manual/vagrant
vagrant up
```

Verify that you have three virtual machines.

```
vagrant status
```

> output

```
master01                  running (virtualbox)
node01                    running (virtualbox)
node02                    running (virtualbox)
```
