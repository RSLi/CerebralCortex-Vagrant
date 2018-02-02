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
  config.vm.box = "centos/7"
  # config.vm.box = "ubuntu/xenial64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  config.vm.network "forwarded_port", guest: 80, host: 80

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  machine_ip = "10.100.100.5"
  config.vm.network "private_network", ip: machine_ip

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"



  config.vm.hostname = "cerebralcortex"

  
  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
    # Customize the amount of memory on the VM:
    vb.memory = 8192
    vb.cpus = 4

    vb.name = "cerebralcortex"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Force CentOS/7 to turn on the second ethernet interface
  config.vm.provision "shell", inline: "ifup eth1", run: "always"
  
  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL


  config.vm.provision "shell", inline: <<-SHELL
    yum install -y git
    rm -rf CerebralCortex*
    git clone https://github.com/MD2Korg/CerebralCortex-DockerCompose
    git clone https://github.com/MD2Korg/CerebralCortex -b 2.1.2
    git clone https://github.com/MD2Korg/CerebralCortex-APIServer
    git clone https://github.com/MD2Korg/CerebralCortex-KafkaStreamPreprocessor.git
    git clone https://github.com/MD2Korg/CerebralCortex-Scripts.git
  SHELL

# Installing python3 and Apache Spark
  config.vm.provision "shell", inline: <<-SHELL
    yum install -y https://centos7.iuscommunity.org/ius-release.rpm
    yum install -y python36u python36u-devel python36u-pip java-1.8.0-openjdk libev libev-devel gcc wget 

    ln -s /usr/lib/jvm/java-1.8* /usr/lib/jvm/java

    wget -q http://d3kbcqa49mib13.cloudfront.net/spark-2.2.0-bin-hadoop2.7.tgz
    tar xzf spark-2.2.0-bin-hadoop2.7.tgz -C /usr/local && rm spark-2.2.0-bin-hadoop2.7.tgz
    ln -s /usr/local/spark-2.2.0-bin-hadoop2.7 /usr/local/spark
    pip3.6 install minio==2.2.4 numpy scipy
    pip3.6 install -r CerebralCortex-KafkaStreamPreprocessor/requirements.txt 

  SHELL
  

# Setting the log4j properties for Spark
  config.vm.provision "shell", inline: <<-SHELL
    set -x
    echo "log4j.rootLogger=INFO, file" > /usr/local/spark/conf/log4j.properties
    echo "log4j.appender.file=org.apache.log4j.RollingFileAppender" >> /usr/local/spark/conf/log4j.properties
    echo "log4j.appender.file.File=/tmp/logs/spark.log" >> /usr/local/spark/conf/log4j.properties
    echo "log4j.appender.file.MaxFileSize=10MB" >> /usr/local/spark/conf/log4j.properties
    echo "log4j.appender.file.MaxBackupIndex=10" >> /usr/local/spark/conf/log4j.properties
    echo "log4j.appender.file.layout=org.apache.log4j.PatternLayout" >> /usr/local/spark/conf/log4j.properties
    echo "log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss}%-5p%c{1}:%L - %m%n" >> /usr/local/spark/conf/log4j.properties
    echo "log4j.logger.org.apache.spark.repl.Main=WARN" >> /usr/local/spark/conf/log4j.properties
    echo "log4j.logger.org.spark_project.jetty=WARN" >> /usr/local/spark/conf/log4j.properties
    echo "log4j.logger.org.spark_project.jetty.util.component.AbstractLifeCycle=ERROR" >> /usr/local/spark/conf/log4j.properties
    echo "log4j.logger.org.apache.spark.repl.SparkIMain$exprTyper=INFO" >> /usr/local/spark/conf/log4j.properties
    echo "log4j.logger.org.apache.spark.repl.SparkILoop$SparkILoopInterpreter=INFO" >> /usr/local/spark/conf/log4j.properties
    echo "log4j.logger.org.apache.parquet=ERROR" >> /usr/local/spark/conf/log4j.properties
    echo "log4j.logger.parquet=ERROR" >> /usr/local/spark/conf/log4j.properties
    echo "log4j.logger.org.apache.hadoop.hive.metastore.RetryingHMSHandler=FATAL" >> /usr/local/spark/conf/log4j.properties
    echo "log4j.logger.org.apache.hadoop.hive.ql.exec.FunctionRegistry=ERROR" >> /usr/local/spark/conf/log4j.properties

  SHELL


# Setting the environment variables
#  config.vm.provision "shell", run: "always", inline: <<-SHELL
#    set -x
#    [ -z $JAVA_HOME ] && echo "export JAVA_HOME=/usr/lib/jvm/java/jre" >> ~/.bashrc
#  SHELL


  config.vm.provision :docker
  config.vm.provision :docker_compose, yml: "/home/vagrant/CerebralCortex-DockerCompose/docker-compose.yml", env: { "MACHINE_IP" => "#{machine_ip}" }, run: "always"

  # Temporary fix for mysql-apiserver startup problem
  config.vm.provision "shell", run: "always", inline: <<-SHELL
    set -x
    cd /home/vagrant/CerebralCortex-DockerCompose
    /usr/local/bin/docker-compose restart apiserver
  SHELL
  
  # creating default database in influxdb
  config.vm.provision "shell", inline: <<-SHELL
    set -x
    curl -G http://localhost:8086/query --data-urlencode "q=CREATE DATABASE cerebralcortex_raw"
  SHELL

  # Temporary fix for mysql-apiserver startup problem
  config.vm.provision "shell", inline: <<-SHELL
    set -x
    cd /home/vagrant/CerebralCortex
    /usr/bin/python3.6 setup.py install
    cd /home/vagrant/CerebralCortex-KafkaStreamPreprocessor
    /usr/bin/python3.6 setup.py install
  SHELL

  
  config.vm.provision "shell", run: "always", inline: <<-SHELL
    set -x
    cd /home/vagrant/CerebralCortex-KafkaStreamPreprocessor
    /usr/bin/sh ./run_vagrant.sh >> /tmp/spark_stderr 2>&1 &
  SHELL

  # Getting test data from mHealth
  config.vm.provision "shell", inline: <<-SHELL
    set -x
    cd /home/vagrant/CerebralCortex-DockerCompose/data/
    ggID=1HKH6sxPgbKTqm1THAEIEzPVmUwlrZibS
    ggURL='https://drive.google.com/uc?export=download'
    filename="$(curl -sc /tmp/gcokie "${ggURL}&id=${ggID}" | grep -o '="uc-name.*</span>' | sed 's/.*">//;s/<.a> .*//')"
    getcode="$(awk '/_warning_/ {print $NF}' /tmp/gcokie)"
    curl -Lb /tmp/gcokie "${ggURL}&confirm=${getcode}&id=${ggID}" -o "${filename}"
    tar -xf mCerebrum_test_data.tar.bz2
    rm -f mCerebrum_test_data.tar.bz2
    cd /home/vagrant/CerebralCortex-Scripts/data_replay
    python3.6 replay_data.py -b "127.0.0.1:9092" -d "/home/vagrant/CerebralCortex-DockerCompose/data/636fcc1f-8966-4e63-a9df-0cbaa6e9296c/"
  SHELL
  # config.vm.provision "docker" do |d|
  #   d.pull_images "hello-world"
  # end

end
