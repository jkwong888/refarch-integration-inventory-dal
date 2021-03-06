
# The Liberty server configuration
The goal of this article is to give you some details on the server configuration. The source definition for server.xml can be found under src/main/liberty/config folder.

## Download and install

* Download the server library from [WAS dev](https://developer.ibm.com/wasdev/downloads/download-latest-stable-websphere-liberty-runtime/)

* Install additional features like the Admin Center [optional] using the command:
```
$WLP_HOME/bin/installUtility install adminCenter-1.0
```
* Create a server named appServer
```
$WLP_HOME/bin/server create appServer
```
* Modify the server.xml or copy the one from this project src/main/liberty/config folder, to define the webapp and datasources

## Datasource configuration

To expose SOAP interface and use JPA as a data access layer we need to add the following features
```
<feature>jpa-2.0</feature>
<feature>jaxws-2.2</feature>
```
The following commands were done:
```
cd wlp/bin
./server create appServer
./installUtility install adminCenter-1.0
./installUtility install jaxws-2.2
./installUtility install jpa-2.0
```

To avoid changing configuration and parameters we set static port definition, for the HTTP end point as
```
<httpEndpoint httpPort="9080" httpsPort="9443"
    id="defaultHttpEndpoint" />
```
As the project is using a single DB2 database instance the data source is defined to use db2 jdbc driver and to control the connection pool size so failover and testing can be done on capacity.
```
<library id="DB2JCC4Lib">
   <fileset dir="${shared.config.dir}/lib" includes="db2jcc4.jar"/>
</library>
<dataSource id="db2" jndiName="jdbc/invdbds" type="javax.sql.ConnectionPoolDataSource">
     <connectionManager maxPoolSize="8" minPoolSize="2"
                       connectionTimeout="5s" agedTimeout="10m"/>
     <jdbcDriver libraryRef="DB2JCC4Lib"/>
     <properties.db2.jcc databaseName="INVDB" portNumber="50000" serverName="172.16.254.xx"
           currentLockTimeout="30s" user="DB2INST1" password="xxxxx"/>
   </dataSource>
```
You will need to adapt the IP address and password.

## Security
The firewall was disabled using
```
# See the status of firewall:
$ systemctl status firewalld

$ service firewalld stop

$ systemctl disable firewalld
```

## Development environment setup

As root user git was installed as well as JDK 1.8. (The JDK was extracted under home/admin/IBM
```
yum install git
yum list installed java\*
```
The admin account was modified by adding the following exports into .bashrc file
```
export JAVA_HOME=~/IBM/jdk1.8.0_131
export PATH=$JAVA_HOME/bin:$PATH
```

The first time the server was set up the following script was executed to update the server.xml and add the DB2 JDBC driver jar to the liberty shared config folder.
```
./updateAppServer.sh
```

## Server start and test

Under the ~/IBM/wlp/bin do ./server start appServer --clean then locally point to the URL http://localhost:9080/inventory/ws?wsdl to access the web app SOAP operation definition.
