# nifi-impala
Examples for integrating NiFi and Impala

## Download the Cloudera QuickStart VM
- These instructions are based on using the Docker version of the QuickStarts for CDH 5.13
- The CDH 5.13 QuickStart VM Docker image can be downloaded from: https://www.cloudera.com/downloads/quickstart_vms/5-13.html

## Starting the Cloudera QuickStart VM
- Please use the "Getting Started" docker documentation at: https://www.cloudera.com/documentation/enterprise/5-13-x/topics/quickstart_docker_container.html
- Documentation for administering the cluster, including usernames and passwords: https://www.cloudera.com/documentation/enterprise/5-13-x/topics/quickstart_vm_administrative_information.html
- Example to start the QuickStart container
  - `docker run --hostname=quickstart.cloudera --name=quickstart.cloudera --privileged=true -t -i -d [IMAGE] /usr/bin/docker-quickstart`
- Start Cloudera Manager.  Depending on your host, it may take a while to start.
  - `docker exec quickstart.cloudera /home/cloudera/cloudera-manager --express`
- Make sure an entry for `quickstart.cloudera` is in `/etc/hosts` that resolves to the QuickStart container's IP.
- Watch the containers logs: `docker logs quickstart.cloudera`

## After starting Cloudera Manager
- From the Cloudera Manager main page, click the `Cloudera QuickStart ...` dropdown and click `Start` to make sure all the services are started.

## Download the Impala JDBC driver
- As of the creation of this README, 2.6.4
- The driver can be downloaded from: https://www.cloudera.com/downloads/connectors/impala/jdbc/2-6-4.html

## Add the Impala JDBC driver to NiFi
1. Once the driver has been downloaded, unzip `impala_jdbc_2.6.4.1005.zip`. Multiple zips and a docs folder will be created after extraction, in a directory named `ClouderaImpalaJDBC-2.6.4.1005`.
1. In the `ClouderaImpalaJDBC-2.6.4.1005` directory, extract `ClouderaImpalaJDBC41-2.6.4.1005.zip`, which will result in `ImpalaJDBC41.jar` being created in `ClouderaImpalaJDBC-2.6.4.1005`.
1. For simplicity and ease of use with the NiFi template `nifi-impala-integration.xml`, copy `ImpalaJDBC41.jar` to the NiFi lib dir.
1. Restart NiFi.

## Example Flow
Import template `nifi-impala-integration.xml` into NiFi, which will create a process group called `NiFi Impala Integration`.  Within that process group are several process groups that demonstrate various examples of retrieving data from external sources and creating tables in Impala to make that data queryable.
- Enable all controller services from the `NiFi Impala Integration` process group's `Configuration`, under `Controller Services`.

## Download the HDFS client configs
1. Go to: http://quickstart.cloudera:7180/cmf/home
1. Click on `HDFS` in the services list.
1. Click the `Actions` dropdown, and select `Download Client Configuration`
1. Unzip `hdfs-clientconfig.zip`, and move the `hadoop-conf` dir to a directory to which the user running NiFi has read access.
1. Go into the `NiFi Impala Integration` process group.
1. Right click on the canvas, and update the `hadoop-conf` variable with the path to the directory containing the HDFS client configs.

## Troubleshooting
### Host Health issues
  - `ntp` may not be started, or may not be set to start at boot.
    - `service ntpd restart`
### Docker For Mac
  - Several ports should be added to the `docker run` command when starting the QuickStart container.  More information about ports used by Cloudera QuickStart can be found at: https://www.cloudera.com/documentation/enterprise/5-13-x/topics/cm_ig_ports.html
    - `-p 7051:7051` (Kudu Master)
    - `-p 7180:7180` (Cloudera Manager UI)
    - `-p 21050:21050` (Impala Daemon Frontend)
    - `-p 50070:50070` (HDFS HTTP NameNode)
    - TBD

## TODO
- Provide examples that integrate with a kerberized Impala
