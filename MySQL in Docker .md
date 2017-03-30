# MySQL in Docker 
## Setup MySQL in Docker on MacBook
1. Start the Docker
2. Create local data files

	```
	$ mkdir -p ~/Docker/masterdb/data ~/Docker/masterdb/data
	$ mkdir -p ~/Docker/masterdb/cnf ~/Docker/slavedb/cnf
	
	```
3. Create Configuration file for master and slave

	```
	$ vi ~/Docker/masterdb/cnf/config-file.cnf
		# Config Settings:
		[mysqld]
		server-id=1
		binlog_format=ROW
		log-bin

	$ vi ~/Docker/slavedb/cnf/config-file.cnf
		# Config Settings:
		[mysqld]
		server-id=2
		
	```
4. Create the container 

	```
	$ docker run --name mysql-master -v ~/Docker/masterdb/cnf:/etc/mysql/conf.d -v ~/Docker/masterdb/data:/var/lib/mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:latest

	$ docker run --name mysql-slave -v ~/Docker/slavedb/cnf:/etc/mysql/conf.d -v ~/Docker/slavedb/data:/var/lib/mysql -p 3307:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:latest

	```

5. Create the replication user in master db

	```
	$ docker exec -ti mysql-master 'mysql' -uroot -p123456 -vvv -e"GRANT REPLICATION SLAVE ON *.* TO repl@'%' IDENTIFIED BY 'slavepass'"
	
	```

## Setup the MySQL Replication in Docker
1. Setup replication on slave. (Log file and log position used in the second command are coming from the first command)

	```
	$ docker exec -ti mysql-master 'mysql' -uroot -p123456 -e"SHOW MASTER STATUS"
	

	$ docker exec -ti mysql-slave 'mysql' -uroot -p123456 -e'change master to master_host="mysql",master_user="repl",master_password="slavepass",master_log_file="02cb9916fc4d-bin.000001",master_log_pos=154;' -vvv

	```
	
2. Start the replication

	```
	$ docker exec -ti mysql-slave 'mysql' -uroot -p123456 -e"START SLAVE;" -vvv

	$ docker exec -ti mysql-slave 'mysql' -uroot -p123456 -e"SHOW SLAVE STATUS" -vvv

	```