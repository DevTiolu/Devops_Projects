# LAMP STACK IMPLEMENTATION
- **Linux** 
- **Apache** 
- **MySQL** 
- **PHP**

## Installing Apache Web Server

- Update a list of packages in **Ubuntu’s** package manager, **apt**. 

`sudo apt update`

![sudo apt update](<Images/apt update.png>)

- Install **Apache** using the package manager.

`sudo apt install apache2`

![sudo apt install apache2](<Images/apt install apache2.png>)

- Verify that **Apache** is running in your OS using the following command;

`sudo systemctl status apache2`

![systemctl status](<Images/sudo systemctl status apache2.png>)

- Check if the server can be accessed locally using either of these; 

 `curl http://localhost:80`  

`curl http://127.0.0.1:80`

![check local connection](<Images/port 80.png>)

- Check if the Apache HTTP server can respond to requests from the internet.

`http://<Public-IP-Address>:80`

## Installing MySQL

- Use **apt** to install **MySQL**.

`sudo apt install mysql-server`

![apt install mysql](<Images/apt install MySQL server.png>)

- Log into MySQL console.

`sudo mysql`

![sudo mysql](<Images/sudo mysql.png>)

- Set a password for the root user.

`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';`

![password and exit](<Images/change password and exit.png>)

- Run SQL security script.

`sudo mysql_secure_installation`

![mysql security](<Images/mysql security.png>)

- To log into MySQL and exit it, use the following commands;

`sudo mysql -p`

`mysql> exit`

![enter and exit mysql](<Images/enter and exit.png>)

## Installing PHP

