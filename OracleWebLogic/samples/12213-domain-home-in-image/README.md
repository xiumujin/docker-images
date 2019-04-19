Example Image with a  WebLogic Server Domain
=============================================
This Dockerfile extends the Oracle WebLogic image by creating a sample WebLogic Server  12.2.1.3 domain and cluster into a Docker image.

A domain is created inside the image and utility scripts are copied into the image, enabling users to start an Administration Server and a Managed Server, each running in separate containers. 

**Note:** In this sample, the WebLogic Servers are configured with a blank listen address; when running JTA transactions, you must use a DNS server to configure the listen addresses to use DNS names.

## Providing the Administration Server user name and password


**During Docker Build:** The user name and password must be supplied in the `domain_security.properties` file.  The property file is located in the directory `docker-images/OracleWebLogic/samples/12213-domain-home-in-image/properties/docker_build` in the HOST. This property file gets copied into the image directory `/u01/oracle/properties`.

**During Docker Run:** The user name and password must be supplied in a `security.properties` file.  The property file is located in a `docker-images/OracleWebLogic/samples/12213-domain-home-in-image/properties/docker_run` directory in the HOST. On the Docker run command line, the `-v` option maps the property file into the image directory `/u01/oracle/properties`.


The security property files enable the scripts to configure the correct authentication for the WebLogic Administration Server and Managed Servers. The format of the `security.properties` and `domain_security.properties` files are key=value pairs, for example:

	username=myadminusername
	password=myadminpassword

**Note:** Oracle recommends that the `domain_security.properties` and `security.properties` files be deleted or secured after the container and the WebLogic Server are started so that the user name and password are not inadvertently exposed.

## How to Build and Run
At build time, the `domain.properties` file is used to pass in the Docker arguments and configuration parameters for the WebLogic domain.


**During Docker Build:** The domain configuration parameters must be supplied in the `domain.properties` file.  This file is located in the directory `properties/docker_build` in the HOST. This property file gets copied into the image directory `/u01/oracle/properties`.


The domain property file enables you to customize the parameters to configure the WebLogic domain. The format of the `domain.properties` are key=value pairs, for example:

	ADMIN_NAME=admin-server
	ADMIN_HOST=wlsadmin
	MANAGED_SERVER_NAME_BASE=managed-server
	CONFIGURED_MANAGED_SERVER_COUNT=2
	CLUSTER_NAME=cluster-1
	DEBUG_FLAG=true
	PRODUCTION_MODE_ENABLED=true
	CLUSTER_TYPE=DYNAMIC

**NOTE:** Before invoking the build make sure you have built `oracle/weblogic:12.2.1.3-developer`.


Under the directory `docker-images/OracleWebLogic/samples/12213-domain-home-in-image/container_scripts` find the script `setEnv.sh`. This script extracts the following Docker arguments and passes them as a `--build-arg` to the Dockerfile.


* Domain Name:           `DOMAIN_NAME`         (default: `base_domain`)  
* Admin Port:            `ADMIN_PORT`          (default: `7001`)          
* Managed Server Port:   `MANAGED_SERVER_PORT` (default: `8001`)          
* Debug Port:            `DEBUG_PORT`          (default: `8453`)
* Database Port:         `DB_PORT`             (default: `1527`)
* Admin Server Name:     `ADMIN_NAME`          (default: `admin-server`)
* Admin Server Host:     `ADMIN_HOST`          (default: `wlsadmin`)

**NOTE:** The `DOMAIN_HOME` will be persisted in the image directory `/u01/oracle/user-projects/domains/$DOMAIN_NAME`.

To build this sample, run:

 	$ . container-scripts/setEnv.sh ./properties/docker-build/domain.properties
 	$ docker build $BUILD_ARG  --force-rm=true -t 12213-domain-home-in-image .


**During Docker Run:** of the Administration and Managed Servers, the user name and password need to be passed in as well as some optional parameters. The property file is located in a `docker-images/OracleWebLogic/samples/12213-domain-home-in-image/properties/docker_run` in the HOST. On the Docker run command line, add the `-v` option which maps the property file into the image directory `/u01/oracle/properties`.


To start the containerized Administration Server, run:

	$ docker run -d --name wlsadmin --hostname wlsadmin -p 7001:7001 \
          -v <HOST DIRECTORY TO PROPERTIES FILE>/properties/docker-run:/u01/oracle/properties \
          12213-domain-home-in-image

To start a containerized Managed Server (MS1) to self-register with the Administration Server above, run:

	$ docker run -d --name MS1 --link wlsadmin:wlsadmin -p 8001:8001 \
          -v <HOST DIRECTORY TO PROPERTIES FILE>/properties/docker-run:/u01/oracle/properties \
          -e MANAGED_SERV_NAME=managed-server1 12213-domain-home-in-image startManagedServer.sh

To start a second Managed Server (MS2), run:

	$ docker run -d --name MS2 --link wlsadmin:wlsadmin -p 8002:8001 \
          -v <HOST DIRECTORY TO PROPERTIES FILE>/properties/docker-run:/u01/oracle/properties \
          -e MANAGED_SERV_NAME=managed-server2 12213-domain-home-in-image startManagedServer.sh

The above scenario from this sample will give you a WebLogic domain with a cluster set up on a single host environment.

You may create more containerized Managed Servers by calling the `docker` command above

# Copyright
Copyright (c) 2014-2018 Oracle and/or its affiliates. All rights reserved.
