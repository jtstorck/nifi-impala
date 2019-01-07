# nifi-impala
Examples for integrating NiFi and Impala

## Download the Impala JDBC driver
- As of the creation of this README, 2.6.4
- The driver can be downloaded from: https://www.cloudera.com/downloads/connectors/impala/jdbc/2-6-4.html

## Add the Impala JDBC driver to NiFi
1. Once the driver has been downloaded, unzip "impala_jdbc_2.6.4.1005.zip" in a directory to which the user running NiFi has access. Multiple zips and a docs folder will be created after extraction, in a directory named "ClouderaImpalaJDBC-2.6.4.1005"
1. cd to "ClouderaImpalaJDBC-2.6.4.1005" and extract "ClouderaImpalaJDBC41-2.6.4.1005.zip", which will result in "ImpalaJDBC41.jar" being created in "ClouderaImpalaJDBC-2.6.4.1005".
1. For simplicity and ease of use with the NiFi template, copy ImpalaJDBC41.jar to the nifi lib dir.
1. Restart NiFi.

## Example Flow
Import nifi-impala-template into NiFi, which will create a process group called "NiFi Impala Integration".  Within that process groups are several process groups that demonstrate various examples of retrieving data from external sources and creating tables in Impala to make that data queryable.
