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
- Enable all controller services from the `NiFi Impala Integration` process group's `Configuration`, under `Controller Services`.

## Download the HDFS client configs
1. Go to: http://quickstart.cloudera:7180/cmf/home
1. Click on `HDFS` in the services list.
1. Click the `Actions` dropdown, and select `Download Client Configuration`
1. Unzip `hdfs-clientconfig.zip`, and move the `hadoop-conf` dir to a directory to which the user running NiFi has read access.
1. Go into the `NiFi Impala Integration` process group.
1. Right click on the canvas, and update the `hadoop-conf` variable with the path to the directory containing the HDFS client configs.

---

## Enabling Kerberos on the QuickStart VM
### Update the QuickStart VM
A blog detailing the setup is available at: https://blog.cloudera.com/blog/2015/03/how-to-quickly-configure-kerberos-for-your-apache-hadoop-cluster/  
You may want to review the installation script, there are various comments describing the actions invoked by the script: https://github.com/git4impatient/quickKerberos/blob/master/goKerberos_beforeCM.sh  
In summary, these instructions can be followed to enable Kerberos on the QuickStart VM:
1. The JCU Unlimited Strength policies should be downloaded from https://www.oracle.com/technetwork/java/javase/downloads/jce-7-download-432124.html, and `UnlimitedJCEPolicyJDK7.zip` should placed in `/root` on the QuickStart VM container.
1. Make sure that Cloudera Manager is running
1. As root, on the QuickStart VM container, run the following script: https://github.com/git4impatient/quickKerberos/blob/master/goKerberos_beforeCM.sh
   - This can be downloaded from Github and placed in the `/root` directory of the QuickStart VM.  Make sure to set the execute bit so the script can run.
1. Log into Cloudera Manager
1. Click on `Administration` -> `Security` -> `Enable Kerberos`
1. Check the four boxes on the first page of the wizard, then click `Continue`
1. On the next page, supply the following values, then click `Continue`
   - Kerberos Security Realm: `CLOUDERA`
   - KDC Server Host: `quickstart.cloudera`
   - Kerberos Encryption Types: `aes256-cts-hmac-sha1-96`
1. On the next page, check the box for `Manage krb5.conf through Cloudera Manager`, then click `Continue`
1. On the next page, `KDC Account Manager Credentials`, enter `cloudera-scm/admin@CLOUDERA` for the username, `cloudera` for the password, then click `Continue`
1. The `Import KDC Account Manager Credentials Command` will run for a bit and then should finish successfully, click `Continue`
1. On the `Kerberos Principal` page, leave the default values and click `Continue`
1. On the `Configure Ports` page, leave the default values and check the box for `Yes, I am ready to restart the cluster now`, then click `Continue`
1. Wait for the steps to complete
1. Test Kerberos by attaching to the QuickStart VM container, or using `docker exec -ti quickstart.cloudera bash`:
   - `kinit hdfs@CLOUDERA`
   - `hadoop fs -mkdir /eraseme`
   - `hadoop fs -rmdir /eraseme`
   - `kdestroy`
    
### Copy `cloudera` Keytab and krb5.conf to the NiFi host
1. Export the cloudera principal's keytab from quickstart.cloudera kdc 
   - `xst -k /root/cloudera.keytab cloudera@CLOUDERA`
1. Copy `/root/cloudera.keytab` from the QuickStart VM to the host path `/tmp/cloudera.keytab`
1. Copy `/etc/krb5.conf` from the QuickStart VM to the host path `/tmp/krb5.conf`

### Configure NiFi to use the KDC installed on the QuickStart VM
1. Set the `nifi.kerberos.krb5.file` property in nifi.properties to `/tmp/krb5.conf`
1. Download HDFS client configs after kerberizing, as defined in [Download the HDFS client configs](README.md#Download-the-HDFS-client-configs). 
1. Restart NiFi
1. Verify the `hadoop-conf` variable in the `NiFi Impala Integration` process group is set to the path to the directory containing the kerberos-enabled HDFS client configs. 
1. Modify the `Impala JDBC` controller service configuration to use a `KeytabCredentialsService`
   - `Kerberos Principal` set to `cloudera@CLOUDERA`
   - `Kerberos Keytab` set to `/tmp/cloudera.keytab`
1. Enable the `Impala JDBC` and `KeytabCredentialService` controller services.

### Set permissions on HDFS directory `/tmp/randomuser`
1. Set the permissions on the HDFS directory storing the Randomuser.me data:
   - `kinit hdfs@CLOUDERA`
   - `hdfs dfs -chmod a+w /tmp/randomuser`

### Update the `Impala JDBC` DBCPConnectionPool controller service config
Update the `Database Connection URL` property to: `jdbc:impala://quickstart.cloudera:21050/;AuthMech=1;KrbRealm=CLOUDERA;KrbHostFQDN=quickstart.cloudera;KrbServiceName=impala`

## Troubleshooting
### Host Health issues
- `ntp` may not be started, or may not be set to start at boot.
  - `docker exec quickstart.cloudera service ntpd restart`
### Kerberos Issues
- `krb5kdc` may not be running
  - `docker exec quickstart.cloudera service krb5kdc restart`
- Inability to authenticate with KDC due to supported encryption algorithms
  - Make sure the JCE Unlimited Strength polices are in place for the Java 7 JRE used by the QuickStart VM: `/usr/java/jdk1.7.0_67-cloudera/jre/lib/security`
- Client configurations may need to be redeployed once krb5kdc, and at least Zookeeper is succesfully able to be started.
### QuickStart VM Kerberos wizard leaves cluster in inoperable state
- The Kerberos wizard requires that services be installed from parcels, rather than packages.  The QuickStart VM 5.13 (at least for the Docker image) has components installed as packages.  Downloading and activating the 'cdh' parcel should resolve this issue.
### Docker For Mac
- Several ports should be added to the `docker run` command when starting the QuickStart container.  More information about ports used by Cloudera QuickStart can be found at: https://www.cloudera.com/documentation/enterprise/5-13-x/topics/cm_ig_ports.html
  - `-p 7051:7051` (Kudu Master)
  - `-p 7180:7180` (Cloudera Manager UI)
  - `-p 21050:21050` (Impala Daemon Frontend)
  - `-p 50070:50070` (HDFS HTTP NameNode)
  - TBD
## TODO
- Provide examples that integrate with a kerberized Impala
