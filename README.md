# Store HTTP access logs to MySQL DB.

  mod_log_sql is a logging module for Apache 1.3 and 2.0 which logs all requests to a database.
  
  # Steps
 
 1. Install httpd and mysql package to your server.
      ```
      yum install -y httpd*
      yum install -y mysql*
      
 2. Download and install mod_log_sql package
      ```
      cd /usr/local/src
      wget http://www.outoforder.cc/downloads/mod_log_sql/mod_log_sql-1.101.tar.bz2
      tar fxjv mod_log_sql-1.101.tar.bz2
      cd mod_log_sql-1.101
      ./configure
      make
      make install
  
  3. Configure HTTP
     
        ```
        vi /etc/httpd/conf/httpd.conf
        ```
    
   Search for LoadModule insert the following underneath
    
        LoadModule log_sql_module modules/mod_log_sql.so
        LoadModule log_sql_mysql_module modules/mod_log_sql_mysql.so
        <IfModule mod_ssl.c>
        LoadModule log_sql_ssl_module moduels/mod_log_sql_ssl.so
        </IfModule>
        LogSQLLoginInfo mysql://alogger:apasskey@/apachelogs
        #LogSQLCreateTables on
        LogSQLDBParam socketfile /var/lib/mysql/mysql.sock
        
        
  4. Setup password for mysql database
      
      Stop mysql service
      
      ```
      mysqld_safe --skip-grant-tables &
      ```
      
      Start MySQL without a password
      
      ```
      mysqld_safe --skip-grant-tables &
      ```
      
      Connect to MySQL
      
      ```
      mysql -uroot
      ```
    
      Set a new MySQL root password
      
      ```
      use mysql;
      update user set password=PASSWORD("mynewpassword") where User='root';
      flush privileges;
      quit
      ```
      
      Restart the MySQL service
      ```
      /etc/init.d/mysqld restart
      
  5. Setup HTTP - MySQL Logging
  
    cd /usr/local/src/mod_log_sql-1.101/contrib/
    mysql -uroot -p
    mysql>
    create database apachelogs;
    mysql>
    use apachelogs
    Database changed mysql>
    source create_tables.sql
    mysql>
    
    grant insert on apachelogs.* to alogger@localhost identified by 'alogger_password';
    flush privileges;
    quit
    
  6. Enable full logging of your MySQL daemon for trouble shooting
  
    ```
    vi /etc/my.cnf
    under the [mysqld] section add the following
    log=/var/log/mysqld.log
    service mysqld restart
    
  7. Create a Virtual Host in httpd.conf file
  
     ```
     <VirtualHost *:80>
     ServerAdmin root@localhost
     DocumentRoot /var/www/html
     ServerName abc.com
     ErrorLog logs/abc.com-error_log
     CustomLog logs/abc.com-access_log combined
     LogSQLTransferLogTable access_log
     LogSQLTransferLogFormat AbfHhlMmpRrSsTtUuv
     </VirtualHost>

  8. Create a index.html in /var/www/html
     
     ```
     echo "Hello World!!" > index.html

  9. Restart Apache 
  
    ```
    /etc/init.d/httpd restart
    
  10. Test the access logs in MySQL db
  
    ```
    tail -f /var/log/mysqld.log
    ```
    
    OR
    
    ```
    mysql -uroot -p
    mysql>
    use apachelogs;
    mysql>
    select count(*) from access_log;
    ```
