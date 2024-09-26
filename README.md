
# Project Title

WEB STACK IMPLEMENTATION (LEMP STACK)

The purpose of this project is to set up a LEMP stack using NGINX as the webserver.


## Introduction

This project demonstrates the implementation of the LEMP stack using Ubuntu and Gitbash for command execution. LEMP stands for:

Linux: Operating system

Engine-X (Nginx): Web server

MySQL: Database management system

PHP: Scripting language for dynamic web pages

 
## Launch an EC2 Instance

In my AWS console, I launched an EC2 instance using Ubuntu Server. I configured the security group with:

1. SSH (Port 22) for remote access.

2. HTTP (Port 80) for web access.

![Lunch Instance](https://github.com/ChideraA080/LEMP-PROJECT/blob/main/LEMP%20IMAGE/Screenshot%20Showing%20EC2%20instance%20running%20with%20public%20IP.png)
## Connect to the EC2 Instance

To connect to the EC2 instance I used Git Bash.I launched Git Bash and run this command to connect to the EC2:

```ssh -i "your-key.pem" ubuntu@<your-ec2-public-ip>```

![Connect to instance](https://github.com/ChideraA080/LEMP-PROJECT/blob/main/LEMP%20IMAGE/Screenshot%20of%20Gitbash%20Launch.png)


## Installing the Nginx Web Server

The first step is to update the server's package index and install the Nginx web server.

Purpose: To updates the package lists for the server and upgrades any outdated packages. Ensures that the server has the latest patches and security updates.

Commands to use:

```
sudo apt update

sudo apt install nginx

sudo systemctl status nginx  # Check Nginx status
```

If Nginx is running correctly, you will see active (running in green) as the output.

![Nginx status](https://github.com/ChideraA080/LEMP-PROJECT/blob/main/LEMP%20IMAGE/Screenshot%20Showing%20Nginx%20Installed.png)
## To Access Nginx locally on the Ubuntu shell

```
curl http://localhost:80
```

![Local URL](https://github.com/ChideraA080/LEMP-PROJECT/blob/main/LEMP%20IMAGE/Screenshot%20showing%20we%20can%20access%20our%20nginx%20locally%20in%20Ubuntu.png)
## To Test Nginx with Public IP address 

Go to your browser,navigate and paste this command:

```
http://<ec2-instance-public-ip>
```

You should see the Nginx welcome page like this below:


![Nginx Default Page](https://github.com/ChideraA080/LEMP-PROJECT/blob/main/LEMP%20IMAGE/Screenshot%20showing%20how%20nginx%20respond%20to%20Web%20browser%20.png)


## Installing MYSQL

Next is to install the MySQL database management system.

Purpose: Installs MySQL to manage databases for web applications. This is essential for storing and managing data for your website.

Commands to use:

```
sudo apt install mysql-server -y
```

 This command installs MySQL to manage databases for web applications. This is essential for storing and managing data for your website.
```
sudo mysql_secure_installation 
```
 This command runs a security script that removes insecure default settings, like test databases and anonymous users. It also sets a root password for MySQL.
#follow prompts to secure MYSQL 
## Create a New MySQL User and Database

1. Log in to the MySQL Shell:

```
sudo mysql
```
Purpose: Grants access to the MySQL root console with administrative privileges.

2. Set a password for the root user:
```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Password.1';
```
Purpose: This command switches the MySQL root user's authentication method to mysql_native_password

3. Create a database and user for your application

```
CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'Password.1';

GRANT ALL ON example_database.* TO 'example_user'@'%';

FLUSH PRIVILEGES;

EXIT;
```
![MYSQL Security Script](https://github.com/ChideraA080/LEMP-PROJECT/blob/main/LEMP%20IMAGE/Screenshot%20after%20running%20mysql%20script%20and%20Password%20testing.png)

## Installing PHP

Use this command:

```
sudo apt install php-fpm php-mysql -y
```
Purpose: Above command Installs PHP and its MySQL extension, which allows PHP to communicate with the MySQL database.
## Configuring  Nginx to use PHP processor

To do this,
1.  Create a root web directory for your_domain as follows:

```
sudo mkdir /var/www/projectLEMP
```

2. Assign the directory ownership with $USER enviroment variable, which refrence your current system user.Use this command:

```
sudo chown -R $USER:$USER /var/www/projectLEMP
```

3. Create a new configuration file in Nginx’s “sites-available” directory:

```
sudo vim /etc/nginx/sites-available/projectLEMP
```

Paste in the following bare-bones configuration:

```
server {
  listen 80;
  server_name projectLEMP www.projectLEMP;
  root /var/www/projectLEMP;

  index index.html index.htm index.php;

  location / {
    try_files $uri $uri/ =404;
  }

  location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
  }

  location ~ /\.ht {
    deny all;
  }
}
```

Purpose: The above configuration file configures the server to process .php files and define the root directory for the website.

 Here’s what each directives and location blocks does:

- __listen__ - Defines what port nginx listens on. In this case it will listen on port 80, the default port for HTTP.

- __root__ - Defines the document root where the files served by this website are stored.

- __index__ - Defines in which order Nginx will prioritize the index files for this website. It is a common practice to list index.html files with a higher precedence than index.php files to allow for quickly setting up a maintenance landing page for PHP applications. You can adjust these settings to better suit your application needs.

- __server_name__ - Defines which domain name and/or IP addresses the server block should respond for. Point this directive to your domain name or public IP address.

- __location /__ - The first location block includes the try_files directive, which checks for the existence of files or directories matching a URI request. If Nginx cannot find the appropriate result, it will return a 404 error.

- __location ~ \.php$__ - This location handles the actual PHP processing by pointing Nginx to the fastcgi-php.conf configuration file and the php7.4-fpm.sock file, which declares what socket is associated with php-fpm.

- __location ~ /\.ht__ - The last location block deals with .htaccess files, which Nginx does not process. By adding the deny all directive, if any .htaccess files happen to find their way into the document root, they will not be served to visitors.

4. Activate the configuration by linking to the config file from Nginx’s sites-enabled directory:

```
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
```

5. Test configuration for syntax error:

```
sudo nginx -t
```

![Test syntax](https://github.com/ChideraA080/LEMP-PROJECT/blob/main/LEMP%20IMAGE/Screenshot%20showing%20nginx%20syntax%20Configuration%20working.png)

6. Disable the default Nginx host that currently configured to listen on port 80:

```
sudo unlink /etc/nginx/sites-enabled/default
```

7. Reload Nginx to apply the changes:

```
sudo systemctl reload nginx
```

8. The new website is now active but the web root /var/www/projectLEMP is still empty. Create an index.html file in this location so to test the virtual host work as expected.:

```
sudo echo ‘Hello LEMP from hostname’ $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) ‘with public IP’ $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
```

9. Open the website on a browser using IP address:

```
http://<Public-IP Address>:80
```

If you see the text from 'echo' command you to index.html file, then it means your Nginx is working as expected. At this point, your LEMP stack is fully configured.


## Testing PHP with Nginx

1. Create info.php file in the default website root

```
sudo vim /var/www/info.php
```

2. Add the following PHP code:

```
<?php
phpinfo();
```

Go to your browser and paste:

```
http://<ec2-instance-public-ip>/info.php
```

![PHP page](https://github.com/ChideraA080/LEMP-PROJECT/blob/main/LEMP%20IMAGE/Screenshot%20of%20PHP%20after%20setting%20nginx%20configuration.png)

Don't forget to remove the info.php when done as it holds very sensitive information about you web server.To do that run :

```
sudo rm /var/www/projectLEMP/info.php
```

You can always regenerate this file if you need it later.


## Retrieving  Data from MySQL database with PHP

Create a new user with the mysql_native_password authentication method in order to be able to connect to MySQL database from PHP.

Create a database named example_database and a user named example_user

1.  connect to the MySQL console using the root account.
```
sudo mysql -p
```

2. Create a new database

CREATE DATABASE example_database;


3. Create a new user and grant the user full privileges on the new database.

```
CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
```

4. GRANT ALL ON example_database.* TO 'example_user'@'%';


5. exit

6. Login to MySQL console with the user custom credentials and confirm that you have access to todo_database.

```
mysql -u example_user -p
```

The -p flag will prompt for password used when creating the example_user


```
SHOW DATABASES;
```

![Show database](https://github.com/ChideraA080/LEMP-PROJECT/blob/main/LEMP%20IMAGE/Screenshot%20Showing%20example_user%20DATABASES.png)

7. Create a table called Todo_list,FRom MYSQL console,run the following statement:

```
CREATE TABLE example_database.todo_list (
  item_id INT AUTO_INCREMENT,
  content VARCHAR(255),
  PRIMARY KEY(item_id)
);
```

8. Insert a few rows of content to the test table

```
INSERT INTO example_database.todo_list (content) VALUES ("My first important item");

INSERT INTO example_database.todo_list (content) VALUES ("My second important item");

INSERT INTO example_database.todo_list (content) VALUES ("My third important item");
```

9. To confirm the data was successfully saved to your table run:

```
SELECT * FROM example_database.todo_list;
```
![Query table](https://github.com/ChideraA080/LEMP-PROJECT/blob/main/LEMP%20IMAGE/Screenshot%20showing%20%20todo_list%20data%20output%20via%20Mysql%20console.png)

10. exit
## Configuring Nginx for PHP Processing

1. Create a new PHP file in the custom web root directory

```
sudo vim /var/www/projectLEMP/todo_list.php
```

Copy the content below into the todo_list.php script.
```
<?php
$user = "example_user";
$password = "PassWord.1";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
?>
```

Save and close the file when you are done editing.
## Testing Application

Havigate to your Web browser and paste EC2 Public IP Address

```
http://<Public-IP>/todo_list.php
```

The content will be displayed as below

![Updated site with dns](https://github.com/ChideraA080/LEMP-PROJECT/blob/main/LEMP%20IMAGE/Screenshot%20showing%20todo_list%20.php%20Script%20output%20on%20web%20browser.png)
## Challenges Faced

 1. 404 Not Found Error

Problem: I encountered a 404 Not Found error when trying to access the website.

Solution: This was resolved by checking the Nginx configuration file (/etc/nginx/sites-available/projectLEMP) to ensure the root directive was correctly pointing to the web directory (/var/www/projectLEMP). Additionally, I ensured that the symbolic link was properly created and that permissions were correctly set.

Reason: The permissions were set incorrectly, preventing modifications by the current user.

 2. Timeout Error During SSH Connection

Problem: When connecting to the EC2 instance, I encountered a ssh: connect to host 54.xxx.xxx.xxx port 22: Connection timed out error.

Solution: The issue was resolved by verifying the EC2 instance's security group settings to ensure that inbound rules allowed SSH access (port 22) from my IP address.

Reason: The problem was due to restrictive firewall rules that blocked SSH traffic.
## Conclusion

Implementing the LEMP stack provided valuable insights into configuring and managing a web server environment. Through the process, I gained experience troubleshooting various issues such as file permissions and server configuration errors. By understanding the purpose of each command and configuration, I successfully deployed a functional web application with PHP interacting with a MySQL database on an Nginx server.

Overall, the project emphasized the importance of server security (using appropriate authentication methods and permissions), and it showcased the efficiency of the LEMP stack for deploying web applications on an Ubuntu server environment.
