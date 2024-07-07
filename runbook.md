### SecureCertAppDeploy Runbook

#### Contents

1. [Objectives](#objectives)
2. [Requirements](#requirements)
   - [Standard CA Certificates](#standard-ca-certificates)
   - [Custom CA Certificates](#custom-ca-certificates)
   - [Application Deployment](#application-deployment)
   - [Security](#security)
3. [Deliverables](#deliverables)
4. [Appendices](#appendices)
   - [CA Certificates](#ca-certificates)
   - [Example Application](#example-application)
   - [Run Script](#run-script)
   - [Configuration File](#configuration-file)
   - [Variables](#variables)

---

### 1. Objectives

The primary aim of this exercise is to assess your knowledge and familiarity with Ansible. To do so, you should produce an Ansible playbook that will do the following:
1. Configure standard Certificate Authority (CA) certificates on the target hosts.
2. Install custom CA certificates in each target’s trust store.
3. Deploy a Python application into a virtual environment.

---

### 2. Requirements

After the execution of the playbook, the following conditions should be met.

#### 2.1 Standard CA Certificates

The target hosts should trust any certificate signed by the standard set of CAs on GNU/Linux.

#### 2.2 Custom CA Certificates

The target hosts should trust any certificate signed by a CA in the list of custom CA certificates supplied to the playbook.

##### 2.2.1 Validity

Each custom CA certificate should be checked for validity (i.e. that it has not expired).

##### 2.2.2 Files

You should think carefully about which files need to be deployed to the target hosts as not every file is required.

#### 2.3 Application Deployment

A Python Flask application is provided in the form of a wheel file. This application needs to be deployed and run on the target hosts.

##### 2.3.1 Location

The application should be deployed to `/opt/example`.

##### 2.3.2 Virtual Environment

A virtual environment should be created in the deployment location and the application should be deployed using that virtual environment.

##### 2.3.3 Application Configuration

Inside the deployment folder for the application, you should create a folder called `instance` (e.g. in this example the instance folder will be located at `/opt/example/instance`). This folder should contain the configuration files for the application, which are specified in the appendix.

##### 2.3.4 Variables

The following items should be parameterized using variables. The actual values of the variables that should be deployed are given in Appendix B:
- The port on which the application will run.
- The location of the deployment.
- The name of the wheel file to deploy.
- The instance path.
- Configuration variables:
  - `SECRET_KEY`
  - `DB_PATH`
  - `ADMIN_GROUPS`

##### 2.3.5 Execution

The application is run using Gunicorn via a script called `run.sh`. You need to deploy this script into the deployment folder and parameterize the variables within it using Ansible.

You should then create a system service file called `example.service` to execute this script. Bear in mind that when `run.sh` is executed, it must have the `INSTANCE_PATH` environment variable set to the instance path of the application (`/opt/example/instance` in this case).

##### 2.3.6 Expected File Structure

The file structure of the application deployment location should end up looking like this:

```
/opt/example
├── instance
│   └── config.py
├── venv
├── run.sh
```

#### 2.4 Security

When developing this Ansible project, you must be security conscious. Bear in mind what information you are including in the repository and publishing for all to see. You will not be invited to interview if critical security vulnerabilities are found in your project.

---

### 3. Deliverables

There is only one deliverable for this exercise: the Ansible project in a zip file. As an absolute minimum, the zip file should contain:
- The playbook used to meet the requirements in section 2.
- The files to be deployed to the target host(s)
- The `.git` folder containing the commits made while developing the project.

Bear in mind that the Ansible project should almost certainly contain more than just a single playbook, so it is expected that the zip file be larger than just the items mentioned above.

---

### 4. Appendices

#### Appendix A.1: CA Certificates

**CA1.crt/CA1.key**
The first custom CA certificate that needs to be deployed in requirement 2.2.

**CA2.crt/CA2.key**
The second custom CA certificate that needs to be deployed in requirement 2.2.

**CA3.crt/CA3.key**
The third custom CA certificate that needs to be deployed in requirement 2.2.

#### Appendix A.2: Example Application

**Example-1.1.2-py3-none-any.whl**
This is the example application that needs to be deployed.

#### Appendix A.3: Run Script

**run.sh**
This file will need to be deployed into the deployment folder. You can then run the Flask application in the virtual environment by executing: `/opt/example/run.sh`. Remember that this file needs to be executed with the `INSTANCE_PATH` environment variable set to the correct instance path.

#### Appendix A.4: Configuration File

**config.py**
This is the application configuration file. It needs to be deployed into the instance folder and the variables inside it should be parameterized by Ansible.

#### Appendix B: Variables

Below are the default values for the Ansible variables. You should give them sensible names and store them in a sensible location.

##### Appendix B.1: Port

**Default Value:** 5000

##### Appendix B.2: Deployment Folder

**Default Value:** /opt/example

##### Appendix B.3: Wheel File

**Default Value:** Example-1.1.2-py3-none-any.whl

##### Appendix B.4: Instance Path

**Default Value:** /opt/example/instance

##### Appendix B.5: Secret Key

**Default Value:** AVerySafeExampleKey123!

##### Appendix B.6: DB Path

**Default Value:** postgresql://example_user:myVerySecretDatabasePassword@localhost:5432/example_db

##### Appendix B.7: Admin Groups

**Default Values:**
- Read-only
- Read-write
- Super

---

### Project File Structure

```SecureCertAppDeploy
.
├── README.md
├── ansible.cfg
├── files
│   ├── config.py
│   ├── example.service
│   └── run.sh
├── group_vars
│   └── all.yml
├── host_vars
│   ├── localhost.yml
│   └── target1.yml
├── inventory
│   └── hosts.ini
├── playbooks
│   └── main.yml
├── roles
│   ├── ca_certificates
│   │   ├── defaults
│   │   ├── files
│   │   └── tasks
│   │       └── main.yml
│   └── python_app
│       ├── defaults
│       │   └── main.yml
│       ├── files
│       │   ├── Example-1.1.2-py3-none-any.whl
│       │   └── app.py
│       ├── tasks
│       │   └── main.yml
│       ├── templates
│       │   ├── config.py.j2
│       │   ├── example.service.j2
│       │   └── run.sh.j2
│       └── vars
│           └── main.yml
├── runbook.md
├── vault
│   ├── ca_certificates
│   │   ├── CA1.crt
│   │   ├── CA1.key
│   │   ├── CA2.crt
│   │   ├── CA2.key
│   │   ├── CA3.crt
│   │   └── CA3.key
│   └── secrets.yml
└── vault_password_file.txt
```

---

### Ansible Configuration (ansible.cfg)

```ini
[defaults]
inventory = inventory/hosts.ini
remote_user = ec2-user
host_key_checking = False
private_key_file = ~/.ssh/your_private_key 
retry_files_enabled = False
stdout_callback = yaml
timeout = 30

[privilege_escalation]
become = True
become_method = sudo
become_user = root

[ssh_connection]
pipelining = True
ssh_args = -o ControlMaster=auto -o ControlPersist=60s

[defaults]
vault_password_file = vault_password_file.txt

[log_path]
log_path = ansible.log

[galaxy]
server_list = release_galaxy
```

### Example Vault Password File (vault_password_file.txt)

```
movvamanoj
```

### Example Inventory File (inventory/hosts.ini)

```ini
[targets]
localhost
#target1 ansible_host=192.168.0.1 ansible_user=your_user ansible_ssh_private_key_file=/path/to/private_key
```

### Example Host Variables (host_vars/localhost.yml)

```yaml
ansible_connection: local
```

### Group Variables (group_vars/all.yml) (Optional)

```yaml
---
app_ip: 127.0.0.1
app_port: 5000
db_path: "{{ lookup('vault', 'secret/ansible/db_path') }}"
secret_key: "{{ lookup('vault', 'secret/ansible/secret_key') }}"
admin_groups: "{{ lookup('vault', 'secret/ansible/admin_groups') }}"
```

### Python App Role Variables (roles/python_app/vars/main.yml)

```yaml
# Python application settings
python_app_flask_package: flask
python_app_gunicorn_package: gunicorn
python_app_wheel_name: Example-1.1.1-py3-none-any.whl
python_app_wheel_path: "{{ python_app_wheel_name }}"


# Packages to install
python_packages:
  - python3
  - python3-pip

python_app_packages:
  - flask
  - gunicorn

# Flask application environment variables
flask_app_name: example
flask_env: production
gunicorn_workers: 2
gunicorn_threads: 2

# Paths and directories
python_app_deployment_folder: /opt/example
python_app_instance_folder: "{{ python_app_deployment_folder }}/instance"
python_app_venv_path: "{{ python_app_deployment_folder }}/venv"

# Configurations
python_app_config_template: config.py.j2
python_app_run_script_template: run.sh.j2
python_app_service_template: example.service.j2
python_app_service_name: example

# Application-specific variables
app_ip: 0.0.0.0 #127.0.0.1  #This ensures Gunicorn listens on all interfaces (0.0.0.0).
app_port: 5000

# System and environment variables
deploy_path: "{{ python_app_deployment_folder }}"
instance_path: "{{ python_app_instance_folder }}"
service_name: "{{ python_app_service_name }}"
user: root

# Secrets and sensitive information (to be retrieved from Ansible Vault)
db_path: "{{ lookup('vault', 'secret/ansible/db_path') }}"
secret_key: "{{ lookup('vault', 'secret/ansible/secret_key') }}"
admin_groups: "{{ lookup('vault', 'secret/ansible/admin_groups') }}"
```

### Example Secrets File (vault/secrets.yml)

```yaml
---
vault:
  secret_key: "AVerySafeExampleKey123!"
  db_path: "postgresql://example_user:myVerySecretDatabasePassword@localhost:5432/example_db"
  admin_groups:
    - Read-only
    - Read-write
    - Super
```

### Playbook (playbooks/main.yml)

```yaml
---
- name: Deploy Secure Certificate Application
  hosts: all
  become: true

  roles:
    - role: ca_certificates
    - role: python_app
```


### Python App Role Variables (roles/python_app/vars/main.yml)

```yaml
# vault_ca_certificate_files: "{{ lookup('fileglob', 'SecureCertAppDeploy/vault/ca_certificates/*.crt') }}"
# vault_ca_private_key_files: "{{ lookup('fileglob', 'SecureCertAppDeploy/vault/ca_certificates/*.key') }}"
vault_ca_certificate_files: "{{ lookup('fileglob', 'vault/*.crt') }}"
vault_ca_private_key_files: "{{ lookup('fileglob', 'vault/*.key') }}"

```
### CA Certificates Role Task File (roles/ca_certificates/tasks/main.yml)

```yaml
---
- name: Install standard CA certificates
  package:
    name: ca-certificates
    state: present

- name: Install custom CA certificates
  copy:
    src: "{{ item }}"
    dest: /usr/local/share/ca-certificates/
    mode: 0644
    decrypt: yes
  with_fileglob:
    - "{{ vault_ca_certificate_files }}"
  
- name: Install custom CA private keys
  copy:
    src: "{{ item }}"
    dest: /usr/local/share/ca-certificates/
    mode: 0600
    decrypt: yes
  with_fileglob:
    - "{{ vault_ca_private_key_files }}"
 
- name: Update CA certificates
  command: update-ca-certificates
  become: true

```

### Python App Role Task File (roles/python_app/tasks/main.yml)

```yaml
---
- name: Ensure Python 3 and pip are installed
  become: true
  apt:
    name: "{{ item }}"
    state: present
  with_items: "{{ python_packages }}"


- name: Ensure Python 3 venv is installed
  become: true
  apt:
    name: python3-venv
    state: present

- name: Create deployment directory
  file:
    path: "{{ python_app_deployment_folder }}"
    state: directory
    mode: 0755

- name: Create virtual environment
  command: python3 -m venv "{{ python_app_venv_path }}"
  args:
    chdir: "{{ python_app_deployment_folder }}"

- name: Install Flask and Gunicorn in virtual environment
  pip:
    name: "{{ item }}"
    virtualenv: "{{ python_app_venv_path }}"
    state: present
  with_items: "{{ python_app_packages }}"

- name: Copy application wheel file
  copy:
    src: "{{ python_app_wheel_path }}"
    dest: "{{ python_app_deployment_folder }}"
    mode: 0644

- name: Install application dependencies in virtual environment
  pip:
    name: "{{ python_app_deployment_folder }}/{{ python_app_wheel_name }}"
    virtualenv: "{{ python_app_venv_path }}"
    state: present

- name: Create instance folder for application configurations
  file:
    path: "{{ python_app_instance_folder }}"
    state: directory
    mode: 0755

- name: Deploy application configuration file
  template:
    src: "{{ python_app_config_template }}"
    dest: "{{ python_app_instance_folder }}/config.py"
    mode: 0644  

- name: Deploy run.sh script
  template:
    src: "{{ python_app_run_script_template }}"
    dest: "{{ python_app_deployment_folder }}/run.sh"
    mode: 0755

- name: Create systemd service file for the application
  template:
    src: "{{ python_app_service_template }}"
    dest: "/etc/systemd/system/{{ python_app_service_name }}.service"
    mode: 0644
  notify: 
    - Start and enable the application service

- name: Start and enable the application service
  systemd:
    name: "{{ python_app_service_name }}"
    enabled: yes
    state: started

- name: Reload systemd for changes to take effect
  systemd:
    daemon_reload: yes

```

### Templates (roles/python_app/templates)

#### config.py.j2

```python
SECRET_KEY = "{{ secret_key }}"
DB_PATH = "{{ db_path }}"
ADMIN_GROUPS = {{ admin_groups }}
```

#### example.service.j2

```ini
[Unit]
Description=Example Flask Application
After=network.target

[Service]
User={{ user }}
WorkingDirectory={{ deploy_path }}
ExecStart={{ deploy_path }}/run.sh
Environment="INSTANCE_PATH={{ instance_path }}"
Environment=PATH=/bin:/usr/bin:/sbin:/usr/sbin:{{ deploy_path }}/venv/bin
Restart=always

[Install]
WantedBy=multi-user.target

```

#### run.sh.j2

```bash
#!/bin/bash

source {{ deploy_path }}/venv/bin/activate

# Set environment variables
export FLASK_APP={{ flask_app_name }}
export FLASK_ENV={{ flask_env }}
export DB_PATH={{ db_path }}
export SECRET_KEY={{ secret_key }}
export ADMIN_GROUPS="{{ admin_groups }}"
export INSTANCE_PATH="{{ instance_path }}"

# Start the Flask application with Gunicorn
exec gunicorn -b {{ app_ip }}:{{ app_port }} --workers {{ gunicorn_workers }} --threads {{ gunicorn_threads }} app:app

```

### Git (.gitignore)

```.gitignore
# Ignore Ansible Vault password file # chmod 600 vault_password_file.txt # This give access to current user

vault_password_file.txt

# # Ignore sensitive vault files
vault/secrets.yml
vault/ca_certificates/


# # Ignore Python cache files
__pycache__/
*.py[cod]
*.pyo
*.pyd

# # Ignore virtual environments
venv/
ENV/
env/
.venv/
.ENV/
.env/

# # Ignore Ansible logs
*.log

# # Ignore configuration files
*.ini
*.cfg

# # Ignore specific temporary files
*.tmp
*.swp

# # Ignore deployment artifacts
*.tar.gz
*.zip
*.whl

# # Ignore IDE and editor specific files
.vscode/
.idea/
*.iml
*.sublime-project
*.sublime-workspace

# # Ignore system files
.DS_Store
Thumbs.db

# # Ignore compiled code
*.o
*.so
*.a
*.out

# # Ignore additional sensitive files
*.crt
*.key

```

This concludes the detailed runbook for the SecureCertAppDeploy project. It provides a comprehensive guide on setting up and deploying the application using Ansible, ensuring security, and managing CA certificates.


Step by step:

---

### 5. Running the Project Securely

To ensure the security and proper functioning of the SecureCertAppDeploy project, follow these steps:

#### 5.1 Prerequisites

Before running the project, ensure the following prerequisites are met:

- **Ansible Installed**: Ansible must be installed on your control machine where you will execute the playbook.
- **Inventory Configuration**: Ensure your Ansible inventory (`inventory/hosts.ini`) includes the correct target hosts or localhost configurations.
- **Vault Password File**: Have a `vault_password_file.txt` file containing the password to decrypt Ansible Vault secrets.

#### 5.2 Setting Up Secrets

1. **Create Vault Secrets File**:
   - Ensure `vault/secrets.yml` contains the necessary secrets (`secret_key`, `db_path`, `admin_groups`).

2. **Encrypt Secrets**:
   - Encrypt the `vault/secrets.yml` file using Ansible Vault:
     ```bash
     ansible-vault encrypt vault/secrets.yml --vault-password-file vault_password_file.txt
     ```
To encrypt the `vault/secrets.yml` and the files in `vault/ca_certificates/`, you can use Ansible Vault to ensure these sensitive files are securely stored. Here’s how you can do it step-by-step:

### Encrypting `vault/secrets.yml`

1. **Encrypt `secrets.yml`**:
   Use the following command to encrypt the `secrets.yml` file. Ansible Vault will prompt you to enter and confirm a password for encryption.
   ```bash
   ansible-vault encrypt vault/secrets.yml
   ```
   After encryption, the file will be securely stored and cannot be read without decrypting it first.

2. **Decrypt `secrets.yml`** (if needed):
   To decrypt and edit the file, use:
   ```bash
   ansible-vault edit vault/secrets.yml
   ```
   This command will prompt for the vault password to decrypt and open the file for editing.

### Encrypting Files in `vault/ca_certificates/`

1. **Encrypt CA Certificate and Key Files**:
   Encrypt each file individually within the `ca_certificates/` directory using Ansible Vault. You'll need to encrypt each `.crt` and `.key` file separately.
   ```bash
   ansible-vault encrypt vault/ca_certificates/CA1.crt
   ansible-vault encrypt vault/ca_certificates/CA1.key
   ansible-vault encrypt vault/ca_certificates/CA2.crt
   ansible-vault encrypt vault/ca_certificates/CA2.key
   ansible-vault encrypt vault/ca_certificates/CA3.crt
   ansible-vault encrypt vault/ca_certificates/CA3.key
   ```
2. **Decrypt CA Files** (if needed):
   To decrypt and use these files, you'll need to decrypt them individually:
   ```bash
   ansible-vault decrypt vault/ca_certificates/CA1.crt
   ansible-vault decrypt vault/ca_certificates/CA1.key

   ansible-vault decrypt vault/ca_certificates/CA2.crt
   ansible-vault decrypt vault/ca_certificates/CA2.key

   ansible-vault decrypt vault/ca_certificates/CA3.crt
   ansible-vault decrypt vault/ca_certificates/CA3.key   
   ```
    ```bash
    ansible-vault encrypt vault/* --vault-password-file vault_password_file.txt
    ```

    ### Explanation:
    - **`ansible-vault encrypt vault/*`**: This command uses Ansible Vault to encrypt all files (`*`) within the `vault/` directory.
    - **`--vault-password-file vault_password_file.txt`**: Specifies the path to a file (`vault_password_file.txt`) containing the vault password.

### Managing Encrypted Files

- **Edit Encrypted Files**: Use `ansible-vault edit` to open and edit encrypted files securely.
  ```bash
  ansible-vault edit vault/secrets.yml
  ansible-vault edit vault/ca_certificates/CA1.crt
  ```
- **Decrypt Temporarily**: Use `ansible-vault view` to temporarily view the contents without decrypting to a file.

  ```bash
  ansible-vault view vault/secrets.yml
  ansible-vault view vault/ca_certificates/CA1.crt
  ```

Ensure to securely manage the vault password file (`vault_password_file.txt`) and restrict access to authorized personnel only. This approach ensures that sensitive information such as secret keys and certificates are protected throughout their lifecycle in your Ansible-managed environment.

#### 5.3 Running the Playbook

1. **Execute the Playbook**:
   - Run the main playbook (`playbooks/main.yml`) to deploy the application and configure CA certificates:
     ```bash
     ansible-playbook playbooks/main.yml --vault-password-file vault_password_file.txt
     ```

#### 5.4 Verifying Deployment

1. **Check Application Deployment**:
   - Verify that the Python Flask application is running correctly. You can check the application's endpoint to ensure it responds as expected.

     - Command to verify Flask application status:
     ```bash
     curl http://127.0.0.1:5000/healthcheck
     ```
2. **Verify CA Certificates**:
   - Ensure both standard and custom CA certificates are correctly installed and trusted on the target hosts.

     ```bash
     openssl s_client -connect yourdomain.com:443 -CAfile /etc/ssl/certs/ca-certificates.crt
     ```
     
#### 5.5 Security Measures

1. **Secure Vault Access**:
   - Keep `vault_password_file.txt` secure and accessible only to authorized personnel.
      
      - Command to ensure permissions:
     ```bash
     chmod 600 vault_password_file.txt
     ```

2. **Limit Access to Secrets**:
   - Restrict access to the encrypted secrets file (`vault/secrets.yml`) based on the principle of least privilege.

      - Restrict permissions on `vault/secrets.yml`:
     ```bash
     chmod 600 vault/secrets.yml
     ```

3. **Review Security Logs**:
   - Regularly review Ansible logs and system logs for any unauthorized access attempts or security incidents.

     - Regularly check Ansible logs:
     ```bash
     tail -n 50 /var/log/ansible.log
     ```
     - Check system logs for security incidents:
     ```bash
     journalctl -xe
     ```

#### 5.6 Post-Deployment Maintenance

1. **Update and Patch**:
   - Regularly update and patch both the application and the operating system to mitigate security vulnerabilities.
    - Update the system and application:
        ```bash
        sudo apt update && sudo apt upgrade -y
        ```
2. **Monitor Application Performance**:
   - Monitor the performance and security of the deployed application to ensure it meets operational requirements.
       - Monitor application logs:
        ```bash
        tail -n 50 /var/log/app.log
        ```
        - Check system resources:
            ```bash
            top
            ```
3. **Backup and Disaster Recovery**:
   - Implement a backup and disaster recovery plan for both application data and configuration files.
    - Backup application data and configuration:
        ```bash
        rsync -avz /opt/example /backup/location
        ```
    - Test disaster recovery plan:
        ```bash
        systemctl status example.service
        systemctl stop example.service
        systemctl start example.service
        ```
---

Following these steps ensures that the SecureCertAppDeploy project is deployed securely and efficiently using Ansible, adhering to best practices in security and operations.