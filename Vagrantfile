# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
ENV["LC_ALL"] = "en_US:zh_CN.UTF-8"

Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "centos/7"

  (1..1).each do |i|
    config.vm.define "node-#{i}" do |node|
      node.vm.network "private_network", ip: "172.16.0.10#{i}"
      node.vm.network "public_network", bridge: "en1: Wi-Fi (AirPort)"
      node.vm.provision "shell",
        inline: <<-SCRIPT
        echo starting from node $1
        echo '开始安装shadowsocks'
        yum -y install epel-release
        yum -y install python-pip
        pip install --upgrade pip
        pip install shadowsocks
        mkdir /etc/shadowsocks

        cat >/etc/shadowsocks/shadowsocks.json <<EOF
{
  "server":"35.201.231.175",
  "server_port":443,
  "local_address": "127.0.0.1",
  "local_port":1080,
  "password":"pwd284659736",
  "timeout":300,
  "method":"aes-256-cfb",
  "workers": 1
}
EOF
        echo '开始配置shadows服务'
        cat >/etc/systemd/system/shadowsocks.service <<EOF
[Unit]
Description=Shadowsocks
[Service]
TimeoutStartSec=0
ExecStart=/usr/bin/sslocal -c /etc/shadowsocks/shadowsocks.json
[Install]
WantedBy=multi-user.target
EOF
        echo '开始启动shadows客户端'
        systemctl enable shadowsocks.service
        systemctl start shadowsocks.service
        systemctl status shadowsocks.service

        echo '安装privoxy'
        yum install wget gcc autoconf -y
        mkdir /opt/privoxy && cd /opt/privoxy
        wget https://github.com/ilisin/rc/raw/master/privoxy-3.0.26-stable-src.tar.gz
        tar -zxvf privoxy-3.0.26-stable-src.tar.gz
        cd privoxy-3.0.26-stable
        useradd privoxy
        autoheader && autoconf
        ./configure
        make && make install

        echo "forward-socks5t    /    127.0.0.1:1080 ." >> /usr/local/etc/privoxy/config

        privoxy --user privoxy /usr/local/etc/privoxy/config

        echo "export http_proxy=http://127.0.0.1:8118" >> /etc/profile
        echo "export https_proxy=http://127.0.0.1:8118" >> /etc/profile

        source /etc/profile
        
        echo '安装docker'
        yum install -y docker
        systemctl enable docker && systemctl start docker

        echo '安装kubelete kubeadm kubectl'
        cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
        setenforce 0
        yum install -y kubelet kubeadm kubectl
        systemctl enable kubelet && systemctl start kubelet

        echo '这里需要检查docker 及 kubelet cgroup driver是否一致'

        SCRIPT
        args = [i]
    end
  end

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

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

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

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end
