# PostgreSQL Installation and Configuration Playbook

This repository contains an Ansible playbook for installing and configuring **PostgreSQL 17.2** on **CentOS**-based systems. The playbook performs a series of tasks to set up PostgreSQL, configure a custom data directory, and allow access via a specific IP address.

## Key Features

1. **Directory Creation**: Creates the directory `/opt/pgsql/17` to store PostgreSQL data.
2. **Repository Configuration**: Adds the PostgreSQL repository for version **17.2**.  
   - **Note**: SSL certificate verification for the repository has been disabled due to a potential missing GPG key issue.
3. **Package Installation**: Installs PostgreSQL packages, including `postgresql17`, `postgresql17-server`, and `postgresql17-contrib`.
4. **Custom Data Directory**: Configures PostgreSQL to use a custom data directory (`/opt/pgsql/17/data`).
5. **Service Initialization**: Initializes the database with `--data-checksums` and sets up the PostgreSQL service for automatic startup.
6. **Firewall Configuration**: Configures the system firewall to allow PostgreSQL traffic and reloads the firewall to apply changes.
7. **Logging**: Fetches and displays the PostgreSQL logs from the last hour.

## Prerequisites

Ensure Ansible is installed on your control node. Follow the steps below to set it up if not already installed.

### Installing Ansible

For **CentOS/RHEL**-based systems, use the following commands to install Ansible:

```bash
sudo yum install epel-release -y
sudo yum install ansible -y
```

Verify the installation:
```bash
ansible --version
```

## Inventory configuration

Modify the `inventory.yml` file to match your environment. Replace the `ansible_host, ansible_ssh_user, and ansible_ssh_pass` values with the appropriate details for your target servers. Below is an example:

```yaml
all:
  children:
    webservers:
      hosts:
        AnsibleTarget1:
          ansible_host: <YOUR_TARGET1_IP>
          ansible_ssh_user: <YOUR_SSH_USER>
          ansible_ssh_pass: <YOUR_SSH_PASSWORD>
          ansible_become: yes
          ansible_become_method: sudo
          ansible_become_user: root
          ansible_become_pass: <YOUR_ROOT_PASSWORD>

        AnsibleTarget2:
          ansible_host: <YOUR_TARGET2_IP>
          ansible_ssh_user: <YOUR_SSH_USER>
          ansible_ssh_pass: <YOUR_SSH_PASSWORD>
          ansible_become: yes
          ansible_become_method: sudo
          ansible_become_user: root
          ansible_become_pass: <YOUR_ROOT_PASSWORD>
```

## Playbook Execution
By default, the playbook runs only on `AnsibleTarget1`. If you want to run the playbook on a different host or set of hosts, modify the target host(s) in the corresponding task. To change the target, update the `hosts` parameter in the following task:

```yaml
- name: Install PostgreSQL 17.2 and configure repository
  hosts: AnsibleTarget1  # Change this to your target host group or specific host
```

Additionally, in the task where the IP is added to **pg_hba.conf**, make sure to modify the IP address to reflect your controller machine's IP:

```yaml
- name: Add IP to pg_hba.conf
  lineinfile:
    path: /opt/pgsql/17/data/pg_hba.conf
    line: "host    all             all             192.168.0.195/32            scram-sha-256"  # Update this IP to your controller's IP
    insertafter: EOF
    create: true
```

To execute the playbook, use the following command:
```bash
ansible-playbook -i inventory.yml playbooks/install_postgresql.yml
```

##### Expected Behavior
If **PostgreSQL** is already installed and running, the playbook will ensure it is configured correctly without unnecessary changes.
If the **GPG key** for the **PostgreSQL** repository is not pre-installed, the playbook will bypass key verification during repository installation.

## Headers Overview
**1. Directory Creation**
This step ensures the custom directory /opt/pgsql/17 is created with the correct permissions.

**2. Repository Configuration**
Adds the PostgreSQL repository required for installing version 17.2. Note that SSL certificate verification is disabled to avoid issues with missing GPG keys.

**3. PostgreSQL Installation**
Installs PostgreSQL packages and prepares the environment with custom settings for the data directory.

**4. Service Management**
Initializes and starts the PostgreSQL service, enabling it to run at startup.

**5. Firewall Configuration**
Configures and reloads the firewall to allow PostgreSQL traffic through the specified ports.

**6. Logging and Debugging**
Displays the PostgreSQL logs from the last hour on the console for debugging purposes.

## Troubleshooting
**Repository Download Fails**
If the repository download fails with an SSL error, ensure that the system time is correct. You can synchronize the time using:
```bash
sudo timedatectl set-ntp true
```
If time is still not correct, you can turn off synchronization and set it manually to disable automatic time synchronization use:
```bash
sudo timedatectl set-ntp false
```

Then Set the date and time manually in the format "YYYY-MM-DD HH:MM"
```bash
sudo timedatectl set-time "2025-01-20 21:04"
```

Verify the current time and settings
```bash
timedatectl
```

**PostgreSQL Service Issues**
If PostgreSQL fails to start, check the service logs:
```bash
journalctl -u postgresql-17
```

For further assistance, please refer to the official PostgreSQL documentation or the Ansible documentation.
- [timedatectl Command Reference](https://man7.org/linux/man-pages/man1/timedatectl.1.html)
- [PostgreSQL Repository Setup](https://www.postgresql.org/download/linux/redhat/)
- [Ansible Documentation](https://docs.ansible.com/)  

## License
This project is licensed under the MIT License. For more details, see the [LICENSE](https://opensource.org/licenses/MIT) file.