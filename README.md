# **Documentation: Automated Deployment of Multi-VM Architecture**

This project automates the deployment of a multi-VM architecture using **Vagrant** and **Ansible**. The infrastructure includes an Active Directory server, a mail server, a Windows and a Linux client machines, all provisioned and configured automatically.

---

## **Directory Structure**

```bash
.
├── playbook-ansible-ad.yml         # Playbook for Active Directory configuration
├── playbook-ansible-client-linux.yml  # Playbook for Linux client configuration
├── playbook-ansible-mail.yml       # Playbook for mail server configuration
├── playbook-ansible-win10.yml      # Playbook for Windows 10 client configuration
├── templates/                      # Ansible templates for service configurations
│   ├── 10-auth.conf.j2
│   ├── 10-mail.conf.j2
│   ├── 10-master.conf.j2
│   ├── chrony.conf.j2
│   ├── dovecot-ldap.conf.ext.j2
│   ├── main.cf.j2
│   ├── passwd.j2
└── Vagrantfile                     # Main Vagrant configuration file
```

---

## **Project Goals**

The primary objectives of this project are:  

- Automating the deployment of an enterprise-like infrastructure.  
- Configuring services such as Active Directory, email, and client machines automatically.  
- Ensuring a seamless and reproducible deployment process with minimal manual intervention.

---

## **Architecture**

This project provisions the following virtual machines:  

1. **Active Directory Server**
   - Base image: Windows Server 2019 Standard
   - IP: `192.168.56.10`
   - Configured for domain services using Ansible.  

2. **Mail Server**
   - Base image: Ubuntu 22.04 LTS
   - IP: `192.168.56.20`
   - Configured with Postfix and Dovecot for email services.  

3. **Linux Client**
   - Base image: Ubuntu Desktop 22.04
   - IP: `192.168.56.30`
   - Configured for domain integration and basic client operations.  

4. **Windows 10 Client**
   - Base image: Windows 10 Professional
   - IP: `192.168.56.40`
   - Configured to integrate with Active Directory.  

---

## **Vagrant Configuration**

The `Vagrantfile` defines the infrastructure and automates provisioning:  

### **Global Settings**
- VMs are provisioned using VirtualBox.
- Default configurations include 2 CPUs, 2 GB RAM, and linked clones for efficiency.

### **Per-VM Configurations**
Each VM has custom configurations tailored to its role.  
- **Windows VMs**: Configured for WinRM communication with necessary firewall rules.  
- **Linux VMs**: Configured for SSH and Ansible management.  

---

## **Ansible Playbooks**

Ansible playbooks handle the configuration of each VM:  

1. **playbook-ansible-ad.yml**
   - Sets up Active Directory roles.  
   - Configures domain services.  

2. **playbook-ansible-mail.yml**
   - Installs and configures Postfix and Dovecot for mail services.  
   - Uses Jinja2 templates for configuration files.  

3. **playbook-ansible-client-linux.yml**
   - Configures a Linux desktop client.  
   - Integrates the client with Active Directory for domain authentication.  

4. **playbook-ansible-win10.yml**
   - Configures a Windows 10 client.  
   - Integrates with the Active Directory domain.  

---

## **Networking**

All VMs are connected to a private network:  

- Subnet: `192.168.56.0/24`
- Gateway: VirtualBox NAT
- DNS Server: Active Directory server (`192.168.56.10`)  

---

## **How to Use**

### Prerequisites
1. Install **Vagrant**, **VirtualBox**, and **Ansible** on your system.  
    ```bash
    sudo apt update && sudo apt install -y virtualbox ansible python3-pip
    wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
    echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
    sudo apt update && sudo apt install -y vagrant
    ```
2. Install the necessary vagrant plugins:  
   ```bash
   vagrant plugin install winrm winrm-elevated winrm-fs
   ```

### Deployment Steps
1. Start the Vagrant environment:  
   ```bash
   vagrant up
   ```
2. Wait for Vagrant to provision the VMs.  
3. Verify the deployment:  
   - Check Active Directory services on `192.168.56.10`.  
   - Access the mail server on `192.168.56.20`.  
   - Log in to the client machines using their respective IPs.

---

## **Customization**

Modify the `Vagrantfile` or Ansible playbooks to:  
- Add more VMs or services.  
- Adjust resource allocation (e.g., CPU, memory).  
- Customize configurations using the templates in the `templates` folder.

---

## **Troubleshooting**

1. **Provisioning Issues**
   - Ensure all required plugins for Vagrant and VirtualBox are installed.  
   - Verify that Ansible is correctly installed and configured.  

2. **Networking Issues**
   - Ensure your host's firewall allows VirtualBox NAT.  
   - Confirm IPs and DNS settings are correct.  

3. **Service Configuration**
   - Check logs for errors (`/var/log` on Linux, Event Viewer on Windows).  
   - Validate the Ansible playbook execution with verbose mode:  
     ```bash
     ansible-playbook -i inventory playbook-ansible-ad.yml -vv
     ```

---

## **Future Enhancements**

- Automate deployment of additional services such as web servers or databases.  
- Integrate monitoring tools like Prometheus and Grafana.  
- Expand to cloud environments using tools like Terraform.  

---

This documentation provides a comprehensive guide to deploying and managing the automated VM architecture. For further assistance, refer to the respective tool documentation. 
