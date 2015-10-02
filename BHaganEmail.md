

##Why did I do this?
Just because.
To demonstrate that you can store OLAP data in HBASE and perform reporting and analytics via Phoenix in my favorite BI Tool.
To help demonstrate all of the existing reports using HBASE.

##Software Used
PDI 5.4.0.1-130
BA Server 5.4.0.1-130
HDP 2.3 Sandbox

Use PDI to copy the tables from Hypersonic to MySQL – I always like data in MySQL – you don’t have to do this because I’ve included all tables as csv. This is just here to show what I did.

Why Not Use PDI for the whole thing? - See Troubleshooting at the end

Update NULL fields in Existing SampleData Tables – Phoenix import did not like NULL values – Again, you don’t have to do this because I’ve included all tables as csv. This is just here to show what I did.
update orders set comments = IFNULL(comments, "NONE”);
update orders set shippeddate = IFNULL(shippeddate, "2020-01-01”);
update orderfact set comments = IFNULL(comments, "NONE”);
update orderfact set shippeddate = IFNULL(shippeddate, "2020-01-01”);
update offices set ADDRESSLINE2 = IFNULL(ADDRESSLINE2, "NONE”);
update employees set ADDRESSLINE2 = IFNULL(ADDRESSLINE2, "NONE”);
update employees set reportsto = IFNULL(reportsto, -1);
update customer_w_ter set employeenumber = IFNULL(employeenumber, -1);

Export SampleData Tables from MySQL to CSV Files – CSV Files Attached
select CUSTOMERNUMBER, CUSTOMERNAME,CONTACTLASTNAME,CONTACTFIRSTNAME,PHONE,CITY,STATE,POSTALCODE,COUNTRY,SALESREPEMPLOYEENUMBER,CREDITLIMIT  into outfile '/tmp/customers.csv' fields optionally enclosed by '"' terminated by ',' lines terminated by '\n' from customers;
select CUSTOMERNUMBER, CUSTOMERNAME,CONTACTLASTNAME,CONTACTFIRSTNAME,PHONE,CITY,STATE,POSTALCODE,COUNTRY,EMPLOYEENUMBER,CREDITLIMIT,TERRITORY,TERRITORY_COLOR  into outfile '/tmp/customer_w_ter.csv' fields optionally enclosed by '"' terminated by ',' lines terminated by '\n' from customer_w_ter;
select REGION, MANAGER_NAME, EMAIL into outfile '/tmp/department_managers.csv' fields terminated by ',' lines terminated by '\n' from department_managers;
select TIME_ID,MONTH_ID,QTR_ID,YEAR_ID,MONTH_NAME,MONTH_DESC,QTR_NAME,QTR_DESC  into outfile '/tmp/dim_time.csv' fields optionally enclosed by '"' terminated by ',' lines terminated by '\n' from dim_time;
select EMPLOYEENUMBER, LASTNAME, FIRSTNAME, EXTENSION, EMAIL, OFFICECODE, REPORTSTO, JOBTITLE into outfile '/tmp/employees.csv' fields terminated by ',' lines terminated by '\n' from employees;
select officecode, city, phone, addressline1, addressline2, state, country, postalcode, territory  into outfile '/tmp/offices.csv' fields terminated by ',' lines terminated by '\n' from offices;
select ordernumber, productcode, quantityordered, priceeach, orderlinenumber into outfile '/tmp/orderdetails.csv' fields terminated by ',' lines terminated by '\n' from orderdetails;
select ordernumber, productcode, quantityordered, priceeach, orderlinenumber,totalprice,orderdate,requireddate,shippeddate,status,comments,customernumber,time_id,qtr_id,month_id,year_id into outfile '/tmp/orderfact.csv' fields optionally enclosed by '"' terminated by ',' lines terminated by '\n' from order fact;
select ordernumber, orderdate,requireddate,shippeddate,status,comments,customernumber into outfile '/tmp/orders.csv' fields optionally enclosed by '"' terminated by ',' lines terminated by '\n' from orders;
select customernumber,checknumber,paymentdate,amount into outfile '/tmp/payments.csv' fields terminated by ',' lines terminated by '\n' from payments;
select productcode,productname,productline,productscale,productvendor,productdescription,quantityinstock,buyprice,msrp into outfile '/tmp/products.csv' fields optionally enclosed by '"' terminated by ',' lines terminated by '\n' from products;

Copy the CSV Files to root’s home directory on the Sandbox
On the Mac,
cd /tmp

scp customers.csv root@sandbox.hortonworks.com:~
scp customer_w_ter.csv root@sandbox.hortonworks.com:~
scp department_managers.csv root@sandbox.hortonworks.com:~
scp dim_time.csv root@sandbox.hortonworks.com:~
scp employees.csv root@sandbox.hortonworks.com:~
scp offices.csv root@sandbox.hortonworks.com:~
scp orderdetails.csv root@sandbox.hortonworks.com:~
scp orderfact.csv root@sandbox.hortonworks.com:~
scp orders.csv root@sandbox.hortonworks.com:~
scp payments.csv root@sandbox.hortonworks.com:~
scp products.csv root@sandbox.hortonworks.com:~

Create Tables in Phoenix – Used PDI copy tables job to get the DDL – Had to change a couple data types
On the Sandbox,
cd /usr/hdp/current/phoenix-server/bin
./sqlline.py sandbox.hortonworks.com:16000/hbase

CREATE TABLE CUSTOMERS (   CUSTOMERNUMBER INTEGER PRIMARY KEY, CUSTOMERNAME VARCHAR(50) , CONTACTLASTNAME VARCHAR(50) , CONTACTFIRSTNAME VARCHAR(50) , PHONE VARCHAR(50) , ADDRESSLINE1 VARCHAR(50) , ADDRESSLINE2 VARCHAR(50) , CITY VARCHAR(50) , STATE VARCHAR(50) , POSTALCODE VARCHAR(15) , COUNTRY VARCHAR(50) , SALESREPEMPLOYEENUMBER INTEGER , CREDITLIMIT BIGINT ) ;

CREATE TABLE CUSTOMER_W_TER (   CUSTOMERNUMBER INTEGER PRIMARY KEY, CUSTOMERNAME VARCHAR(50) , CONTACTLASTNAME VARCHAR(50) , CONTACTFIRSTNAME VARCHAR(50) , PHONE VARCHAR(50) , CITY VARCHAR(50) , STATE VARCHAR(50) , POSTALCODE VARCHAR(15) , COUNTRY VARCHAR(50) , EMPLOYEENUMBER INTEGER , CREDITLIMIT BIGINT, TERRITORY VARCHAR(10), TERRITORY_COLOR VARCHAR(7) ) ;

CREATE TABLE DEPARTMENT_MANAGERS (   REGION VARCHAR(50) PRIMARY KEY, MANAGER_NAME VARCHAR(50) , EMAIL VARCHAR(50) ) ;

CREATE TABLE DIM_TIME (   TIME_ID VARCHAR(10) PRIMARY KEY, MONTH_ID  INTEGER , QTR_ID  INTEGER , YEAR_ID  INTEGER , MONTH_NAME VARCHAR(3) , MONTH_DESC VARCHAR(9) , QTR_NAME VARCHAR(4) , QTR_DESC VARCHAR(9) ) ;

CREATE TABLE EMPLOYEES (   EMPLOYEENUMBER INTEGER PRIMARY KEY, LASTNAME VARCHAR(50) , FIRSTNAME VARCHAR(50) , EXTENSION VARCHAR(10) , EMAIL VARCHAR(100) , OFFICECODE VARCHAR(20) , REPORTSTO INTEGER , JOBTITLE VARCHAR(50) ) ;

 CREATE TABLE OFFICES( OFFICECODE VARCHAR(3) PRIMARY KEY, CITY VARCHAR(20), PHONE VARCHAR(20), ADDRESSLINE1 VARCHAR(30), ADDRESSLINE2 VARCHAR(10), COUNTRY VARCHAR(15), POSTALCODE VARCHAR(10), TERRITORY VARCHAR(10));

CREATE TABLE ORDERDETAILS (   ORDERNUMBER  INTEGER PRIMARY KEY, PRODUCTCODE VARCHAR(50) , QUANTITYORDERED  INTEGER , PRICEEACH BIGINT , ORDERLINENUMBER  INTEGER ) ;

CREATE TABLE ORDERFACT (   ORDERNUMBER INTEGER PRIMARY KEY, PRODUCTCODE VARCHAR(50) , QUANTITYORDERED INTEGER , PRICEEACH DECIMAL(17) , ORDERLINENUMBER INTEGER , TOTALPRICE DECIMAL(17) , ORDERDATE  DATE , REQUIREDDATE  DATE , SHIPPEDDATE  DATE , STATUS VARCHAR(15) , COMMENTS VARCHAR(255) , CUSTOMERNUMBER INTEGER , TIME_ID VARCHAR(10) , QTR_ID BIGINT , MONTH_ID BIGINT , YEAR_ID BIGINT ) ;

CREATE TABLE ORDERS  (   ORDERNUMBER INTEGER  PRIMARY KEY, ORDERDATE  DATE , REQUIREDDATE  DATE , SHIPPEDDATE  DATE , STATUS VARCHAR(15) , COMMENTS VARCHAR(255) , CUSTOMERNUMBER INTEGER ) ;

CREATE TABLE PAYMENTS (   CUSTOMERNUMBER INTEGER PRIMARY KEY, CHECKNUMBER VARCHAR(50) , PAYMENTDATE  DATE , AMOUNT BIGINT ) ;

CREATE TABLE PRODUCTS (   PRODUCTCODE VARCHAR(50) PRIMARY KEY, PRODUCTNAME VARCHAR(70) , PRODUCTLINE VARCHAR(50) , PRODUCTSCALE VARCHAR(10) , PRODUCTVENDOR VARCHAR(50) , PRODUCTDESCRIPTION VARCHAR(10000) , QUANTITYINSTOCK INTEGER , BUYPRICE BIGINT , MSRP BIGINT ) ;


Load the data into HBASE
On the Sandbox,
cd /usr/hdp/current/phoenix-server/bin

./psql.py -t CUSTOMERS sandbox.hortonworks.com ~/customers.csv
./psql.py -t CUSTOMER_W_TER sandbox.hortonworks.com ~/customer_w_ter.csv
./psql.py -t DEPARTMENT_MANAGERS sandbox.hortonworks.com ~/department_managers.csv
./psql.py -t DIM_TIME sandbox.hortonworks.com ~/dim_time.csv
./psql.py -t EMPLOYEES sandbox.hortonworks.com ~/employees.csv
./psql.py -t OFFICES sandbox.hortonworks.com ~/offices.csv
./psql.py -t ORDERDETAILS sandbox.hortonworks.com ~/orderdetails.csv
./psql.py -t ORDERFACT sandbox.hortonworks.com ~/orderfact.csv
 ./psql.py -t ORDERS sandbox.hortonworks.com ~/orders.csv
./psql.py -t PAYMENTS sandbox.hortonworks.com ~/payments.csv
./psql.py -t PRODUCTS sandbox.hortonworks.com ~/products.csv

Install the Phoenix driver into BA Server
On the Sandbox,
scp /usr/hdp/2.3.0.0-2557/phoenix/phoenix-4.4.0.2.3.0.0-2557-client.jar bhagan@yourhost:~/Downloads/biserver-ce/tomcat/lib

Create a new BA Server Connection using the Phoenix Driver
Name: Phoenix
Type: Generic Database
Custom Connection URL: jdbc:phoenix:sandbox.hortonworks.com:16000/hbase
Custom Driver Class: org.apache.phoenix.jdbc.PhoenixDriver

Create a new BA Server Data Source - Attached Schema
Export SteelWheels Analysis DataSource. This should give you the mondrian.xml schema.
In the xml, change the schema name of the schema to HBASESampleData
In the xml, change the cube name of the schema to HBASESteelWheelsSampleData
Save the file as HBASESampleData.xml
Import the HBASESampleData.xml file

Now you can create new reports and analysis. Also, you can copy your exisiting samples and put them in an HBASE directory. Then edit all reports to use the Hbase connections and schema.


Troubleshooting
Using the same Phoenix driver that I used in the BA Server, I am unable to make a new Generic Connection in PDI. I also added the Phoenix core jar, but that didn’t help:

Error connecting to database [Phoenix] : org.pentaho.di.core.exception.KettleDatabaseException: 
Error occurred while trying to connect to the database

Error connecting to database: (using class org.apache.phoenix.jdbc.PhoenixDriver)
ERROR 2006 (INT08): Incompatible jars detected between client and server. Ensure that phoenix.jar is put on the classpath of HBase in every region server: tried to access method com.google.common.base.Stopwatch.<init>()V from class org.apache.hadoop.hbase.zookeeper.MetaTableLocator


org.pentaho.di.core.exception.KettleDatabaseException: 
Error occurred while trying to connect to the database

Error connecting to database: (using class org.apache.phoenix.jdbc.PhoenixDriver)
ERROR 2006 (INT08): Incompatible jars detected between client and server. Ensure that phoenix.jar is put on the classpath of HBase in every region server: tried to access method com.google.common.base.Stopwatch.<init>()V from class org.apache.hadoop.hbase.zookeeper.MetaTableLocator


at org.pentaho.di.core.database.Database.normalConnect(Database.java:428)
at org.pentaho.di.core.database.Database.connect(Database.java:358)
at org.pentaho.di.core.database.Database.connect(Database.java:311)
at org.pentaho.di.core.database.Database.connect(Database.java:301)
at org.pentaho.di.core.database.DatabaseFactory.getConnectionTestReport(DatabaseFactory.java:80)
at org.pentaho.di.core.database.DatabaseMeta.testConnection(DatabaseMeta.java:2686)
at org.pentaho.ui.database.event.DataHandler.testDatabaseConnection(DataHandler.java:546)
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
at java.lang.reflect.Method.invoke(Method.java:606)
at org.pentaho.ui.xul.impl.AbstractXulDomContainer.invoke(AbstractXulDomContainer.java:313)
at org.pentaho.ui.xul.impl.AbstractXulComponent.invoke(AbstractXulComponent.java:157)
at org.pentaho.ui.xul.impl.AbstractXulComponent.invoke(AbstractXulComponent.java:141)
at org.pentaho.ui.xul.swt.tags.SwtButton.access$500(SwtButton.java:43)
at org.pentaho.ui.xul.swt.tags.SwtButton$4.widgetSelected(SwtButton.java:138)
at org.eclipse.swt.widgets.TypedListener.handleEvent(Unknown Source)
at org.eclipse.swt.widgets.EventTable.sendEvent(Unknown Source)
at org.eclipse.swt.widgets.Display.sendEvent(Unknown Source)
at org.eclipse.swt.widgets.Widget.sendEvent(Unknown Source)
at org.eclipse.swt.widgets.Widget.sendEvent(Unknown Source)
at org.eclipse.swt.widgets.Widget.sendEvent(Unknown Source)
at org.eclipse.swt.widgets.Widget.notifyListeners(Unknown Source)
at org.eclipse.swt.widgets.Display.runDeferredEvents(Unknown Source)
at org.eclipse.swt.widgets.Display.readAndDispatch(Unknown Source)
at org.eclipse.jface.window.Window.runEventLoop(Window.java:820)
at org.eclipse.jface.window.Window.open(Window.java:796)
at org.pentaho.ui.xul.swt.tags.SwtDialog.show(SwtDialog.java:389)
at org.pentaho.ui.xul.swt.tags.SwtDialog.show(SwtDialog.java:318)
at org.pentaho.di.ui.core.database.dialog.XulDatabaseDialog.open(XulDatabaseDialog.java:116)
at org.pentaho.di.ui.core.database.dialog.DatabaseDialog.open(DatabaseDialog.java:59)
at org.pentaho.di.ui.trans.step.BaseStepDialog$4.widgetSelected(BaseStepDialog.java:752)
at org.eclipse.swt.widgets.TypedListener.handleEvent(Unknown Source)
at org.eclipse.swt.widgets.EventTable.sendEvent(Unknown Source)
at org.eclipse.swt.widgets.Display.sendEvent(Unknown Source)
at org.eclipse.swt.widgets.Widget.sendEvent(Unknown Source)
at org.eclipse.swt.widgets.Widget.sendEvent(Unknown Source)
at org.eclipse.swt.widgets.Widget.sendEvent(Unknown Source)
at org.eclipse.swt.widgets.Widget.notifyListeners(Unknown Source)
at org.eclipse.swt.widgets.Display.runDeferredEvents(Unknown Source)
at org.eclipse.swt.widgets.Display.readAndDispatch(Unknown Source)
at org.pentaho.di.ui.trans.steps.tableinput.TableInputDialog.open(TableInputDialog.java:435)
at org.pentaho.di.ui.spoon.delegates.SpoonStepsDelegate.editStep(SpoonStepsDelegate.java:124)
at org.pentaho.di.ui.spoon.Spoon.editStep(Spoon.java:8712)
at org.pentaho.di.ui.spoon.trans.TransGraph.editStep(TransGraph.java:3061)
at org.pentaho.di.ui.spoon.trans.TransGraph.mouseDoubleClick(TransGraph.java:747)
at org.eclipse.swt.widgets.TypedListener.handleEvent(Unknown Source)
at org.eclipse.swt.widgets.EventTable.sendEvent(Unknown Source)
at org.eclipse.swt.widgets.Display.sendEvent(Unknown Source)
at org.eclipse.swt.widgets.Widget.sendEvent(Unknown Source)
at org.eclipse.swt.widgets.Widget.sendEvent(Unknown Source)
at org.eclipse.swt.widgets.Widget.sendEvent(Unknown Source)
at org.eclipse.swt.widgets.Widget.notifyListeners(Unknown Source)
at org.eclipse.swt.widgets.Display.runDeferredEvents(Unknown Source)
at org.eclipse.swt.widgets.Display.readAndDispatch(Unknown Source)
at org.pentaho.di.ui.spoon.Spoon.readAndDispatch(Spoon.java:1319)
at org.pentaho.di.ui.spoon.Spoon.waitForDispose(Spoon.java:7939)
at org.pentaho.di.ui.spoon.Spoon.start(Spoon.java:9190)
at org.pentaho.di.ui.spoon.Spoon.main(Spoon.java:654)
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
at java.lang.reflect.Method.invoke(Method.java:606)
at org.pentaho.commons.launcher.Launcher.main(Launcher.java:92)
Caused by: org.pentaho.di.core.exception.KettleDatabaseException: 
Error connecting to database: (using class org.apache.phoenix.jdbc.PhoenixDriver)
ERROR 2006 (INT08): Incompatible jars detected between client and server. Ensure that phoenix.jar is put on the classpath of HBase in every region server: tried to access method com.google.common.base.Stopwatch.<init>()V from class org.apache.hadoop.hbase.zookeeper.MetaTableLocator

at org.pentaho.di.core.database.Database.connectUsingClass(Database.java:592)
at org.pentaho.di.core.database.Database.connectUsingClass(Database.java:4697)
at org.pentaho.di.core.database.Database.normalConnect(Database.java:414)
... 63 more
Caused by: java.sql.SQLException: ERROR 2006 (INT08): Incompatible jars detected between client and server. Ensure that phoenix.jar is put on the classpath of HBase in every region server: tried to access method com.google.common.base.Stopwatch.<init>()V from class org.apache.hadoop.hbase.zookeeper.MetaTableLocator
at org.apache.phoenix.exception.SQLExceptionCode$Factory$1.newException(SQLExceptionCode.java:386)
at org.apache.phoenix.exception.SQLExceptionInfo.buildException(SQLExceptionInfo.java:145)
at org.apache.phoenix.query.ConnectionQueryServicesImpl.checkClientServerCompatibility(ConnectionQueryServicesImpl.java:990)
at org.apache.phoenix.query.ConnectionQueryServicesImpl.ensureTableCreated(ConnectionQueryServicesImpl.java:869)
at org.apache.phoenix.query.ConnectionQueryServicesImpl.createTable(ConnectionQueryServicesImpl.java:1215)
at org.apache.phoenix.query.DelegateConnectionQueryServices.createTable(DelegateConnectionQueryServices.java:112)
at org.apache.phoenix.schema.MetaDataClient.createTableInternal(MetaDataClient.java:1902)
at org.apache.phoenix.schema.MetaDataClient.createTable(MetaDataClient.java:744)
at org.apache.phoenix.compile.CreateTableCompiler$2.execute(CreateTableCompiler.java:186)
at org.apache.phoenix.jdbc.PhoenixStatement$2.call(PhoenixStatement.java:303)
at org.apache.phoenix.jdbc.PhoenixStatement$2.call(PhoenixStatement.java:295)
at org.apache.phoenix.call.CallRunner.run(CallRunner.java:53)
at org.apache.phoenix.jdbc.PhoenixStatement.executeMutation(PhoenixStatement.java:293)
at org.apache.phoenix.jdbc.PhoenixStatement.executeUpdate(PhoenixStatement.java:1236)
at org.apache.phoenix.query.ConnectionQueryServicesImpl$12.call(ConnectionQueryServicesImpl.java:1893)
at org.apache.phoenix.query.ConnectionQueryServicesImpl$12.call(ConnectionQueryServicesImpl.java:1862)
at org.apache.phoenix.util.PhoenixContextExecutor.call(PhoenixContextExecutor.java:77)
at org.apache.phoenix.query.ConnectionQueryServicesImpl.init(ConnectionQueryServicesImpl.java:1862)
at org.apache.phoenix.jdbc.PhoenixDriver.getConnectionQueryServices(PhoenixDriver.java:180)
at org.apache.phoenix.jdbc.PhoenixEmbeddedDriver.connect(PhoenixEmbeddedDriver.java:132)
at org.apache.phoenix.jdbc.PhoenixDriver.connect(PhoenixDriver.java:151)
at java.sql.DriverManager.getConnection(DriverManager.java:571)
at java.sql.DriverManager.getConnection(DriverManager.java:233)
at org.pentaho.di.core.database.Database.connectUsingClass(Database.java:578)
... 65 more
Caused by: java.lang.IllegalAccessError: tried to access method com.google.common.base.Stopwatch.<init>()V from class org.apache.hadoop.hbase.zookeeper.MetaTableLocator
at org.apache.hadoop.hbase.zookeeper.MetaTableLocator.blockUntilAvailable(MetaTableLocator.java:596)
at org.apache.hadoop.hbase.zookeeper.MetaTableLocator.blockUntilAvailable(MetaTableLocator.java:580)
at org.apache.hadoop.hbase.zookeeper.MetaTableLocator.blockUntilAvailable(MetaTableLocator.java:559)
at org.apache.hadoop.hbase.client.ZooKeeperRegistry.getMetaRegionLocation(ZooKeeperRegistry.java:61)
at org.apache.hadoop.hbase.client.ConnectionManager$HConnectionImplementation.locateMeta(ConnectionManager.java:1185)
at org.apache.hadoop.hbase.client.ConnectionManager$HConnectionImplementation.locateRegion(ConnectionManager.java:1152)
at org.apache.hadoop.hbase.client.ConnectionManager$HConnectionImplementation.relocateRegion(ConnectionManager.java:1126)
at org.apache.hadoop.hbase.client.ConnectionManager$HConnectionImplementation.locateRegionInMeta(ConnectionManager.java:1331)
at org.apache.hadoop.hbase.client.ConnectionManager$HConnectionImplementation.locateRegion(ConnectionManager.java:1155)
at org.apache.hadoop.hbase.client.ConnectionManager$HConnectionImplementation.locateRegion(ConnectionManager.java:1139)
at org.apache.hadoop.hbase.client.ConnectionManager$HConnectionImplementation.locateRegion(ConnectionManager.java:1096)
at org.apache.hadoop.hbase.client.ConnectionManager$HConnectionImplementation.getRegionLocation(ConnectionManager.java:931)
at org.apache.phoenix.query.ConnectionQueryServicesImpl.getAllTableRegions(ConnectionQueryServicesImpl.java:429)
at org.apache.phoenix.query.ConnectionQueryServicesImpl.checkClientServerCompatibility(ConnectionQueryServicesImpl.java:943)
... 86 more

Custom URL     :  jdbc:phoenix:sandbox.hortonworks.com:16000/hbase
Custom Driver Class:org.apache.phoenix.jdbc.PhoenixDriver