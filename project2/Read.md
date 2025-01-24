## Purpose:
The task involves automating AWS EC2 instance creation using Ansible, setting up secure authentication between nodes, and managing instances (e.g., shutting them down) via Ansible playbooks.

---

### Step 1: **Reduce Repetitive Tasks Using Loops and Automation**
To avoid manual and repetitive tasks, we use loops and Ansible automation. Ansible's `loop` simplifies tasks like creating multiple EC2 instances with different configurations.

---

### Step 2: **Create an IAM User for Ansible**
1. **Create IAM User**:  
   - Go to AWS Management Console → IAM → Users → Add User.  
   - Set username (e.g., `ansible-user`) and enable **Programmatic access**.
   - Attach the policy **AmazonEC2FullAccess** to the user. This grants the user full permissions to manage EC2 resources.

2. **Generate Access Key**:  
   - After creating the user, download the access key (`Access Key ID` and `Secret Access Key`) securely for later use.

---

### Step 3: **Set Up Ansible Control Node for AWS Integration**
1. **Install Required Tools**:
   - **Boto3**: Install AWS SDK for Python (used by Ansible modules to interact with AWS):  
     ```bash
     pip install boto3
     ```
   - **AWS Ansible Collection**: Install AWS modules for Ansible:  
     ```bash
     ansible-galaxy collection install amazon.aws
     ```

2. **Configure Ansible Vault for Credentials Security**:  
   - Create a vault password file:  
     ```bash
     echo "your-vault-password" > vault.pass
     ```
   - Encode the AWS access key and secret key using the vault:  
     ```bash
     ansible-vault encrypt_string 'your-access-key' --name ec2_access_key > vars.yml
     ansible-vault encrypt_string 'your-secret-key' --name ec2_secret_key >> vars.yml
     ```
     > **Note**: This stores encrypted credentials in `vars.yml`.

   - Use `vault.pass` to unlock the encrypted credentials.

---

### Step 4: **Write the Playbook to Launch EC2 Instances**
```yaml
---
- name: Launch EC2 Instances on AWS
  hosts: localhost
  connection: local

  vars_files:
    - vars.yml  # Encrypted variables file

  tasks:
    - name: Launch EC2 instances
      amazon.aws.ec2_instance:
        name: "{{ item.name }}"
        key_name: "har.pem"  # Update with your key pair name
        instance_type: t2.micro
        security_group: default
        region: ap-south-1
        aws_access_key: "{{ ec2_access_key }}"  # Decrypted from vault
        aws_secret_key: "{{ ec2_secret_key }}"  # Decrypted from vault
        network:
          assign_public_ip: true
        image_id: "{{ item.image }}"
        tags:
          environment: "{{ item.name }}"
      loop:
        - { image: "ami-0e1d06225679bc1c5", name: "manage-node-1" }
        - { image: "ami-0f58b397bc5c1f2e8", name: "manage-node-2" }
        - { image: "ami-0f58b397bc5c1f2e8", name: "manage-node-3" }
```

---

### Step 5: **Set Up Passwordless Authentication**
1. **Generate SSH Key on Control Node**:
   ```bash
   ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa_ansible
   ```
2. **Copy Public Key to Managed Nodes**:
   - Use the EC2 instance’s public IP (from AWS Console or `aws ec2 describe-instances`) and copy the public key:
     ```bash
     ssh-copy-id -i ~/.ssh/id_rsa_ansible.pub ec2-user@<managed-node-ip>
     ```

---

### Step 6: **Create an Inventory File**
- Define managed nodes in an inventory file (e.g., `inventory.ini`):
  ```ini
  [managed_nodes]
  manage-node-1 ansible_host=<public-ip-of-node-1> ansible_user=ec2-user ansible_ssh_private_key_file=~/.ssh/id_rsa_ansible
  manage-node-2 ansible_host=<public-ip-of-node-2> ansible_user=ec2-user ansible_ssh_private_key_file=~/.ssh/id_rsa_ansible
  manage-node-3 ansible_host=<public-ip-of-node-3> ansible_user=ec2-user ansible_ssh_private_key_file=~/.ssh/id_rsa_ansible
  ```

---

### Step 7: **Playbook to Shutdown Ubuntu Instances**
```yaml
---
- name: Shutdown Ubuntu Instances
  hosts: all
  become: true

  tasks:
    - name: Shutdown Ubuntu instances only
      ansible.builtin.command: /sbin/shutdown -h now
      when: ansible_facts['os_family'] == "Debian"
```

---

### Explanation:
1. **Loops**:
   - Instead of repeating tasks for each instance, `loop` dynamically creates multiple instances with different AMI IDs and names.

2. **Secure Credentials with Vault**:
   - The access and secret keys are encrypted using `ansible-vault`, ensuring sensitive data isn’t exposed in plain text.

3. **Passwordless Authentication**:
   - SSH keys are configured for seamless communication between the control and managed nodes.

4. **Inventory File**:
   - Lists all managed nodes for easier reference and playbook targeting.

5. **Shutdown Playbook**:
   - The `when` condition ensures only Debian/Ubuntu-based instances are affected.  

