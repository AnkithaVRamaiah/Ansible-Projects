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

### Goal: Set up Passwordless SSH Authentication in Ansible

You will be able to SSH into a remote machine (like an EC2 instance) without needing to type in your password every time. This is essential for Ansible, as it automates SSH-based tasks.

### Step 1: Install Ansible on the Control Node
Before setting up SSH authentication, you need to install Ansible on the machine that will control your remote nodes (referred to as the **control node**).

For **Ubuntu-based systems**, install Ansible with the following commands:

```bash
sudo apt update
sudo apt install ansible -y
```

This ensures you have Ansible set up on the control node to manage remote nodes.

### Step 2: SSH Key Authentication (Passwordless Login Using SSH Key)

#### A. Generate an SSH Key Pair (If You Don't Already Have One)

Open your terminal and create an SSH key pair by running:

```bash
ssh-keygen -t rsa -b 2048
```

This will generate two files:

- `id_rsa` (private key — **keep this secure**)
- `id_rsa.pub` (public key — **share this with other systems**)

When asked where to save the key, press **Enter** (the default location `~/.ssh/id_rsa` is fine).

#### B. Copy the Public Key to the Remote Machine

Now, you need to tell your remote machine (e.g., an EC2 instance) to trust your control node. 

**If you're using an EC2 instance:**

1. Ensure you have the **private key** (`my-key.pem`) you used to launch the EC2 instance.
2. Move this key to your control node's `~/.ssh/` directory.
3. Use `ssh-copy-id` to copy your public key:

```bash
ssh-copy-id -i pathof_.ssh/id_rsa.pub ubuntu@<remote-node-ip>
```

**Explanation:**
- Replace `<remote-node-ip>` with the actual IP address of your remote instance (e.g., EC2).
- This command will ask for the password of the remote machine for the first time. After entering the password, the SSH key will be copied, and future logins will not require a password.

#### C. Verify Passwordless SSH Login

To confirm that you can log in without a password, run:

```bash
ssh ubuntu@<remote-node-ip>
```

You should now be able to log in to the remote instance without being asked for a password.

### Step 3: Password Authentication (Passwordless Login Using Password)

If you want to use **password-based SSH** but still set up passwordless login, follow these steps:

#### A. Edit SSH Configuration File on the Remote Machine

Log into the remote machine using the password:

```bash
ssh -i keypair.pem ubuntu@<remote-node-ip>
```

Once logged in, edit the SSH configuration file to allow password-based login:

```bash
sudo vi /etc/ssh/sshd_config
```

Look for the line:

```bash
PasswordAuthentication yes
```

Ensure it's set to `yes`. If it's not present, add it.

**Explanation:**
- Allowing password authentication might seem counterintuitive to passwordless login, but it’s required for enabling passwordless login via SSH key after setting up the user password.

Save the file (in nano, press **Ctrl+X**, then **Y**, and hit **Enter**).

#### B. Restart SSH Service

To apply the changes, restart the SSH service:

```bash
sudo systemctl restart sshd
```

#### C. Set a Password for the User

Ensure that the user (e.g., `ubuntu`) has a password set. If not, set one:

```bash
sudo passwd ubuntu
```

Enter a password when prompted then logout

#### D. Copy SSH Key to the Remote Machine Using `ssh-copy-id`

Now, copy your public key to the remote machine so you can log in without entering a password:

```bash
ssh-copy-id ubuntu@<remote-node-ip>
```

This will ask for the password you just set. After providing it, your SSH key will be copied.

#### E. Verify Passwordless Login

Finally, test if you can log in without entering a password:

```bash
ssh ubuntu@<remote-node-ip>
```

You should now be able to log in without entering a password.

### Summary of Methods

- **SSH Key Authentication (Recommended for security):**
  1. Generate an SSH key pair (`ssh-keygen`).
  2. Copy the public key to the remote machine (`ssh-copy-id`).
  3. Log in without a password.

- **Password Authentication (Less secure, but works):**
  1. Modify SSH settings to allow password authentication.
  2. Set a password for the user.
  3. Use `ssh-copy-id` to copy the public key and enable passwordless login.
  4. Log in without entering a password.

### Why Does This Matter?

Setting up passwordless SSH (either with keys or passwords) is crucial for automating tasks with tools like Ansible. Once you've configured passwordless SSH, Ansible can automate configuration management, software deployment, and system monitoring across multiple servers without requiring manual password input. This streamlines operations and helps maintain consistency across environments.

---
