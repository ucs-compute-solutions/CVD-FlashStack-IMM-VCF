[![published](https://static.production.devnetcloud.com/codeexchange/assets/images/devnet-published.svg)](https://developer.cisco.com/codeexchange/github/repo/ucs-compute-solutions/IMM-VCF-MgmtDomain)
# VMware Cloud Foundation management domain host setup for Intersight Managed (IMM) Cisco UCS rack servers
This repository contains Ansible playbooks for configuring Cisco Intersight managed C-Series (tested Cisco UCS C240 M5) servers. This repository aligns with the following design guide: https://www.cisco.com/c/en/us/td/docs/unified_computing/ucs/UCS_CVDs/flexpod_vcf_design.html. This repository can be used to setup various pools, policies, Server Profile Template, and to perform initial configuration of the ESXi hosts for deploying VCF using Cloud Builder. To run these playbooks, Cisco UCS C-Series servers should be connected via Cisco UCS Fabric Interconnects and managed using IMM as shown in the figure.
<img width="848" alt="IMM-Overview" src="https://user-images.githubusercontent.com/89957595/206804516-95da06c8-2be1-4739-b067-7392359896e6.png">

# Cisco Intersight Managed Mode (IMM) and ESXi Configuration
The playbooks in this repository perform following high level functions:
## Cisco UCS
- Create various pools required to setup a Server Profile Template
- Create various policies required to setup a Server Profile Template
- Create VCF Mangement Host Server Profile Templates
After successfully executing UCS playbooks, you can use the server profile template to derive 4 server profiles and install ESXi software on local disk.

__NOTE:__ The addition of UCS to Intersight Account or configuration of Domain Profile to setup UCS is not part of this repository and will have to be performed manually before executing the playbooks.

__NOTE:__ The playbooks do not create an organization and assume an organization (default or otherwise) has already been setup under Intersight account. The organization name must be updated in group_vars/all.yml(org_name) for successful execuation of the playbooks.
## ESXi Config
- Setup NTP server
- Setup hostname, domain and DNS server
- Enable SSH on the hosts
- Set vSwitch0 MTU to 9000
- Set default port-group VLAN (management)
After running the ESXi playbooks, log into each ESXi host using SSH and re-generate self signed certificates using the following commands:
```
/sbin/generate-certificates
/etc/init.d/hostd restart && /etc/init.d/vpxa restart
```
__Note__ An Ansible playbook _regenerate_esxi_hosts_certs.yml_ has also been provided to re-generate the certificates. 
The ESXi hosts are now ready for VCF cloud builder configuration. 

# Execution Package Requirement
To execute various ansible playbooks, a linux based system will need to be setup with Ansible and following packages:
- https://galaxy.ansible.com/cisco/intersight?extIdCarryOver=true&sc_cid=701f2000001OH7YAAW
- https://github.com/ansible-collections/community.vmware 

# Intersight Access Requirement
To execute the playbooks against your Intersight account, you need to complete following additional steps of creating an API key and saving the Secrets_File: https://community.cisco.com/t5/data-center-and-cloud-documents/intersight-api-overview/ta-p/3651994

The API key and Secrets_Filename information is added to the group_vars/all.yml. The Secrets_File value can be updated in all.yml. SecretKey.txt file is typically copied to the same folder/directory where Ansible Playbooks are cloned (alongside inventory file).

# Setting up Variables
All the variables used in this framework are defined in the following locations:
- Variable that require customer inputs are part of group_vars/all.yml
- Variable that do not typically require customer input (e.g., descriptions of policies etc) are present under role_name/defauls/main.yml.

# Playbook Execution
To execute the playbooks, you will need to follow these steps:
1. Physically connect the hardware and perform the initial configurations to setup UCS in IMM mode
2. Add Cisco UCS to Intersight account and create a domain profile so the servers get discovered and ports and port-channels are configured
3. Setup a Linux (or similar) host and install Ansible, git and required UCS and VMware packages
4. Clone the repository using git
5. Update the inventory file to provide the access information for ESXi host information. Add the Intersight API key to all.yml and copy the SecretKey.txt to appropriate folder and update the location in all.yml
6. Update variables in group_vars/all.yml to match your environment
7. Execute the create_pools.yml playbook to setup various pools `ansible-playbook ./create_pools.yml -i inventory`
8. Execute the create_server_policies.yml playbook to setup various policies `ansible-playbook ./create_server_policies.yml -i inventory`
9. Execute the create_server_profile_template.yml playbook to setup server profile template for VCF management host `ansible-playbook ./create_server_profile_template.yml -i inventory`
10. Manually derive 4 Server Profiles to deploy 4 VCF Mgmt Host servers
11. Install ESXi on the 4 servers and configure the management interface to be able to access ESXi hosts
12. Execute the prepare_esxi_host.yml playbook to prep the ESXi hosts `ansible-playbook ./prepare_esxi_hosts.yml -i inventory`
13. Regenerate the self signed certificates on all 4 ESXi hosts manually or use the playbook: `ansible-playbook ./regenerate_esxi_hosts_certs.yml -i inventory`
14. Update enic or fnic drivers if applicable (please consult the associated CVD)

At this time, ESXi servers will be ready for VCF cloud builder to setup the management domain. 
