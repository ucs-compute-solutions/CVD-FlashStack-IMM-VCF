[![published](https://static.production.devnetcloud.com/codeexchange/assets/images/devnet-published.svg)](https://developer.cisco.com/codeexchange/github/repo/ucs-compute-solutions/IMM-VCF-MgmtDomain)
# VMware Cloud Foundation (VCF) setup for Intersight Managed (IMM) Cisco UCS X-series blades as part of a Cisco FlashStack and rackmount servers
This repository contains Ansible playbooks for configuring Cisco Intersight managed X-Series servers, when used as part of a Cisco FlashStack, to operate as a VMware Cloud Foundation Workload Domain. In addition, these playbooks can configure a set of vSAN-ready Cisco Intersight managed C-series servers to operate as the VCF management domain. This repository is offered as an additional deployment option to use Infrastructure as Code (IaC) to automate a large portion of the installation process, as opposed to the manual installation process outlined in the accompanying Cisco Validated Design (CVD) document. These playbooks can be used to setup various pools, policies, Server Profile Template, and to perform initial configuration of the ESXi hosts for deploying VCF using Cloud Builder. To run these playbooks, Cisco UCS C-Series servers should be connected via Cisco UCS Fabric Interconnects and managed using IMM as shown in the figure, and the Cisco UCS X-series blades should be configured and available as part of a Cisco FlashStack installation.
<br><br>The Cisco Validated Design (CVD) deployment document which accompanies these playbooks is available here: https://www.cisco.com/c/en/us/td/docs/unified_computing/ucs/UCS_CVDs/flashstack_vcf.html

<img src="https://github.com/ucs-compute-solutions/CVD-FlashStack-IMM-VCF/blob/main/images/Overview.jpg">

# Cisco Intersight Managed Mode (IMM) and ESXi Configuration
The playbooks in this repository perform following high level functions:
## Cisco UCS
- Creates the various pools required to setup the Server Profile Templates
- Creates the various policies required to setup the Server Profile Templates
- Creates a VCF Mangement domain Server Profile Template, which boots from local disk and run vSAN
- Creates a VCF Workload domain Server Profile Template, which boots from SAN via the Pure Storage FlashArray, and uses FC based VMFS datastores
<br><br>After successfully executing UCS playbooks, you can use the server profile template to derive 4 server profiles for the management hosts, and 3 or more server profiles for the workload domain hosts, then install ESXi software on all of the servers. Afterwards, configure the management interfaces and hostnames of the ESXi hosts via the virtual KVM feature in Cisco Intersight.

__NOTE:__ The addition of UCS to Intersight Account or configuration of Domain Profile to setup UCS is not part of this repository and will have to be performed manually before executing the playbooks.

__NOTE:__ The playbooks do not create an organization and assume an organization (default or otherwise) has already been setup under Intersight account. The organization name must be updated in group_vars/all.yml(org_name) for successful execuation of the playbooks.
## ESXi Config
- Setup NTP server
- Set vSwitch0 MTU to 9000
- Set default port-group VLAN (management)
<br><br>After running the ESXi playbooks, log into each ESXi host using SSH and re-generate self signed certificates using the following commands:
```
/sbin/generate-certificates
/etc/init.d/hostd restart && /etc/init.d/vpxa restart
```
__NOTE:__ An Ansible playbook _regenerate_esxi_hosts_certs.yml_ has also been provided to re-generate the certificates. 
The ESXi hosts are now ready for VCF cloud builder configuration. 

# Execution Package Requirement
To execute various ansible playbooks, a linux based system will need to be setup with Ansible and following packages:
- https://galaxy.ansible.com/cisco/intersight?extIdCarryOver=true&sc_cid=701f2000001OH7YAAW
- https://github.com/ansible-collections/community.vmware 

# Intersight Access Requirement
To execute the playbooks against your Intersight account, you need to complete following additional steps of creating an API key and saving the Secret Key file: https://community.cisco.com/t5/data-center-and-cloud-documents/intersight-api-overview/ta-p/3651994

The <API_KEY_ID> and <SECRET_KEY_FILE> information is added to the group_vars/all.yml file. The SecretKey.txt file is typically copied to the same folder/directory where the Ansible Playbooks are cloned (alongside inventory file).

# Setting up Variables
All the variables used in this framework are defined in the following locations:
- Enter the hostnames for the ESXi hosts and their root passwords in the inventory file
- Variables that require customer inputs are part of group_vars/all.yml
- Variable that do not typically require customer input (e.g., descriptions of policies etc) are present under role_name/defauls/main.yml

# Playbook Execution
To execute the playbooks, you will need to follow these steps:
1. Physically connect the hardware and perform the initial configurations to setup UCS in IMM mode
2. Add Cisco UCS to Intersight account and create a domain profile so the servers get discovered and ports and port-channels are configured
3. Setup a Linux (or similar) host and install Ansible, git and required UCS and VMware packages
4. Clone the repository using git
5. Update the inventory file to provide the ESXi host information and passwords 
6. Add the Intersight API key to group_vars/all.yml and copy the SecretKey.txt to appropriate folder and update the location in group_vars/all.yml
7. Review and update the variables in group_vars/all.yml to match your environment
8. Execute the update_all_inventory.yml playbook to populate the inventory from Cisco Intersight, and to verify proper access and connectivity `ansible-playbook update_all_inventory.yml -i inventory`
9. Execute the create_pools.yml playbook to setup the various pools `ansible-playbook create_pools.yml -i inventory`
10. Execute the create_server_policies.yml playbook to setup the various policies `ansible-playbook create_server_policies.yml -i inventory`
11. Execute the create_server_profile_template.yml playbook to setup the server profile templates for VCF management and workload hosts `ansible-playbook create_server_profile_template.yml -i inventory`
12. From the two templates created, manually derive 4 Server Profiles to deploy the 4 VCF Mgmt Host servers and then derive 3 or more FC boot Server Profiles for the VCF Workload Domain hosts.
13. Create a bootable ESXi ISO image for ESXi 7.0 U3g, including the compatible Cisco nenic and nfnic drivers (instructions are in the associated CVD document)
14. Install ESXi on all the servers and configure the management interface to be able to access ESXi hosts
15. Execute the prepare_esxi_host.yml playbook to prep the ESXi hosts `ansible-playbook prepare_esxi_hosts.yml -i inventory`
16. Regenerate the self signed certificates on all the ESXi hosts manually or use the playbook: `ansible-playbook regenerate_esxi_hosts_certs.yml -i inventory`

At this time, ESXi servers will be ready for VCF cloud builder to setup the management domain. Afterwards, VMware SDDC can be used to commission the workload domain hosts and create the workload domain. All of these steps can be found in the associated CVD document listed at the top of this page.
