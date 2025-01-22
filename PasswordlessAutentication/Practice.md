# Passwordless Authentication 

Passwordless authentication can be achieved using two primary methods: **SSH keys** and **password-based authentication**. This method is crucial for environments where seamless and secure communication between nodes is required, such as:

- AWS EC2 servers communication
- Jenkins master-slave setup
- Ansible control node to managed nodes

---

## **1. Using SSH Key-Based Authentication**
SSH key-based authentication is a secure and widely used method for passwordless login. Below are the steps to set it up for an Ansible control node and managed nodes.

### Prerequisites:
- Control Node: Can be your local machine (e.g., with WSL) or an EC2 instance with Ansible installed.
- Managed Node: An EC2 instance (Ubuntu in this example).

---
### **Steps to Set Up SSH Key-Based Authentication**

#### Step 1: Log in to the Instances
- If using an EC2 instance as the control node:
  ```bash
  ssh -i <path_to_pem_file> ubuntu@<control_node_ip_address>
  ```
- If using your local machine as the control node, no need to log in. Proceed with the next steps directly.

- Log in to the managed node:
  ```bash
  ssh -i <path_to_pem_file> ubuntu@<managed_node_ip_address>
  ```

---
#### Step 2: Generate SSH Keys on the Control Node
Run the following command on the control node to generate an SSH key pair:
```bash
ssh-keygen
```
This will create two files in the `~/.ssh/` directory:
- `id_rsa`: Private key (keep this secure).
- `id_rsa.pub`: Public key.

---
#### Step 3: Copy the Public Key to the Managed Node (Manual Method)
- Open the public key file on the control node:
  ```bash
  vi ~/.ssh/id_rsa.pub
  ```
- Copy the contents of the public key.

- On the managed node, append the copied key to the `authorized_keys` file:
  ```bash
  vi ~/.ssh/authorized_keys
  ```
- Paste the key into the file and save it.

Ensure the permissions for `~/.ssh` and `authorized_keys` are correct:
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

---
#### Step 4: Test Passwordless SSH Login
From the control node, attempt to log in to the managed node:
```bash
ssh ubuntu@<managed_node_ip_address>
```
You should be able to log in without being prompted for a password.

---
#### **Automated Method (ssh-copy-id)**
To avoid manually copying and pasting keys, use the `ssh-copy-id` command:
```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub ubuntu@<managed_node_ip_address>
```
This will copy the public key to the managed node and append it to the `authorized_keys` file.

Now, you can log in without a password:
```bash
ssh ubuntu@<managed_node_ip_address>
```

---

## **2. Using Password-Based Authentication**
Password-based authentication can also be used to set up passwordless communication, but it requires manual setup and is generally considered less secure than SSH key-based authentication.

### **Steps to Set Up Password-Based Authentication**

#### Step 1: Enable Password Authentication on the Managed Node
By default, AWS EC2 instances have password authentication disabled. To enable it:
- Log in to the managed node:
  ```bash
  ssh -i <path_to_pem_file> ubuntu@<managed_node_ip_address>
  ```
- Edit the SSH configuration file:
  ```bash
  sudo vi /etc/ssh/sshd_config
  ```
- Update the following lines:
  ```
  PasswordAuthentication yes
  PermitRootLogin no
  ```
- Save the file and restart the SSH service:
  ```bash
  sudo systemctl restart sshd
  ```

---
#### Step 2: Set a Password for the User
Set a password for the `ubuntu` user on the managed node:
```bash
sudo passwd ubuntu
```
Enter and confirm the new password.

---
#### Step 3: Use SSH to Log In with the Password
From the control node, log in to the managed node using the password:
```bash
ssh ubuntu@<managed_node_ip_address>
```
Enter the password when prompted.

---
#### Step 4: Configure Ansible for Password-Based Authentication
If you're using Ansible, you can specify the password in the inventory file or as a variable in the playbook. However, this method is less secure compared to SSH key-based authentication.

Example:
- Add the managed node to the Ansible inventory file:
  ```
  [managed_nodes]
  <managed_node_ip_address> ansible_ssh_user=ubuntu ansible_ssh_pass=<password>
  ```

---

### **Why Use Passwordless Authentication?**
- **Convenience**: Automates login without entering passwords.
- **Security**: SSH keys are more secure than passwords as they are less prone to brute-force attacks.
- **Efficiency**: Essential for automated tools like Ansible, Jenkins, and other CI/CD pipelines where repetitive logins occur.
- **Scalability**: Easier to manage and scale in multi-node environments.

---

### **Best Practices**
1. Always prefer SSH key-based authentication for better security.
2. Use strong passwords and change them regularly if using password-based authentication.
3. Restrict SSH access to specific IP ranges using security groups or firewalls.
4. Regularly audit and rotate keys or passwords to maintain security.
5. Use tools like Ansible Vault to secure sensitive credentials.

