# Installing and Configuring Resilient-Circuits Integration Server

**Table of Contents**

- [Installing and Configuring Resilient-Circuits Integration Server](#installing-and-configuring-resilient-circuits-integration-server)
    + [Install CentOS within a Virtual Environment](#install-centos-within-a-virtual-environment)
    + [Installing Python 3](#installing-python-3)
    + [Install Resilinet-Circuits](#install-resilinet-circuits)
    + [Configure Resilient-Circuits for First-Use](#configure-resilient-circuits-for-first-use)
    + [Using Keyring to Store you Secrets](#using-keyring-to-store-you-secrets)
    + [Set Resilient-Circuits to run as a Service on Startup](#set-resilient-circuits-to-run-as-a-service-on-startup)


### Install CentOS within a Virtual Environment

1. Install CentOS Minimal with the following Specs:

	>100 GB Storage
	>
	>4 GB of RAM
	>
	>1 CPU with 2 cores
	
- Enable the Network Interface in the install screen
- Set the root password
- Define the default user (I use intadmin to align with other admin accounts, e.i. resadmin and appadmin).

	<sup>Note: This user should be an admin (gives the ability to use sudo).</sup>

2. **If the network did not get enabled on install**, follow the next step; otherwise skip ahead to installing Python.
- Access the Network Interface configuration tool:
```
nmtui
```
- Go to "Edit a Connection"
- Select the interface you want to enable (should just be 1)
- Select "IPv6 <Automatic>" and change to "<Ignore>"
- Below that Check the box for "Automatically connect"
- Select "OK"
- Leave the nmtui GUI by Selecting "Back" and "Quit"
- If you want to look at network config from ifconfig, install it through yum with the following command:
```
sudo yum install -y net-tools
```


### Installing Python 3

1. Upgrade the Server
```
sudo yum update -y
sudo yum upgrade -y
```
2. Install Python 3 and pip
```
sudo yum install -y python3 python3-pip
```
3. Upgrade PIP
```
sudo python3 -m pip install --upgrade pip
```


### Install Resilinet-Circuits

1. Install Resilient Circuits with PIP
```
pip3 install --upgrade resilient-circuits
```
2. Verify that keyring and idna installed the correct versions
```
pip3 list
```
  <sub>Note: Keyring should be version 7.3 and idna should be version 2.6.</sub>
  
- If these are not the correct version, uninstall the incorrect version using the command below and reinstall with the correct version
  
```
pip3 uninstall [keyring or idna]
pip3 install [keyring or idna]==[required_version]
```


### Configure Resilient-Circuits for First-Use

1. Create the configuration files:
```
resilient-circuits config -c
```
2. Edit and configure the config file that was generated:
```
vi /home/[username]/.resilient/app.config
```
<sub>Note: If you are using keyring to store secrets(passwords), Skip to the next section before continuing on.</sub>

3. Once the config file and secrets are established, verify that resilient-circuits is ready to run without error.
```
resilient-circuits run
```
4. If no errors, move on; else, resolve the errors in the config file.


### Using Keyring to Store you Secrets

1. Add a keyring variable name into the config file using a carrot and the desired variable name (each name must be unique).

>api_key_secret=^SOAR_API_SECRET

2. Establish the config file for the Keyring to search for variables
```
export APP_CONFIG_FILE=/home/[username]/.resilient/app.config
```
3. Run the Resilient Keyring app to save your secrets.
```
res-keyring
```
<sub>Note: This will search through your app.config file for any variables. If the variable is already associated to a secret, you can click enter to leave unchanged; otherwise, enter and confirm your secret.</sub>


### Set Resilient-Circuits to run as a Service on Startup

1. Create a new service file in Linux
```
sudo vi /etc/systemd/system/resilient_circuits.service
```
2. Add the service information to the new file, editing the text below where the username is.

>[Unit]
>
>escription=Resilient-Circuits Service
>
>
>[Service]
>
>Type=simple
>
>User=[username]
>
>WorkingDirectory=/home/[username]
>
>ExecStart=/home/[username]/.local/bin/resilient-circuits run
>
>Restart=always
>
>TimeoutSec=10
>
>Environment=APP_CONFIG_FILE=/home/[username]/.resilient/app.config
>
>Environment=APP_LOCK_FILE=/home/[username]/.resilient/resilient_circuits.lock
>
>
>[Install]
>WantedBy=multi-user.target

3. Edit the file permissions:
```
sudo chmod 664 /etc/systemd/system/resilient_circuits.service
```
4. Enable the newly added service
```
sudo systemctl daemon-reload 
sudo systemctl enable resilient_circuits.service
```
5. Start the service
```
sudo systemctl start resilient_circuits
```
