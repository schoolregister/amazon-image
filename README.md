# Schoolregister.com AWS EC2 machine

To get good idea of pricing, look here
http://www.ec2instances.info/?cost=monthly

We are running on one `m3.medium` now

## Users

Pass userdata while booting, cloud-init will take care of the rest:

	#cloud-config
	groups:
	  - schoolregister
	users:
	  - name: schoolregister
	    gecos: Schoolregister.com
	    primary-group: schoolregister
	    system: true
	  - name: jelle
	    gecos: Jelle Herold
	    primary-group: wheel
	    sudo: ALL=(ALL) NOPASSWD:ALL
	    no-user-group: true
	    ssh-authorized-keys:
	      - <ssh-rsa AAAA....>

## Elasticsearch

	rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch

Add to `/etc/yum.repos.d/elasticsearch.repo`

	[elasticsearch-1.5]
	name=Elasticsearch repository for 1.5.x packages
	baseurl=http://packages.elastic.co/elasticsearch/1.5/centos
	gpgcheck=1
	gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch
	enabled=1

Then install it

	yum install elasticsearch

Also ensure it runs on startup

	chkconfig --add elasticsearch

Modify the config `/etc/elasticsearch/elasticsearch.yml`.

	# cluster.name: pick-a-cluster-name
	# discovery.type: ec2
	# discovery.ec2.host_type: public_ip
	# discovery.ec2.groups:
	# discovery.ec2.ping_timeout: 5m
	# cloud.aws.region:
	# script.disable_dynamic: true

Start it now

	service elasticsearch start

Check it

	curl localhost:9200
	
### Further tuning

Disable heap dumps, in `/usr/share/elasticsearch/bin/elasticsearch.in.sh` make sure this option is not set:

	#JAVA_OPTS="$JAVA_OPTS -XX:+HeapDumpOnOutOfMemoryError"


## NodeJS

Extra Packages for Enterprise Linux ([EPEL](https://fedoraproject.org/wiki/EPEL#What_packages_and_versions_are_available_in_EPEL.3F))
has a [NodeJS package](http://dl.fedoraproject.org/pub/epel/7/x86_64/repoview/nodejs.html).

Either install that, using 

	sudo yum install nodejs npm --enablerepo=epel

Or, build from source:

Install build deps (and leave them, we sometimes need it to build build NodeJS extentions)

	yum install gcc-c++ openssl-devel

Download and extract

	cd /usr/local/src
	curl -LO http://nodejs.org/dist/node-latest.tar.gz
	tar -xzvf node-latest.tar.gz
	cd node-v*/

Configure and compile

	./configure
	make
	make install

This installs node into `/usr/local/`, but this is not in our path.

Add the following to a file `/etc/profile.d/nodejs.sh`

	export PATH=$PATH:/usr/local/bin

Logout, login. Boom.
