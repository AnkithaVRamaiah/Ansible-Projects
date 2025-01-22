### Passwordless Authentication in Ansible

In Ansible, a key prerequisite is **passwordless authentication** between the control node and the managed nodes. 

#### Example Scenario:

If you want to install a service like **Nginx** on a managed node, you can either:
- Write a **YAML playbook** to define the task.
- Use an **Ad-hoc command** (a simple one-line command) to perform the installation directly.

When the control node tries to connect to a managed node to execute this task, it requires **authentication**. This can be done in two ways:
1. **Password-based authentication**: Entering the password every time the control node connects to a managed node.
2. **SSH key-based authentication**: Using **SSH keys** to establish a secure connection without repeatedly entering passwords.

Using passwords every time for multiple managed nodes is time-consuming. To automate this, we set up **passwordless authentication**.

#### What is Passwordless Authentication?

Passwordless authentication allows the control node to connect to a managed node without being prompted for a password each time. Here's how it works:
- The first time you connect to a managed node, you provide the password or **SSH key**.
- Once set up, future connections from the control node to the managed node will not require entering the password again.

---

### Steps to Set Up Passwordless SSH Authentication in Ansible

#### 1. **Install Ansible on the Control Node**:
   - First, install **Ansible** on your control node. For example, if you're using **Windows**, you can use **Windows Subsystem for Linux (WSL)** to install Ansible or use a Linux-based machine.

#### 2. **Create a Managed Node**:
   - Create an **instance** (for example, an **AWS EC2 instance**) to serve as the managed node. 

#### 3. **Generate SSH Key Pair**:
   - On the control node, generate an **SSH key pair** (if you don't have one already) using the following command:
     ```
     ssh-keygen
     ```
   - This will create a public and private SSH key (typically located in `~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub`).

#### 4. **Copy the SSH Key to the Managed Node**:
   - Use the `ssh-copy-id` command to copy the public key to the managed node:
     ```
     ssh-copy-id -i ~/.ssh/id_rsa.pub ubuntu@<Public_IP_of_Managed_Node>
     ```
   - Replace `<Public_IP_of_Managed_Node>` with the actual public IP address of your managed node.

   - This command will prompt you for the **password** of the **ubuntu** user on the managed node. Once authenticated, the SSH key will be added to the `~/.ssh/authorized_keys` file on the managed node.

#### 5. **Test the SSH Connection**:
   - Now, try logging into the managed node without entering a password:
     ```
     ssh ubuntu@<Public_IP_of_Managed_Node>
     ```
   - If everything is set up correctly, you should be able to log in **without** being prompted for a password, using the SSH key instead.

#### 6. **Configure SSH for Password Authentication (if needed)**:
   - If you're unable to log in or need to enable **password authentication** on the managed node, follow these steps:
     1. SSH into the managed node and open the SSH configuration file for editing:
        ```
        sudo nano /etc/ssh/sshd_config
        ```
     2. Find the line `PasswordAuthentication` and make sure it’s set to `yes`:
        ```
        PasswordAuthentication yes
        ```
     3. If the line is commented (i.e., starts with `#`), uncomment it by removing the `#` symbol.
     4. Save the file and restart the SSH service to apply changes:
        ```
        sudo systemctl restart ssh
        ```

#### 7. **Create a Password for the User (if necessary)**:
   - If you’ve enabled password authentication, you need to set a password for the `ubuntu` user:
     ```
     sudo passwd ubuntu
     ```
   - Provide the password when prompted, and remember it for future logins.

#### 8. **Use SSH Copy-ID Again (if needed)**:
   - In case the SSH key was not copied previously, or you changed the password configuration, you can use the following command again:
     ```
     ssh-copy-id ubuntu@<Public_IP_of_Managed_Node>
     ```
   - This will ensure the public key is copied to the managed node, allowing you to log in without a password.

#### 9. **Log in with SSH**:
   - Finally, attempt to log in again using SSH:
     ```
     ssh ubuntu@<Public_IP_of_Managed_Node>
     ```
   - You should now be able to log in **without** a password prompt, using **passwordless SSH authentication**.

---

### Summary:

With this setup, you’ve achieved passwordless authentication, allowing the **control node** (where Ansible is installed) to securely connect to the **managed node** without needing to enter a password each time. This is an essential step for automating tasks across multiple nodes using Ansible.