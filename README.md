
# Monitoring Tools Setup on Debian VM (VirtualBox)

This document outlines the installation and basic configuration of Cacti, Icinga 2, and LibreNMS on a Debian VM running in VirtualBox with 10 GB storage and 1GB of RAM.  The focus is on lightweight and efficient monitoring to suit the VM's specifications.

**Why These Tools Need MySQL/MariaDB and PHP**

These tools (Cacti, Icinga 2, and LibreNMS) need MySQL/MariaDB and PHP for the following key reasons:

*   **MySQL/MariaDB (Database):**  Stores configuration settings, network data, user information, and performance metrics.  It provides persistent storage, allows for complex data relationships, and enables efficient querying of historical data.

*   **PHP (Server-Side Scripting Language):** Powers the web interface. Handles user requests, generates dynamic web pages displaying graphs and data, processes alerts, and provides interactivity.

In short, the database provides storage and organization, while PHP provides the dynamic web interface for interaction and visualization.

## 1. Cacti Installation

### Step 1: Update System

```bash
sudo apt update
sudo apt upgrade -y
```

### Step 2: Install Dependencies

```bash
sudo apt install apache2 mysql-server php php-cli php-mysql php-snmp php-gd php-xml snmp snmpd rrdtool build-essential librrds-perl libapache2-mod-php -y
```

### Step 3: Install Cacti

```bash
sudo apt install cacti -y
```

### Step 4: Configure MySQL for Cacti

```bash
sudo mysql -u root -p
```

(Enter your root password when prompted.)

In the MySQL shell:

```sql
CREATE DATABASE cacti;
GRANT ALL PRIVILEGES ON cacti.* TO 'cactiuser'@'localhost' IDENTIFIED BY 'password';
FLUSH PRIVILEGES;
EXIT;
```

**Important:** Replace `'password'` with a strong password.

### Step 5: Configure Cacti

```bash
sudo nano /etc/cacti/db.php
```

Modify the following lines:

```php
$db_hostname = "localhost";
$db_name = "cacti";
$db_user = "cactiuser";
$db_pass = "password";  // Use the password you set in MySQL.
```

**Important:** Use the same password you set in the MySQL shell.

### Step 6: Setup Apache

```bash
sudo a2ensite cacti
sudo systemctl restart apache2
```

Access Cacti through your browser at `http://<your_VM_IP>/cacti`. Follow the web installer instructions to finish the installation.

## 2. Icinga 2 Installation

### Step 1: Install the Icinga 2 repository

```bash
wget -O - https://packages.icinga.com/icinga.key | sudo tee /etc/apt/trusted.gpg.d/icinga.asc
```

```bash
echo "deb https://packages.icinga.com/icinga2/debian icinga-2-stable main" | sudo tee /etc/apt/sources.list.d/icinga.list
```

### Step 2: Update Package List and Install Icinga 2

```bash
sudo apt update
sudo apt install icinga2 icinga2-ido-mysql -y
```

### Step 3: Configure Icinga 2

```bash
sudo icinga2 feature enable command
sudo icinga2 feature enable ido-mysql
sudo icinga2 feature enable api
```

### Step 4: Create MySQL database for Icinga 2

```bash
sudo mysql -u root -p
```

(Enter your root password when prompted.)

In the MySQL shell:

```sql
CREATE DATABASE icinga2;
GRANT ALL PRIVILEGES ON icinga2.* TO 'icinga2user'@'localhost' IDENTIFIED BY 'password';
FLUSH PRIVILEGES;
EXIT;
```

**Important:** Replace `'password'` with a strong password.

### Step 5: Apply the database schema

```bash
sudo icinga2 ido-mysql setup
```

Answer the prompts, using the database name, user, and password you just created.

### Step 6: Start and enable Icinga 2 service

```bash
sudo systemctl start icinga2
sudo systemctl enable icinga2
```

### Step 7: Configure the web interface

```bash
sudo apt install icingaweb2 icingacli -y
```

```bash
sudo icingacli setup config webserver apache
sudo systemctl restart apache2
```

Access Icinga Web 2 via `http://<your_VM_IP>/icingaweb2`.

## 3. LibreNMS Installation

### Step 1: Install dependencies

```bash
sudo apt install apache2 mariadb-server php php-cli php-mysql php-gd php-snmp php-xml php-mbstring php-bcmath snmp snmpd git rrdtool librrds-perl -y
```

### Step 2: Download LibreNMS

```bash
cd /opt
sudo git clone https://github.com/librenms/librenms.git
sudo chown -R www-data:www-data librenms
```

### Step 3: Set up the database

```bash
sudo mysql -u root -p
```

(Enter your root password when prompted.)

In the MySQL shell:

```sql
CREATE DATABASE librenms;
GRANT ALL PRIVILEGES ON librenms.* TO 'librenms'@'localhost' IDENTIFIED BY 'password';
FLUSH PRIVILEGES;
EXIT;
```

**Important:** Replace `'password'` with a strong password.

### Step 4: Configure LibreNMS

```bash
cd /opt/librenms
sudo ./lnms install
```

Follow the on-screen prompts.  When asked for the database password, enter the password you set in the previous step.

### Step 5: Set up Apache

```bash
sudo nano /etc/apache2/sites-available/librenms.conf
```

Add the following configuration:

```apache
<VirtualHost *:80>
    DocumentRoot /opt/librenms/html
    ServerName librenms.local

    <Directory /opt/librenms/html>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

```bash
sudo a2ensite librenms.conf
sudo systemctl restart apache2
```

### Step 6: Set up Cron Jobs

```bash
sudo crontab -u librenms -e
```

Add the following cron job:

```bash
* * * * * /opt/librenms/poller.php 1>> /dev/null 2>&1
```

(Select your editor of choice if prompted, typically `nano` is easiest)

### Step 7: Finalize Installation

Access LibreNMS through `http://<your_VM_IP>/` and complete the installation via the web interface.

## Final Tips:

*   Monitor system resource usage (CPU, RAM) when running these tools, especially when running them simultaneously.  Consider staggering polling intervals if performance is an issue.
*   You can use these tools independently or integrate them.
*   Configure firewall rules as needed for remote access.
*   Choose strong passwords!
*   Adjust the memory allocation for the PHP processes.

This guide provides a basic installation process. You may need to adjust the configuration files and dependencies based on your specific needs and environment.  Good luck!

