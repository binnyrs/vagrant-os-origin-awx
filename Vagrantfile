# -*- mode: ruby -*-
# vi: set ft=ruby :

BOX_IMAGE = "centos/7"
MASTER_NAME = "origin-master"
BRIDGE_IF = "enp6s0"
#BRIDGE_IF = "wlp5s0"
ROUTING_SUFFIX = ".xip.io"
INSECURE_REGISTRY = "172.30.0.0/16"

#MACHINE DETAILS
CPUS = "2"
MEMORY = "4096"
DISK = "40G"

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
  config.vm.box = BOX_IMAGE
  config.vm.box_check_update = false
  config.vbguest.auto_update = false
  config.vm.hostname = MASTER_NAME
  config.hostmanager.enabled = true
  config.hostmanager.manage_guest = true
  config.hostmanager.manage_host = true
  config.hostmanager.ignore_private_ip = false

  config.hostmanager.ip_resolver = proc do |machine|
    result = ""
    machine.communicate.execute("hostname -I | cut -d ' ' -f 2") do |type, data|
      result << data if type == :stdout
    end
    ip = result.split("\n").first[/(\d+\.\d+\.\d+\.\d+)/, 1]
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
  config.vm.network "public_network", type: "dhcp", :bridge => BRIDGE_IF

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
  config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
    vb.memory = MEMORY
    vb.cpus = CPUS
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "provision", type: "shell", inline: <<-SH_PRO
    yum -y install epel-release
    yum -y update
    yum -y install wget git docker golang make gcc zip mercurial krb5-devel bsdtar bc rsync bind-utils file jq tito createrepo openssl gpgme gpgme-devel libassuan libassuan-devel ansible
    cat <<EOF | tee /etc/docker/daemon.json
{
    "insecure-registries" : [ "#{INSECURE_REGISTRY}" ]
}
EOF
    systemctl daemon-reload && systemctl start docker && systemctl enable docker
    wget https://github.com/openshift/origin/releases/download/v3.7.0/openshift-origin-client-tools-v3.7.0-7ed6862-linux-64bit.tar.gz -O /tmp/oc.tar.gz
    tar -zxvf /tmp/oc.tar.gz --strip-components=1 -C /usr/bin/
    MASTER_IP=$(hostname -i|cut -f2 -d ' ')
    # oc cluster up --public-hostname=#{MASTER_NAME} --routing-suffix="${MASTER_IP}"#{ROUTING_SUFFIX} --metrics=true --service-catalog=true
    oc cluster up --public-hostname=#{MASTER_NAME} --routing-suffix="${MASTER_IP}"#{ROUTING_SUFFIX}
    if [ ! -d awx ]
    then
      git clone https://github.com/ansible/awx
      cd awx
    else
      cd awx
      git remote update
      git pull --ff-only
    fi
    sed -i 's/openshift_host=.*/openshift_host=#{MASTER_NAME}:8443/' installer/inventory
    sed -i '/openshift_host/s/^#//g' installer/inventory
    sed -i '/awx_openshift_project/s/^#//g' installer/inventory
    sed -i '/openshift_user/s/^#//g' installer/inventory
    sed -i '/awx_node_port/s/^#//g' installer/inventory
    ansible-playbook -i installer/inventory installer/install.yml -e openshift_password=developer  -e docker_registry_password=$(oc whoami -t)
  SH_PRO
end
