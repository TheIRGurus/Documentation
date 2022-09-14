# Setup Instructions for using FN-SCHEDULER
The following instructions will be used to install Postgres on a CentOS/RedHat Server. This will allows us to install the FN-Scheduler app which we be installed and configured at the very end.

### Installing Postgres
Update your system:
```
sudo yum update && sudo yum upgrade
```
Install the Postgres repo:
```
sudo yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```
Note: We will be installing **Postgres v14 (newest version as of date of creation)**, but if you want to install a different version of Postgres run the following command to see the versions available. You will need to install the server version
```
sudo yum list postgresql*
```
Install Postgres v14 Server, initiate the DB for first use, and then set to run on startup:
```
sudo yum install postgresql14-server

sudo /usr/pgsql-14/bin/postgresql-14-setup initdb
sudo systemctl start postgresql-14
sudo systemctl enable postgresql-14
```

### Configuring the DB
Create the user for the scheduler DB:
```
sudo -u postgres createuser --interactive
```
<sub>Note: Use the following options when building your user.</sub>
>role name: scheduler
>
>superuser: Yes
Create the DB for the scheduler app:
```
sudo -u postgres createdb scheduler
```
### Securing the DB
#### The default password for the user created will be the username, let's improve security by changing that password!

Start by logging into the Postgres DB:
```
sudo -u postgres psql
```
Update the password by running the following code, but change the Bracketed area with the wanted password:
```
ALTER ROLE scheduler WITH PASSWORD '[your_password]';
```

### Allowing Remote Access to Postgres DB
Edit the DB config file to configure what remote systems can access the DB:
```
sudo vi /var/lib/pgsql/14/data/pg_hba.conf
```
Add the following line at the bottom of the document within the chart:
>host    scheduler       scheduler       [IP Range - 172.16.100.1/23]         trust

<sub>The above line of code in the file will do the following:</sub>
  
<sub>  - Host - Allows connections from outside of the local system</sub>
  
<sub>  - scheduler - which user is allowed to connect</sub>
  
<sub>  - scheduler - which db the user is allowed to connect</sub>
  
<sub>  - IP Range - Provide the IP or CIDR range to accept connections from</sub>
  
<sub>  - trust - Allow connection attempts following the above rules</sub>

Edit the DB Listener Config
```
sudo vi /var/lib/pgsql/14/data/postgresql.conf
```
Change the following configs within the file:

>listen_address = '*'

<sub>note: uncomment as well</sub>
>port = 5432

<sub>note: uncomment as well</sub>

After changing the above configurations, restart the postgres service:
```
sudo systemctl restart postgresql-14
```
Add a firewall rule to the box to allow inbound traffic and restart the firewall service:
```
sudo firewall-cmd --zone=public --add-port=5432/tcp --permanent
sudo systemctl restart firewalld
```

### Verify Network Connections
Check that the DB Service is listening on the 5432 port:
```
netstat -na | grep 5432
```
Ensure that a connection can be established from the postgres server.
```
nc -v [server_name or IP] 5432
```
<sub>Note: Netcat is not installed on the server, you may need to install it or an alternative like wget.</sub>

Finally from the Apphost server, verify that a connection can be established remotely.
```
nc -v [server_name or IP] 5432
```
<sub>Note: Netcat is not installed on the server, you may need to install it or an alternative like wget.</sub>

### Install Scheduler
Use the following link to grab the newest version of the [FN-SCHEDULER app](https://exchange.xforce.ibmcloud.com/hub/extension/4917b8a4bb53c46a7c63efa4e65238e4).
Install the App within IBM SOAR by navigating to the Apps page within the Administrator Settings. From here you can click the install button, provide the entire zip file from the download above, and click upload.
After completing the installation process, we will use the instructions below to configure and deploy the app.

<sub>Note: This part of the instructions assume that you have already added an apphost. If you have not done that please following the instructions in the KB Article IBM provides for their apphost installation and deployment.</sub>

Edit the app.config file within the Configuration tab, ensuring that the settings below match up:
>timezone = America/New_York
>
>#datastore_dir
>
>db_url = postgresql+pypostgresql://scheduler:[your_password]@[server_name or IP]:5432/scheduler

<sub>Note: Be sure to add your password and server name or IP in the configuration. It is also good to note that if you have any special characters within your password you may wish to turn the entire postgres connector string into a SECRET to ensure the special characters don't interfere with Syntax.

At this point select the apphost to deploy to and run "Test Configuration". Once this is successful, Click "Save and Push Changes" at the top, then in the Details tab, Deploy the App.
