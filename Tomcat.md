# **_Apache Tomcat installation and Configuration_**
This documentation is to iinstall and configure the Apache Tomcat distribution on _**Ubuntu 16.0.4**_ only.

---

## Table of contents
* [Prerequisites](#Prerequisites)
* [Java installation](#java-installation)
* [Tomcat user and group creation](#tomcat-user-group)
* [Tomcat installation](#tomcat-installation)
* [File permission update](#file-permission-update)
* [Tomcat service file](#system-service-file)
* [Firewall configuration](#firewall-config)
* [Tomcat web management configuration](#management-config)

---
<a name="Prerequisites"/>

### Prerequisites
* Ubuntu 16.04 Server edition
* curl
* `sudo` privilege 
* Run `sudo apt -y update && sudo apt -y upgrade`

<a name="java-installation"/>

### Java installation
Install the Java Development Kit
```
sudo apt -y install default-jdk
```

<a name="tomcat-user-group"/>

### Tomcat user and group creation
Create tomcat group
```
sudo groupadd tomcat
```
Create new tomcat user with following configuration
    
   * `/opt/tomcat` as home directory
   * Add to new `tomcat` group
   * Restrict account login `/bin/false`
  
```
sudo useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat
```

<a name="tomcat-installation"/>

### Tomcat installation
Download the preferred [Tomcat distribution](https://tomcat.apache.org/index.html) or acquire the direct link for the `tar.gz` file.

I.E `http://apache.mirrors.nublue.co.uk/tomcat/tomcat-8/v8.5.24/bin/apache-tomcat-8.5.24.tar.gz`

   * Use `curl` to download the binary file
   * Create `tomcat` directory in `/opt`
   * Extract binary file content to `/opt/tomcat`
    
```
curl -O http://apache.mirrors.nublue.co.uk/tomcat/tomcat-8/v8.5.24/bin/apache-tomcat-8.5.24.tar.gz
sudo mkdir /opt/tomcat
sudo tar xzvf apache-tomcat-8*tar.gz -C /opt/tomcat --strip-components=1
```

<a name="file-permission-update"/>

### File permission update
   * Change to `tomcat` directory
   * Give group `tomcat` installation directory ownership
   * Change `conf` folder access permission
   * Give `tomcat` ownership of following folders
       * `webapps/`
       * `work/`
       * `temp/`
       * `logs/`
```
cd /opt/tomcat
sudo chgrp -R tomcat /opt/tomcat
sudo chmod -R g+r conf
sudo chmod g+x conf
sudo chown -R tomcat webapps/ work/ temp/ logs/
```
<a name="system-service-file"/>

### Tomcat service file
   * Create a service file in `/etc/systemd/system/` folder to run Tomcat as service
   * Update `JAVA_HOME` path reflect this system
   * Run following command to find `JAVA_HOME` path.
        ```
        sudo update-java-alternatives -l
        ```
   * Reload the systemd daemon
   * Start Tomcat service
   * Verify the service status
```
sudo vim /etc/systemd/system/tomcat.service
```

   ###### Copy the following content to the file and replace `<PATH-JAVA-HOME>` `JAVA_HOME` path
   
   ###### Update `CATALINA_OPTS` if necessary
   
   
   ```
   [Unit]
   Description=Apache Tomcat Web Application Container
   After=network.target
   
   [Service]
   Type=forking
   
   Environment=JAVA_HOME=<PATH-JAVA-HOME>/jre
   
   Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
   Environment=CATALINA_HOME=/opt/tomcat
   Environment=CATALINA_BASE=/opt/tomcat
   Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
   Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'
   
   ExecStart=/opt/tomcat/bin/startup.sh
   ExecStop=/opt/tomcat/bin/shutdown.sh
   
   User=tomcat
   Group=tomcat
   UMask=0007
   RestartSec=10
   Restart=always
    
   [Install]
   WantedBy=multi-user.target
   ```
```
sudo systemctl daemon-reload
sudo systemctl start tomcat
sudo systemctl status tomcat
```

<a name="firewall-config"/>

### Firewall configuration
   * Allow traffic to port `8080`
   * Enable Tomcat service
```
sudo ufw allow 8080
sudo systemctl enable tomcat
```

<a name="management-config"/>

### Tomcat web management configuration
   * Add Tomcat users to `<tomcat-users . . .></tomcat-users>` block
   * Enable remote management console access.
   * Restart Tomcat service
   
```
sudo vim /opt/tomcat/conf/tomcat-users.xml
```
   ###### Add user account
   ```
   <tomcat-users . . .>
      <user username="username" password="password" roles="manager-gui,admin-gui,etc"/>
   </tomcat-users>
   ```
```
sudo vim /opt/tomcat/webapps/manager/META-INF/context.xml
sudo vim /opt/tomcat/webapps/host-manager/META-INF/context.xml
```
   ###### Comment the following line
   ```
   <Valve className="org.apache.catalina.valves.RemoteAddrValve"
            allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
   ```
```
sudo systemctl restart tomcat
```