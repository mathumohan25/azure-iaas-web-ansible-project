# azure-iaas-web-ansible-project
This project contains the ansible script to create Azure IaaS resources for python(flask-web)-mysql(db) application.

# Project Description

This Ansible project comprises Ansible roles and playbooks designed to deploy Azure resources according to the Azure resource diagram. Furthermore, it fetches code from a specified Git repository, which contains a Flask-MySQL application, and runs it on the server.

_Note: This project was created for an assignment in the course._

# Commands to Set Up a Linux Machine (Ubuntu) from Scratch [Brief of overall document]

1. **Setup Python Virtual Environment and Install Ansible and Azure Modules:**
   - Create a Python virtual environment and setup ansible:
     ```bash
     python3 -m venv venv-test && . venv-test/bin/activate && pip3 install --upgrade pip && pip3 install wheel
     pip3 install ansible==2.10.0
     ansible-galaxy collection install azure.azcollection
     pip3 install -r ~/.ansible/collections/ansible_collections/azure/azcollection/requirements-azure.txt
     ```
    
    Referred from [Link][https://github.com/ansible-collections/azure/issues/699]

2. **Install OpenVPN Client & Azure CLI:**
   - Install OpenVPN client:
     ```bash
     curl -fsSL https://swupdate.openvpn.net/repos/openvpn-repo-pkg-key.pub | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/openvpn-repo-pkg-keyring.gpg
     DISTRO=$(lsb_release -c | awk '{print $2}')
     sudo apt-get install openvpn -y
     sudo curl -fsSL https://swupdate.openvpn.net/community/openvpn3/repos/openvpn3-$DISTRO.list -o /etc/apt/sources.list.d/openvpn3.list
     sudo apt update
     sudo apt install openvpn3
     ```
     Referred from [Link][https://openvpn.net/cloud-docs/owner/connectors/connector-user-guides/openvpn-3-client-for-linux.html]

   - Install Azure CLI (for Ubuntu):
     ```bash
     curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
     ```

3. **Place Self-Signed/CA-Signed Certificate in the Azure Role Directory:**
   - Copy your self-signed or CA-signed certificate to your Azure role  directory _roles\az-vpn-p2s-connection\files_ .
    Referred from [Link][https://learn.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-certificates-point-to-site]

4. **Change the Variables (if Required):**
   - Update any necessary variables in your Ansible playbook, roles, or inventory files. Make sure on the variables are referred in other playbooks and roles. 

5. **Execute the Ansible Command to Setup VPN Gateway:**
   - Run the Ansible playbook:
     ```bash
     ansible-playbook 1-network-infra-setup.yml
     ```

6. **Run OpenVPN Client Command to Connect to the Created VPN from the Previous Step:**
   - Connect using OpenVPN:
     ```bash
     openvpn3 session-start --config output_files/OpenVPN/vpnconfig.ovpn
     ```

7. **Execute the Ansible Command to Setup VMs:**
   - Run the Ansible playbook:
     ```bash
     ansible-playbook 2-compute-infra-setup.yml
     ```

8. **Execute the Ansible Command to Deploy the Application:**
   - Run the Ansible playbook:
     ```bash
     ansible-playbook 3-web-db-app-setup.yml -i inventory/dynamic_inventory_azure_rm.yml --ssh-extra-args '-o StrictHostKeyChecking=no'
     ```
_For more details refer section Roles and playbooks_


# Prerequisites

Preparation Before Running Ansible Playbooks

1. **Run Python in a Virtual Environment and Install Ansible and Its Azure Modules:**
   To run this project, ensure that Linux machines have Ansible and Azure modules installed. To avoid issues with Azure Ansible modules, it is recommended to run Ansible within a Python virtual environment. You can find details about known issues [here][link]. Below are the commands to create a Python virtual environment and install Ansible and Azure modules.

   ```bash
     python3 -m venv venv-test && . venv-test/bin/activate && pip3 install --upgrade pip && pip3 install wheel
     pip3 install ansible==2.10.0
     ansible-galaxy collection install azure.azcollection
     pip3 install -r ~/.ansible/collections/ansible_collections/azure/azcollection/requirements-azure.txt
   ```

2. **Placing Azure Credentials File and Installing Azure CLI:**

   Since these playbooks create Azure resources, it is essential to have the Azure credentials file, which contains the service principal details, located at ~/.azure/credentials. These secrets are required by certain Azure modules that execute Azure CLI commands within the playbook. For information on creating and placing the credentials file, please refer to the [documentation][https://learn.microsoft.com/en-us/azure/developer/ansible/install-on-linux-vm?tabs=azure-cli].

3. **Creating a VPN Client Certificate:**

   To create a VPN client certificate, you will need a CA-signed or self-signed certificate. This certificate will be used in the P2S connection configuration of the VPN Gateway. The required files, P2SRootCert.cer and P2SClientCert.pfx, should be placed in the az-vpn-p2s-connection\files directory, as specified in the role az-vpn-p2s-connection. Instructions for creating a self-signed certificate using PowerShell can be found [here][https://learn.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-certificates-point-to-site#clientcert].

4. **OpenVPN 3 Client Setup:**

   The first playbook will set up the VPN, and the VMs created in the second playbook will be used for deploying the web-db application (roles from the third playbook). To establish a VPN P2S connection, you will need to install the OpenVPN client on the Ansible control machine. Please note that the installation steps for the OpenVPN client are not included in the playbook script.

   Installation steps for the OpenVPN client can be found [here][https://openvpn.net/cloud-docs/owner/connectors/connector-user-guides/openvpn-3-client-for-linux.html].

   Once installed, you can use the OpenVPN client command to connect to the created VPN.

[reference]: https://docs.microsoft.com/azure/developer/ansible/playbooks/setup-ansible#provide-azure-credentials
[self_signed_cert]: https://your-self-signed-cert-link
[openvpn_installation]: https://your-openvpn-install-link

# About Playbooks:

This section covers the details about the playbooks

1.  **Network Setup:**

    This playbook will set up the Azure VPN gateway and download a zip file containing VPN client files. These  files will be extracted and placed in the output_files directory. The contents of the generated OpenVPN configuration file [.ovpn] will be modified by Ansible itself. For this, it is required to place the  self-signed certificate files [P2SClientCert.pfx, P2SRootCert.cer] in the path  roles\az-vpn-p2s-connection\files, where this file is generated manually in Windows.


    ***Ansible roles used:***
       - az-resource-group
       - az-virtual-network
       - az-subnet
       - az-public-ip
       - az-virtual-network-gateway
       - az-vpn-p2s-connection

    ***Requirements:***
       - Azure Variables
       - Credentials file placed in the .azure/credentials file

    ***Output:***
       - Azure resources:
          - Resource group
          - Public IP associated with VPN Gateway
          - Vnet with gateway subnet associated with VPN Gateway
          - Azure VPN Gateway and P2S connection enabled
       - VPN client files placed in the output_files folder

````bash
   # command to run the playbook fo setting up Network
   ansible-playbook 1-network-infra-setup.yml
````

2.  **Setting Up Compute Machines and Load-Balancer:**

    This playbook will first create an SSH key, which will be stored in the .ssh folder. With this SSH key, it will set up three VMs for deploying the web-sql application (2 VMs dedicated to the web application, 1 VM dedicated to the DB application). The web machine has a separate subnet named "web-subnet," and the DB has a separate "db-subnet." These VMs are not associated with any public IP; they only have private IPs associated using NICs. For the further deployment process in these VMs (in step 3), it will make use of the VPN client file generated in Step 1.
 
    ***Ansible roles used:***
       - ssh_key_filename
       - az-resource-group
       - az-virtual-network
       - az-network-security-group
       - az-subnet
       - az-network-interface-card
       - az-virtual-machine
       - az-virtual-network-peer
       - az-public-ip
       - az-external-loadbalancer

    ***Requirements:***
       - Azure Variables
       - OpenVPN client file

    ***Output:***
       - SSH Key file generated in the .ssh directory
       - Azure resources:
          - Resource group for VMs and VM and its dependent resources (Vnet, Subnet (DB, web), Network security groups, Network Interface card)
          - Public IP associated with Azure LoadBalancer (In terminal output, note the  message "Please click this URL IP:80," where the deployed application can be accessed)
          - Vnet peered with VM's VNet
````bash
   # Command to run the playbook for setting up Compute Machines and Load-Balancer
   ansible-playbook 2-compute-infra-setup.yml
````

3.  **Web-DB Application Deployment:**

    This playbook is referred from the GitHub repo [link] for the assignment. Since the project refers to an older version of packages, a few changes have been added to the playbook. For the hosts input, the Azure dynamic inventory file is used, which returns the resources created in the 2nd playbook.

    ***Ansible roles used:***
       - Python3
       - mysql-db
       - web-falsk-app

    ***Requirements:***
       - Group_vars can be added for MySQL database, tables, and users.

    ***Output:***
       - MySQL DB deployed in the DB VM
       - Python-Flask application deployed in two web VMs

_Note: Dynamic inventory file name should suffixed with 'azure_rm'_

````bash
   # Run the playbook for Web-DB Application Deployment
   ansible-playbook 3-web-db-app-setup.yml -i inventory\dynamic_inventory_azure_rm.yml --ssh-extra-args '-o StrictHostKeyChecking=no'
````

# About Roles:

|No | Roles | Description | Variables required |
|---|-----------------|-----------------|-----------------|
|1| az-external-loadbalancer    | To create the standard SKU azure external loadbalancer which will be assoicated with public IP to access over the internet   | <ul><li>resource_group_name<li>region</li><li>public_ip_name</li><li>load_balancer_name</li><li>az_client_id</li><li>az_client_secret</li><li>az_tenant_id</li><li>subscription_id</li><li>target_resource_group_name</li><li>target_vnet_name</li><li>network_interfaces</li></ul>    |
|2| az-network-interface-card    | To create NIC to associate to the VMs which will be further used to deploy the application and add in load balancing rule(only web NICs)    | <ul><li>resource_group_name</li><li>network_interface_name</li><li>virtual_network_name</li><li>subnet_name</li><li>network_security_group_name</ul>    |
|3| az-network-security-group    | This NSG will have rules to allow inbound traffic to the VMs (80 in web dedicated VMs and 22 in all machines)    | <ul><li>resource_group_name</li><li>network_security_group_name</li><li>rule_name</li><li>rule_protocol</li><li>port_range</li><li>access_type</li><li>priority</li><li>direction</li></ul>    |
|4| az-public-ip  | To create azure public IP, which will have a random public IP address required in Azure Load balancer and  Azure VPN Gateway   | <li>resource_group_name</li><li>public_ip_name</li></ul>    |
|5| az-resource-group    | To Create the resource group, which will have the location of resources need to be created and contains respective resources.    | <ul><li>resource_group_name<li>region</li><ul>    |
|6| az-subnet    | To create the Azure subnet from the parent VNet CIDR    | <ul><li>resource_group_name</li><li>virtual_network_name</li><li>subnet_name</li><li>subnet_range</li></ul>    |
|7| az-virtual-machine    | To create the azure compute instances to deploy the application and its environment    | <ul><li>resource_group_name</li><li>nic_name</li><li>key_data</li><li>vm_name</li></ul>    |
|8| az-virtual-network    | To create the Azure VNet with the given CIDR, where the azure resorces like VMs and VPN Gateway will be associated    | <ul><li>resource_group_name</li><li>region</li><li>virtual_network_name</li><li>virtual_network_cidr</li></ul>    |
|9| az-virtual-network-gateway    | To create Azure VPN Gateway associated with VNet which will further peered with VNet of VMs to allow P2S connection    | <ul><li>resource_group_name</li><li>region</li><li>virtual_network_name</li><li>virtual_network_cidr</li><li>subnet_name</li><li>subnet_range</li><li>public_ip_name</li><li>vpn_gateway_name</li><li>az_client_id</li><li>az_client_secret</li><li>az_tenant_id</li></ul>   |
|10| az-virtual-network-peer    | To create azure VNet peering to allow in-out bound traffic between the VPN Gateway network to VMs network   | <ul><li>peering_name</li> <li>resource_group_name</li> <li>vnet_name</li> <li>remote_resource_group_name</li><li>remote_vnet_name</li> <li>use_remote_gateway</li> <li>allow_gateway_transit</li></ul>    |
|11| az-vpn-p2s-connection   | To configure P2S connection on Azure VPN Gateway with the VPN Certificate and place & modify the OpenVPN client configuration file to connect via openvpn.    | <ul><li>resource_group_name</li><li>region</li><li>virtual_network_name</li><li>virtual_network_cidr</li><li>subnet_name</li><li>subnet_range</li><li>public_ip_name</li><li>vpn_gateway_name</li><li>az_client_id</li><li>az_client_secret</li><li>az_tenant_id</li></ul>   |
|12| mysql-db    | Install MariaDB-server, pymysql and configure the MySQL server in the VM _(VM name should have the DB as per the condition in inventory\dynamic_inventory_azure_rm.yml)_    | _add the variables of Mysql user, password, database in the group_vars\all\all.yml_    |
|13| python3   | Dependent library for MySqlDB and Flask it should be installed in web & DB machines   | -    |
|14| ssh-key-generate    | Generate the SSH key file which will be used as authentication key for the VMs. (_This variables referred in other playbook as well as in the group_vars\all\all.yml_)  | <ul><li>ssh_key_filename</li><li>    ansible_user</li>    |
|15| web-falsk-app   | To run the web application , this flask is required and its dependent on python    | -    |

# Overall flow of the Azure resources and its dependency
![Local Image](https://github.com/mathumohan25/azure-iaas-web-ansible-project/blob/main/flow.png)

# Azure Resource Diagram
![Local Image](https://github.com/mathumohan25/azure-iaas-web-ansible-project/blob/main/res_diagram.png)

# Improvments can be done
- OpenVPN Connect is not included in the Ansible script because the passphrase is not prompted by the OpenVPN client.

- DNS configuration is not included in the script due to the Azure Load Balancer being at Level 4. SSL cannot be enabled in this use case.

# References
- Web-DB application: https://github.com/mmumshad/simple-webapp
- Reference project in GCP: https://github.com/mmumshad/udemy-ansible-assignment/tree/master
- OpenVPN client setup:https://openvpn.net/cloud-docs/owner/connectors/connector-user-guides/openvpn-3-client-for-linux.html
- Azure VPN Gateway Setup:https://learn.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal
- Azure Dynamic inventory:https://learn.microsoft.com/en-us/azure/developer/ansible/dynamic-inventory-configure?tabs=azure-cli









