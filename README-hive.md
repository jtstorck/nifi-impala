## Hive Integration
### Build NiFI with the Hive 1.1 profile
mvn clean install -Pinclude-hive1_1

### Import `nifi-impala-integration` template
The "Randomuser Hive 1.1" process group contains a `Hive_1_1ConnectionPool` controller service configured to access Hive on the QUickstart VM, secured by Kerberos.

### Download the Hive client configs
1. Go to: http://quickstart.cloudera:7180/cmf/home
1. Click on `Hive` in the services list.
1. Click the `Actions` dropdown, and select `Download Client Configuration`
1. Unzip `hive-clientconfig.zip`, and move the `hive-conf` dir to a directory to which the user running NiFi has read access.
1. Go into the `NiFi Impala Integration` process group.
1. Right click on the canvas, and update the `hive-conf` variable with the path to the directory containing the Hive client configs.

### Enable Kerberos on the QuickStart VM
Instructions for enabling Kerberos on the QuickStart VM are available here: [Enabling Kerberos on the QuickStart VM](https://github.com/jtstorck/nifi-impala/blob/master/README-kerberos.md#enabling-kerberos-on-the-quickstart-vm)
### Enable the `Hive_1_1ConnectionPool` controller service
In the `NiFi Impala Integration` process group, open the `Configure` dialog.
Enable the `Hive_1_1ConnectionPool` service
  - The `Database Connection URL` property is set to `jdbc:hive2://quickstart.cloudera:10000/default;principal=hive/_HOST@CLOUDERA.COM` by default.
