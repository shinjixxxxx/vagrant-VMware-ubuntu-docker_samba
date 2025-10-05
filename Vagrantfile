# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|

	# SSH できなくなった時 以下解放
	# config.ssh.username = "vagrant"
	# config.ssh.insert_key = false
	# config.ssh.private_key_path = "~/.vagrant.d/insecure_private_key"

	config.vm.box = "gyptazy/ubuntu22.04-arm64"
	config.vm.box_version = "1.0.1"

	# Define virtual machine name.
	config.vm.define "sloopstash-ubuntu-22-04-server"

	# Set virtual machine hostname.
	config.vm.hostname = "sloopstash-ubuntu-22-04-server"

	# Disable automatic box update checking. If you disable this, then
	# boxes will only be checked for updates when the user runs
	# `vagrant box outdated`. This is not recommended.
	config.vm.box_check_update = false

	###
	###
	config.vm.network "forwarded_port", guest: 5173, host: 5173, host_ip: "0.0.0.0"
	config.vm.network "forwarded_port", guest: 3000, host: 3000, host_ip: "0.0.0.0"
	###
	###

	# config.vm.network "private_network"

	config.vm.network "public_network", ip: "192.168.1.200" ,bridge: "en0: Ethernet"

	config.vm.synced_folder ".", "/vagrant", disabled: true

	config.vm.provider "vmware_fusion" do |vb|
		# Disable GUI when booting the virtual machine.
		vb.gui = false
		# Allocate memory to the virtual machine.
		vb.memory = "2048"
		# Allocate processors to the virtual machine.
		vb.cpus = "2"
 	end
  


	###########################################################################
	# Enable provisioning with a shell script. Additional provisioners such as
	# Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
	# documentation for more information about their specific syntax and use.
	###########################################################################
	config.vm.provision "shell", inline: <<-SHELL

		###########################################################################
		# set -eux
		###########################################################################
		export DEBIAN_FRONTEND=noninteractive

		apt-get -yq update
		apt-get -yq upgrade
		apt-get -yq install wget vim net-tools gcc make tar git unzip sysstat tree emacs

		###################
		###################
		# docker 設定
		# リポジトリの追加を行うので先に実行
		###################
		###################
		# 【簡単な4つの方法】UbuntuにDockerをインストールするには
		# https://kinsta.com/jp/blog/install-docker-ubuntu/?fbclid=IwAR0ON33QfuacgwlWS8e6TvN3JUEcGn0DR1Z8a4T6jkItDiwc-vr7o8B6f9Y

		apt-get -yq install ca-certificates curl gnupg lsb-release

		# Docker repo（★ jammy を明示）
		# install -d -m 0755 /etc/apt/keyrings
		# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg

		install -d -m 0755 /etc/apt/keyrings
		curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
			| gpg --dearmor --batch --yes \
			| tee /etc/apt/keyrings/docker.gpg >/dev/null
		chmod a+r /etc/apt/keyrings/docker.gpg

		echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu jammy stable" > /etc/apt/sources.list.d/docker.list
		apt-get -yq update
		apt-get -yq install docker-ce docker-ce-cli containerd.io


		###################
		# docker samba 設定
		###################
		if [ ! -d /dockerd ]; then
		mkdir /dockerd
		fi
		chown -R vagrant:vagrant /dockerd
		chmod 777 /dockerd

		# dockerd 作成直後、ディレクトリが空のうちに viewer をクローン 
		if [ ! -d /dockerd/.git ]; then
		git clone https://github.com/shinjixxxxx/js-project-quick-viewer-for-vitepj /dockerd
		fi  



# 起動スクリプトを作成（既存を上書き）
cat >/dockerd/docker_samba.sh <<'EOS'
#!/usr/bin/env bash
set -euo pipefail
docker rm -f samba >/dev/null 2>&1 || true
exec docker run -d --name samba \
  --restart unless-stopped \
  -p 139:139 -p 445:445 \
  -v /dockerd:/dockerd \
  -e USERID=1000 -e GROUPID=1000 \
  dperson/samba \
  -u "vagrant;vagrant" \
  -s "dockerd;/dockerd;yes;no;yes;all;vagrant"
EOS
		chmod +x /dockerd/docker_samba.sh
		# 念のため所有権も揃える
		chown -R vagrant:vagrant /dockerd
		# すぐ起動
		/dockerd/docker_samba.sh

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
		
		apt-get install -y nodejs npm
		npm install -g -y n
		n latest
		apt-get purge -y nodejs npm
		apt-get autoremove -y


		###################
		# apache2 php  
		# インストール
		###################    
		apt-get -yq install apache2 php libapache2-mod-php || true
		PHPV=$(php -r 'echo PHP_MAJOR_VERSION.".".PHP_MINOR_VERSION;' 2>/dev/null || echo "")
		if [ -n "$PHPV" ] && [ -f "/etc/apache2/mods-available/php${PHPV}.load" ]; then
		a2enmod "php${PHPV}"
		a2enmod rewrite
		systemctl enable --now apache2
		else
		apt-get -yq install php-fpm
		a2enmod proxy_fcgi setenvif
		a2enconf "php*-fpm" || a2enconf "php${PHPV}-fpm" || true
		systemctl enable --now php*-fpm.service || true
		systemctl reload apache2 || systemctl restart apache2
		fi
		php -v


# 仮想ホストを書き込み（echoよりヒアドキュメントが安全）
cat >/etc/apache2/sites-available/001-docker.conf <<'VHOST'
<VirtualHost *:80>
ServerAdmin webmaster@localhost
DocumentRoot /dockerd

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined

<Directory /dockerd/>
	Options Indexes FollowSymLinks
	AllowOverride None
	Require all granted
</Directory>
</VirtualHost>
VHOST


		# サイトの有効/無効化は専用コマンドで
		a2ensite 001-docker.conf
		a2dissite 000-default.conf || true

		# Apache の再読み込み/再起動
		systemctl reload apache2 || systemctl restart apache2

	SHELL


	# --- 2) vagrant ユーザーで Node/npm セット & express を入れる ---
	config.vm.provision "shell", privileged: false, inline: <<-'SHELL'
		set -eux
	    echo 'alias pj="cd /dockerd"' > ~/.bash_aliases
		cd /dockerd

		sudo chown -R vagrant:vagrant /dockerd
		# express をインストール（-y は不要）
		npm install express

		# 確認
		node -p "require.resolve('express')" || node -p "import.meta.url" >/dev/null
		SHELL

		config.vm.provision "shell", run: "always", inline: <<-SHELL
		/dockerd/start-distviewer.sh
	SHELL

end
