
# Monitoring services with Nagios in Docker 
This project implements a monitoring system with Nagios, using Docker to manage containers for services such as MySQL and NGINX. It allows monitoring the status and availability of these services in real time.

## Contents

1. Introduction to Nagios

2. Lab Environment Setup using Docker

3. Basic Nagios Configuration

4. Infrastructure Monitoring

5. Server Management

6. Notifications Configuration

7. Backup and Recovery

8. Time Management

9. Event Handlers

10. Practical Exercises

 

## 1. Introduction to Nagios

 

### What is Nagios?

Nagios Core is an open-source monitoring system that enables organizations to identify and resolve IT infrastructure problems before they affect critical business processes. It can monitor:

- Network services (SMTP, POP3, HTTP, NNTP, PING, etc.)

- Host resources (processor load, disk usage, etc.)

- Server and network hardware (temperature, power, etc.)

- Applications and services

 

### Key Features

- Network monitoring

- Infrastructure monitoring

- Server and host monitoring

- Applications and services monitoring

- Alert system

- Performance data graphing

- Log management

 

## 2. Lab Environment Setup

 

We'll use Docker Compose to set up our Nagios environment. This ensures consistency across different systems and easy setup.

 

### Prerequisites

- Docker Desktop installed

- Basic understanding of YAML

- Terminal/Command Prompt access

 

### Setup Steps
0. Create a codespaces on Github / ensure it's path is nagios-monitor-services

```bash
pwd
```
    

2. Create a new directory for your MySQL data:

```bash
mkdir -p data/mysql
```

 

2. Create a `docker-compose.yml` file:

```yaml
version: '3'
services:
  nagios:
    image: jasonrivers/nagios:latest
    ports:
      - "8080:80"
    volumes:
      - ./nagios/config:/opt/nagios/etc
      - ./nagios/plugins:/opt/nagios/libexec
    environment:
      - NAGIOSADMIN_USER=nagiosadmin
      - NAGIOSADMIN_PASS=nagios123
    networks:
      - monitoring_network
    restart: always

  mysql:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=rootpass
      - MYSQL_DATABASE=testdb
      - MYSQL_USER=testuser
      - MYSQL_PASSWORD=testpass
    volumes:
      - ./data/mysql:/var/lib/mysql
    ports:
      - "3307:3306"
    networks:
      - monitoring_network
    restart: always

  nginx:
    image: nginx:latest
    ports:
      - "8081:80"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
    networks:
      - monitoring_network
    restart: always

networks:
  monitoring_network:
    driver: bridge
```
 

4. Start the environment:

```bash
docker-compose up -d
```

5. Access Nagios web interface:

- Click on the codespace (ports and open the browser link) 
- Username: nagiosadmin
- Password: nagios123

 

## 3. Basic Nagios Configuration

### Understanding Configuration Files
Key configuration files in Nagios:

```

/opt/nagios/etc/
├── nagios.cfg           # Main configuration file
├── cgi.cfg             # CGI configuration
├── objects/
    ├── commands.cfg    # Command definitions
    ├── contacts.cfg    # Contact definitions
    ├── timeperiods.cfg # Time period definitions
    ├── templates.cfg   # Template definitions
    └── localhost.cfg   # Host definitions
```

### Basic Host Monitoring Configuration

1. Create a new host configuration file:

```bash
docker exec -it nagios-lab_nagios_1 bash
cd /opt/nagios/etc/objects/
apt-get update
apt-get install vim
```

2. Add a new host configuration (`target-host.cfg`):

```ini

define host {
    use                     linux-server
    host_name               target-host
    alias                   Target Host
    address                 target-host
    max_check_attempts      5
    check_period           24x7
    notification_interval   30
    notification_period    24x7
}

define service {
    use                     generic-service
    host_name               target-host
    service_description     PING
    check_command          check_ping!100.0,20%!500.0,60%
}

```

 

3. Verify and reload configuration:

```bash
/opt/nagios/bin/nagios -v /opt/nagios/etc/nagios.cfg
```


## 4. Infrastructure Monitoring

### Setting Up Basic Service Checks

 

1. HTTP Monitoring:

```ini

define service {

    use                     generic-service
    host_name               target-host
    service_description     HTTP
    check_command          check_http
    check_interval         5
}

```

 

2. System Resource Monitoring:

```ini
define service {

    use                     generic-service
    host_name               target-host
    service_description     CPU Load
    check_command          check_nrpe!check_load
}

```

 

