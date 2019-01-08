# nifi-impala
Examples for integrating NiFi and Impala

## Download the Cloudera QuickStart VM
- These instructions are based on using the Docker version of the QuickStarts for CDH 5.13
- The CDH 5.13 QuickStart VM Docker image can be downloaded from: https://www.cloudera.com/downloads/quickstart_vms/5-13.html

## Starting the Cloudera QuickStart VM
- Please use the "Getting Started" docker documentation at: https://www.cloudera.com/documentation/enterprise/5-13-x/topics/quickstart_docker_container.html
- Documentation for administering the cluster, including usernames and passwords: https://www.cloudera.com/documentation/enterprise/5-13-x/topics/quickstart_vm_administrative_information.html
- Example to start the QuickStart container
  - `docker run --hostname=quickstart.cloudera --name=quickstart.cloudera --privileged=true -t -i -d -2701f6c9ad88 /usr/bin/docker-quickstart`
- Start Cloudera Manager.  Depending on your host, it may take a while to start.
  - `docker exec quickstart.cloudera /home/cloudera/cloudera-manager --express`
- Make sure an entry for `quickstart.cloudera` is in `/etc/hosts` that resolves to the QuickStart container
- Watch the containers logs: `docker logs quickstart.cloudera`

## Download the Impala JDBC driver
- As of the creation of this README, 2.6.4
- The driver can be downloaded from: https://www.cloudera.com/downloads/connectors/impala/jdbc/2-6-4.html

## Add the Impala JDBC driver to NiFi
1. Once the driver has been downloaded, unzip `impala_jdbc_2.6.4.1005.zip` in a directory to which the user running NiFi has access. Multiple zips and a docs folder will be created after extraction, in a directory named `ClouderaImpalaJDBC-2.6.4.1005`
1. cd to `ClouderaImpalaJDBC-2.6.4.1005` and extract `ClouderaImpalaJDBC41-2.6.4.1005.zip`, which will result in `ImpalaJDBC41.jar` being created in `ClouderaImpalaJDBC-2.6.4.1005`.
1. For simplicity and ease of use with the NiFi template `nifi-impala-template`, copy `ImpalaJDBC41.jar` to the nifi lib dir.
1. Restart NiFi.

## Example Flow
Import `nifi-impala-template` into NiFi, which will create a process group called `NiFi Impala Integration`.  Within that process groups are several process groups that demonstrate various examples of retrieving data from external sources and creating tables in Impala to make that data queryable.

## TODO
- Provide examples that integrate with a kerberized Impala
