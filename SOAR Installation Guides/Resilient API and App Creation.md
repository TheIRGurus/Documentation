# Installing and Configuring Resilient for Development
<sub>Expectations/Requirements: Python 3 is already installed</sub>

This guide is to be used for building development environments for using the Resilient API and creating integrations to increase automation of the platform.

**Table of Contents**

  * [Building a Virtual Environment for App Development](#building-a-virtual-environment-for-app-development)
    + [Setup a Virtual Environment for Development](#setup-a-virtual-environment-for-development)
    + [Install Resilient for API and App Building](#install-resilient-for-api-and-app-building)
    + [Setting up Resilient_Circuits for Authorization](#setting-up-resilient_circuits-for-authorization)
    + [Installing Docker](#installing-docker)
  * [Using the API](#using-the-api)
    + [Accessing API](#accessing-api)
    + [Using finfo](#using-finfo)
    + [Using Gadget](#using-gadget)
  * [Developing Apps](#developing-apps)
    + [Building the Base Code for an Integration](#building-the-base-code-for-an-integration)
    + [Build and Run the Docker Container for your App](#build-and-run-the-docker-container-for-your-app)
    + [Documenting Your App](#documenting-your-app)
  * [Publishing and Using a Private Registry](#publishing-and-using-a-private-registry)
    + [Connecting and Uploading to Registry from Docker](#connecting-and-uploading-to-registry-from-docker)
    + [Configuring the App and AppHost for Registry Use](#configuring-the-app-and-apphost-for-registry-use)
    + [Import the Private App into Resilient](#import-the-private-app-into-resilient)


## Building a Virtual Environment for App Development

### Setup a Virtual Environment for Development

1. Install virtualenv for python:
	```
	pip install virtualenv
	```
2. Make a virtual environment root folder:
	```
	mkdir ~\.python_envs
	```
3. Create virtual environment:
	```
	virtualenv --python=python3 ~\.python_envs\<name_of_environment>
	```
4. Activate the virtual environment:
	```
	~\.python_envs\<name_of_environment\Scripts\activate
	```


### Install Resilient for API and App Building

1. (Windows Only) Enable Long Paths
  - Open gpedit.msc
	
  - Navigate to the NTFS Filesystem:

		[Local Computer Policy\\Computer Configuration\\Adminstrative Templates\\System\\Filesystem]
		
  - Set "Enable Win32 long paths" to Enabled

2. Install the Resilient Python Modules:
	```
	pip install resilient
	pip install resilient-circuits
	pip install resilient-sdk
	```
3. (Optional) Install modules for Testing:
	```
	pip install pytest-resilient-circuits
	pip install resilient-lib
	```


### Setting up Resilient_Circuits for Authorization

1. Configure Resilien-Circuits to run using the Python Virtual Environment
  - Navigate to the Virtual Environment version of Resilien-Circuits:
	  
		[~\.python_envs\<name_of_environment>\Lib\site-packages\resilient_circuits]
		
  - Edit app.py
  - Add the code below just under the imports at the very top:
  	```
    	os.environ["APP_CONFIG_FILE"] = r"C:\\.python_envs\\<name_of_environment>\\app.config"
  	```
2. Create the app.config file needed for authentication:
    ```
    resilient-circuits config -c
    ```
3. Edit the Config file:

		[~\.python_envs\<name_of_environment>\app.config]
	
	- Add Server Name
	- Add Authentication information
	- Set log file location
	- Add Resilient Cert Location OR Set cert to not be verified


### Installing Docker

1. Install Docker from [Dockers Website](https://www.docker.com/get-started)
2. (Windows Only) Install [Linux Kernal](https://docs.microsoft.com/en-us/windows/wsl/install-win10#step-4---download-the-linux-kernel-update-package)
3. Set Docker up for Use on CommandLine
	- Open Environment Variables
	
		>Shortcut: WIN + R
		>
		>Type: systempropertiesadvanced
		>
		>Click On: Environment Variables
		
	- Edit the System variables Path
	- Add a new entry with the Docker Install Location
	
			(Default: C:\Program Files\Docker\Docker\resources)
		
4. Run Docker (as admin)



## Using the API

### Accessing API

1. Start Python in Commandline:
	```
	python
	```
2. Authenticating to Resilient

	Method 1(Preferred):
	
	- Import the Resilient Python Module:
		```py
		import resilient
		```
	- Parse the config file:
		```py
		parser = resilient.ArgumentParser(config_file=resilient.get_config_file())
		opts = parser.parse_args()
		```
	- Create the Client (used to connect to Resilient):
		```py
		client = resilient.get_client(opts)
		```
	
	Method 2:
	
	- Import the Resilient Python Module:
		```py
		import resilient
		```
	- Create the client (Must know org_name and Resilient_URL):
		```py
		client = resilient.SimpleClient(org_name="<org_name>",base_url="<url_of_server>", proxies=<None | dict_of_proxies>, verify=<False | path_of_PEM_file>)
		```
	- Connect using User Email and Password:
		```py
		client.connect('<username>','<password>')
		```
3. Using the Client, start browsing the API.

	><sub>Interactive API Help: <Resilient_URL>/docs/rest-api/ui/index.html</sub>
	>
	><sub>Basic API Help: <Resilient_URL>/docs/rest-api/index.html</sub>
	>
	><sub>Documentation on Resilient Client Options: https://ibmresilient.github.io/resilient-python-api/pages/resilient/resilient.html</sub>
	
- Using the client, use all kinds of http options: get,put,delete,patch,post,search
	```py
	client.get(<uri>)
	client.put(<uri>, <payload_dict>)
	client.post(<uri>, <payload_dict>)
	client.delete(<uri>)
	client.get_put(<uri>, <function>)
	```


### Using finfo

   <sub>KB for finfo: https://www.ibm.com/support/pages/rest-api</sub>


### Using Gadget

   <sub>KB for Gadget: https://www.ibm.com/support/pages/rest-api</sub>



## Developing Apps

### Building the Base Code for an Integration

1. Build the Playbook/Workflow within Resilient before moving on.

	><sub>IBM KB Detailing Playbook Design: https://www.ibm.com/docs/en/rsoa-and-rp/42?topic=pdg-introduction</sub>
	
2. Use Resilient-SDK to generate base code by defining the internal parts of the app.
	```
	resilient-sdk codegen -p <name_of_package> -m <message_destination> -f <list_of_functions> -w <list_of_workflows> -s "<list_of_scripts>" -r "<list_of_rules>" -a <list_of_artifactTypes> -fd <list_of_fields> -d <list_of_dataTables> -t <list_of_tasks>
	```
	<sub>Note: If you change any parts within the Resilient Application for the app, use the script below to reload the codegen code</sub>
	
	```
	resilient-sdk codegen -p <package_name> --reload
	```
3. Build the app from the code the generator layed out.
	
	><sub>IBM KB Detailing Code: https://www.ibm.com/docs/en/rsoa-and-rp/42?topic=guide-write-function-processor</sub>
	
4. Ensure that the config/setup files have been edited, including: docker, setup, and any other file that might need changes.

	><sub>IBM KB Detailing Package Config Files: https://www.ibm.com/docs/en/rsoa-and-rp/42?topic=guide-package-your-app</sub>
	
5. Package the integration:
	```
	resilient-sdk package -p ./<package_name>/
	```
6. Install the integration in editable mode:
	```
	pip install -e ./<package_name>/
	```


### Build and Run the Docker Container for your App

1. Build the Docker Container
	```
	docker build ./<package_name> -t resilient/<package_name>:<version_number>
	```
2. Run the Docker Container (must have a app.config file build for this)
	```
	docker run -v <path_to_app.config>:/etc/rescircuits/app.config resilient/<package_name>:<version_number>
	```
3. Now go into Resilient and ensure the app is functioning properly!

	><sub>Note: If you make any changes to the code of your app, you will need to redo these instructions to rebuild the container</sub>


### Documenting Your App

More to come...


## Publishing and Using a Private Registry

<sub>Expectations/Requirements:</sub>

<sub>1. You already have a Private Registry:</sub>

<sub>- Github, Gitlab, or another public or private container registry.</sub>

<sub>2. You already have perferably 2 apphosts installed and configured to your Resilient server.</sub>
><sub>Note: One will be for IBM Published apps and the other will be for your custom apps.</sub>

### Connecting and Uploading to Registry from Docker

1. Log into the private registry:
	```
	docker login <registry_url> -u <username>
	```
2. Tag the container for upload:
	```
	docker tag resilient/<package_name>:<version_number> <registry_path>/ibmresilient/<package_name>:<version_number>
	```
3. Push the container to the registry:
	```
	docker push <registry_path>/ibmresilient/<packages_name>:<version_number>
	```


### Configuring the App and AppHost for Registry Use

1. Log into a AppHost system.	
2. Configure AppHost to use the private registries base URL (ex. ghcr.io)
	```
	sudo manageAppHost registry --registry <registry_url> --user <username>
	```
3. Restart the AppHost Box (I have found it easier to reboot vs restart services)
	
	><sub>This apphost will no longer allow IBM Published apps, that is why it is recommended to have 2 AppHost systems.</sub>

4. Alter the App JSON Files
	- Open in notepad the app.json file in the root directory of the packaged app zip file.
	- Under current_installation and executables, change the image value to the proper URI.
	
			ex. ibmresilient/fn_relations:1.0.1 -> <username>/ibmresilient/fn_relations:1.0.1


### Import the Private App into Resilient

1. Install the App within IBM SOAR
	- Navigate to the Apps page within the Administrator Settings.
	- From here, click the "Install" button
	- Provide the entire zip file of the packaged app, and click "Upload".
	- After completing the installation process, move on to the conifugration steps.
2. Configure the App by editing the app.config file on the "Configuration" tab.
3. At the bottom of the tab, choose the Private AppHost before running the "Test Configuration".
4. Once the test comes back successful, Click "Save and Push Changes".
5. On the "Details" tab, Deploy the app.
