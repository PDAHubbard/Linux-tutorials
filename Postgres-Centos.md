# Quick Introduction to Postgresql Administration under CENTOS


This tutorial assumes:
* CentOS 6.x is installed
* An internet connection is available
* The user has a basic knowledge of Linux
* The user has root access to the machine
* The user has a basic knowledge of the vi editor.

## 1. Postgres Installation
1.1 You must EXCLUDE postgres from the default Centos repositories by editing the following file:

	# /etc/yum.repos.d/CentOS-Base.repo

1.2 Run this script to perform the change:

	# mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak; awk '/gpgkey/{print;print "exclude=postgresql*";next}1' /etc/yum.repos.d/CentOS-Base.repo.bak > /etc/yum.repos.d/CentOS-Base.repo

1.3 Browse http://yum.postgresql.org to find the correct PGDG file. Here I assume Centos 6 (i386) and PostgreSQL 9.4:

	http://yum.postgresql.org/9.4/redhat/rhel-6-i386/pgdg-centos94-9.4-1.noarch.rpm

	# yum -y localinstall http://yum.postgresql.org/9.4/redhat/rhel-6-i386/pgdg-centos94-9.4-1.noarch.rpm

1.4 If you want, you can browse the available packages by running this command:

	# yum list postgres*

1.5 Install the server with this command:

	# yum -y install postgresql94-server


## 2. Initial setup and starting postgresql

2.1 You must first INITIALISE the database. This command must only be run once:

	# service postgresql-9.4 initdb

2.2 You can START and STOP the database using the following commands:

	# service postgresql-9.4 start

	# service postgresql-9.4 stop

2.3 You can configure Centos to start the database automatically at startup with this command:

	# chkconfig postgresql-9.4 on


## 3. Postgres config files and DB location

3.1 The config file is 

	/var/lib/pgsql/9.4/data/postgresql.conf

3.2 View or edit the config file with this command:

	# vi /var/lib/pgsql/9.4/data/postgresql.conf

3.3 Some changes require that you restart the database with this command:

	# service postgresql-9.4 restart

3.4 The default Database location is 

	/var/lib/pgsql/9.4/data


## 4. Creating a Postgresql user

4.1 Create a new Linux user:

	# adduser user1

4.2 Give the new user a password:

	# passwd user1

Type the password twice.

4.3 Login as the Postgres superuser:

	# su - postgres

4.4 Connect to the database server:

	$ psql template1

4.5 Add the user 'user1' to the database with the password 'password1':

	template1=# CREATE USER user1 WITH PASSWORD 'password1';

4.6 Create a new database for the user:

	template1=# CREATE DATABASE user1db;

4.7 Grant ALL privleges to user1:

	template1=# GRANT ALL PRIVILEGES ON DATABASE user1db to user1;

4.8 Quit the Postgresql shell by typing '\q':

	template1=# \q

4.9 Log in as 'user1':

	$ su - user1

4.10 Connect to the Database you just created:

	$ psql -d user1db -U user1

4.11 Quit the shell by typing '\q':

	user1db=> \q


## 5. Recovering/Changing the PostgreSQL root password
NB: You should only change this password by using the 'psql' command below. DO NOT try to change the password of the Linux 'postgres' account.
All these commands need to be run as 'root'

5.1 Shut down PostgreSQL
	
	# service postgresql stop

5.2 Reset the authentication mechanism (assuming defaults are already being used)

Edit the /var/lib/pgsql/9.4/datapg_hba.conf file

	# vi /var/lib/pgsql/9.4/data/pg_hba.conf

Navigate down to the line that says

	local all all peer

Edit it to

	local all postgres trust

And now save the file.

5.3 Start PostgreSQL

	# service postgresql start

5.4 Log in and change the password

	# su - postgres

	$ psql -U postgres template1 -c "alter user postgres with password 'new_password';"

5.5 Start PostgreSQL
	# service postgresql-9.4 start


## 6. Postgresql Command Line

You can run SQL commands in two ways:
* Directly from the Linux command line with the "-c" parameter:

	  # su - user1
	  $ psql -d user1db -U user1 -c "select * from user1table;"

* Directly in the Postgres shell:

	  # su - user1
	  $ psql -d user1db -U user1
	  user1db=> select * from user1table;

This part assumes you will run the commands from the Linux command line.

6.1 CREATE a new table:

	$ psql -d user1db -U user1 -c "CREATE TABLE COMPANY(
 	ID SERIAL PRIMARY KEY     NOT NULL,
    NAME           TEXT    NOT NULL,
    AGE            INT     NOT NULL,
    ADDRESS        CHAR(50),
    SALARY         REAL
	);"

Note that "SERIAL" defines an automatically incremented ID value.


6.2 INSERT data into the table:

	$ psql -d user1db -U user1 -c "INSERT INTO COMPANY(NAME, AGE, ADDRESS, SALARY) VALUES ('John Smith', '52', '1 Round Way, Hertfordshire', '2000.0');" 

6.3 UPDATE existing data in the table:

	$ psql -d user1db -U user1 -c "UPDATE COMPANY set SALARY=25000.0 WHERE SALARY=2000.0;"

6.4 SELECT data from the table:

 	$ psql -d user1db -U user1 -c "SELECT * from COMPANY WHERE SALARY>=20000.0;"

6.5 DELETE data from the table:

	 $ psql -d user1db -U user1 -c "DELETE from COMPANY WHERE NAME='John Smith';"


## 7. Full Database backup and restore

The simplest way to create a backup is to dump the entire database.
This takes a 'snapshot' of the database at the time the backup is run.
The dump can then be used to recreate the database on other machines as well.

7.1 Back up an entire Postgresql Database:

Become the Postgres superuser:

	# su - postgres

Run the pg_dump command:

	$ pg_dump -C user1db > /tmp/user1db.sql

This will create a dump of the database 'user1db' and place it in a file called 'user1db.sql' in your /tmp folder.


7.2 Restore an entire Postgresql Database:

Become the Postgres superuser:

	# su - postgres

Ensure that the database does not exist. WARNING: Be sure you have the correct backup file before you remove the database:

	$ psql -c "DROP DATABASE user1db;"

Create the database again:

	$ psql -c "CREATE DATABASE user1db;"

Restore the data into the database:

	$ psql user1db < /tmp/user1db.sql


## 8. Single Table backup and restore

The method for backing up and restoring tables is almost the same as for full databases. 

8.1 Back up a single table

Become the Postgres superuser:

	# su - postgres

Run the pg_dump command:

	$ pg_dump -C user1db -t COMPANY > /tmp/user1db_table_COMPANY.sql


8.2 Restore a single table

Become the Postgres superuser:

	# su - postgres

Ensure that the table does not exist. WARNING: Be sure you have the correct backup file before you remove the table:

	$ psql -d user1db -c "DROP table COMPANY;"

Restore the data into the table:

	$ psql user1db < /tmp/user1db_table_COMPANY.sql
