Wordpress website hosting on EC2 with RDS with LAMP stack (Linux Apache MySQL PHP)
==========================================

--> Launch a Amazon Linux 2023 image EC2 instance
--> Create a RDS MySQL Engine (Private= Public Access NO) - with testdb database
--> Allow protocols in SG , MySQL or anywhere

=============== MySQL Client =======================

--> Install MySQL client

dnf install -y mariadb105

--> Export mysql endpoint as MySQL_HOST Variable, 

export MYSQL_HOST=mysqldb.cdbmlufgqkjd.ap-south-1.rds.amazonaws.com


====================== Connect to RDS and create DB =========================

--> Connect to RDS mysql to create required database and users

mysql -h mysqlwp.cdbmlufgqkjd.ap-south-1.rds.amazonaws.com -P 3306 -u admin -p

--> Create a database 

CREATE DATABASE wordpress;
CREATE USER 'wpuser' IDENTIFIED BY 'root123456';
GRANT ALL PRIVILEGES ON wordpress.* TO wpuser;
FLUSH PRIVILEGES;
Exit
=======================================================================
Now Install and configure apache
======================================================================

sudo yum install -y httpd

sudo service httpd start

Download a wordpress template and unzip the wordpress

wget https://wordpress.org/latest.tar.gz

tar -xzf latest.tar.gz

ls

cd wordpress

--> create a wp-config file from sample file already provided

cp wp-config-sample.php wp-config.php

cat wp-config.php 

--> edit the wp-config.php file to point to database

vi wp-config.php

--> Replace the Database_name_here , "username_here", "password_here" and "rds-endpoint-name" with valid info

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'database_name_here-wordpress' );

/** MySQL database username */
define( 'DB_USER', 'username_here-wordpressuser' );

/** MySQL database password */
define( 'DB_PASSWORD', 'password_here-root123456' );

/** MySQL hostname */
define( 'DB_HOST', 'rds-endpoint-name-rds endpoint' );

--> Now go to below link and it provides some information to update the wp-config file. It looks like below shared one.

https://api.wordpress.org/secret-key/1.1/salt/ --> open this in browser , it will generate few tokens and replace in this file

define('AUTH_KEY',         'mHHL8hQ}sF %WwgLl7.7[k&Cjgsbf@WwM`*Sm4^n!8fC|uN)AxS/yX~=te6dYV%K');
define('SECURE_AUTH_KEY',  'ZBe[xk@/XbLJ|:I^$i:*^S5j3kd-YXCz#`>e% (Cu9x,6>PdI3Jwgp;DbEc53`/j');
define('LOGGED_IN_KEY',    'N{q^HL7Z/I2`.D~Ef>yHIMXVnX#t7(cTmWD5-d=@|V!1R/@9/$RQ`%c6Bu4BDRd,');
define('NONCE_KEY',        '7)2V4-|o~FB#+;GDxT<nVtFP{Q{EW>|R:^ Mb+O;OOzsbVeT7h4:5Bs^7#*&<ty,');
define('AUTH_SALT',        '|q P.sq`8CAa%&N-Q?8+;gc2]SXystOf4n&p7-j3Cm&w~;Y<$0VpNcmnYE>~pTv|');
define('SECURE_AUTH_SALT', 'o23,r#f&Bky`Xb,T<87(NW/m6)fi<!e^PL{(Q10e94?KV`Y3Qx{>2lc|%Qfqm2,J');
define('LOGGED_IN_SALT',   '0;?-0R-x(*~-u6qDK.1y,%b/m|T)<!HfHyl.]hY3F|?R{>)Vf84oO  AmxMw -/k');
define('NONCE_SALT',       'cq#/FZ$+8eHy&ae^Qh@6p&Uvna5>qYtWsTP[#/-HB/3Mc|oe++2FAb,y;CO(zgET');

--> Now install dependencies

dnf install -y php8.4 php-mysqlnd

--> come back to home directory and copy all content to /var/www/html, Then restart the service.

cd /home/ec2-user

sudo cp -r wordpress/* /var/www/html/

sudo service httpd restart

systemctl enable httpd

place public ip to the browser : it ask for username and password and install wordpress

IP/wp-admin --> for admin page
Just IP for website


==============================

RDS Blue Green Deployment

Setup another EC2 instance with same application 
   (either manual(skip database creation steps))
   (create a AMI(wordpressAMI) and launch EC2 instance)
Create a Target Group - health check / or readme.html or index.php
Create a Load Balancer

Create a Launch Template while selecting AMI select WordpressAMI
Create a ASG with Launch Template

Access application using ALB