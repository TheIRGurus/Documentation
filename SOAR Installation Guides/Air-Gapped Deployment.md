# Installing and Configuring SOAR for Air-Gapped Deployment
<sub>Expectations/Requirements: SOAR and AppHost are already deployed in the air-gapped environment.</sub>

This guide is to be used for deploying IBM Security SOAR in an air-gapped environment. This will include building a app registry, deploying and re-configuring a AppHost server, and connecting the SOAR System to your AppHost.

**Table of Contents**

  * [Creating your own Private Registry](#creating-your-own-private-registry)
    + [Installing and Setting up Docker](#installing-and-setting-up-docker)
    + [Establishing the Required Security Files for the Registry](#establish-the-required-security-files-for-the-registry)
    + [Building, Mirroring, and Deploying the Container Registry](#building-mirroring-and-deploying-the-container-registry)
  * [Configuring AppHost for Air-Gapped Environment](#configuring-apphost-for-air-gapped-environment)
    + [Configuring Kubernetes to Use the Private Registry](#configuring-kubernetes-to-use-the-private-registry)
    + [Establishing the AppHost Pairing](#establishing-the-apphost-pairing)

## Creating your own Private Registry
<sub>Expectations/Requirements: The commands below are if you are installing on a CentOS or RHEL box.</sub>

### Installing and Setting up Docker

1. Install docker and docker-compose
  ```bash
  sudo yum install -y docker docker-compose
  ```
2. Enable Docker service on boot. 
  ```bash
  sudo systemctl enable docker
  ```
3. Start docker service 
  ```bash
  sudo systemctl start docker
  ```

---

### Establish the Required Security Files for the Registry
1. Create a registry file structure for your docker registry environment. Below are the commands to create the path I will be using.
  ```bash
  mkdir -p appdata/registry/auth
  mkdir -p appdata/registry/certs
  mkdir -p appdata/registry/data
  ```
  - Note: If you are using a system with SELinux running, you might need to use the following command to add context to the files to allow Docker to interact.
    ```bash
    chcon -Rt svirt_sandbox_file_t appdata/registry/
    ```

2. Change your operating directory to the root registry folder created above.
  ```bash
  cd appdata/registry
  ```

3. Create the TLS Certs and Keys using OpenSSL.
  ```bash
  openssl genrsa -out certs/ca.key 2048
  openssl req -new -x509 -days 365 -key certs/ca.key -subj "/C={Country_Code}/ST={State_Code}/L={Location}/O={Organization}/CN={Company_Name} Root CA" -out certs/ca.crt

  openssl req -newkey rsa:2048 -nodes -keyout certs/registry.key -subj "/C={Country_Code}/ST={State_Code}/L={Location}/O={Organization}/CN=registry.{company.domain}" -out certs/registry.csr
  openssl x509 -req -extfile <(printf "subjectAltName=DNS:registry.{company.domain}") -days 365 -in certs/registry.csr -CA certs/ca.crt -CAkey certs/ca.key -CAcreateserial -out certs/registry.crt
  ```

4. We will now need a copy of the `ca.crt` file in the docker certs folder. Use the command below to make a copy for docker.
  ```bash
  sudo mkdir /etc/docker/certs.d/localhost
  sudo cp certs/ca.crt /etc/docker/certs.d/localhost/
  sudo update-ca-trust
  ```

5. Next, establish a credential file for basic auth into the registry.
  ```bash
  sudo docker run --entrypoint htpasswd httpd:2 -Bbn {username} {password} > auth/htpasswd
  ```

6. Create the `docker-compose.yml` file to allow you to quickly start and stop the registry.
  ```bash
  vi docker-compose.yml
  ```

7. Using the YAML code below, update the volumes section with your full paths to finish the file.
  ```yaml
  version: "3"

  services:
    registry:
      image: registry:latest
      container_name: registry
      ports:
        - 443:443
      environment:
        REGISTRY_HTTP_ADDR: 0.0.0.0:443
        REGISTRY_HTTP_TLS_CERTIFICATE: certs/registry.crt
        REGISTRY_HTTP_TLS_KEY: certs/registry.key
        REGISTRY_AUTH: htpasswd
        REGISTRY_AUTH_HTPASSWD_PATH: auth/htpasswd
        REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
      tty: true
      stdin_open: true
      restart: always
      volumes:
        - {starting_path}/appdata/registry/data:/var/lib/registry
        - {starting_path}/appdata/registry/certs:/certs
        - {starting_path}/appdata/registry/auth:/auth
  ```

  >Note: This will add basic security like Auth and TLS. If you would like to add any additional security, please consult the container documentation: https://docs.docker.com/registry/

---

### Building, Mirroring, and Deploying the Container Registry
1. Stand-up the docker container using the yaml file.
  ```bash
  sudo docker-compose up -d
  ```
  - If you need to bring the container down just run the down command.
    ```bash
    sudo docker-compose down
    ```

2. Install jq for the script we will be using later.
  ```bash
  sudo yum install -y jq
  ```

3. Copy over the `mirror-all-images.sh` script found at the link below.
  >https://raw.githubusercontent.com/ibmresilient/resilient-community-apps/master/.scripts/mirror-containers/mirror-all-images.sh
  ```bash
  curl -X GET https://raw.githubusercontent.com/ibmresilient/resilient-community-apps/master/.scripts/mirror-containers/mirror-all-images.sh > mirror-all-images.sh
  ```

  - IBM Security SOAR KB Article: https://www.ibm.com/docs/en/sqsp/49?topic=repository-mirroring-quayio

4. Log into your registry with Docker using the credentials created above.
  ```bash
  sudo docker login localhost --username <username>
  ```

5. Run the Mirror script in bash to mirror your registry with the IBM registry. 
  ```bash
  sudo bash mirror-all-images.sh localhost docker
  ```
  - Note: While you can add `latest_tag` to the end of the above command to only grab the latest container of each app, for an air-gapped environment you will need to run the full mirror script to get the required versions of certain apps to run in the air-gapped environment.

6. Lastely, We will also need to grab the Rancher images for our registry.
  ```bash
  sudo docker pull rancher/mirrored-metrics-server:v0.6.2
  sudo docker pull rancher/local-path-provisioner:v0.0.23
  sudo docker pull rancher/mirrored-coredns-coredns:1.9.4

  sudo docker tag rancher/mirrored-metrics-server:v0.6.2 localhost/rancher/mirrored-metrics-server:v0.6.2
  sudo docker tag rancher/local-path-provisioner:v0.0.23 localhost/rancher/local-path-provisioner:v0.0.23
  sudo docker tag rancher/mirrored-coredns-coredns:1.9.4 localhost/rancher/mirrored-coredns-coredns:1.9.4

  sudo docker push localhost/rancher/mirrored-metrics-server:v0.6.2
  sudo docker push localhost/rancher/local-path-provisioner:v0.0.23
  sudo docker push localhost/rancher/mirrored-coredns-coredns:1.9.4
  ```

7. Once the script has finished running, you can verify that the transfer happened and is in your registry by doing the following command to see all of the available apps.
  ```bash
  curl -X GET https://localhost/v2/_catalog -u "<username>:<password>" --insecure
  ```
  >Note: The `--insecure` flag is just to not verify the TLS cert which is self-signed.

8. At this point, you can move your VM to the air-gapped environment and set a static IP.

---

## Configuring AppHost for Air-Gapped Environment
<sub>Using the following guide with some improvements: https://www.ibm.com/docs/en/sqsp/49?topic=host-virtual-appliance-in-air-gap-environment</sub>

### Configuring Kubernetes to Use the Private Registry

1. Download Rancher k3s air-gapped tar images. 
    - For version 49, you must use v1.23.6+k3s1: https://github.com/k3s-io/k3s/releases/tag/v1.23.6%2Bk3s1

2. Once the tar file is on the AppHost, make the correct directory and move it to the rancher images location using the commands below.
  ```bash
  sudo mkdir -p /var/lib/rancher/k3s/agent/images/
  sudo mv <k3s-airgap-images tar file> /var/lib/rancher/k3s/agent/images/
  ```

3. Move the `registry.crt`, `registry.key`, and `ca.crt` over to the AppHost server. Then create the registry folder and move the registry cert and key into the structure.
  ```bash
  mkdir -p .registry/certs
  mv registry.* .registry/certs/
  ```

4. Make a copy of the cert into the trusted certificates.
  ```bash
  sudo mv ca.crt /etc/pki/ca-trust/source/anchors/
  sudo update-ca-trust extract
  ```

5. Open the `/etc/hosts` file to add a DNS entry.
  ```bash
  sudo vi /etc/hosts
  ```

6. Using the template below, add the DNS entry for your registry server:
  ```
  <Registry_IP> registry.{company.domain}
  ```

7. Create the `registries.yaml` directory and file using the following commands
  ```bash
  sudo mkdir -p /etc/rancher/k3s/
  sudo vi /etc/rancher/k3s/registries.yaml
  ```

8. Add the following code to the registry file, changing your domain and credentials.
  ```yaml
  mirrors:
    docker.io:
      endpoint:
        - "registry.{company.domain}"
    quay.io:
      endpoint:
        - "registry.{company.domain}"
  configs:
    "registry.{company.domain}":
      auth:
        username: <username>
        password: <username>
      tls:
        cert_file: /home/appadmin/.registry/certs/registry.crt
        key_file: /home/appadmin/.registry/certs/registry.key
  ```

9. Determine which interface you are using to establish a default path to the network the registry and apphost are on. Then using the interface name and the following path, `/etc/sysconfig/network-scripts/`, create a route file to house your default route.
  >Example: if the interface is `eth0`, you will create a the file `route-eth0` using the command below.
  ```bash
  sudo vi /etc/sysconfig/network-scripts/route-eth0
  ```

10. Add the following line to the file above using the IP schema and interface used.
  ```bash
  <IP Network> via <Network Gateway> dev <interface>
  ```
  >Example: `172.16.160.1/24 via 172.16.160.1 dev eth0`

11. Restart the network service
  ```bash
  sudo systemctl restart NetworkManager
  ```

12. Restart the K3s service for Kubernetes.
  ```bash
  sudo systemctl restart k3s
  ```

13. Verify that all of your pods are running.
  ```bash
  sudo kubectl get pods -A
  ```
  - if any of the pods are errored redeploy them using the command below:
      ```bash
      sudo kubectl rollout restart deployments -n kube-system
      ```

---

### Establishing the AppHost Pairing
1. Now proceed to follow the steps in the KB to create a Pairing for the AppHost.
  >https://www.ibm.com/docs/en/sqsp/49?topic=installation-create-pairing

2. Then using that pairing info you created, establish the link between SOAR and the AppHost using the command below:
  ```bash
  sudo manageAppHost install
  ```
  >Note: You can just paste the pairing directly into your putty session when prompted to.

3. Configure the AppHost to also use the registry.
  ```bash
  sudo manageAppHost registry --registry <host name/ip>[:port] --user <username>
  ```

4. Set your DNS within the AppHost.
  ```bash
  sudo manageAppHost dns --set --ip=<registry_ip> --hostname=registry.{company.domain}
  ```

5. Finally verify that the Pairing has taken place and the AppHost shows in a <span style="color:green">Running</span> state within the admin settings of SOAR.

6. Now you can deploy your apps just like you would in a non-air-gapped environment, with the caveat that you can't use anything that communicated outside of your air-gapped network.
