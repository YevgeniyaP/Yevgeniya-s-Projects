## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

(Diagrams/Network Diagram.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the Network Diagram may be used to install only certain pieces of it, such as ELK Server and Filebeat.

  - Automated ELK Stack Deployment

This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology
The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly performing, in addition to restricting incoming external traffic to the network.
- Load balancers allow to distribute incoming traffic evenly between the local machines in the network. It protects from Denial of Service attack (DoS) by means that if one of the machines is not accessible, others will be still running. Using the Jumpbox machine (Jumpbox1 on the Network Diagram) playes a gateway role by routing all traffic through a single node. In that case we should be focused only on the connection between Jumpbox1 and Local Machine instead of connections between multiple machines. 

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the file and system logs. For that purpose we use Filebeat and Metricbeat which perform the following functions:
- Filebeat generates data about file system; it monitors log files and collects log events on the specified locations, and then transfers them to Kibana for visualising gathered data and providing analytics. The example of data presentation is shown on screenshot (Images/Kibana Filebeat Dashboard.png).
- Metricbeat collects data about machin metrics, for example CPU usage, memory, etc. And the same as Filebeat it sends gathered data to Kibana for data presentation and analytics.

The configuration details of each machine may be found below:

| Name       | Function                                                     | IP Address    | Operating  System |
|------------|--------------------------------------------------------------|---------------|-------------------|
| Jumpbox1   | Gateway;Provides access to other VMs on the Internal Network | 52.160.90.104 | Linux             |
| ELK-Server | Holds ELK docker container; provides public access to Kibana | 138.91.141.1  | Linux             |
| DVWA-VM1   | Connected to load balancer; provides access to DVWA website  | 10.0.0.10     | Linux             |

### Access Policies
The machines on the internal network are not exposed to the public Internet. 

Only the Jumpbox1 machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- 52.160.90.104

Machines within the network can only be accessed by SSH from Jumpbox1 using public key that we generated for all machines.

A summary of the access policies in place can be found in the table below.

| Name       | Publicly Accessible | Allowed IP Addresses    |
|------------|---------------------|-------------------------|
| Jumpbox1   | Yes                 | 52.160.90.104; 10.0.0.5 |
| ELK-Server | No                  | 138.91.141.1; 10.0.0.11 |
| DVWA-VM1   | No                  | 10.0.0.10               |


### Elk Configuration
Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because Ansible allows to deploy configurations on any environment by creating and 
running Ansible playbook. Ansible playbook contains the automation of tasks, which are written in a simple language. The machine, which has Ansible installed on it (in our case it is Jumpbox1 VM) is able to run configurations and provisioning on any server that it manages. 

The playbook implements the following tasks:
- install docker.io - that command installs docker engine which will allow to run containers
- install python-pip - package used to install Python software (this is required by ELK)
- increase virtual memory, which is required for better performance of ELK container
- download and launch docker container and configure container  to the following ports: 5601, 9200 and 5044. This will inform docker that ELK container listenes to specified ports at its runtime. For example, port 5601 allows access to web application Kibana (in other words, it will allow us to access Kibana web application from our ELK machine); ports 9200 and 5044 - allow access to Elasticsearch application. We will use both Kibana and Elasticsearch applications further in our work. 
 
The following screenshots display an example of install-elk.yml playbook and the result of running `docker ps` after successfully configuring the ELK instance: (Images/Install Elk.png) and (Images/Docker PS.png).

### Target Machines & Beats
This ELK server is configured to monitor DVWA-VM1 machine (IP 10.0.0.10). It could be configured similarly to monitor more machines from the network, yet on a current stage we are not able to add more machines to the internal network due to technical limitations of our Azur working environment. Therefore, all examples will be provided for one machine. The configuration of ELK server will allow us to install Beats necessary for data collection from our machine DVWA-VM1.
First, we have installed the following Beats on DVWA-VM1 machine:
- Filebeat and Metricbeat.
Though, ELK support eight Beats configured to collect specific data from different places: Filebeat, Metricbeat, Winlogbeat, Packetbeat, Functionbeat, Auditbeat, Hearbeat and Journalbeat. 

For the purpose of our demo we installed two Beats. These Beats allow us to collect the following information:
- Filebeat - this Beat is used for collecting log files, which are the evidence of what has been done on the machine: eg. successfully loaded files; attempted or successful connections, etc. For the example of syslog logs refer to the screenshot (Images/Filebeat Syslog.png)
- Metricbeat - it collects and presents various system-level metrics, for example CPU usage, memory usage, etc. For the example refer to the screenshot (Images/Kibana Metricbeat Dashboard.png).

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the  filebeat-configuration.yml (configuration file fouind in (Files/Filebeat-Configuration.pages)) to Ansible control node (in our example, Jumpbox1 holds the Ansible control node). From Ansible we will have to copy this config file into our DVWA-VM1 machine. In order to do that, we will update "Drop in filebeat.yml" command in filebeat-playbook.yml as shown in the screenshot (Images/Filebeat-Playbook.png), where command "dest" indicates on destination on DVWA-VM1 machine. 
- Update the above configuration file to include IP of ELK server, because we will be running Filebeat playbook on ELK machine. To do that, navigate to (Files/Filebeat-Configuration.pages), scroll to line #1106 and change the IP address into ELK-Server's IP (in our case we have changed it for 10.0.0.11). Then, scroll down to line #1806 and replace the IP address with the IP address of your ELK machine (10.0.0.11 in our example).
- Run the playbook (the example of filebeat-playbook.yml is provided in (Images/Filebeat-Playbook.png). In order to specify on which machine we are installing ELK server and on which - Filebeat, we need to update "hosts" command in the playbook. Since we have created playbook to install Filebeat, then hosts will be "webservers" (in our example, webservers is DVWA_VM1 machine). For installing ELK, hosts will be "elkservers". 
Then after running playbook navigate to DVWA-VM1 machine (ssh to 10.0.0.10) to check that the installation worked as expected.
For that, on your DVWA-VM1 machine run the set of following commands:
  ./filebeat modules enable system;
  ./filebeat setup
  ./filebeat -e.
  Those commands will allow us to Install and run filebeat on destination machine - DVWA-VM1. 
  In order to check that the installation works and filebeat runs, navigate to Kibana through http://138.91.141.1:5601 (where 138.91.141.1 - is a Public IP for ELK machine). From Kibana home page navigate to Add Data - System Logs - Module Status - Check Data. If your configuration is correct you should get the same results as on the following screenshots: (Images/Kibana-Filebeat.png) and (Images/Kibana Filebeat Dashboard.png). The same result should be obtained after running Metricbeats: (Images/Kibana-Metricbeat.png) and (Images/Kibana Metricbeat Dashboard.png).
*The above instructions are similar both for Filebeat and Metricbeat configurations.
