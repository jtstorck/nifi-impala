# nifi-impala
Examples for integrating NiFi and Impala

## Download the Cloudera QuickStart VM
- These instructions are based on using the Docker version of the QuickStarts for CDH 5.13
- The CDH 5.13 QuickStart VM Docker image can be downloaded from: https://www.cloudera.com/downloads/quickstart_vms/5-13.html

## Starting the Cloudera QuickStart VM
- Please use the "Getting Started" docker documentation at: https://www.cloudera.com/documentation/enterprise/5-13-x/topics/quickstart_docker_container.html
- Documentation for administering the cluster, including usernames and passwords: https://www.cloudera.com/documentation/enterprise/5-13-x/topics/quickstart_vm_administrative_information.html
- Example to start the QuickStart container
  - Linux
    - `docker run --hostname=quickstart.cloudera --name=quickstart.cloudera --privileged=true -t -i -d [IMAGE] /usr/bin/docker-quickstart`
  - Docker For Mac
    - `docker run --hostname=quickstart.cloudera --name=quickstart.cloudera --privileged=true -t -i -d -p 7051:7051 -p 7180:7180 -p 21050:21050 -p 50070:50070 -p TBD:TBD [IMAGE] /usr/bin/docker-quickstart`
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
- Create and enable a DistributedMapCacheServer controller service, using the property defaults.
  - This is due to https://issues.apache.org/jira/browse/NIFI-1293, controller services and reporting tasks cannot be exported to a flow because no components explicitly reference them.
- Enable all controller services from the `NiFi Impala Integration` process group's `Configuration`, under `Controller Services`.

## Download the HDFS client configs
1. Go to: http://quickstart.cloudera:7180/cmf/home
1. Click on `HDFS` in the services list.
1. Click the `Actions` dropdown, and select `Download Client Configuration`
1. Unzip `hdfs-clientconfig.zip`, and move the `hadoop-conf` dir to a directory to which the user running NiFi has read access.
1. Go into the `NiFi Impala Integration` process group.
1. Right click on the canvas, and update the `hadoop-conf` variable with the path to the directory containing the HDFS client configs.

---

Please see [Enabling Kerberos on the QuickStart VM](https://github.com/jtstorck/nifi-impala/blob/master/README-kerberos.md#enabling-kerberos-on-the-quickstart-vm) to configure the QuickStart VM to be secured by a KDC.

---

Please see [README-hive.md](https://github.com/jtstorck/nifi-impala/blob/master/README-hive.md) for instructions for using Hive 1.1 with NiFi.

---

## Troubleshooting
### Host Health issues
- `ntp` may not be started, or may not be set to start at boot.
  - `docker exec quickstart.cloudera service ntpd restart`
### Docker For Mac
- It may be easier to run NiFi on the QuickStart VM to avoid needing to map ports to localhost.
  - Docker has an internal SOCKS proxy (an experimental feature) as of 18.03.0-ce-rc2-mac56 (23206) that can be used to allow access to the container by its name
    - Portmapping in the `docker run` command isn't needed make HTTP ports available to localhost
    - This [github/docker/for-mac issue comment](https://github.com/docker/for-mac/issues/2670#issuecomment-372365274) provides instructions for setting up the experimental SOCKS proxy feature
    - Rather than adding the SOCKS proxy at the OS level, an alternate web browser from the one typically used can be configured with the SOCKS proxy settings.
    - UIs for Cloudera Manager, NiFi, etc, will be available at `http://quickstart.cloudera:[PORT]`
- If NiFi is not running on the QuickStart VM
  - References to the hostname `quickstart.cloudera` in the `NiFi Impala Integration` template will need to be replaced with `localhost`.
  - Several ports should be added to the `docker run` command when starting the QuickStart container.  More information about ports used by Cloudera QuickStart can be found at: https://www.cloudera.com/documentation/enterprise/5-13-x/topics/cm_ig_ports.html
    - `-p 7051:7051` (Kudu Master)
    - `-p 7180:7180` (Cloudera Manager UI)
    - `-p 21050:21050` (Impala Daemon Frontend)
    - `-p 50070:50070` (HDFS HTTP NameNode)
    - TBD

## TODO
- Set up a proxy container to allow resolution to the `quickstart.cloudera` container
- Scripts to speed up configuration, starting, stopping the containers.
