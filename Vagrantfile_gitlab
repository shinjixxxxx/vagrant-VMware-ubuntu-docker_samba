# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
#  config.vm.box = "ubuntu"
#  config.vm.box = "ubuntu/trusty64"
#  config.vm.box = "ubuntu/xenial64"
  config.vm.box = "ubuntu/bionic64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
  # config.vm.network "forwarded_port", guest: 8080, host: 8080

  # webpack-dev-server のポートをフォワード provate networkと共存可能
  config.vm.network "forwarded_port", guest: 8082, host: 8082
  # 以下のsamba共有はなぜかうまくいかない
  # config.vm.network "forwarded_port", guest: 139, host: 139
  # config.vm.network "forwarded_port", guest: 445, host: 445

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.

  config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
# config.vm.synced_folder "../data", "/vagrant_data"
  config.vm.synced_folder "../", "/var/www/html"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.


  #########################################################
  # プロビジョニング
  #########################################################
  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.

config.vm.provision "shell", inline: <<-SHELL
  # 参考:
  # https://qastack.jp/server/500764/dpkg-reconfigure-unable-to-re-open-stdin-no-file-or-directory
  # https://docs.openstack.org/liberty/ja/install-guide-ubuntu/debconf/debconf-concepts.html
  # 設定のインストラクションを非対話モードに設定

  export DEBIAN_FRONTEND=noninteractive

  ###################
  # apt-get update
  # apache2 php emacs 
  # インストール
  ###################

  apt-get update
  
  apt-get install -y apache2
  apt-get install -y php
  apt-get install -y emacs

  ###################
  # docker
  ###################
  # Install Docker Engine on Ubuntu
  # https://docs.docker.com/engine/install/ubuntu/

  # SET UP THE REPOSITORY
  # 1
  apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

  # 2
  ### 
  # 参考:
  # https://qiita.com/jacob_327/items/e99ca1cf8167d4c1486d
  export APT_KEY_DONT_WARN_ON_DANGEROUS_USAGE=1

  curl -fsSL https://download.docker.com/linux/ubuntu/gpg > /tmp/___ubuntu_shinji_docker_pkg
  apt-key add /tmp/___ubuntu_shinji_docker_pkg

  # 3
  add-apt-repository  -y \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

  # INSTALL DOCKER ENGINE
  apt-get update -y
  apt-get install -y docker-ce docker-ce-cli containerd.io

  mkdir /dockerd
  chmod 777 /dockerd
  echo 'sudo docker rm -f samba ; sudo docker run -it --name samba -p 139:139 -p 445:445 -v /dockerd:/dockerd -d dperson/samba -s "dockerd;/dockerd;yes;no"' > /dockerd/docker_samba.sh
  chmod +x /dockerd/docker_samba.sh
  
  ###################
  # docker samba 起動設定
  ###################

  echo '#!/bin/sh -e\n' > /etc/rc.local
  echo 'sudo /dockerd/docker_samba.sh\n' >> /etc/rc.local
  chmod +x /etc/rc.local
  /etc/rc.local

  ###################
  # apache2設定
  ###################
  
  echo '\nListen 8080\n' >> /etc/apache2/ports.conf
  echo  '<VirtualHost *:8080>\n\
        ServerAdmin webmaster@localhost\n\
        DocumentRoot /dockerd\n\
        ErrorLog ${APACHE_LOG_DIR}/error.log\n\
        CustomLog ${APACHE_LOG_DIR}/access.log combined\n\
        <Directory /dockerd/>\n\
          Options Indexes FollowSymLinks\n\
          AllowOverride None\n\
          Require all granted\n\
      	</Directory>\n\
  </VirtualHost>\n\
  ' > /etc/apache2/sites-available/001-docker.conf

  ln -s /etc/apache2/sites-available/001-docker.conf /etc/apache2/sites-enabled/001-docker.conf

  systemctl restart apache2

  ###################
  # node npm n 設定
  ###################

  # https://www.trifields.jp/how-to-install-node-js-on-ubuntu1804-3544
  
  apt install -y nodejs npm
  npm install -g -y n
  n latest
  apt purge -y nodejs npm
  apt autoremove -y


SHELL
end
