+++
date = "2017-10-23T13:17:29-05:00"
title = "Python Development Environment"
draft = false

tags = ["Python","DevOps","Vagrant","Ansible","Python"]

categories = ["Development"]

+++

# Python Development Environment

I set out to create an extensible, portable python development that would be consistent if I moved it between operating environments and architectures. When I first set out on this project, I wanted to use [Docker](https://www.docker.com/) to deploy a dev environment on top of a Debian based system. I didn't realize this when I started, nor did Google seem to know that Docker isn't meant for a dev environment, only for pushing about apps after they're made. Docker is strictly a deployment tool, so using it as a dev environment is a no-go in case you're as confused as I was.

The repo for the two files I made are here: https://github.com/asnair/OtherScripts

The specific files are the [Vagrantfile](https://github.com/asnair/OtherScripts/blob/master/Vagrantfile) and the [setup.sh](https://github.com/asnair/OtherScripts/blob/master/setup.sh) files.

## Setup

So after ruling out Docker, I decided [Vagrant](https://www.vagrantup.com/) might be the way to go with a full Debian dev environment. Vagrant deploys Virtual Machines through a provider such as [Virtualbox](https://www.virtualbox.org/wiki/Downloads/) or VMware (Note Fusion or Workstation must be used) instead of containers as Docker does. This is often paired with a provisioning tool such as [Ansible](https://www.ansible.com/) or [Chef](https://www.chef.io/chef/) that let you mass deploy VMs. This is often also deployed as the framework behind container deployments. In a sense, vagrant could be thought of helping bridge the gap between IT infrastructure and DevOps, although this is purely for thinking about how tools relate to each other and not how this would work in practice, as it's still a DevOps activity.

Once I figured out I wanted to use Vagrant, I installed the application on my Ubuntu laptop and set up a vagrant environment using `vagrant init`.

The Vagrantfile I use (vagrant's config file per instance) and shell set up I use are on Github. The important part of the Vagrantfile is below:

```python
config.vm.box = "debian/jessie64"

config.vm.provision :shell, :privileged => true, :path => "setup.sh"

config.vm.network "forwarded_port", guest: 5000, host: 5000
```

 You can find a list of VMs to use instead of debian on [vagrant's box search](https://app.vagrantup.com/boxes/search). Note vagrant terminology for a VM is a *box*.

In the setup.sh file, you can set up your environment using bash and install any dependencies or packages you need. For the forwarded ports, the guest port is the port exposed on the guest you would configure your app to use, and the host port is the port you would connect to on your host to access the internal app from the host machine.

**The setup.sh file by default installs git, tmux, vim, build-essential, apache2, and sets up a usable MySQL DB with username and password. Look at the code for everything that's done specifically.**

## Using the Environment

If you have both the Vagrantfile and the setup.sh file in the same directory, you can run `vagrant up` to begin building the box. It will automatically download the VM and configure it. This means wherever someone has vagrant installed, they can use your Vagrantfile and script to make an environment exactly as yours. This is one of the main goals of the portable dev environment.

Once the environment is done building, you should be able to `vagrant ssh` into the box and be presented with the shell and environment. In a future update a git repo should automatically be downloaded, for now however, you have to download a repo manually using `git clone URL`.

When you are done developing in your environment, you can exit the shell by typing `exit` until you get to outside the box. Once outside the box, you can type `vagrant destroy` in the current directory, or supply the ID of a specific instance using `vagrant global-status` then `vagrant destroy ID`. If you just type `vagrant`, you can see all commands available. These scripts will get periodic updates in the Github repo, so keep an eye out for them.

Thanks for reading.

![An Idea](https://gooddebate.org/images/anidea.png)