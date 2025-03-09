# Network Monitoring Setup on Debian (VirtualBox)

Set up **Cacti**, **Netdata**, and **Nagios Core** on a Debian VM for comprehensive network monitoring. Follow these steps to install, configure, and troubleshoot each tool.

## Prerequisites
* Debian VM in VirtualBox (minimal installation)
* Bridged or NAT network configured for VM access
* Sudo/root access
* Terminal access to the VM
* LAMP stack (Apache, MySQL/MariaDB, PHP) installed:
  ```bash
  sudo apt install -y apache2 mariadb-server php libapache2-mod-php php-mysql php-snmp php-xml php-gd php-curl
  sudo systemctl start apache2 mariadb
  sudo systemctl enable apache2 mariadb
  ```
* Basic security configuration for MySQL:
  ```bash
  sudo mysql_secure_installation
  ```

## 1. Install Cacti (SNMP Graphing)
### Installation

```bash
# Update package repositories
sudo apt update

# Install Cacti and its dependencies
sudo apt install -y cacti snmpd snmp

# During installation:
# - Choose to configure database with dbconfig-common: Yes
# - Enter MySQL password when prompted
# - Select Apache2 when prompted for web server
```

### Configure SNMP Service

```bash
# Edit SNMP configuration
sudo nano /etc/snmp/snmpd.conf

# Add or modify these lines:
# Replace 'public' with your preferred community string
rocommunity public
syslocation "Your Location"
syscontact admin@example.com

# Restart SNMP service
sudo systemctl restart snmpd
sudo systemctl enable snmpd
```

### Access Cacti
Visit `http://<VM_IP>/cacti` in your browser.

Initial login credentials:
- Username: admin
- Password: admin

Follow the setup wizard to complete installation.

### Basic Configuration
1. Add a device to monitor:
   - Go to Console > Create > New Device
   - Enter device IP address and SNMP community string
   - Select appropriate template

2. Create graphs:
   - Select device from the list
   - Choose "Create Graphs for this Host"
   - Select metrics you want to graph

### Troubleshooting
- If you can't access the web interface, check Apache status:
  ```bash
  sudo systemctl status apache2
  ```
- Check logs for errors:
  ```bash
  sudo tail -30 /var/log/apache2/error.log
  ```
- For database issues:
  ```bash
  sudo tail -30 /var/log/mysql/error.log
  ```

## 2. Install NetData (Real-time Monitoring)
### Installation

```bash
# Install dependencies
sudo apt install -y curl wget

# Download and run the installation script
wget -O /tmp/netdata-kickstart.sh https://my-netdata.io/kickstart.sh
sudo sh /tmp/netdata-kickstart.sh --stable-channel
```

### Access NetData
Visit `http://<VM_IP>:19999` in your browser to access the NetData dashboard.

### Basic Configuration
NetData works out of the box, but you can customize:

```bash
# Edit main configuration file
sudo nano /etc/netdata/netdata.conf

# Configure data retention
[global]
  history = 3996    # Default is 1 hour, increase as needed
```

### Configure Alerts (Optional)

```bash
# Edit the health configuration
sudo nano /etc/netdata/health_alarm_notify.conf

# Configure email alerts
sudo nano /etc/netdata/health.d/cpu.conf
```

### Troubleshooting
- If NetData isn't running, check service status:
  ```bash
  sudo systemctl status netdata
  ```
- Check logs for errors:
  ```bash
  sudo tail -30 /var/log/netdata/error.log
  ```
- If port 19999 isn't accessible, verify firewall settings:
  ```bash
  sudo ufw status
  ```

## 3. Install Nagios Core (Service Monitoring)
### Installation

```bash
# Install Nagios and plugins
sudo apt update
sudo apt install -y nagios4 nagios-plugins nagios-nrpe-plugin
```

### Configure Web Access

```bash
# Enable CGI module for Apache
sudo a2enmod cgi

# Enable Nagios configuration in Apache
sudo a2enconf nagios4-cgi

# Restart Apache
sudo systemctl restart apache2
```

### Start and Enable Nagios Service

```bash
# Start Nagios service
sudo systemctl start nagios4

# Enable Nagios to start on boot
sudo systemctl enable nagios4

# Verify Nagios is running
sudo systemctl status nagios4
```

### Set Nagios Admin Password

```bash
# Create or update admin user password
sudo htpasswd -c /etc/nagios4/htpasswd.users nagiosadmin

# Restart Apache to apply changes
sudo systemctl restart apache2
```

### Access Nagios
Visit `http://<VM_IP>/nagios` or `http://<VM_IP>/nagios4` in your browser and log in with:
- Username: nagiosadmin
- Password: (the password you set)

### Basic Configuration
1. Add a new host to monitor:
   ```bash
   sudo nano /etc/nagios4/conf.d/myserver.cfg
   ```
   
   Add the following content:
   ```
   define host {
       use                     generic-host
       host_name               myserver
       alias                   My Server
       address                 192.168.1.100  # Replace with actual IP
       max_check_attempts      5
       check_period            24x7
       notification_interval   30
       notification_period     24x7
   }
   
   define service {
       use                     generic-service
       host_name               myserver
       service_description     PING
       check_command           check_ping!100.0,20%!500.0,60%
       max_check_attempts      5
       check_interval          5
       retry_interval          1
   }
   ```

2. After making changes, verify and reload:
   ```bash
   sudo nagios4 -v /etc/nagios4/nagios.cfg
   sudo systemctl restart nagios4
   ```

### Troubleshooting
- If Nagios web interface isn't accessible:
  ```bash
  # Check if Nagios service is running
  sudo systemctl status nagios4
  
  # Check Apache configuration
  sudo apache2ctl -S
  
  # Check Apache error logs
  sudo tail -30 /var/log/apache2/error.log
  
  # Ensure CGI is enabled
  sudo a2enmod cgi
  sudo systemctl restart apache2
  ```
- Test with a simple page to verify Apache is working:
  ```bash
  echo "Hello World" | sudo tee /var/www/html/test.html
  # Then try accessing http://<VM_IP>/test.html
  ```
- If you have permission issues:
  ```bash
  sudo chown -R nagios:nagios /var/lib/nagios4
  sudo chmod -R 775 /var/lib/nagios4
  ```

## Conclusion
You now have three powerful monitoring tools configured on your Debian VM:
- **Cacti**: For long-term SNMP-based performance graphing
- **NetData**: For real-time system monitoring and visualization
- **Nagios Core**: For service monitoring and alerting

Each tool serves a different monitoring purpose, providing a comprehensive view of your network and system performance.

### Next Steps
- Set up SSL for secure access to web interfaces
- Configure email notifications for alerts
- Add additional hosts and services to monitor
- Customize dashboards and views for your specific needs
