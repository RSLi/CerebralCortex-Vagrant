# CerebralCortex-Vagrant

Cerebral Cortex is the big data cloud companion of mCerebrum designed to support population-scale data analysis, visualization, model development, and intervention design for mobile sensor data.

Vagrant configuration for installing Cerebral Cortex.

You can find more information about MD2K software on our [software website](https://md2k.org/software) or the MD2K organization on our [MD2K website](https://md2k.org/).

## Install

This configuration has been tested on Ubuntu 17.10.

1. To install Vagrant and dependencies

```
$ sudo apt install virtualbox virtualbox-dkms virtualbox-guest-additions-iso
$ wget https://releases.hashicorp.com/vagrant/2.0.1/vagrant_2.0.1_x86_64.deb
$ sudo dpkg -i vagrant_2.0.1_x86_64.deb
```

2. Install the docker-compose plugin for Vagrant
```
$ sudo vagrant plugin install vagrant-docker-compose
$ sudo vagrant plugin install vagrant-vbguest
```
Please consult [Vagrant Documentation](https://www.vagrantup.com/docs/)  if you face any installation errors for step 1 and 2. 

3. Clone this CerebralCortex-Vagrant repository.  Note: Vagrant must be run as superuser to properly forward ports
```
$ git clone https://github.com/MD2KOrg/CerebralCortex-Vagrant -b 2.2.2
$ cd CerebralCortex-Vagrant
$ sudo vagrant up
```

4. Once the 3rd  step is complete, please execute the following command to
   confirm that the installation steps were successfully.
   
```
$ wget http://localhost/api/v1/auth/
--2017-12-21 21:14:20--  http://localhost/api/v1/auth/
Resolving localhost (localhost)... 127.0.0.1
Connecting to localhost (localhost)|127.0.0.1|:80... connected.
HTTP request sent, awaiting response... 401 UNAUTHORIZED

Username/Password Authentication Failed.
```
The `Username/Password Authentication Failed` response confirms that the installation was successful and the system is online.

## Examples

The following commands lists the the status of all the services used by CerebralCortex.  Docker-Compose commands can be used to
interact with Cerebral Cortex's containers.
1. `sudo vagrant ssh`
2. `cd CerebralCortex-DockerCompose/``
3. `docker-compose ps`

`docker-compose ps` displays the status of all the services as shown below. 
```
     Name                    Command               State                                      Ports                                     
     ---------------------------------------------------------------------------------------------------------------------------------------
     md2k-api-server   /entrypoint.sh /start.sh         Up      443/tcp, 80/tcp                                                              
     md2k-cassandra    /bootstrap.sh cassandra -f       Up      7000/tcp, 7001/tcp, 7199/tcp, 0.0.0.0:9042->9042/tcp, 0.0.0.0:9160->9160/tcp 
     md2k-grafana      /run.sh                          Up      0.0.0.0:3000->3000/tcp                                                       
     md2k-influxdb     /entrypoint.sh influxd           Up      0.0.0.0:8086->8086/tcp                                                       
     md2k-jupyterhub   jupyterhub --no-ssl --conf ...   Up      0.0.0.0:32768->8000/tcp                                                      
     md2k-kafka        start-kafka.sh                   Up      0.0.0.0:9092->9092/tcp                                                       
     md2k-minio        /usr/bin/docker-entrypoint ...   Up      0.0.0.0:9000->9000/tcp                                                       
     md2k-mysql        docker-entrypoint.sh mysqld      Up      0.0.0.0:3306->3306/tcp                                                       
     md2k-nginx        nginx -g daemon off;             Up      0.0.0.0:443->443/tcp, 0.0.0.0:80->80/tcp                                     
     md2k-zookeeper    /bin/sh -c /usr/sbin/sshd  ...   Up      0.0.0.0:2181->2181/tcp, 22/tcp, 2888/tcp, 3888/tcp 
```

#### Running test cases
Run system level test-cases to make sure all the things are setup properly. System level test case will generate some sample data, process it using CerebralCortex, store it, retrieve it and verify results with predefined test values.
```
cd /home/vagrant/CerebralCortex/cerebralcortex/core/test_suite/
python3.6 -m unittest discover
```

#### Importing data
UPDATE THIS, pluging the phone.

`/home/vagrant/CerebralCortex-DockerCompose/cc_config_file\cc_vagrant_configuration.yml`
contains the parameter that stores the path from where data can be imported into 
CerebralCortex

Copy data into the directory used in the configuraton file.
```
cd /home/vagrant/CerebralCortex-DockerCompose/data/
wget https://mhealth.md2k.org/images/datasets/mCerebrum_test_data.tar.bz2
tar -xf mCerebrum_test_data.tar.bz2
rm -f mCerebrum_test_data.tar.bz2
```

Import the data into CerebralCortex
```
cd /home/vagrant/CerebralCortex-Scripts/data_replay/
python3.6 store_dirs_to_db.py --conf
/home/vagrant/CerebralCortex-DockerCompose/cc_config_file/cc_vagrant_configuration.yml

cd /home/vagrant/CerebralCortex-KafkaStreamPreprocessor/
sudo sh run_vagrant.sh
```

#### Increasing the available disk space in VM
Edit the Vagrant file to add the following line.
```
config.vm.synced_folder "<host_machine_folder_path>", "<foder_path_in_VM>"
```
This will mount a folder from the host machine in the VM. 

#### Using Jupyter Notebook
You may access Python Jupyter Notebook interface at:
```
http://IP-ADDRESS/jupyterhub
```

Default user for Jupyter notebook is
```
User Name: md2k
Password: mdk2
```
To add another user to Jupyter Notebook:
```
cd CerebralCortex-DockerCompose/
docker-compose exec jupyter bash
useradd -m USER-NAME && echo "USER-NAME:PASSWORD" | chpasswd
```
To create new python script:
```
Authenticate with user credentials
Click on Files tab
Click on new and select pySpark (Spark 2.2.0) (Python 3) to create a new Python script.
```

#### Using mCerebrum

Download the latest mCerebrum from [https://github.com/MD2Korg/mCerebrum-releases/tree/master/2.0/org.md2k.mcerebrum](https://github.com/MD2Korg/mCerebrum-releases/tree/master/2.0/org.md2k.mcerebrum)

Login to the mCerebrum application with the following information:
* Login: string
* Password: string
* Server: http://YOUR_MACHINE_IP_OR_DNS_NAME/

Once the system successfully authenticates, it will download a predefined
configuration file for you to test our the platform.  Once `Start Study` is
pressed the system will begin collecting and uploading to this enviornment.

#### Removing Vagrant image of CC
Run following commands if anything goes wrong and/or you want to uninstall CerebralCortex vagrant image
```
sudo vagrant global-status
Find the IMAGE-NAME. It would be required for next command
sudo vagrant destroy  IMAGE-NAME
```
#### Starting and stopping Cerebral Cortex
When you are done using stop as
PUT THIS IN THE END
to resume
Start Cerebral Cortex
```
$ sudo vagrant up
```

Stop Cerebral Cortex
```
$ sudo vagrant halt
```

Suspend Cerebral Cortex
```
$ sudo vagrant suspend
```

## FAQ
1. System requirements
2. Possible errors that can be encountered during the installation.
```
Pulling grafana (grafana/grafana:latest)...
Get https://registry-1.docker.io/v2/: dial tcp: lookup registry-1.docker.io on
10.0.2.3:53: read udp 10.0.2.15:34893->10.0.2.3:53: i/o timeout
```
The above error is DockerCompose error, please restart initalizing the VM with
the following command.
```
sudo vagrant up --provision
```


## Contributing
Please read our [Contributing Guidelines](https://md2k.org/contributing/contributing-guidelines.html) for details on the process for submitting pull requests to us.

We use the [Python PEP 8 Style Guide](https://www.python.org/dev/peps/pep-0008/).

Our [Code of Conduct](https://md2k.org/contributing/code-of-conduct.html) is the [Contributor Covenant](https://www.contributor-covenant.org/).

Bug reports can be submitted through [JIRA](https://md2korg.atlassian.net/secure/Dashboard.jspa).

Our discussion forum can be found [here](https://discuss.md2k.org/).

## Versioning

We use [Semantic Versioning](https://semver.org/) for versioning the software which is based on the following guidelines.

MAJOR.MINOR.PATCH (example: 3.0.12)

  1. MAJOR version when incompatible API changes are made,
  2. MINOR version when functionality is added in a backwards-compatible manner, and
  3. PATCH version when backwards-compatible bug fixes are introduced.

For the versions available, see [this repository's tags](https://github.com/MD2Korg/CerebralCortex-Vagrant/tags).

## Contributors

Link to the [list of contributors](https://github.com/MD2Korg/CerebralCortex-Vagrant/graphs/contributors) who participated in this project.

## License

This project is licensed under the BSD 2-Clause - see the [license](https://md2k.org/software-under-the-hood/software-uth-license) file for details.

## Acknowledgments

* [National Institutes of Health](https://www.nih.gov/) - [Big Data to Knowledge Initiative](https://datascience.nih.gov/bd2k)
  * Grants: R01MD010362, 1UG1DA04030901, 1U54EB020404, 1R01CA190329, 1R01DE02524, R00MD010468, 3UH2DA041713, 10555SC
* [National Science Foundation](https://www.nsf.gov/)
  * Grants: 1640813, 1722646
* [Intelligence Advanced Research Projects Activity](https://www.iarpa.gov/)
  * Contract: 2017-17042800006

