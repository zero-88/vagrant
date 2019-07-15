# -*- mode: ruby -*-
# vi: set ft=ruby :

$script = <<-SCRIPT
apt-get install -y vim tree net-tools build-essential
apt-get install -y libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev zlib1g-dev

#python3
cd /opt
wget https://www.python.org/ftp/python/3.7.4/Python-3.7.4.tgz
tar xzf Python-3.7.4.tgz
cd Python-3.7.4
./configure --enable-optimizations
make altinstall
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python3.7 get-pip.py

#gcloud
echo "deb http://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
apt-get update && apt-get install google-cloud-sdk -y

#kubectl
apt-get install -y apt-transport-https
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | tee -a /etc/apt/sources.list.d/kubernetes.list
apt-get update && apt-get install -y kubectl

#aws
pip3 install awscli --upgrade

#helm
curl -L https://git.io/get_helm.sh | bash
SCRIPT

$alias = <<-SCRIPT
cat <<EOF >> /home/vagrant/.bashrc

source <(curl -s -S https://raw.githubusercontent.com/google-cloud-sdk/google-cloud-sdk/master/completion.bash.inc > /dev/null)
source <(kubectl completion bash)
source <(helm completion bash)
EOF
echo 'complete -C $(which aws_completer) aws' >> /home/vagrant/.bashrc

echo '#### POSTGRESQL' >> /home/vagrant/.bash_aliases
echo 'POSTGRES_VERSION=10-alpine' >> /home/vagrant/.bash_aliases
echo 'alias psql="docker run --rm -it --network host -v /tmp:/tmp postgres:$POSTGRES_VERSION psql"' >> /home/vagrant/.bash_aliases
echo 'alias pg_dump="docker run --rm -it --network host -v /tmp:/tmp postgres:$POSTGRES_VERSION pg_dump"' >> /home/vagrant/.bash_aliases

echo '#### MYSQL' >> /home/vagrant/.bash_aliases
echo 'MYSQL_VERSION=latest' >> /home/vagrant/.bash_aliases
echo 'alias mysql="docker run --rm -it --network host mysql:$MYSQL_VERSION mysql"' >> /home/vagrant/.bash_aliases

echo '#### CONSUL' >> /home/vagrant/.bash_aliases
echo 'CONSUL_VERSION=latest' >> /home/vagrant/.bash_aliases
echo 'alias consul="docker run --rm -it --network host consul:$CONSUL_VERSION consul"' >> /home/vagrant/.bash_aliases

chown vagrant:vagrant /home/vagrant/.bash_aliases
SCRIPT

$docker = <<-SCRIPT
mkdir -p /etc/systemd/system/docker.service.d
cat > /etc/systemd/system/docker.service.d/override.conf <<EOF
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375
EOF
systemctl daemon-reload
systemctl restart docker
SCRIPT

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
  config.vm.provider "virtualbox"
  config.vm.box = "generic/debian9"
  config.vm.hostname = "zerodebian"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080
  # config.vm.network "forwarded_port", guest: 2375, host: 2375
  # config.vm.usable_port_range = 4000..9999

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network "private_network", ip: "10.10.14.12"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder "C:\\projects\\nubeio", "/nubeio"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    # vb.gui = true
  
    # Customize the amount of memory on the VM:
    vb.name = "debian9"
    vb.cpus = 2
    vb.memory = "2048"
  end
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
  config.vm.provision "docker" do |d|
    d.post_install_provision "shell", inline: $docker
  end
  config.vm.provision "shell", inline: $script
  config.vm.provision "shell", inline: $alias
end
