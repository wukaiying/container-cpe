# IBM Content Platform Engine Overview
IBM® Content Platform Engine (CPE) container is a Docker image that enables you to quickly deploy IBM Content Platform Engine without a traditional software installation. The IBM Content Platform Engine container image is based on the IBM Content Platform Engine v5.5.0 and Liberty v17.0.0.3 releases.

For more details about IBM Content Platform Engine, see the [IBM Knowledge Center](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.5.0/com.ibm.p8toc.doc/welcome_p8.htm).    


# Limitations
The following features are not currently available for this image:
- Client connections to Content Platform Engine using EJB is not supported.
- Content Platform Engine WebSphere Virtual Member Manager (VMM) directory provider is not supported.

# Known Issues

The following issues have been observed:

- ACCE restricted access does not work on Liberty

# Requirements and prerequisites

Before you deploy and run the IBM Content Platform Engine container image, confirm the following prerequisites:

- A Docker runtime environment (a Linux host or virtual machine with Docker installed)    
- Supported LDAP provider (Microsoft Active Directory or IBM Security Directory Server)
- Supported database provider (IBM DB2 v10.5 or higher)


# Preparing for container installation

## 1. Prepare the database.

Before you can use the Content Platform Engine container, you must create the GCD database and the object store database.
#### 1.1 Create the GCD database.
Use the following information to create the GCD database: [IBM Knowledge Center](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.5.0/com.ibm.p8.planprepare.doc/p8ppi258.htm)

#### 1.2 Create the Object Store database.
Create object store database based on [IBM Knowledge Center](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.2.1/com.ibm.p8.planprepare.doc/p8ppi168.htm)

## 2. Create the container configuration.

