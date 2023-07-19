These steps will walk you through setting up Standalone (site manager) SOAR. 

- [Prerequisites](#prerequisites)
    - [Download the SOAR OVA file](#download-the-soar-ova-file)
    - [Download the SOAR updates and optional files from FixCentral](#download-the-soar-updates-and-optional-files-from-fixcentral)
    - [Ensure you have a VMWare hypervisor tool](#ensure-you-have-a-vmware-hypervisor-tool)
- [Installing the OVA](#installing-the-ova)
- [Configure license](#configure-license)
- [Update SOAR Appliance](#update-soar-appliance)
- [Set system timezone](#set-system-timezone)
- [Create the Initial User](#create-the-initial-user)


# Prerequisites
### Download the SOAR OVA file 

### Download the SOAR updates and optional files from [FixCentral](https://www.ibm.com/support/fixcentral)
- In the top right, click the person icon and make sure you're logged in
- Once logged in, on the Fix Central page you'll see a ```Find Product``` and ```Select Product``` buttons. You can use either to search for QRadar SOAR. 
  - In this example I used ```Find Product```, you simply just type in ```SOAR``` and it'll pop up with a suggestion for ```IBM Security QRadar SOAR (formerly Resilient)```, select it
  - For Installed Version, select the second newest dot version for whichever SOAR OVA you downloaded
    - For Example, if you downloaded SOAR V49, select the second newest version starting with 49. At the time of this writing, the newest version was 49.2.34, so we'll select 49.1.52
    - The purpose is that the tool is asking what you have installed so it can show you whats new. 
  - For Platform, select ```All```
  - Click Continue
  - On the ```Identity fixes``` screen, make no changes and select Continue at the bottom. 
  - Now that the tool has searched, you may see a few options, select (click the checkbox) the newest version .run file available for your version. 
  - Click Continue
  - Click I agree on the pop up to continue
  - At the bottom you'll see three .run files to download. Download all three
    - ```soar-<version>.run``` is the main update file for the SOAR platform 
    - ```soar-appliance-security-update-<version>.run``` is the update file for the underlying operating system the platform is installed on
    - ```soar-optional-packages-repo-<version>.run``` installs an additional repo on your appliance. This is used for additional tools you may need on the appliance. This file is needed for Disaster Recovery. If may be needed for other processes, the IBM documentation will tell you when it's needed. 

### Ensure you have a VMWare hypervisor tool
- Official IBM Documentation for QRadar SOAR has a user install the platform on a VMWare ESXI server, if you have this available you can follow the [IBM Documentation](https://www.ibm.com/docs/en/sqsp/49?topic=guide-deployment)
- For this demo, I am running QRadar SOAR with VMWare Workstation on my endpoint. 
  - Disk: 185GB
  - CPU Cores: 4
  - Memory: 8GB Minimum, 16GB recommended

# Installing the OVA
These directions are for deploying on VMWare Workstation, they are very similiar to the VMWare ESXI directions found on the [IBM Documentation](https://www.ibm.com/docs/en/sqsp/49?topic=guide-deployment) site. 
- Open VMWare Workstation
- Click File > Open 
- Navigate to the OVA file and open it
- An End User License Agreement window will open, Click the ```I accept``` checkbox and then click Next
- Give the new virtual appliance a name
  - This is what it will be called in VMWare Workstation
  - This is the default hostname for the system, but it can be changed after it installs
- The Storage path for the virtual machine will auto populate, but if you'd like to change where it saves, click Browse and select the path
- Click Import, it may take a few minutes to import
- Now that the system has imported, click ```Edit virtual machine settings```
- On the Virtual Machine Settings you can modify the Memory and network settings as needed. 
  - On my endpoint, I lowered the Memory to ```8192MB```
  - For Network settings, I leave it ```Bridged (Automatic)```
- Click OK 
- Power on the Virtual Machine
- Allow the software to install, this may take up to 15 minutes 
- Once the install is complete, you'll be prompted to provide a root password
  - Use a strong password with at least four character classes, such as lowercase, uppercase, digits, and symbols.
  - If these are not met, it'll say Bad Password, however you can move forward
- Now you'll be asked for the ```resadmin``` password
  - Use a strong password with at least four character classes, such as lowercase, uppercase, digits, and symbols.
  - If these are not met, it'll say Bad Password, however you can move forward
- Next the system will ask if the network settings are correct, by default it uses DHCP. 
  - **NOTE:** You can modify the network settings and hostname at a later time by using the command ```nmtui```
  - If acceptable, Type ```Y``` and press Enter. 
  - If you'd like to use a Static IP, type ```N``` and press Enter
    - Walk through the prompts, pressing Enter to continue
      - Static or DHCP
      - IP address
      - Gateway address
      - Netmask address
      - DNS Server IP
      - DNS Search Domain
    - Verify the settings and if correct, type ```Y``` and press Enter
- The system will reboot now

# Configure license 
If you need a license, please check the bookmarks in the [#soar_global_se](https://ibm-security.slack.com/archives/C4ANS80KZ) Slack channel 
- Log in to the SOAR platform using the resadmin user
- Copy the license key, including the header and footer
  - Example
    ```
    -----BEGIN LICENSE-----
    AB+LCAAAAAAAAACFkMtKxUAMhl+lzNZi59rSAUHxioqchecB0rlgse2UyZTjUXx3p7iyLk
    42gS/J/yf5ImBSHya8izC6Q4jvtxN0g7NEp7i4ksAMxv1FBrZgwRRGF1+yBNHk+nVX3A+h
    g6F44AWnXBSkJG7ZTLmPuY+wmhPNGlrXStV0jZLMgJh3ya2k9YzxjoPgILlnraGtAAXWet
    5I5WVWjjDZMObex+fdGM6ubp4+Dxcrd3PYxyEX3lKaUVeVtB1jtKWU2a5ujZS+BQnC1Q2n
    QlmlT7ldrprnJgg8Yk6jllJUK8t2CIBEexgwH4fOLLFPx803F3T/QJx+33ZqOfL9AyFB7G
    itAQAA:MCwCFEGjRtzCTP9o8gBMUV7dP13VBANaAhR0hIlfc05aPoQjkH5vAWZCSPZIGA=
    =
    -----END LICENSE-----
    ```
- Create a new file and paste the key
  ```
  vi license.txt
  ``` 
  - Within vi, press the ```i``` key to change to Insert mode
  - Paste the license, on my system the pasting takes a minute 
  - I am on a Windows system, and when I paste the key, it adds extra blank lines. Remove these while in insert mode
  - Press ```Esc``` to leave Insert mode
  - Enter ```:wq``` to write and quit vi
  - Confirm file is there
    ```
    ls
    ```
  - Confirm file is populated   
    ```
    cat <filename>
    ```
- Import the license 
  - Run the following command to load in the license key 
    ```
    sudo license-import -file <License File>
    ```
  - You'll see a similar message if the license import was successful
    ```
    Successfully imported license
    Customer name:  <customer>
    Expiration:  Not Defined
    US regulators enabled:  true
    CA regulators enabled:  true
    EU regulators enabled:  true
    APAC regulators enabled:  true
    Security module enabled:  true
    Actions framework enabled: true
    Users:  Not Defined
    ```

# Update SOAR Appliance
Now that the base SOAR appliance is installed. Let's update it to the latest version. In the Prerequisites we downloaded the update files. Now we'll transfer them to the system and install them. 
- Transfer the update files (```soar-<version>.run``` and ```soar-appliance-security-update-<version>.run```) to the SOAR appliance 
  - On my Windows system, I have install WSL and am able to use Linux commands 
  - Using SCP 
    ```
    scp <path to file> resadmin@<SOAR IP or FQDN>:/home/resadmin
    ```
- Install the platform update
  ```
  sudo bash soar-<version>.run
  ```
- Enter the resadmin password when prompted
- Install the appliance update
  ```
  sudo bash soar-appliance-security-update-<version>.run
  ```
- Enter the resadmin password when prompted
- Reboot the system
  - If you're unable to reboot the system, you can restart resilient-messaging.service to finish the update
  - However if there are kernel updates, it'll prompt to reboot anyways.
       ```
       sudo systemctl restart resilient-messaging.service
       ```
  ```
  reboot
  ```

# Set system timezone
- View list of available timezones
  ```
  timedatectl list-timezones
  ```
- Set timezone
  ```
  sudo timedatectl set-timezone <time_zone>
  ```
- Restart resilient-messaging.service
  ```
  sudo systemctl restart resilient-messaging.service
  ```

# Create the Initial User
- Log into the SOAR CLI with the Resadmin user account 
- Run the following command to view all options available 
  ```
  sudo resutil newuser -help
  ```
- Create the user
  ```
  sudo resutil newuser -createorg -email "jsmith@example.com" -first "John" -last "Smith" -org "My Company, Inc."
  ```
- The system will prompt for a new password for that user

- **NOTE** If you'd like to create multiple orgs on your platform, run the command again
  - ```-first``` and ```-last``` are only required for the initial user configuration
- **NOTE** To edit an org, run the following to see your options
  ```
  sudo resutil editorg -help
  ```
