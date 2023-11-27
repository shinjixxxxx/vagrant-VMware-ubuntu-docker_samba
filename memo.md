

## github Vagrantfile サンプル

https://github.com/sloopstash/kickstart-linux/blob/master/vagrant/ubuntu-22-04/vmware/arm64/server/Vagrantfile?fbclid=IwAR0n9AjZNNTC-cVoUsP1u6AcCJGywb75XNiPsLyU2rGUss_HeoSx9O78pSs


## What is Vagrant?
https://developer.hashicorp.com/vagrant/tutorials/getting-started/getting-started-index?fbclid=IwAR17n-v2eRj15rb7cjYQEktEe6lUBgqP64O5nLLIrnsJfkzinPvu5XBc_So



## AWS lightsailで素のubuntuにapache2を入れてhttps化したときに接続できないエラーの対処
https://zenn.dev/kz23szk/scraps/c1639f38f231f5

### 環境変数が読み込めていない

apache2 -V
Config variable ${APACHE_RUN_DIR} is not defined
apache2: Syntax error on line 80 of /etc/apache2/apache2.conf: DefaultRuntimeDir must be a valid directory, absolute or relative to ServerRoot

source /etc/apache2/envvarsで読み込み直したら上記のエラーは出なくなった。

sudo service apache2 restartで再起動したが変わらず。

curl https://localhost
curl: (35) error:1408F10B:SSL routines:ssl3_get_record:wrong version number


## UbuntuにDocker Engineをインストールする方法
https://kinsta.com/jp/blog/install-docker-ubuntu/


## AWS lightsailで素のubuntuにapache2を入れてhttps化したときに接続できないエラーの対処
https://zenn.dev/kz23szk/scraps/c1639f38f231f5

