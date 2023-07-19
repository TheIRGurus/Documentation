These directions are for the setup and configuration of Disaster Recovery for an on-site QRadar SOAR appliance. This example is using V49 of QRadar SOAR. 

These are an interpretation of the official IBM documents found [here](https://www.ibm.com/docs/en/sqsp/49?topic=guide-overview). 

# Prerequisites 
- Create second SOAR instance with same version as Primary/Production
  - Check SOAR version
    ```
    sudo resutil -v
    ```
- Resadmin password should be same on both systems
- Make sure to have the optional packages from FixCentral (```soar-optional-packages-repo-<version>.run```)
- Setup SSH Keys for resadmin on both systems

    ```
    ssh-keygen -t rsa -b 4096 -C "resadmin@machine_a" 
    ssh-copy-id -i /home/resadmin/.ssh/id_rsa.pub resadmin@machine_a
    ```

    ```
    ssh-keygen -t rsa -b 4096 -C "resadmin@machine_b" 
    ssh-copy-id -i /home/resadmin/.ssh/id_rsa.pub resadmin@machine_b
    ```

- Make sure you can SSH between the systems

    ```
    ssh resadmin@machine_a  
    ssh resadmin@machine_b
    ```	

- Ports 22 (ssh) and 5432 (postgres) must be open between the systems. 

# Installing DR packages - Perform on both systems
```
sudo yum install -y resilient-dr
```
To check dr version
```
sudo yum info resilient-dr
```

# Setting Up - Perform on both systems
**Reminder: The resadmin sudo password needs to be the same on both systems**


Copy the soar-optional-packages-repo.run file to the /tmp directory 
  - /tmp so that file can be auto removed later by system
    - Example SCP command
        ```
        scp <Path to file on local machine> resadmin@<SOAR IP/Hostname>:/tmp
        ```
Install Optional package
  ```
  sudo bash soar-optional-packages-repo.run
  ```

Install the lsyncd package (this also installs rsync)
  ```
  sudo yum install lsyncd
  ```
Generate a public/private ssh key pair on the appliance for the resfilesync user
    - When prompted where to save the file, set the path to ```/usr/share/resilient-dr/ansible/files/id_rsa```
    - When prompted for a password, press Enter to leave blank. It's required not to be specified. 
  ```
  ssh-keygen -t rsa -b 4096 -C "resfilesync@res-dr"
  ```

Copy the ```/usr/share/resilient-dr/ansible/templates/ssh_vault.template.yml``` file to ```/usr/share/resilient-dr/ansible/files``` and rename to ```ssh_vault.yml```
    ```
    cp /usr/share/resilient-dr/ansible/templates/ssh_vault.template.yml /usr/share/resilient-dr/ansible/files/ssh_vault.yml
    ```

Copy the public and private keys you created earlier and paste into the ```/usr/share/resilient-dr/ansible/files/ssh_vault.yml``` file
  - Make sure to maintain correct indentation and spacing, else the YAML file won't work
  - If using ```vi``` to modify the file. Instead of manually spacing each line, you can use the following commands:
    - Show number lines
      ```
      :set number
      ```
    - Change the first and last number to correspond to the line number of the header and footer of the key. 
      - Note there are 8 spaces between the last / and the preceding /
      ```
      :9,59s/^/        /
      ```
  
  
- ```ssh_vault.yml``` before modification
  ```
    # Place your private and public ssh keys here
    ssh_key_files:
      /home/resfilesync/.ssh/id_rsa:
        owner: "resfilesync"
        group: "resfilesync"
        mode: "0600"
        content: |
          <INSERT_PRIVATE_KEY_HERE>
      /home/resfilesync/.ssh/id_rsa.pub:
        owner: "resfilesync"
        group: "resfilesync"
        mode: "0644"
        content: |
          <INSERT_PUBLIC_KEY_HERE>
  ```
- Example after modification
  ```
    # Place your private and public ssh keys here
    ssh_key_files:
      /home/resfilesync/.ssh/id_rsa:
      owner: "resfilesync"
      group: "resfilesync"
      mode: "0600"
      content: |
        -----BEGIN RSA PRIVATE KEY-----
        MIIJKQIBAAKCAgEAsO4znU2B6rESLao5MetjScpuTefV89PflTRnh0iEVuzck/QY
        No0y5oWyC1r8BRRveTjYJM8uf4QC2RyN1VMQFA66sqPBpsegHayQmrEfssIgcHh6
        u3m7DReh6l0PyEb86fG92yAywAvzeLyFqRfjaUUws79/bDInBNWRNjilufCovGaE
        PV+l3wOvi6mZ3L8Ps4khxxeXqmrgAKSG/ElJYuE9JFhCFN1JpNAgvxArxoR8gKaO
        oa4X/vvuhp2K0cFvNTTXA/rzsiiazkt4b812IJnLYTHwLxY1eL6PGKJyUBUc4531
        wdefwdfadfaerfaef233refdzvdfsavq34rtefcdsfv4fr234feq3fqervavdvvc
        eddfvqerfgyougettheideavszdfgvsbdfgbnrstghwergsvddvaegfvaefgsfbg
        9bVm01CVKUJtxcbd77nDDLp1LGb3FsAnvj4atbiu/q0LPygKkIAFXedEV0psWlwH
        gCM9n3Dzs9qm7r9kyfH0ud9YYS/nz6x0i6ZRF5jwhNy0C+PWQeyVzWlv3QfN9ALY
        Z1IPNVScxAjJCzDjqhP_____SAMPLE_____iQIv/UQGws1nyKJWFaeffeffFEFEF
        YvOafmpudnpMFlILHNl8t1c4AZEL/1Jt0TF9bCbBnQ0tlEZ9WkrvFCcHw1OhpIHH
        1KMpSGSWUfVw9TatZf0icwQRPxwImC2vAoIBAQCYoHcAPeYlFXcNZfVvLUQtA6qr
        nlrxrkLZDCZTrgCt4QxU5egGWFlosjHk0TqCElBq5aB4fjPTck/PNLGvdLcxlsnN
        piafGrbCbxuZ1kqqPhbVCA7bjsgW5X7ljLwhhE4B6iOuB7j/sHFD7kMhYamtVJRR
        B+C9KUNx3hTc8ILDbnC4O6lijJzPmx1F3L3OeHbzHB/tXNZdLIrL5Gt/ckA3+0ow
        NuOuaZV2FLlyVa0UjUaAKirN/c68d/G3MJdQFEsARI9TNc+uhOCY+8rTr35dAliG
        LfsxTtDA+1ftKmt+I28PdgZpPz0X9QAws6Wd/QCIAI3BF5Dz7rT02KveJKPOEFJU
        -----END RSA PRIVATE KEY-----
    /home/resfilesync/.ssh/id_rsa.pub:
      owner: "resfilesync"
      group: "resfilesync"
      mode: "0644"
      content: |
        ssh-rsa AAAAB3NzaC1yc2EAAADFVBNJR56SO2djdvdvvgyuoldfgwADA==resfilesync@resilient.localdomain
  ```

Activate the Ansible environment so that we can encrypt the ```ssh_vault.yml``` file 
  ```
  source /opt/ansible-venv/python/ansible-python-env-latest/bin/activate 
  ```
 Encrypt the ssh_vault.yml file to protect the keys 
  - Use the same password on both appliances 
  - Below assumes you are in the ```/usr/share/resilient-dr/ansible/files/``` folder
  ```
  ansible-vault encrypt ssh_vault.yml
  ```

# Configuring Postgres for SSL 
The following steps are required to ensure a secure connection between the two SOAR systems and prevent the possibility of a Man-in-the-Middle (MITM) attack. 

For full documentation on this step, please refer to the [IBM Documentation](https://www.ibm.com/docs/en/sqsp/49?topic=up-step-3-configuring-postgres-ssl) and [PostgreSQL Documentation](https://www.postgresql.org/docs/current/libpq-ssl.html#LIBPQ-SSL-PROTECTION)

Below are the SSL options, from most secure to least 
- ```verify-full``` - This requires a ```server.crt```,```server.key```, and ```root,crt``` file. 
- ```verify-ca``` - This requires a ```server.crt```,```server.key```, and ```root,crt``` file. 
  - Might not verify the common name, depending on the certificate authority
- ```require```
  - Does not provide server identity verification and does not require the server to be trusted by the Certificate Authority.
  - However, it does provide data encryption between the SOAR appliances. 

For demo purposes, I followed this [blog](https://www.cherryservers.com/blog/how-to-configure-ssl-on-postgresql) to create self-signed certificates. 

## Create keys and certs
Create a server key, you'll be prompted to set a password. Type and create it.
```
openssl genrsa -aes128 2048 > server.key 
```

To use this key, we now need to remove the password.
```
openssl rsa -in server.key -out server.key
```

Create server certificate based on server key 
```
openssl req -new -key server.key -days 365 -out server.crt -x509
```

Since we're using a self signed certificate for this demo, the server.crt will be the root.crt as well
```
cp server.crt root.crt
```

Once these three files are created, follow the "Manually installing postgres SSL certificates" [section](https://www.ibm.com/docs/en/sqsp/49?topic=up-step-3-configuring-postgres-ssl#task_wbb_y1h_v2b__title__1)

## Copy certs and keys to appropriate places 
1. On one of the appliances, create the /crypt/postgresql directory with the owner:group set to postgres:postgres and the permissions set to 0750.
```
sudo install -d -m 750 -g postgres -o postgres /crypt/postgresql
```

2. Place the server.crt in the /crypt/postgresql directory with the owner:group set to postgres:postgres and the permissions set to 0644.
```
sudo cp <path to server.crt> /crypt/postgresql/server.crt && sudo chown postgres:postgres /crypt/postgresql/server.crt && sudo chmod 0644 /crypt/postgresql/server.crt
```
3. Place the server.key in the /crypt/postgresql directory with the owner:group set to postgres:postgres and the permissions set to 0600.
```
sudo cp <path to server.key> /crypt/postgresql/server.key && sudo chown postgres:postgres /crypt/postgresql/server.key && sudo chmod 0600 /crypt/postgresql/server.key
```
4. On the other appliance:
   1. Create the .postgresql directory in the home directory of the postgres user, for example, /var/lib/pgsql/.postgresql with the owner:group set to postgres:postgres and the permissions set to 0731.
       - Find directory of postgres user
        ```
        -bash-4.2$ ~postgres
        -bash: /var/lib/pgsql: Is a directory
        ```
       - Create directory and set owner/permissions
        ```
        sudo install -d -m 731 -g postgres -o postgres /var/lib/pgsql/.postgresql
        ``` 

   2. To enable the postgres client to trust the server.crt, place the root.crt in the .postgresql directory with the owner:group set to postgres:postgres and the permissions set to 0644.
        - Transfer root.crt from other appliance
        ```
        sudo scp resadmin@<other appliance>:<path to root.crt> /var/lib/pgsql/.postgresql/root.crt
        ```
        - Set owner:group and permissions
        ```
        sudo chown postgres:postgres /var/lib/pgsql/.postgresql/root.crt && sudo chmod 0644 /var/lib/pgsql/.postgresql/root.crt
        ```

**Tip:** To check that OpenSSL on the appliance can verify the server certificates using the root certificate, run the following command as the root or postgres user from the /crypt/postgresql directory on either of the appliances:
```
openssl verify -CAfile root.crt server.crt
```

# Creating Ansible inventory files
### Create the inventory file on the primary appliance.
1. Copy the /usr/share/resilient-dr/ansible/templates/inventory.template.yml file and save it to the /usr/share/resilient-dr/ansible/inventories folder with a name that represents the primary appliance, such as resilient_hosts_primary_machine_a.yml.
        ```
        cp /usr/share/resilient-dr/ansible/templates/inventory.template.yml /usr/share/resilient-dr/ansible/inventories/resilient_hosts_primary_machine_a.yml
        ```
   
2. Edit the file and make the following changes:
        
        | Variable      |  Value |
        |---------------|--------|
        | master_hosts  | Change <REPLACE_ME_WITH_AN_IP_OR_FQDN> to the IP address or fully qualified domain name of appliance A. <br />This is used by Ansible for targeting when running the playbooks.|
        | receiver_hosts | Change <REPLACE_ME_WITH_AN_IP_OR_FQDN> to the fully qualified domain name or IP address of appliance B. <br />This is used by Ansible for targeting when running the playbooks. |
        | inv_vars_master_host | Change <REPLACE_ME_WITH_AN_FQDN> to the fully qualified domain name of appliance A. <br /> This must match the common name of the SSL server certificates for this instance. |
        | inv_vars_receiver_host | Change <REPLACE_ME_WITH_AN_FQDN> to the fully qualified domain name of appliance B.<br /> This must match the common name of the SSL server certs for this instance. |
        | inv_vars_master_host_firewalld_range | Specify the range of IP addresses that can interact with the appliance through the postgres port. <br />The range is enforced using firewalld on the primary appliance only. The range must be set using a netmask, and the IP address must be the IP address of the receiver host. |
        | inv_vars_master_host_firewalld_network_zone | Specify the network zone to which you want Ansible to add the firewalld postgres connection rule.<br /> This is set to the default value of internal. |
        | inv_vars_postgres_ssl_certs_vault_file | Specify the user-created SSL certs vault file (created from ansible/templates/ssl_certs_vault.template.yml) within ansible/files used to store the master host SSL certificates (server.crt, server.key, and root.crt).<br /> This must be the set of certs created for this machine. Do not specify the inv_vars_postgres_ssl_certs_vault_file variable unless you are using the supply method for the postgres SSL certs, which is specified in the group_vars/all/vault file.|
    

3. Copy ```/usr/share/resilient-dr/ansible/inventories/resilient_hosts_primary_machine_a.yml``` to the secondary appliance
    ```
    scp /usr/share/resilient-dr/ansible/inventories/resilient_hosts_primary_machine_a.yml resadmin@<secondary appliance IP or FQDN>:/usr/share/resilient-dr/ansible/inventories/
    ```


### Create the inventory file for the secondary appliance.
1. Copy the /usr/share/resilient-dr/ansible/templates/inventory.template.yml file, and save it to the /usr/share/resilient-dr/ansible/inventories folder with a name that represents the secondary appliance, such as resilient_hosts_secondary_machine_b.yml.
   ```
   cp /usr/share/resilient-dr/ansible/templates/inventory.template.yml /usr/share/resilient-dr/ansible/inventories/resilient_hosts_primary_machine_b.yml
   ```

2. Edit the file and make the following changes:
    | Variable | Value |
    |----------|-------|
    | master_hosts | Change <REPLACE_ME_WITH_AN_IP_OR_FQDN> to the IP address or fully qualified domain name of appliance B. <br /> This is used by Ansible for targeting when running the playbooks. |
    | receiver_hosts | Change <REPLACE_ME_WITH_AN_IP_OR_FQDN> to the fully qualified domain name or IP address of appliance A. <br /> This is used by Ansible for targeting when running the playbooks. |
    | inv_vars_master_host | Change <REPLACE_ME_WITH_AN_FQDN> to the fully qualified domain name of appliance B. <br /> This must match the common name of the SSL server certificates for this instance. |
    | inv_vars_receiver_host | Change <REPLACE_ME_WITH_AN_FQDN> to the fully qualified domain name of appliance A. <br /> This must match the common name of the SSL server certs for this instance. |
    | inv_vars_master_host_firewalld_range | Specify the range of IP addresses that can interact with the appliance through the postgres port. <br /> The range is enforced using firewalld on the primary appliance only. The range must be set using a netmask, and the IP address must be the IP address of the receiver host. |
    | inv_vars_master_host_firewalld_network_zone | Specify the network zone to which you want Ansible to add the firewalld postgres connection rule. <br /> This is set to the default value of internal. |
    | inv_vars_postgres_ssl_certs_vault_file | Specify the SSL certificate vault file (created from ansible/templates/ssl_certs_vault.template.yml) within ansible/files/ used to store the master_host ssl certificates (server.crt, server.key and root.crt). This must be the set of certs created for this machine. <br /> Do not specify the inv_vars_postgres_ssl_certs_vault_file variable unless you are using the supply method for the postgres SSL certs, which is specified in the group_vars/all/vault file.|

3. Copy ```/usr/share/resilient-dr/ansible/inventories/resilient_hosts_primary_machine_b.yml``` to the primary appliance
    ```
    scp /usr/share/resilient-dr/ansible/inventories/resilient_hosts_primary_machine_b.yml resadmin@<primary appliance IP or FQDN>:/usr/share/resilient-dr/ansible/inventories/
    ```

# Creating Ansible vault files 

The following directions are to be performed on both appliances.

1. Copy the vault.template file to /usr/share/resilient-dr/ansible/group_vars/all/ and rename vault
    ```
    cp /usr/share/resilient-dr/ansible/templates/vault.template /usr/share/resilient-dr/ansible/group_vars/all/vault
    ```
2. Open ```/usr/share/resilient-dr/ansible/group_vars/all/vault``` in a text editor of your choice and edit the following fields: 

    | Variable | Value |
    |----------|-------|
    | vault_ssl_certs_supply_method | If you are managing your certs manually, make sure this variable is set to manual, which is the default. <br /> If you want to use the supply method, set this variable value to supply. <br /> <br /> **Note:** You do not need to set the vault_root_cn variable, specified in the group_vars/all/vault, file unless you are using the generate option for the postgres SSL certificates. The generate option is for test purposes only and is not recommended for production environments.|
    | vault_postgres_ssl_security_level | Specify the security option that you are using for postgres replication streaming. The supported security options are verify-full, verify-ca, and require. The default is verify-full, which is the highest security setting. It requires the common name to match the server identity and verifies using the root cert that the server is a trusted host. <br /><br /> **Note:** Do not change the vault_postgres_base_path variable unless you have a custom Postgres data directory path. Do not change the vault_postgres_service_name variable unless you are using a different version of Postgres than v12. This is unlikely on the SOAR appliance, unless you have upgraded Postgres to a version other than v12.|
    | vault_postgres_replication_db_password | change <REPLACE_WITH_REP_DB_PASSWORD> to the password to be used when the system creates the postgres replication user. <br /> This password is used when the receiver appliance is connecting to the master appliance to replicate Postgres data. <br /><br /> Note: Do not use the following characters in the password: ```" ' ) ( ; .```|
    | vault_vars_maximum_async_wait_in_minutes | **Optional** <br /> The parameter sets the maximum amount of time to wait for a single Asynchronous Ansible task to complete in minutes. This is used for long running tasks such as backing up the receiver db and performing a pg_basebackup of the master db to the receiver. The default value is 180 minutes, although very large databases may require more time, depending on db size and in some cases the network conditions. <br /> <br /> **IMPORTANT:** This parameter must have a value in the vault file.|
    |  vault_vars_poll_interval_in_seconds | **Optional** <br /> The parameter sets the poll interval time (in seconds) specifying how long to wait between each new ssh connection to check if the Asynchronous task has completed. The default value in the template is 15 seconds, which means a new ssh connection is established every 15 seconds to check for task completion until the timeout value is reached or the task itself completes. <br /><br /> **IMPORTANT:** This parameter must have a value in the vault file.|

3. Activate the Ansible environment 
   ```
   source /opt/ansible-venv/python/ansible-python-env-latest/bin/activate 
   ```
4. Encrypt the vault file in ansible/group_vars/all by running the following command from the /usr/share/resilient-dr/ansible directory:
    ```
    ansible-vault encrypt group_vars/all/vault
    ```

# Disaster Recovery Playbooks

Now that the configuration files are setup, we can now confirm the appliances can communicate and enable the Disaster Recovery Ansible playbooks. 

1. Activate the Ansible environment
    ```
    source /opt/ansible-venv/python/ansible-python-env-latest/bin/activate 
    ```
2. Check that SSH is set up correctly.
    ```
    ansible all -i inventories/<resilient_hosts_master_machine_a.yml> -m ping
    ```

When you run the playbooks, you must run them from the /usr/share/resilient-dr/ansible directory.

The playbook prompts you for both the sudo password and the vault password.

**Note:** DR Ansible playbooks can run for several hours, depending on the size of their data. To avoid problems with brief network interruptions, it is recommended to run the commands in the background by using the Linux® screen tool or other tools.

## Enabling the Disaster Recovery Playbooks

We will now enable the DR playbooks. These will be activate from the Primary appliance. 

Before you enable DR for the first time, or after you upgrade, complete a system back up on the receiver host. After you enable DR, you cannot do a database restore on the receiver host to return it to its previous state. For more information about the different backup methods that are available, see [Backup and restore](https://www.ibm.com/docs/en/SSBRUQ_49.0.0/doc/install/backup_restore.html).

1. Activate the Ansible Environment 
   ```
   source /opt/ansible-venv/python/ansible-python-env-latest/bin/activate 
   ```
2. Ensure you're in the ```/usr/share/resilient-dr/ansible``` directory
   ```
   cd /usr/share/resilient-dr/ansible
   pwd
   ```
3. Enable DR from the primary appliance 
   ```
   ansible-playbook -K -i inventories/<resilient_hosts_primary_machine_a.yml> enable_dr.yml --extra-vars "skip_receiver_db_backup=true"
   ```
   - ```--extra-vars "skip_receiver_db_backup=true"``` reduces downtime by not backing up the receiver database.
   - You'll be prompted for the resadmin and vault passwords

## Checking the status of the DR sync
- Run the following on the primary appliance
  - This binary gets installed when the enable_dr.yml playbook is ran. If it is not found, then there was an error in the playbook.
  ```
  sudo /usr/bin/resDrStatus
  ```
- Ensure the resilient-filesync service is running on the primary appliance. 
  - If running correctly, the service is active and running the following process:
    - ```/usr/bin/lsyncd -nodaemon /usr/share/co3/conf/lsyncd/lsyncd.conf.lua```

  ```
  sudo systemctl status resilient-filesync
  ```
- 