#### 2.1 Download the sample configuration files.  
Collect the following configuration files for your container environment:
- XML configuration file for LDAP ([ldapAD.xml](https://github.com/wukaiying/container-cpe/blob/master/examples/ldapAD.xml))<br>
- XML configuration file for Db2 JDBC Driver ([DB2JCCDriver.xml
](https://github.com/wukaiying/container-cpe/blob/master/examples/DB2JCCDriver.xml))<br>
- JAR file for Db2 JDBC Driver ([db2jcc4.jar](https://github.com/wukaiying/container-cpe/blob/master/examples/db2jcc4.jar),[db2jcc_license_cu.jar](https://github.com/wukaiying/container-cpe/blob/master/examples/db2jcc_license_cu.jar))<br>
- XML configuration file for GCD data source ([FNGCDDS.xml](https://github.com/wukaiying/container-cpe/blob/master/examples/FNGCDDS.xml))<br>
- XML configuration file for Object Store data source ([OS1DS.xml](https://github.com/wukaiying/container-cpe/blob/master/examples/OS1DS.xml))<br>
- JAR file for building bootstrap properties (1 of 2) ([BootstrapConfig.jar](https://github.com/wukaiying/container-cpe/blob/master/examples/BootstrapConfig.jar))
- JAR file for building bootstrap properties (2 of 2) ([BootstrapConfigProps.jar](https://github.com/wukaiying/container-cpe/blob/master/examples/BootstrapConfigProps.jar))

#### 2.2 Create the LDAP configuration.
- Modify the ldapAD.xml file to reflect your LDAP configuration. The example configuration is for Microsoft Active Directory. At minimum, update the following values:
  - host: LDAP host
  - port: LDAP port
  - baseDN: base DN for searching for users
  - bindDN: LDAP bind user
  - bindPassword: LDAP bind user password

You can find additional information and example configurations for Microsoft Active Directory and IBM® Directory Server [here.](https://www.ibm.com/support/knowledgecenter/en/SSAW57_liberty/com.ibm.websphere.wlp.nd.multiplatform.doc/ae/twlp_sec_ldap.html)<br>

#### 2.3 Create the GCD configuration.
- Modify the FNGCDDS.xml file to reflect your DB2 configuration for the GCD. Update the configuration in both the non-XA (FNGCDDS) and XA (FNGCDDSXA) sections.  At minimum, update the following values:
  - serverName: your DB2 host
  - password: the DB2 user password (db2user)
- The following must be modified if you created a GCD with a different name in section 1.1:
  - databaseName: your GCD database name

You can find additional information on data source properties [here.](https://www.ibm.com/support/knowledgecenter/en/SSAW57_8.5.5/com.ibm.websphere.wlp.nd.doc/ae/rwlp_config_dataSource.html)

#### 2.4 Modify the Object Store configuration.
- Modify the OS1DS.xml file to reflect your Db2 configuration for the Object Store. Update the configuration in both the non-XA (OS1DS) and XA (OS1DSXA) sections. At minimum, update the following values:
  - serverName: your DB2 host
  - password: the DB2 user password (db2user)
- The following must be modified if you created an Object Store with a different name in section 1.2
  - databaseName: your Object Store database name

You can create additional Object Stores. The best practice is to create a new database for each object store. Note that you must have distinct 'id' and 'jndiId' attributes for each 'dataSource' tag.

You can find additional information on data source properties [here.](https://www.ibm.com/support/knowledgecenter/en/SSAW57_8.5.5/com.ibm.websphere.wlp.nd.doc/ae/rwlp_config_dataSource.html)

#### 2.5 Create folders on shared or local storage.
Create folders on shared or local storage to hold the deployment-specific configuration files as well as data that will live outside the container.<br>The host directory examples here use a base directory of `/home/cpe_data`, but you can modify this if required. If you choose an alternate location, make a note of the location so that you can pass these mount points to the container later when you run it.

Container folder | Host directory example | Description
------------ | ------------- | -------------
/opt/ibm/wlp/usr/servers/defaultServer/configDropins/overrides | /home/cpe_data/configDropins/overrides | Configuration files for Liberty |
/opt/ibm/asa  | /home/cpe_data/asa | Content storage volume for advanced storage area  |
/opt/ibm/textext | /home/cpe_data/textext | Text extraction volume used by CSS |
/opt/ibm/wlp/usr/servers/defaultServer/logs | /home/cpe_data/logs |Content Platform Engine & Liberty logs
/opt/ibm/wlp/usr/servers/defaultServer/FileNet | /home/cpe_data/FileNet | FileNet logs
/opt/ibm/icmrules | /home/cpe_data/icmrules | Rules for ICM |
/opt/ibm/wlp/usr/servers/defaultServer/lib/bootstrap | /home/cpe_data/bootstrap | Content Platform Engine bootstrap file location

```
[root@docker_host:/]# mkdir -p /home/cpe_data/configDropins/overrides /home/cpe_data/asa /home/cpe_data/textext /home/cpe_data/logs /home/cpe_data/FileNet /home/cpe_data/icmrules /home/cpe_data/bootstrap
```


#### 2.6 Generate the bootstrap properties.
A bootstrap JAR file is generated so that the Content Platform Engine container can start correctly. This file contains login and GCD info that mimics the information that you set up during section 2.2 and 2.3.<br>
Ensure you are in the same directory as the `BootstrapConfig.jar` and `BootstrapConfigProps.jar` files that you downloaded in step 2.1. Replace the following values in this command to match your configuration:<br>

- GCDDS: The jndiName that you used for the non-XA GCD data source in FNGCDDS.xml
- GCDDSXA: The jndiName that you used for the XA GCD data source in FNGCDDS.xml
- bootstrap-user: The short username that you used for bindDN in ldap.xml
- bootstrap-user-password: The password that you used for bindPassword in ldap.xml

```
[root@docker_host:/tmp/cpeFiles]# java -Dfile.encoding=utf-8 -jar BootstrapConfig.jar -s <GCDDS> -x <GCDDSXA> -u <bootstrap-user> -p <bootstrap-user-password> -e BootstrapConfigProps.jar -b 256 -c AES -k -o true
```

With the example values, the command looks like this:

```
[root@docker_host:/tmp/cpeFiles]# java -Dfile.encoding=utf-8 -jar BootstrapConfig.jar -s FNGCDDS -x FNGCDDS -u P8Admin -p IBMFileNetP8 -e BootstrapConfigProps.jar -b 256 -c AES -k -o true
```

This command modifies BootstrapConfigProps.jar with your information. Then you must extract the updated props.jar file from BootstrapConfigProps.jar:

```
[root@docker_host:/tmp/cpeFiles]# jar xvf BootstrapConfigProps.jar APP-INF/lib/props.jar
```

This command extracts APP-INF/lib/props.jar to the current directory. Copy the props.jar file to the bootstrap host directory that you created in section 2.1:

```
[root@docker_host:/tmp/cpeFiles]# cp APP-INF/lib/props.jar /home/cpe_data/bootstrap/
```


#### 2.7 Copy configuration files to host directory and set security.

Copy the configuration files that you generated, the JBDC driver, and the JBDC license into the `/home/cpe_data/configDropins/overrides` host directory.<br><br>If you changed the base directory earlier in this step, use that instead of /home/cpe_data in the following examples.

```
[root@docker_host:/tmp/cpeFiles]# cp {ldap.xml,DB2JCCDriver.xml,FNGCDDS.xml,OS1DS.xml,db2jcc4.jar,db2jcc_license_cu.jar} /home/cpe_data/configDropins/overrides/
```

Change owner permission on the host directories.
Since the image is already security hardened and runs services as a non-root user (`uid=501` and `gid=500`), you must set user and group ownership on these host directories:

```
[root@docker_host:/home/cpe_data]# chown -R 501:500 /home/cpe_data/
```

Validate your directories:

```
[root@docker_host:/]# cd /home/cpe_data/
[root@docker_host:/home/cpe_data]# find . -type d -print0|xargs -0 ls -ld
drwxr-xr-x 9 501 500 4096 Nov  9 13:12 .
drwxr-xr-x 2 501 500 4096 Nov  9 13:12 ./asa
drwxr-xr-x 2 501 500 4096 Nov  9 13:18 ./bootstrap
drwxr-xr-x 3 501 500 4096 Nov  9 13:12 ./configDropins
drwxr-xr-x 2 501 500 4096 Nov  9 13:17 ./configDropins/overrides
drwxr-xr-x 2 501 500 4096 Nov  9 13:12 ./FileNet
drwxr-xr-x 2 501 500 4096 Nov  9 13:12 ./icmrules
drwxr-xr-x 2 501 500 4096 Nov  9 13:12 ./logs
drwxr-xr-x 2 501 500 4096 Nov  9 13:12 ./textext

```

Validate your files:

```
[root@docker_host:/]# cd /home/cpe_data/
[root@docker_host:/home/cpe_data]# find . -type f -print0|xargs -0 ls -l
-rw-r--r-- 1 501 500   19675 Nov  9 13:18 ./bootstrap/props.jar
-rwxr-xr-x 1 501 500 3700005 Nov  9 13:17 ./configDropins/overrides/db2jcc4.jar
-rw-r--r-- 1 501 500     221 Nov  9 13:17 ./configDropins/overrides/DB2JCCDriver.xml
-rwxr-xr-x 1 501 500    1015 Nov  9 13:17 ./configDropins/overrides/db2jcc_license_cu.jar
-rw-r--r-- 1 501 500     929 Nov  9 13:17 ./configDropins/overrides/FNGCDDS.xml
-rw-r--r-- 1 501 500     560 Nov  9 13:17 ./configDropins/overrides/ldap.xml
-rw-r--r-- 1 501 500     871 Nov  9 13:17 ./configDropins/overrides/OS1DS.xml
```

# Quickstart

## 1. Pull the Content Platform Engine Docker image

Use the following Docker commands with your credentials to log in and pull the Docker image:
- ```docker login -u mydockerid -p mydockerpw```
- ```docker pull ibmcorp/filenet_content_platform_engine:earlyadopters-gm5.5```

## 2. Set the environment variables (optional)
These optional variables can be set on your Docker container. You can pass these to the container when you run the image by using additional ```-e FLAG=value``` arguments.

<table style="width:100%">
  <tr>
    <th>Name</th>
    <th>Description</th>
    <th>Required</th>
    <th>Default Value</th>
  </tr>
  <tr>
    <td>JVM_HEAP_XMS</td>
    <td>Initial Java heap size</td>
    <td>NO</td>
    <td>512m</td>
  </tr>
  <tr>
    <td>JVM_HEAP_XMX</td>
    <td>Maximum Java heap size</td>
    <td>NO</td>
    <td>1024m</td>
  </tr>
   <tr>
    <td>TZ</td>
    <td>Time Zone, take https://en.wikipedia.org/wiki/List_of_tz_database_time_zones as a reference for expected TZ.</td>
    <td>NO</td>
    <td>Etc/UTC</td>
  </tr>
  <tr>
    <td>CONTAINERTYPE</td>
    <td>The type of container</td>
    <td>NO</td>
    <td>1</td>
  </tr>
  <tr>
    <td>CPESTATICPORT</td>
    <td>If you set <b>CPESTATICPORT=true</b>, add <b>-p 38241:38241</b> in your Docker run command.</td>
    <td>NO</td>
    <td>false</td>
  </tr>
  </table>

For monitoring environment variables, check [ECM Monitoring Github](https://github.ibm.com/ecm-container-service/ecm-container-monitoring#environment-variables).

## 3. Run the container in the Docker environment

A Linux host or virtual machine with Docker engine installed is needed to run this image. You can refer to [the Docker installation information](https://docs.docker.com/engine/installation/) for details.
The following command, which uses sample values, runs the Content Platform Engine container:  

```
docker run -d --name cpe -p 9080:9080 -p 9443:9443 --hostname=hikes1 -v /home/cpe_data/asa:/opt/ibm/asa -v /tmp/data/textext:/opt/ibm/textext -v /home/cpe_data/icmrules:/opt/ibm/icmrules -v /home/cpe_data/logs:/opt/ibm/wlp/usr/servers/defaultServer/logs -v /home/cpe_data/FileNet:/opt/ibm/wlp/usr/servers/defaultServer/FileNet -v /home/cpe_data/configDropins/overrides:/opt/ibm/wlp/usr/servers/defaultServer/configDropins/overrides -v /home/cpe_data/bootstrap:/opt/ibm/wlp/usr/servers/defaultServer/lib/bootstrap ecm-containerization-docker-local.artifactory.swg-devops.com/cpe:stable
```

Verify the Content Platform Engine deployment http://your-host-ip:9080/cpe or https://your-host-ip:9443/cpe 

# Usage

## Run CPE container with monitoring

Connect to Bluemix metrics service by using IBM Cloud monitoring metrics writer for space or organization scope and connect to Bluemix logging service using Bluemix multi-tenant lumberjack writer:

```
docker run -d -e MON_METRICS_WRITER_OPTION=2 -e MON_METRICS_SERVICE_ENDPOINT=metrics.ng.bluemix.net:9095 -e MON_BMX_GROUP=com.ibm.ecm.monitor. -e MON_BMX_METRICS_SCOPE_ID={space or organization guid} -e MON_BMX_API_KEY={IAM API key} -e MON_LOG_SHIPPER_OPTION=2 -e MON_BMX_SPACE_ID={tenant id} -e MON_LOG_SERVICE_ENDPOINT=logs.opvis.bluemix.net:9091 -e MON_BMX_LOGS_LOGGING_TOKEN={log logging token} --name cpe -p 9080:9080 -p 9443:9443 --hostname=hikes1 -v /home/cpe_data/asa:/opt/ibm/asa -v /tmp/data/textext:/opt/ibm/textext -v /home/cpe_data/icmrules:/opt/ibm/icmrules -v /home/cpe_data/logs:/opt/ibm/wlp/usr/servers/defaultServer/logs -v /home/cpe_data/FileNet:/opt/ibm/wlp/usr/servers/defaultServer/FileNet -v /home/cpe_data/configDropins/overrides:/opt/ibm/wlp/usr/servers/defaultServer/configDropins/overrides -v /home/cpe_data/bootstrap:/opt/ibm/wlp/usr/servers/defaultServer/lib/bootstrap ecm-containerization-docker-local.artifactory.swg-devops.com/cpe:stable
```

## Run the IBM Content Platform Engine on Kubernetes.  

1. Prepare persistence volumes:  

Refer to [kubernetes document](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) for information on persistence volume preparation.

2. Create a persistence volume claim ([sample YAML file for create PVC](https://github.ibm.com/ecm-container-service/navigator-docker/tree/master/examples/ecmcfgstore.yml)):
```
kubectl apply ecmcfgstore.yml
```
Use the following command to check persistence volume information:
```
kubectl describe pvc <PVC_NAME>
```
You can use the command to bind the persistence volume name to this persistence volume claim.

3. Provision data in the storage volume.
- Mount the configuration storage on the Kubernetes client:
```
mkdir /cfgstore
mount -t nfs4 -o hard,intr <PV_HOST>:/<PV_FOLDER> /cfgstore
```
Where PV_HOST is the server name of the persistence volume, and PV_FOLDER is the path for the persistence volume. You can check these values by running the following command:
```
kubectl describe pv <PV_NAME>  
```

- Create the following folders under /cfgstore
```
    /cfgstore/cpe/configDropins/overrides

    /cfgstore/cpe/asa

    /cfgstore/cpe/textext

    /cfgstore/cpe/logs

    /cfgstore/cpe/FileNet

    /cfgstore/cpe/icmrules

    /cfgstore/cpe/bootstrap

```
- Follow section 2.6 to generate the props.jar and copy it into /cfgstore/cpe/bootstrap folder.Follow section 2.7 copy the configuration files that you generated, the JBDC driver, and the JBDC license into the `/cfgstore/cpe/configDropins/overrides` folder.

4. Deploy IBM Content Platform Engine ([sample YAML file for deploy CPE](https://github.ibm.com/ecm-container-service/cpe-docker/tree/master/examples/cpe-deploy.yaml)).

```
kubectl create -f cpe-deploy.yaml
```

# Support
Support can be obtained at [IBM® DeveloperWorks Answers](https://developer.ibm.com/answers/)
<br>
Use the ECM-CONTAINERS tag and assistance will be provided.<br>
*Note: Limited support available during Early Adopter Program*
