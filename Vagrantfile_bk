# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|

  config.ssh.insert_key = false
  config.ssh.private_key_path = "~/.vagrant.d/insecure_private_key"


  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "sloopstash/ubuntu-22-04"
  config.vm.box_version = "2.1.1"

 # Define virtual machine name.
  config.vm.define "sloopstash-ubuntu-22-04-server"

  # Set virtual machine hostname.
  config.vm.hostname = "sloopstash-ubuntu-22-04-server"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 443, host: 443, auto_correct: false

  
  ###
  ###
  config.vm.network "forwarded_port", guest: 5173, host: 5173, host_ip: "0.0.0.0"
  config.vm.network "forwarded_port", guest: 3000, host: 3000, host_ip: "0.0.0.0"
  config.vm.network "forwarded_port", guest: 6006, host: 6006, host_ip: "0.0.0.0"
  ###
  ###

  
  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.



  
  #  config.vm.network "private_network", ip: "192.168.201.6" #オリジナル
# config.vm.network "private_network", ip: "192.168.33.10"
#config.vm.network "private_network", ip: "192.168.1.63"

# config.vm.network "private_network"

config.vm.network "public_network", ip: "192.168.1.200" ,bridge: "en0: Ethernet"

  

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # SSH credentials to connect to virtual machine.
  config.ssh.username = "vagrant"
  config.ssh.private_key_path = ["~/.vagrant.d/insecure_private_key"]
  config.ssh.insert_key = false

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "vmware_fusion" do |vb|
    # Disable GUI when booting the virtual machine.
    vb.gui = false
  
    # Allocate memory to the virtual machine.
    vb.memory = "2048"

    # Allocate processors to the virtual machine.
    vb.cpus = "1"



  end
  
  #
  # View the documentation for the provider you are using for more
  # information on available options.




  ###########################################################################
  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  ###########################################################################

  config.vm.provision "shell", inline: <<-SHELL
    apt update
    apt upgrade -y
    apt install -y wget vim net-tools gcc make tar git unzip sysstat tree emacs


    ###################
    ###################
    # docker 設定
    # リポジトリの追加を行うので先に実行
    ###################
    ###################
    # 参考

    # 【簡単な4つの方法】UbuntuにDockerをインストールするには
    # https://kinsta.com/jp/blog/install-docker-ubuntu/?fbclid=IwAR0ON33QfuacgwlWS8e6TvN3JUEcGn0DR1Z8a4T6jkItDiwc-vr7o8B6f9Y

    apt install -y ca-certificates curl gnupg lsb-release

    mkdir -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    chmod a+r /etc/apt/keyrings/docker.gpg

    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt update

    # INSTALL DOCKER ENGINE
    apt install -y docker-ce docker-ce-cli containerd.io

  
    ###################
    # docker samba 設定
    ###################    
    mkdir /dockerd
    chmod 777 /dockerd
    echo 'sudo docker rm -f samba ; sudo docker run -it --name samba -p 139:139 -p 445:445 -v /dockerd:/dockerd -d dperson/samba -s "dockerd;/dockerd;yes;no"' > /dockerd/docker_samba.sh
    chmod +x /dockerd/docker_samba.sh
    ###################
    # docker samba 自動起動設定
    ###################

    echo '#!/bin/sh -e\n' > /etc/rc.local
    echo 'sudo /dockerd/docker_samba.sh\n' >> /etc/rc.local
    chmod +x /etc/rc.local
    /etc/rc.local


    ###################
    # node npm n 設定
    ###################

    # https://www.trifields.jp/how-to-install-node-js-on-ubuntu1804-3544
    
    apt install -y nodejs npm
    npm install -g -y n
    n latest
    apt purge -y nodejs npm
    apt autoremove -y
  

    ###################
    # apt-get update
    # apache2 php  
    # インストール
    ###################    

    # 【Ubuntu 20.04 LTS Server】Apache2とPHPを動かす
    # https://www.yokoweb.net/2020/08/14/ubuntu-20_04-apache-php/
    apt-get install -y apache2
    systemctl enable apache2
    apt-get install -y php libapache2-mod-php

    php -v
    a2enmod php8.1 ###################バージョン注意

    ###################
    # apache2設定
    ###################
    
    
    echo  '<VirtualHost *:80>\n\
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
    rm /etc/apache2/sites-enabled/000-default.conf


    source /etc/apache2/envvars
    systemctl restart apache2


  SHELL



config.vm.provision "shell", run: "always", inline: <<-SHELL
  /dockerd/start-distviewer.sh
SHELL

end
