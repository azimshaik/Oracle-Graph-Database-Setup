### Steps to setup Oracle Graph Database on Oracle Linux 7.6 on Azure

#### Things to download:
------
1.  jdk-11.0.6_linux-x64_bin.rpm(https://www.oracle.com/java/technologies/javase-jdk11-downloads.html)
2.  jdk-8u241-linux-x64.rpm(https://www.oracle.com/java/technologies/javase-jdk8-downloads.html)
3.  jdk-11.0.6_linux-x64_bin.tar.gz.gz(https://www.oracle.com/java/technologies/javase-jdk11-downloads.html)

	... as azimshaik create and save at jdk11/
	```
	   $ gunzip jdk-11.0.6_linux-x64_bin.tar.gz.gz
	   $ tar xvf jdk-11.0.6_linux-x64_bin.tar
	```
4.  oracle-database-ee-19c-1.0-1.x86_64.rpm(https://www.oracle.com/database/technologies/oracle19c-linux-downloads.html)
	```
	yum install -y oracle-database-preinstall-19c
	yum -y localinstall oracle-database-ee-19c-1.0-1.x86_64.rpm
	[root@oraclevm azimshaik]# /etc/init.d/oracledb_ORCLCDB-19c configure
	Configuring Oracle Database ORCLCDB
	.
	.
	.
	Global Database Name: ORCLCDB
	System Identifier(SID): ORCLCDB
	Database configuration completed successfully. The passwords were auto generated, you muct change them by connecting to the database using 'sqlplus / as sysdba' as the oracle user
	[root@oraclevm azimshaik]# passwd oracle
	Changing password for user oracle
	New password:
	Retype new password:
	passwd: all authentication tokens updated successfully
	```
	as azimshaik 
	```
	$ cd /opt/oracle/product/19c/dbhome_1/bin/
	$ ./sqlplus as sysdba
	$ su oracle
	as oracle user
	$ source oraenv.sh
	$ cd /opt/oracle/product/19c/dbhome_1/bin/
	$ ./sqlplus / as sysdba
	SQL> startup
	It will show
		Database mounted.
		Database opened.
	SQL> show con_name;
	CON_NAME
	------------------------------
	CDB$ROOT
	SQL> show pdbs;
	    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
		---------- ------------------------------ ---------- ----------
	 	2 PDB$SEED			  READ ONLY  NO
	 	3 ORCLPDB1			  MOUNTED
	SQL> alter session set container=ORCLPDB1;
	SQL> create user azimshaik identified by azimshaik;
	SQL> grant create session, create sequence, create procedure, create table, create public synonym to azimshaik;
	SQL> connect azimshaik/azimshaik@orclpdb1;
	ERROR:
	SQL> connect / as sysdba
	SQL> show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
	---------- ------------------------------ ---------- ----------
	 2 PDB$SEED			  READ ONLY  NO
	 3 ORCLPDB1			  READ WRITE NO
	quit and login as oracle user
	$ ./sqlplus / as sysdba
	SQL> alter session set container=ORCLPDB1;
	SQL> alter session set container=ORCLPDB1;
	SQL> grant unlimited tablespace to azimshaik;
	SQL> connect azimshaik/azimshaik@ORCLPDB1;
	SQL> create table test(myid number);
	SQL> insert into test values(1);
	SQL> insert into test values(2);
	SQL> select * from test;

      MYID
	----------
			 1
	 		 2
	SQL> create index myidx on test(myid);
	SQL> connect / as sysdba
	SQL> alter session set container=ORCLPDB1;
	SQL> create tablespace hrtbs datafile 'hrtbs.dat' size 1M autoextend on next 100K segment space management auto;
	Tablespace created.
	SQL> create user hr identified by hr default tablespace hrtbs;
	SQL> grant create session, create sequence, create procedure, create table, unlimited tablespace, create public synonym to hr;
	SQL> connect hr/hr@ORCLPDB1;
	SQL> @/home/oracle/load_sample.sql
	.
	.
	.
	SQL> show user;
	USER is "HR"
	SQL> exit
	logout from vm 
	ssh in to vm
	$ su oracle
	$ source oraenv.sh
	$ echo $ORACLE_HOME
	$ sqlplus / as sysdba
	SQL> exit
	$ lsnrctl status
	$ lsnrctl start
	$ lsnrctl status
	$ sqlplus / as sysdba
	SQL> startup
	ORACLE instance started.
	SQL> show pdbs;

    	CON_ID CON_NAME			  OPEN MODE  RESTRICTED
	---------- ------------------------------ ---------- ----------
	 2 PDB$SEED			  READ ONLY  NO
	 3 ORCLPDB1			  MOUNTED
	SQL> alter session set container=ORCLPDB1;
	SQL> connect / as sysdba
	SQL> alter pluggable database ORCLPDB1 open read write;
	SQL> alter session set container=ORCLPDB1;
	SQL> grant create view to hr;
	SQL> connect hr@ORCLPDB1/hr
	SQL> create view works_as as select * from employees;
	SQL> create view works_at as select * from employees;
	SQL> create view managed_by as select * from departments;
	SQL> select tname from tab;
	TNAME
	--------------------------------------------------------
	READEGIONS
	COUNTRIES
	LOCATIONS
	DEPARTMENTS
	JOBS
	EMPLOYEES
	JOB_HISTORY
	SQL> show parameters max_string;
					NAME				     TYPE	 VALUE
	------------------------------------ ----------- ------------------------------
				max_string_size 		     string	 STANDARD
	SQL> ALTER SESSION SET CONTAINER=CDB$ROOT;
	SQL> ALTER SYSTEM SET max_string_size=extended SCOPE=SPFILE;
	SQL> shutdown immediate;
	SQL> startup upgrade;
	SQL> ALTER PLUGGABLE DATABASE ALL OPEN UPGRADE;
	SQL> quit
	$ cd $ORACLE_HOME/rdbms/admin
	$ mkdir /home/oracle/utl32k_cdb_pdbs_output
	$ $ORACLE_HOME/perl/bin/perl $ORACLE_HOME/rdbms/admin/catcon.pl -u SYS -d $ORACLE_HOME/rdbms/admin -l '/home/oracle/utl32k_cdb_pdbs_output' -b utl32k_cdb_pdbs_output utl32k.sql
	.
	.
	.
	Enter Password: //Enter Oracle US OS password
	catcon.pl: completed successfully
	$ sqlplus / as sysdba
	SQL> shutdown immediate;
	Database closed.
	Database dismounted.
	ORACLE instance shut down.
	SQL> startup           
	ORACLE instance started.

	Total System Global Area 1325396400 bytes
	Fixed Size		    9134512 bytes
	Variable Size		  872415232 bytes
	Database Buffers	  436207616 bytes
	Redo Buffers		    7639040 bytes
	Database mounted.
	Database opened.
	SQL> ALTER PLUGGABLE DATABASE ALL OPEN READ WRITE;

	$ mkdir /home/oracle/utlrp_cdb_pdbs_output
	$ $ORACLE_HOME/perl/bin/perl $ORACLE_HOME/rdbms/admin/catcon.pl -u SYS -d $ORACLE_HOME/rdbms/admin -l '/home/oracle/utlrp_cdb_pdbs_output' -b utlrp_cdb_pdbs_output utlrp.sql
	.
	.
	Enter Password: 
	catcon.pl: completed successfully
	$ sqlplus / as sysdba
	SQL> alter session set container=ORCLPDB1;
	SQL> show parameters max_string;

	NAME				     TYPE	 VALUE
	------------------------------------ ----------- ------------------------------
	max_string_size 		     string	 EXTENDED
	SQL> alter session set container=ORCLPDB1;
    ```

5.  oracle-graph-20.1.0.x86_64.rpm
	```
	as azimshaik ~
	$ rpm i oracle-graph-20.1.0.x86_64.rpm
	$ cd /opt/oracle/graph/
	$ cd pgx/
	$ cd bin/
	$ sudo su
	[root@oraclevm bin]# exit
	ssh into VM
	$ java -version
	java version "1.8.0_241"
	Java(TM) SE Runtime Environment (build 1.8.0_241-b07)
	Java HotSpot(TM) 64-Bit Server VM (build 25.241-b07, mixed mode)
	$ sudo su
	[root@oraclevm azimshaik]# ls
	[root@oraclevm azimshaik]# rpm -e oracle-graph
	warning: /etc/oracle/graph/server.conf saved as /etc/oracle/graph/server.conf.rpmsave
	[root@oraclevm azimshaik]# rpm -i oracle-graph-20.1.0.x86_64.rpm
	[root@oraclevm azimshaik]# usermod -a -G oraclegraph 
	[root@oraclevm azimshaik]# exit
	[azimshaik@oraclevm ~]$ groups
	azimshaik oraclegraph
	[azimshaik@oraclevm ~]$ exit
	logout
	ssh into vm
	[azimshaik@oraclevm ~]$ groups
	azimshaik oraclegraph
	[azimshaik@oraclevm ~]$ cd /opt/oracle/graph/pgx/
	[azimshaik@oraclevm pgx]$ cd bin/
	[azimshaik@oraclevm bin]$ pwd
	/opt/oracle/graph/pgx/bin
	[azimshaik@oraclevm bin]$ sudo vi /etc/oracle/graph/server.conf
	[azimshaik@oraclevm bin]$ ./start-server 
	.
	.
	.
	SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
	SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
	Mar 19, 2020 5:57:54 PM org.apache.coyote.AbstractProtocol start
	INFO: Starting ProtocolHandler ["http-nio-7007"]
	```

6.  oracle-graph-client-20.1.0.zip
	```
		as azimshaik at ~
		$ unzip oracle-graph-client-20.1.0.zip
		$ cd oracle-graph-client-20.1.0/bin
		for oracle graph client 
		$ export JAVA_HOME=/home/azimshaik/jdk11/jdk-11.0.6
		at bin/
		$ ./org-rdbms-jshell -b http://localhost:7007
		opg-jshell-rdbms> var jdbcUrl = "jdbc:oracle:thin:@localhost:1521/ORCLPDB1"
		opg-jshell-rdbms> var user = "hr" 
		opg-jshell-rdbms> var pass = "hr"
		opg-jshell-rdbms> var conn = DriverManager.getConnection(jdbcUrl, user, pass)
		opg-jshell-rdbms> conn.setAutoCommit(false)
		opg-jshell-rdbms> var pgql = PgqlConnection.getConnection(conn)
			Note: create file /home/azimshaik/create.pgql
		opg-jshell-rdbms> pgql.prepareStatement(Files.readString(Paths.get("/home/azimshaik/create.pgql"))).execute()
		opg-jshell-rdbms> Consumer<String> query = q -> { try(var s = pgql.prepareStatement(q)) { s.execute(); s.getResultSet().print(); } catch(Exception e) { throw new RuntimeException(e); } }
		query ==> $Lambda$602/0x0000000100762440@3dc4ed6f
		opg-jshell-rdbms> query.accept("select count(v) from hr match (v)")
		+----------+
		| count(v) |
		+----------+
		| 215      |
		+----------+
		opg-jshell-rdbms> query.accept("select count(e) from hr match ()-[e]->()")
		+----------+
		| count(e) |
		+----------+
		| 433      |
		+----------+
		opg-jshell-rdbms> query.accept("select distinct m.FIRST_NAME, m.LAST_NAME, m.SALARY from hr match (v:EMPLOYEES)-[:WORKS_FOR]->(m:EMPLOYEES) order by m.SALARY desc")
		+---------------------------------------+
		| m.FIRST_NAME | m.LAST_NAME | m.SALARY |
		+---------------------------------------+
		| Steven       | King        | 24000.0  |
		| Lex          | De Haan     | 17000.0  |
		| Neena        | Kochhar     | 17000.0  |
		| John         | Russell     | 14000.0  |
		| Karen        | Partners    | 13500.0  |
		| Michael      | Hartstein   | 13000.0  |
		| Alberto      | Errazuriz   | 12000.0  |
		| Shelley      | Higgins     | 12000.0  |
		| Nancy        | Greenberg   | 12000.0  |
		| Den          | Raphaely    | 11000.0  |
		| Gerald       | Cambrault   | 11000.0  |
		| Eleni        | Zlotkey     | 10500.0  |
		| Alexander    | Hunold      | 9000.0   |
		| Adam         | Fripp       | 8200.0   |
		| Matthew      | Weiss       | 8000.0   |
		| Payam        | Kaufling    | 7900.0   |
		| Shanta       | Vollman     | 6500.0   |
		| Kevin        | Mourgos     | 5800.0   |
		+---------------------------------------+
		opg-jshell-rdbms> query.accept("select avg(e.SALARY) from hr match (e:EMPLOYEES) -[h:WORKS_AT]-> (d:DEPARTMENTS) -[:LOCATED_IN]-> (:LOCATIONS) -[:PART_OF]-> (:COUNTRIES) -[:INCLUDED_IN]-> (r:REGIONS) where r.REGION_NAME = 'Americas' and d.DEPARTMENT_NAME = 'Accounting'")
		+---------------+
		| avg(e.SALARY) |
		+---------------+
		| 10150.0       |
		+---------------+
    ```

Users:
1. azimshaik
2. oracle
3. root

$ORACLE_HOME
oraenv.sh
should run everytime 

as azimshaik
vi host.txt (copy the host name here that you see when lstnctl start status)
oraclevm.onxcvxv3dulgdfgqdahgfhfghfghu0u3gbqc0urvlozkf.bx.internal.cloudapp.net

lsnrctl status
lsnrctl stop
lsnrctl start (takes few minutes to start)

Reference:
https://docs.oracle.com/en/database/oracle/oracle-database/20/spgdg/property-graph-overview-spgdg.html#GUID-ED2C4549-7BD4-4070-AA3A-1821F70F08E7

# Oracle-Graph-Database-Setup
test
![Error](https://github.com/azimshaik/Oracle-Graph-Database-Setup/blob/master/sqlplus_sqldba_connect_with_oracle_user%2C_shut_and_startup.png)
