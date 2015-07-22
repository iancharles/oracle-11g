NOTE: This is currently unfinished and will not work in its current form. This note will be removed once this has been successfully run.
# Instant Oracle datase server
A [Docker](https://www.docker.com/) image with [Oracle Database 11g Enterprise Edition Release 11.2.0.1.0](http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html) running in [CentOS 6.6](https://www.centos.org/)
- Default ORCL database on port 1521

## Install
1. [Install Docker](https://docs.docker.com/installation/#installation)
2. You can no longer pull Oracle images from the Docker Hub, so you'll need to [build](https://github.com/iancharles/oracle-11g#build) it yourself

## Run
Create and run a container named orcl:
```
$ docker run --privileged -dP --name orcl wscherphof/oracle-12c
989f1b41b1f00c53576ab85e773b60f2458a75c108c12d4ac3d70be4e801b563
```
Yes, alas, this has to run `privileged` in order to gain permission for the `mount` statement in `/tmp/start` that ups the amount of shared memory, which has a hard value of 64M in Docker; see this [GitHub issue](https://github.com/docker/docker/pull/4981)

## Connect
The default password for the `sys` user is `change_on_install`, and for `system` it's `manager`
The `ORCL` database port `1521` is bound to the Docker host through `run -P`. To find the host's port:
```
$ docker port orcl 1521
0.0.0.0:49189
```
So from the host, you can connect with `system/manager@localhost:49189/orcl`
Though if using [Boot2Docker](https://github.com/boot2docker/boot2docker), you need the actual ip address instead of `localhost`:
```
$ boot2docker ip

The VM's Host only interface IP address is: 192.168.59.103

```
If you're looking for a databse client, consider [sqlplus](http://www.oracle.com/technetwork/database/features/instant-client/index-100365.html)
```
$ sqlplus system/manager@192.168.59.103:49189/orcl

SQL*Plus: Release 11.2.0.4.0 Production on Mon Sep 15 14:40:52 2014

Copyright (c) 1982, 2013, Oracle.  All rights reserved.


Connected to:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

SQL> |
```

## Monitor
The container runs a process that starts up the database, and then continues to check each minute if the database is still running, and start it if it's not. To see the output of that process:
```
$ docker logs orcl

LSNRCTL for Linux: Version 12.1.0.2.0 - Production on 16-SEP-2014 11:34:56

Copyright (c) 1991, 2014, Oracle.  All rights reserved.

Starting /u01/app/oracle/product/12.1.0/dbhome_1/bin/tnslsnr: please wait...

TNSLSNR for Linux: Version 12.1.0.2.0 - Production
Log messages written to /u01/app/oracle/diag/tnslsnr/e90ad7cc75a1/listener/alert/log.xml
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=e90ad7cc75a1)(PORT=1521)))

Connecting to (ADDRESS=(PROTOCOL=tcp)(HOST=)(PORT=1521))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 12.1.0.2.0 - Production
Start Date                16-SEP-2014 11:34:56
Uptime                    0 days 0 hr. 0 min. 0 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Listener Log File         /u01/app/oracle/diag/tnslsnr/e90ad7cc75a1/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=e90ad7cc75a1)(PORT=1521)))
The listener supports no services
The command completed successfully

SQL*Plus: Release 12.1.0.2.0 Production on Tue Sep 16 11:34:56 2014

Copyright (c) 1982, 2014, Oracle.  All rights reserved.

Connected to an idle instance.
ORACLE instance started.

Total System Global Area 1073741824 bytes
Fixed Size		    2932632 bytes
Variable Size		  721420392 bytes
Database Buffers	  343932928 bytes
Redo Buffers		    5455872 bytes
Database mounted.
Database opened.
Disconnected from Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

LSNRCTL for Linux: Version 12.1.0.2.0 - Production on 16-SEP-2014 11:35:24

Copyright (c) 1991, 2014, Oracle.  All rights reserved.

Connecting to (ADDRESS=(PROTOCOL=tcp)(HOST=)(PORT=1521))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 12.1.0.2.0 - Production
Start Date                16-SEP-2014 11:34:56
Uptime                    0 days 0 hr. 0 min. 28 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Listener Log File         /u01/app/oracle/diag/tnslsnr/e90ad7cc75a1/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=e90ad7cc75a1)(PORT=1521)))
Services Summary...
Service "ORCL" has 1 instance(s).
  Instance "ORCL", status READY, has 1 handler(s) for this service...
The command completed successfully
```

## Enter
There's no ssh daemon or similar configured in the image. If you need a command prompt inside the container, consider [nsenter](https://github.com/jpetazzo/nsenter) (and mind the [Boot2Docker note](https://github.com/jpetazzo/nsenter#docker-enter-with-boot2docker) there)

## Build
Should you want to modify & build your own image:

#### Step 1
1) Download `linux.x64_11gR2_database_1of2.zip ` & `linux.x64_11gR2_database_2of2.zip ` from [Oracle Tech Net](http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html)

2) Put the 2 zip files in the `step1` directory

3) `cd` to the `oracle-11g` repo directory

4) `$ docker build -t oracle-11g:step1 step1`