### Installing NRPE on Target Host

 

1. Enter the target container:

```bash
docker exec -it nagios-lab_target-host_1 bash
```

 

2. Install NRPE:

```bash
apt-get update
apt-get install -y nagios-nrpe-server nagios-plugins
```

 

## 5. Server Management

 

### Managing Nagios Services

 

Basic management commands:

```bash
# Start Nagios
systemctl start nagios
# Stop Nagios
systemctl stop nagios
# Restart Nagios
systemctl restart nagios
# Check Status
systemctl status nagios
```

 

### Checking Configuration

```bash
/opt/nagios/bin/nagios -v /opt/nagios/etc/nagios.cfg
```

 

## 6. Notifications Configuration

 

### Setting Up Email Notifications

 

1. Configure contact in `contacts.cfg`:

```ini

define contact {

    contact_name            nagiosadmin
    use                     generic-contact
    alias                   Nagios Admin
    email                   keyon_genesis_kanan@gmail.com
    service_notification_period     24x7
    host_notification_period        24x7
    service_notification_options    w,u,c,r
    host_notification_options       d,u,r
    service_notification_commands   notify-service-by-email
    host_notification_commands      notify-host-by-email
}

```

 

2. Configure email command in `commands.cfg`:

```ini
define command {
    command_name    notify-service-by-email
    command_line    /usr/bin/printf "%b" "Service: $SERVICEDESC$\nHost: $HOSTNAME$\nState: $SERVICESTATE$\n\nDate/Time: $LONGDATETIME$\n\nAdditional Info:\n\n$SERVICEOUTPUT$" | /usr/bin/mail -s "** $NOTIFICATIONTYPE$ Service Alert: $HOSTNAME$/$SERVICEDESC$ is $SERVICESTATE$ **" $CONTACTEMAIL$
}

```

 

## 7. Nagios Core Backups

 

### Backup Script

 

Create a backup script (`backup-nagios.sh`):

```bash

#!/bin/bash

BACKUP_DIR="/opt/nagios/backups"

DATE=$(date +%Y%m%d_%H%M%S)

 

# Create backup directory

mkdir -p $BACKUP_DIR

 

# Backup configuration files

tar -czf $BACKUP_DIR/nagios_conf_$DATE.tar.gz /opt/nagios/etc/

 

# Backup historical data

tar -czf $BACKUP_DIR/nagios_var_$DATE.tar.gz /opt/nagios/var/

 

# Keep only last 7 days of backups

find $BACKUP_DIR -type f -mtime +7 -delete

```

 

## 8. Time Management

 

### Configuring Time Periods

 

Example time period configuration in `timeperiods.cfg`:

```ini

define timeperiod {

    timeperiod_name workhours
    alias           Normal Work Hours
    monday          09:00-17:00
    tuesday         09:00-17:00
    wednesday       09:00-17:00
    thursday        09:00-17:00
    friday          09:00-17:00
}

```

 

## 9. Event Handlers

 

### Creating a Basic Event Handler

 

1. Create a script (`restart-service.sh`):

```bash

#!/bin/bash

SERVICE=$1

systemctl restart $SERVICE

```

 

2. Configure the event handler:

```ini

define command {

    command_name    restart-service

    command_line    $USER1$/restart-service.sh $ARG1$

}

 

define service {

    use                     generic-service

    host_name               target-host

    service_description     HTTP

    check_command          check_http

    event_handler          restart-service!apache2

    event_handler_enabled   1

}

```

 
 

## Troubleshooting Guide

Common issues and solutions

1. Web Interface Not Accessible

```bash
# Check Apache status
docker exec nagios-lab_nagios_1 service apache2 status
 
# Check Nagios logs
docker exec nagios-lab_nagios_1 tail -f /opt/nagios/var/nagios.log
```

 

2. Configuration Verification Fails

```bash
# Check syntax
docker exec nagios-lab_nagios_1 /opt/nagios/bin/nagios -v /opt/nagios/etc/nagios.cfg
```

 

3. Service Checks Not Working

```bash
# Test NRPE
docker exec nagios-lab_nagios_1 /opt/nagios/libexec/check_nrpe -H target-host
```

 

## References
- Nagios Core Documentation: https://www.nagios.org/documentation/
- Nagios Plugins Documentation: https://www.nagios.org/downloads/nagios-plugins/
- NRPE Documentation: https://github.com/NagiosEnterprises/nrpe

 

