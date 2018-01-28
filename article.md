# Configuring multi-host Vagrant

With these upcoming series I want to provide overview on how to create multi-host local setup for writing configurations and creating/testing automation scripts on local workstation.
N.B. I'm using Linux Lubuntu 16.04.3 LTS distribution as target platform.

Requirements for host machine:
- [Vagrant](https://www.vagrantup.com/downloads.html)
- [VirtualBox](https://www.virtualbox.org/wiki/Linux_Downloads)

> Versions that I'm using:
> Vagrant: 2.0.1
> VirtualBox: 5.2.4 r119785

Our main working file for this section will be Vagrantfile where we will define three boxes:
- Controller - will launch provisioning, e.g., ansible;
- Proxy - will serve as proxy to proxy all incoming http requests to their targets;
- Host - will host our services and applications, e.g., jenkins and nodejs apps.

After we have installed requirements for host machine lets proceed with:
```bash
~/project-folder$ vagrant init
```
This will create generic Vagrantfile. Let's remove all unnecessary stuff so that our file looks like this:
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  # Place for our Vagrant box definitions.

end
```

Before adding we need to define few requirements for our hosts:
- Hosts should be able to communicate;
	- ssh keys should be distributed across hosts.
- We should be able to reach hosts by hostnames;
	- mDNS should be installed and configured.
- Controller should have provisioning tools installed (Ansible in our case).

Lets proceed by adding our box definitions:
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.define "controller" do |controller|
    controller.vm.box = "minimal/xenial64"

    controller.vm.hostname = "controller"
    controller.vm.network "private_network", type: "dhcp"

    controller.vm.provider "virtualbox" do |vb|
      vb.memory = "512" # For now lets keep low memory just for tests.
    end
  end

  config.vm.define "proxy" do |proxy|
    proxy.vm.box = "centos/7"

    proxy.vm.network "private_network", type: "dhcp"
    proxy.vm.hostname = "proxy"

    proxy.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
    end
  end

  config.vm.define "host" do |host|
    host.vm.box = "centos/7"

    host.vm.network "private_network", type: "dhcp"
    host.vm.hostname = "host"

    host.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
    end
  end

end
```

As we have written Vagrantfile we should now be able to launch vagrant boxes and ping other hosts:
```bash
~/project-folder$ vagrant up && vagrant ssh controller
~vagrant$ ping -c 2 proxy.local && ping -c 2 host.local
```
