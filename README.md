
# Hosting Wordpress Website on AWS EC2

In this project, we will be creating and hosting a Wordpress website on AWS EC2 with a domain name. We'll manually provision EC2 instance (i.e an AWS virtual machine) to run WordPress using Apache2 Web-Server, PHP and MySQL.




# **Project Flow Step-Wise**

### **Create a EC2 Instance**
We are going to Rent a VM or EC2 Instance with required Configurations on AWS with a SSH Key Pair.
Also assoicate our EC2 Instance with Elastic IP.

### **SSH into the Virutal Machine**
We can SSH into our EC2 Instance either by EC2 Connect without SSH Keys.  

Or can SSH into it by using Windows Powershell as propmt and then go into the folder where our SSH KeyPair is Downloaded then use command

```
ssh -i Keyfilename.pem username@IPv4_EC2
```
Note : The Username is based on AMI of EC2 Instance.
 * For Amazon Linux 2 AMI user name: 'ec2-user'
 * For an Ubuntu AMI user name: 'ubuntu'

 ### **Install Apache Web-Server onto VM**
Once you are successully into your VM.

* We install Apache Web Server in order to run PHP and hence serve us the WordPress Website. Also before installing it we need to download and install the updates for each outdated package and dependency on your system.

```
sudo apt-get update
sudo apt install apache2
```
* After the installation to check if Apache HTTP server is successully installed on our VM we can access "http://IPv4_EC2" on web-browser with Public IPv4 Address of EC2 Instance that should serve us Apache2 Default Page.

* Note: The Apache Server is Installed in /etc of the VM or EC2 instance. And its index.html file is created and served from /var/www/html/ . The instrution which says to serve data from /var/www/html/ path is given from 000default.conf file inside /etc/apache2/sites-available/ 

* Basically it means when user will access Public IPv4 of EC2 , Apache-Server's 000default.conf file says in Document Root serve them files from /var/www/html/ which by defualt at installation contains index.html file contaning apache Page.

### **Installation of PHP Runtime, MySQL connector for PHP and MySQL-Server**
Needed as WordPress Software is build on PHP. As PHP is used for Web-Dev & is used to connect to databases And MySQL is the most popular database system used with PHP.

To Install php runtime and php mysql connector use command
```
sudo apt install php libapache2-mod-php php-mysql
```
Install MySQL server for Managing Databases
```
sudo apt install mysql-server 
```
First we Login or access mysql-server by root whose defualt password is 'password'
```
sudo mysql -u root -p
```
Then to Change defualt or mysql native password of the root user
```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password by 'your_new_password';
```
To login into sql server of our VM use command 
``` 
mysql -u username -p
```

### **Create a new Database User**
We are creating a new User to work with Wordpress Software.  
```
CREATE USER 'wp_user'@localhost IDENTIFIED BY 'your_new_password';
```

### **Create Seperate Database for WordPress**
For creating new Database to use with the WordPress
```
CREATE DATABASE your_DB_Name;
```

### **Grant all Privileges on created database to newly created user**
``` 
GRANT ALL PRIVILEGES ON your_DB_Name.* TO 'new_user'@localhost;
```

### **Download Wordpress Software Installation folder on our VM**
* Now we have fully ready Environment to run WordPress on our VM. We go into /tmp directory and then download Wordpress Installation file using command
```
wget download_link
```
* Command wget stands for web get. wget is a non-interactive file downloader command. Non-interactive means it can work in background even when the  user is not logged in  the system.


* Then we need to unzip the donwloaded Wordpress Zip File by using unzip command so first we will install unzip  
```
sudo apt-get install unzip
unzip download_file.zip
```

* Here we get a unzipped ready File named 'wordpress' next we will move it into the Document root of apache which is /home/var/www/html/ because this is from where the Apache2 serves data onto its server.
```
sudo mv wordpress /var/www/html/
```

### **Access the Wordpress Folder through Browser and start its installation**
We Go to http://IP/wordpress where IP is Elastic IPv4 Address of EC2 Instance. We specified wordpress as path because currently data is served from path which is 
/home/var/www/html/ and our wordpress folder is inside html. Currently html folder consist of index.html which is Default page of Apache2 and wordPress folder. Thus this path will take us into to installation Page of Wordpress Software. 

### **Start by Configuring DB to use with WordPress** 
Now we will configure Database to use with Wordpress info like DB Name,DB User who has access to that DB,Password of that DB User. All about New MySQL-DB and user we created whom we gave all Privileges over that newly created DB.

### **We might get an error saying Unable to write to wp-config.php file**
In this case we create the wp-config.php file manually into wordpress folder in var directory and paste the given text into it and save it.  
To insert press i then paste the text provided and then to save and exit use (esc + :wq)
```
vi wp-config.php
```
 

### **Further Installation Step about User Details**
Next you are directed to a Page to Fill out Info for creating a Wordpress Account so insert your Username,Password,Email,etc.

### **WordPress Site is Served at http://IP/wordpress to serve it at http://IP/**
Now when we access http://IP/wordpress we will have our Site up and hosted. But we want it to be served at http://IP/ so for that we need to Modify Apache Config File.

### **Modifying Apache Config File to Chnage Doucment Root path**
* The instructions for Serving data from a specific path are given in 000default.conf file which is present in  path which is /etc/apache2/sites-available/  
* So we Edit Serving Path called as Document Root by Opening Config File through Root User using command
```
sudo vi 000default.conf
```
* Then we will Change Doucment Root Path in config File from '/home/var/www/html/' to '/home/var/www/html/wordpress'

* Hence we Changed Apache2 Software's Configs for changes to work we need to restart the Server by using command
```
sudo systemctl restart apache2
```

* Hence now our Wordpress Site is Served at http://IP/

### **Linking Domain Name to the WordPress Website**
* We need to Link Domain Name to our Website so we can access it using Domain Name instead the Public IPv4 Address of VM. 
* Hence we go to Domain Name provider and in its DNS/NameServer Settings create a A-Type Record which Points to out IPv4 Address of EC2.

* Then again We need to configure Apache2 Config File so Open the 000default.conf as root user and add there the New Name of Server and its Alias which we have allocated i.e. our Domain Name.
```
ServerName awsprojectsite.online
ServerAlias www.awsprojectsite.online
```
* Save and exit the FIle and restart the Apache2 Server Again. Thus now our Website will be Accessible over the Domain Name Linked to it.

### **Making our Website work over HTTPS to make it Secure**
* We are going to Make our Website work over HTTPS request and responses by using tool called "certbot". First we need to install python 
```
sudo apt install python3
```
* Then we are going to install certbot in apache2's site files which has config files here Certbot will automatically edit the configuration files of Apache, which simplifies configuring HTTPS for your web server.

* We go in path /etc/apache2/sites-available/ and update system packages before installation of certbot for apache  
```
sudo apt-get update
sudo apt install certbot python3-certbot-apache
```

* Next we will run certbot tool by using command
```
sudo certbot --apache
```
* The command "sudo certbot --apache" is used to obtain and install SSL/TLS certificates for an Apache web server using the 'Let's Encrypt' service.

* Then we will get prompt for entering Email and certbot will ask which for Domain name we would like to have HTTPS over we had Main domain i.e. ServerName and Sub-Domain Name i.e. ServerAlias to have HTTPS for both we just press Enter.   
* Thats all our site is now HTTPS Secured with SSL Certificate from Let's Encrypt Service.
