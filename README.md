# oracle21c-docker
A docker container for running Oracle 21c

A while back, Oracle introduced the concept of container databases (CDB) and pluggable databases (PDB). Containers are used for multi-tenancy and contain pluggable databases. Pluggable databases are what you are probably used to, a self contained database that you connect to. By default, this image creates one CDB, and one PDB within that CDB.

Note that Oracle is shifting away from an SID and using service names instead. PDB use service names, CDB use SIDs. However, usernames for CDB must start with C##, ie C##STEVE. So to make things simple, we are focusing on just using the single PDB.

I used to maintain a standalone repository for building these docker images, like my Oracle 12c one here: https://github.com/steveswinsburg/oracle12c-docker

Oracle have since improved their docker images so all we need now are simplified instructions based on the official Oracle docker-images repo.

Before you begin
----------------

1. Clone `https://github.com/oracle/docker-images`.
1. Download the Oracle Database 21c binary `LINUX.X64_213000_db_home.zip` from https://www.oracle.com/database/technologies/oracle21c-linux-downloads.html
1. Put the zip in the `OracleDatabase/SingleInstance/dockerfiles/21.3.0` directory. **Do not unzip it.**
1. In Docker Desktop, ensure you have a large enough amount of memory allocated. These instructions will set the total memory to 4000MB, so make sure Docker has a value higher than that.

Building
--------

````
cd OracleDatabase/SingleInstance/dockerfiles
./buildContainerImage.sh -v 21.3.0 -e
````

If the build fails saying you are out of space, check how much space you have available on your disk AND available to Docker. If it looks ok, prune old Docker images via: 
`yes | docker image prune > /dev/null`

Running
-------

To use the sensible defaults:

```
docker run \
--name oracle21c \
-p 1521:1521 \
-p 5500:5500 \
-e ORACLE_PDB=orcl \
-e ORACLE_PWD=password \
-e INIT_SGA_SIZE=3000 \
-e INIT_PGA_SIZE=1000 \
-v /opt/oracle/oradata \
-d \
oracle/database:21.3.0-ee
```

On first run, the database will be created and setup for you. This will take about 10-15 minutes. Open Docker Dashboard and watch the progress. Then you can connect.

Optionally, you can use the following run command to mount an additional directory on your local machine which can help avoid getting "No disk space" issues as you gradually insert more and more data into your database. Make sure you set the -v command appropriately for your system.

```
docker run \
--name oracle21c \
-p 1521:1521 -p 5500:5500 \
-e ORACLE_PDB=orcl \
-e ORACLE_PWD=password \
-e INIT_SGA_SIZE=3000 \
-e INIT_PGA_SIZE=1000 \
-v /Users/<your-username>/path/to/store/db/files/:/opt/oracle/oradata \
-d \
oracle/database:121.3.0-ee
```

Configuration
-------------

```
   --name:        The name of the container (default: auto generated)
   -p:            The port mapping of the host port to the container port.
                  Two ports are exposed: 1521 (Oracle Listener), 5500 (OEM Express)
   -e ORACLE_SID: The Oracle Database SID that should be used (default: ORCLCDB)
   -e ORACLE_PDB: The Oracle Database Service Name that should be used (default: ORCLPDB1)
   -e ORACLE_PWD: The Oracle Database SYS password (default: auto generated)
   -e INIT_SGA_SIZE: The total memory in MB that should be used for all SGA components. If you bump the memory settings up too much you might need to change your Docker settings to allocate more memory to Docker (optional and if not specified will be auto calculated)
   -e INIT_PGA_SIZE: The target aggregate PGA memory in MB that should be used for all server processes attached to the instance. If you bump this and INIT_SGA_SIZE up to much, you might need to change your Docker settings to allocate more memory to Docker (optional and if not specified will be auto calculated)
   -e ORACLE_CHARACTERSET: The character set to use when creating the database (default: AL32UTF8)
   -v /opt/oracle/oradata
                  The data volume to use for the database.
                  Has to be writable by the Unix "oracle" (uid: 54321) user inside the container!
                  If omitted the database will not be persisted over container recreation.
   -v /Users/<your-username>/path/to/store/db/files/:/opt/oracle/oradata
                  Mount the data volume into one of your local folders.
                  If omitted you might run into a "No disk space" issue at some point as your database keeps growing and docker does not resize its volume.
   -d:            Run in detached mode. You want this otherwise `Ctrl-C` will kill the container.
```

Connecting to Oracle
--------------------

Once the container has been started you can connect to it like any other database. Note we are using `Service Name` and not the SID (since PDB uses Service Name).

For example, SQL Developer:
```
Hostname: localhost
Port: 1521
Service Name: <your service name>
Username: sys
Password: <your password>
Role: AS SYSDBA

```

Changing the password
---------------------

Note: If you did not set the `ORACLE_PWD` parameter, check the docker run output for the password.

The password for the SYS account can be changed via the `docker exec` command. Note, the container has to be running:

First run `docker ps` to get the container ID. Then run:
`docker exec <container id> ./setPassword.sh <new password>`

Getting a shell on the container
--------------------------------
First run `docker ps` to get the container ID. Then run:
`docker exec -it <container id> /bin/bash`

Or as root:
`docker exec -u 0 -it <container id> /bin/bash`

Other questions?
----------------
Check out the [FAQ](https://github.com/oracle/docker-images/blob/main/OracleDatabase/SingleInstance/FAQ.md)
