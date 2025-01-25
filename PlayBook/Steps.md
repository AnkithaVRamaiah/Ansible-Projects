### **Objective:**
We will create a simple playbook that installs the **Nginx** web server on a remote host and ensures that the Nginx service is running.

### **Pre-requisites:**
1. **Ansible installed** on your machine.
2. **Inventory file** that lists the target hosts (servers) where you want to run the playbook.

---

### **Step 1: Set Up Your Inventory File**

An **inventory file** defines the hosts that Ansible will manage. You will either define it in an INI format (commonly used) or a YAML format.

**Example Inventory File (hosts.ini)**:
```ini
[web_servers]
webserver1 ansible_host=192.168.1.10
webserver2 ansible_host=192.168.1.11
```
- **[web_servers]**: This defines a group of hosts, which you can refer to in the playbook.
- **webserver1**: This is a label for the host.
- **ansible_host**: This specifies the IP address of the host (you can use the actual hostname as well).

**Alternatively, using YAML format** (optional):
```yaml
all:
  children:
    web_servers:
      hosts:
        webserver1:
          ansible_host: 192.168.1.10
        webserver2:
          ansible_host: 192.168.1.11
```

### **Step 2: Create the Playbook File**

A **playbook** is a YAML file that defines the tasks you want Ansible to execute. A simple playbook has the following structure:

```yaml
---
- name: Playbook to install Nginx on web servers
  hosts: web_servers    # Specify the group of hosts to run the playbook on
  become: yes           # Use 'sudo' to execute tasks with elevated privileges
  tasks:
    - name: Install Nginx
      apt:               # Use Ansible's 'apt' module for managing packages
        name: nginx      # Name of the package to be installed
        state: present    # Ensure that the package is installed (present)
    - name: Ensure Nginx is started
      service:           # Use Ansible's 'service' module for managing services
        name: nginx      # Name of the service to manage
        state: started    # Ensure the service is running
```

Let's break this down:

---

### **Step 3: Understanding the Playbook Structure**

1. **Playbook Start**:
   - The playbook starts with `---`, which marks the beginning of the YAML file.

2. **The `name` Field**:
   - `name: Playbook to install Nginx on web servers` is a description of the playbook. It provides context for what the playbook is doing. This is helpful when running multiple playbooks or reviewing logs.

3. **The `hosts` Field**:
   - `hosts: web_servers` defines the **target hosts** on which the playbook should run. These are defined in the inventory file. 
   - You could also specify a single host (e.g., `localhost` for local execution) or a specific host.

4. **The `become` Field**:
   - `become: yes` tells Ansible to escalate privileges (i.e., run commands with `sudo`). This is required because installing software typically needs elevated privileges.
   - `become: yes` is equivalent to running commands as a superuser.

5. **The `tasks` Field**:
   - This section lists all the tasks that Ansible will execute on the target hosts. Each task has:
     - A **name**: A description of the task.
     - A **module**: An Ansible module (e.g., `apt`, `service`, `copy`, etc.) that performs the task.
     - Parameters for the module (e.g., `name`, `state`, etc.).

6. **First Task: Install Nginx**:
   ```yaml
   - name: Install Nginx
     apt:
       name: nginx
       state: present
   ```
   - `apt`: This is the Ansible module used to manage packages on Debian-based systems (like Ubuntu).
   - `name: nginx`: This is the package to install.
   - `state: present`: Ensures that the Nginx package is installed. If it is not installed, Ansible will install it. If it is already installed, no action is taken.

7. **Second Task: Start Nginx Service**:
   ```yaml
   - name: Ensure Nginx is started
     service:
       name: nginx
       state: started
   ```
   - `service`: This module is used to manage system services.
   - `name: nginx`: Specifies the service to manage (Nginx service).
   - `state: started`: Ensures that the service is running. If it’s not running, Ansible will start it.

---

### **Step 4: Run the Playbook**

To run the playbook, use the following command:

```bash
ansible-playbook -i hosts.ini install_nginx.yml
```

Here:
- `-i hosts.ini`: Specifies the inventory file where your target hosts are defined.
- `install_nginx.yml`: Specifies the playbook file to run.

### **Step 5: Observe the Output**

Once the playbook is executed, Ansible will display output that looks something like this:

```
PLAY [Playbook to install Nginx on web servers] ***
TASK [Install Nginx] ***
changed: [webserver1] => (item=None) => {
    "msg": "The cache of apt has been updated"
}
TASK [Ensure Nginx is started] ***
changed: [webserver1] => (item=None) => {
    "msg": "The service nginx has been started"
}
```

- `changed`: This indicates that Ansible made changes on the remote host (like installing a package or starting a service).
- `ok`: If the task didn’t require any change (i.e., the state was already as expected), you would see `ok` instead of `changed`.

---

### **Step 6: Verify the Changes**

After the playbook has run, you can manually verify that Nginx is installed and running on the remote host by logging in to the server or using `curl`:

```bash
curl http://<webserver1_ip>
```

You should see the default Nginx web page, confirming that it has been installed and is running.

---

### **Step 7: Additional Considerations**

1. **Error Handling**:
   - You can add error handling to ensure tasks continue even if a task fails using the `ignore_errors: yes` directive.
   
   ```yaml
   - name: Install Nginx
     apt:
       name: nginx
       state: present
     ignore_errors: yes
   ```

2. **Variable Usage**:
   - You can use variables to make your playbook more dynamic. Variables can be defined in a separate YAML file or directly in the playbook.

   ```yaml
   ---
   - name: Install Nginx
     hosts: web_servers
     vars:
       package_name: nginx
     tasks:
       - name: Install the package
         apt:
           name: "{{ package_name }}"
           state: present
   ```

3. **Running Playbooks on Remote Hosts**:
   - Ansible connects to remote hosts using SSH. Ensure that SSH access is configured correctly.
   - You may also need to set up a passwordless SSH key pair for smoother execution.

---

### **Conclusion:**

Writing a simple Ansible playbook involves:
1. Setting up an inventory of hosts.
2. Writing tasks to manage software installation and services.
3. Running the playbook with `ansible-playbook` command.

Ansible playbooks are powerful because they let you automate repetitive tasks, like setting up servers, and ensure that configurations are consistent across all servers.