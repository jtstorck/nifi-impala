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
1. Modify the `Impala JDBC` controller service configuration to use the `KeytabCredentialsService`
   - Verify the config below is set on the `KeytabCredentialsService`.  If one does not exist, create it with the following settings
   - `Kerberos Principal` set to `cloudera@CLOUDERA`
   - `Kerberos Keytab` set to `/tmp/cloudera.keytab`
1. Enable the `Impala JDBC` and `KeytabCredentialService` controller services.

### Set permissions on HDFS directory `/tmp/randomuser`
1. Set the permissions on the HDFS directory storing the Randomuser.me data:
   - `kinit hdfs@CLOUDERA`
   - `hdfs dfs -chmod a+w /tmp/randomuser`

### Update component configs
1. `Impala JDBC` DBCPConnectionPool controller service
   * Update the `Database Connection URL` property to: `jdbc:impala://quickstart.cloudera:21050/;AuthMech=1;KrbRealm=CLOUDERA;KrbHostFQDN=quickstart.cloudera;KrbServiceName=impala`
   * Update the `Kerberos Credentials Service` property to use `KeytabCredentialsService`
1. Update the `PutKudu` config to use the `KerberosCredentialsService` controller service

## Troubleshooting
### Kerberos Issues
- `krb5kdc` may not be running
  - `docker exec quickstart.cloudera service krb5kdc restart`
- Inability to authenticate with KDC due to supported encryption algorithms
  - Make sure the JCE Unlimited Strength polices are in place for the Java 7 JRE used by the QuickStart VM: `/usr/java/jdk1.7.0_67-cloudera/jre/lib/security`
- Client configurations may need to be redeployed once krb5kdc, and at least Zookeeper is succesfully able to be started.
### QuickStart VM Kerberos wizard leaves cluster in inoperable state
- The Kerberos wizard requires that services be installed from parcels, rather than packages.  The QuickStart VM 5.13 (at least for the Docker image) has components installed as packages.  Downloading and activating the 'cdh' parcel should resolve this issue.
