# Deploying a Hestia Panel to a Digital Ocean Ubuntu Droplet

In this guide, we will learn how to deploy a Django web application with MySQL database and Nginx web server to a Digital Ocean droplet. We will use a git repository to store our code and set up a post-receive hook to automate the deployment process. We will also follow some best practices to secure our droplet and optimize our application performance.

## Prerequisites

Before you begin, you will need the following:

- A Digital Ocean account and an API token
- A local machine with git, Python, and pip installed
- A Django project with MySQL database settings. We will use [this example project](https://github.com/redwan-cse/pets-finder) for demonstration purposes.
- A domain name and a DNS record that points to your droplet's IP address

## Step 1: Setup a Digital Ocean Droplet

A droplet is a virtual machine that runs on the Digital Ocean cloud platform. To create a droplet, follow these steps:

- Log in to your Digital Ocean account and click on the **Create** button at the top right corner.
- Select **Droplets** from the menu and choose **Ubuntu 22.04 (LTS) x64** as the image.
- Choose a plan that suits your needs and budget. For this guide, we will use the **Basic** plan with **$6/month** and **1 GB** of memory.
- Choose a datacenter region that is close to your target audience. For this guide, we will use **Singapore** as the region.
- Optionally, you can add additional options such as backups, monitoring, or IPv6.
- Under **Select additional options**, check the box that says **User data** and paste the following script in the text box. This script will run when the droplet is created and will perform some initial setup tasks such as creating a nonroot user, turning off root login, enabling SSH port on the firewall, enabling ufw, allowing password-less SSH authorized key login, and updating the packages.

### Use terminal to ssh into the Droplet you just created

- Once your Droplet is ready, you can find its IP address on the DigitalOcean dashboard. Copy the IP address and open a terminal on your local machine.
- Use the `ssh` command to connect to your Droplet as the root user. Replace the IP address with your Droplet's IP address:

```bash
ssh root@<your_droplet_ip>
```

- You may see a warning message about the authenticity of the host. Type `yes` and press **Enter** to continue. You will not see this message again for this Droplet.
- You should see a welcome message and a prompt like this:

```bash
root@<your_droplet_name>:~#
```

- You are now logged in to your Droplet as the root user.

### Run updates

- Before you install any packages or software on your Droplet, it is a good practice to update the package index and upgrade the existing packages to their latest versions. You can do this by running the following commands:

```bash
apt update && apt upgrade -y
```

- You may be prompted to confirm some actions or restart some services. Follow the instructions on the screen and press **Enter** to continue.

### Create nonroot user with sudo access

- It is not recommended to use the root user for everyday tasks, as it can pose security risks and cause irreversible damage to your system. Therefore, it is better to create a regular user account with sudo privileges, which will allow you to perform administrative tasks by prefixing commands with `sudo`.
- To create a new user account, use the `adduser` command and follow the prompts. Replace `<your_username>` with your desired username:

```bash
adduser <your_username>
```

- You will be asked to set a password and provide some optional information for the user. You can leave the fields blank by pressing **Enter** or fill them as you wish.
- Next, you need to add the new user to the `sudo` group, which will grant the user sudo privileges. You can do this by using the `usermod` command:

```bash
usermod -aG sudo <your_username>
```

- You have now created a nonroot user with sudo access.

### Disable root login

- For security reasons, it is advisable to disable root login via SSH, as it can prevent unauthorized access to your Droplet. To do this, you need to edit the SSH configuration file using a text editor such as `nano`:

```bash
nano /etc/ssh/sshd_config
```

- Find the line that says `PermitRootLogin yes` and change it to `PermitRootLogin no`. If the line is commented out with a `#` symbol, remove the `#` as well.
- Save and close the file by pressing **Ctrl+X**, then **Y**, then **Enter**.
- Restart the SSH service to apply the changes:

```bash
systemctl restart ssh
```

- You have now disabled root login via SSH. From now on, you will need to use the nonroot user account to log in to your Droplet.

### Enable ssh port on the firewall

- To protect your Droplet from unwanted traffic and potential attacks, you can enable a firewall that will only allow connections to specific ports. Ubuntu comes with a firewall tool called `ufw`, which stands for **U**ncomplicated **F**ire**w**all.
- Before you enable the firewall, you need to make sure that the SSH port (22 by default) is allowed, otherwise you will lock yourself out of your Droplet. You can do this by using the `ufw` command:

```bash
sudo ufw allow 22
```

- You should see a message saying `Rules updated` and `Rules updated (v6)`.
- You can now enable the firewall by typing:

```bash
sudo ufw enable
```

- You will be asked to confirm the action. Type `y` and press **Enter**. You should see a message saying `Firewall is active and enabled on system startup`.
- You can check the status of the firewall and the rules that are applied by typing:

```bash
sudo ufw status
```

- You should see something like this:

```bash
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
```

- This means that the firewall is active and only allows connections to the SSH port from anywhere.

### Enable password-less ssh authorized key login

- To further enhance the security and convenience of logging in to your Droplet, you can set up password-less SSH authorized key login, which will allow you to log in to your Droplet without entering a password, using a pair of cryptographic keys.
- You will need to generate a pair of keys on your local machine, if you have not done so already. You can follow this [guide](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server) to generate and add SSH keys to your DigitalOcean droplet.
- Once you have your SSH keys, you need to copy the public key to your Droplet. You can do this by using the `ssh-copy-id` command and providing your nonroot user's username and your Droplet's IP address from local machine:

```bash
ssh-copy-id <your_username>@<your_droplet_ip>
```
- You will be asked to enter the password for your nonroot user. After that, you should see a message saying that your key has been added to your Droplet.
- To test that password-less SSH login works, you can try to log in to your Droplet using the `ssh` command and your nonroot user's username:

```bash
ssh <your_username>@<your_droplet_ip>
```

- You should not be prompted for a password and you should see a prompt like this:

```bash
<your_username>@<your_droplet_name>:~$
```

- You should change ownership & permission

```bash
chown -R <your_username>:<your_username> /home/<your_username>/.ssh
chmod 700 /home/<your_username>/.ssh
chmod 600 /home/<your_username>/.ssh/authorized_keys
```

- You have now enabled password-less SSH authorized key login for your nonroot user.


## Step 2: Increase Swap Space

Swap space is an area on the disk that is used as virtual memory when the physical memory (RAM) is full. Swap space can help improve the performance of your application by preventing out-of-memory errors. To increase the swap space on your droplet, follow these steps:

- To determine whether the system already has a swap file, use the `swapon --show` command. If any files are displayed, this means one or more swap files already exist.

```bash
sudo swapon --show
```

- If a swap file has already been created, it is shown as follows. If there is no swap file, there is no output:

```bash
    NAME    TYPE    SIZE    USED    PRIO
    /swapfile   file  1024M   0B  -2
```

- Confirm swap space has not already been allocated using the `free -h` command.

```bash
free -h
```

- You should see something like this:

```bash
              total        used        free      shared  buff/cache   available
Mem:          969Mi       113Mi       133Mi       0.0Ki       738Mi       728Mi
Swap:            0B          0B          0B
```

- As you can see, there is no swap space allocated. To create a 1 GB swap file, use the following commands:

```bash
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
sudo swapon --show
```

### Making the Swap File Changes Permanent

-At this point, the new swap file is fully functional and the system can use it. Unfortunately, the file is not permanent. The swap file disappears after a system reboot and must be recreated before it can be used again.

- As a precaution, make a backup copy of the existing `fstab` file.

```bash
sudo cp /etc/fstab /etc/fstab.bak
```

- To make the swap file permanent, edit the `/etc/fstab` file

```bash
sudo nano /etc/fstab
```

- And add the following line at the end:

```bash
/swapfile swap swap defaults 0 0
```

- To verify that the swap file is active, use the `free -h` command again.

```bash
free -h
```

- You should see something like this:

```bash
              total        used        free      shared  buff/cache   available
Mem:          969Mi       113Mi       133Mi       0.0Ki       738Mi       728Mi
Swap:         1.0Gi          0B       1.0Gi
```
