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

What we have now are three vagrant boxes which we can launch. But before doing so we need to install mDNS package so that we can ping other hosts from private network using host names.
In order to make this provisioning step 'separate' from each box definition (as we will need mDNS installed on each box) lets put it as last block in our Vagrantfile right before last end statement.

```ruby
...
  config.vm.provision "shell", inline: <<-SHELL

    # N.B. More about configuring mDNS:
    #+ http://blog.uguu.waw.pl/2015/05/21/mdns-netbsd-linux-osx/

    DISTRO=$(cat /etc/os-release | head -1 | grep -o '".*"'  | tr -d '"')

    if [ "${DISTRO}" = "CentOS Linux" ]; then
      yum update -y --quiet
      yum install -y --quiet epel-release
      yum install -y --quiet avahi avahi-tools nss-mdns
      systemctl start avahi-daemon
      systemctl enable avahi-daemon
      service avahi-daemon start
    fi

    if [ "$DISTRO" = "Ubuntu" ]; then
      apt-get update --quiet
      apt-get install -y --quiet avahi-daemon avahi-utils libnss-mdns
    fi

  SHELL
end # Last end statement of our Vagrantfile.
```	


As we have written Vagrantfile we should now be able to launch vagrant boxes and ping other hosts:
```bash
~/project-folder$ vagrant up && vagrant ssh controller
~vagrant$ ping -c 2 proxy.local && ping -c 2 host.local
```