5) `$ docker run --privileged -ti --name step1 oracle-11g:step1 /bin/bash`

6) ` # /tmp/install/install` (takes about 5m)
```
Wed Jul 22 16:29:58 UTC 2015
Starting Oracle Universal Installer...

Checking Temp space: must be greater than 120 MB.   Actual 125233 MB    Passed
Checking swap space: must be greater than 150 MB.   Actual 6141 MB    Passed
Preparing to launch Oracle Universal Installer from /tmp/OraInstall2015-07-22_04-29-58PM. Please wait ...[root@cf45cfa558d2 /]# You can find the log of this install session at:
 /u01/app/oraInventory/logs/installActions2015-07-22_04-29-58PM.log
The following configuration scripts need to be executed as the "root" user. 
 #!/bin/sh 
 #Root scripts to run

/u01/app/oracle/product/11.2.0/dbhome_1/root.sh
To execute the configuration scripts:
	 1. Open a terminal window 
	 2. Log in as "root" 
	 3. Run the scripts 
	 4. Return to this window and hit "Enter" key to continue 

Successfully Setup Software.

```
7) ` <enter>`

8) ` # exit` (the scripts mentioned are executed as part of the step2 build)

9) `$ docker commit step1 oracle-11g:installed`

#### Step 2
1) `$ docker build -t oracle-11g:step2 step2`

2) `$ docker run --privileged -ti --name step2 oracle-11g:step2 /bin/bash`

3) ` # /tmp/create` (takes about 15m)
```
Tue Sep 16 11:07:30 UTC 2014
Creating database...

SQL*Plus: Release 12.1.0.2.0 Production on Tue Sep 16 11:07:30 2014

Copyright (c) 1982, 2014, Oracle.  All rights reserved.

Connected to an idle instance.

File created.

ORACLE instance started.

Total System Global Area 1073741824 bytes
Fixed Size		    2932632 bytes
Variable Size		  721420392 bytes
Database Buffers	  343932928 bytes
Redo Buffers		    5455872 bytes

Database created.


Tablespace created.


Tablespace created.

Disconnected from Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

Tue Sep 16 11:07:50 UTC 2014
Running catalog.sql...

Tue Sep 16 11:08:51 UTC 2014
Running catproc.sql...

Tue Sep 16 11:19:38 UTC 2014
Running pupbld.sql...

Tue Sep 16 11:19:38 UTC 2014
Create is done; commit the container now
```
4) ` # exit`

5) `$ docker commit step2 oracle-11g:created`

#### Step 3
1) `$ docker build -t oracle-11g step3`

## License
[GNU Lesser General Public License (LGPL)](http://www.gnu.org/licenses/lgpl-3.0.txt) for the contents of this GitHub repo; for Oracle's database software, see their [Licensing Information](http://docs.oracle.com/database/121/DBLIC/toc.htm)
