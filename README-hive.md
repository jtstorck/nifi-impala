## Hive Integration
### Build NiFI with the Hive 1.1 profile
mvn clean install -Pinclude-hive1_1

### Import `nifi-impala-integration` template
The "Randomuser Hive 1.1" process group contains a `Hive_1_1ConnectionPool` controller service configured to access Hive on the QUickstart VM, secured by Kerberos.

### Add `cloudera` to the list of `Allowed System Users`
1. Go to: http://quickstart.cloudera:7180/cmf/home
1. Click on `Yarn` in the services list.
1. Click the `Configuration` tab.
1. Click the `Security` properties group.
1. In `Allowed System Users`, add `cloudera`.
1. Restart Yarn with the config redeploy option checked.

### Download the Hive client configs
1. Go to: http://quickstart.cloudera:7180/cmf/home
1. Click on `Hive` in the services list.
1. Click the `Actions` dropdown, and select `Download Client Configuration`
1. Unzip `hive-clientconfig.zip`, and move the `hive-conf` dir to a directory to which the user running NiFi has read access.
1. Go into the `NiFi Impala Integration` process group.
1. Right click on the canvas, and update the `hive-conf` variable with the path to the directory containing the Hive client configs.

### Enable Kerberos on the QuickStart VM
Instructions for enabling Kerberos on the QuickStart VM are available here: [Enabling Kerberos on the QuickStart VM](https://github.com/jtstorck/nifi-impala/blob/master/README-kerberos.md#enabling-kerberos-on-the-quickstart-vm)

### Enable the controller services
1. In the `NiFi Impala Integration` process group, open the `Configure` dialog.
1. Enable the `KeytabCredentialsService` controller service
1. Enable the `Hive_1_1ConnectionPool` controller service
   - The `Database Connection URL` property is set to `jdbc:hive2://quickstart.cloudera:10000/default;principal=hive/_HOST@CLOUDERA.COM` by default.
1. Enable the `Randomuser CSV Writer` controller service
1. Enable the `Randomuser JSON Reader` controller service
