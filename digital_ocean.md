# Deploying a Django + MySQL App to a Digital Ocean Droplet

## 1. Setup a Digital Ocean Droplet

- Log in to your [DigitalOcean account](^1^) and click on the **Create** button at the top right corner. Select **Droplets** from the menu.
- Choose an image for your Droplet. For this guide, we will use **Ubuntu 20.04 (LTS) x64** as the base operating system.
- Choose a plan for your Droplet. You can start with the **Basic** plan and select the size that suits your needs. For this guide, we will use the **$5/month** option with **1 GB** of memory and **25 GB** of disk space.
- Choose a datacenter region for your Droplet. You can select the one that is closest to your location or your target audience.
- Optionally, you can add additional options to your Droplet, such as backups, IPv6, private networking, or monitoring.
- Add your SSH keys to your Droplet. This will allow you to log in to your Droplet securely and without a password. You can follow this [guide](^2^) to generate and add SSH keys to your DigitalOcean account.
- Give your Droplet a name and click on the **Create Droplet** button. It may take a few minutes for your Droplet to be provisioned and ready.

## 2. Use terminal to ssh into the Droplet you just created

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

## 3. Run updates

- Before you install any packages or software on your Droplet, it is a good practice to update the package index and upgrade the existing packages to their latest versions. You can do this by running the following commands:

```bash
apt update
apt upgrade
```

- You may be prompted to confirm some actions or restart some services. Follow the instructions on the screen and press **Enter** to continue.

## 4. Create nonroot user with sudo access

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

## 5. Disable root login

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

## 6. Enable ssh port on the firewall

- To protect your Droplet from unwanted traffic and potential attacks, you can enable a firewall that will only allow connections to specific ports. Ubuntu comes with a firewall tool called `ufw`, which stands for **U**ncomplicated **F**ire**w**all.
- Before you enable the firewall, you need to make sure that the SSH port (22 by default) is allowed, otherwise you will lock yourself out of your Droplet. You can do this by using the `ufw` command:

```bash
ufw allow ssh
```

- You should see a message saying `Rules updated` and `Rules updated (v6)`.
- You can now enable the firewall by typing:

```bash
ufw enable
```

- You will be asked to confirm the action. Type `y` and press **Enter**. You should see a message saying `Firewall is active and enabled on system startup`.
- You can check the status of the firewall and the rules that are applied by typing:

```bash
ufw status
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

## 7. Enable password-less ssh authorized key login

- To further enhance the security and convenience of logging in to your Droplet, you can set up password-less SSH authorized key login, which will allow you to log in to your Droplet without entering a password, using a pair of cryptographic keys.
- You will need to generate a pair of keys on your local machine, if you have not done so already. You can follow this [guide](^2^) to generate and add SSH keys to your DigitalOcean account.
- Once you have your SSH keys, you need to copy the public key to your Droplet. You can do this by using the `ssh-copy-id` command and providing your nonroot user's username and your Droplet's IP address:

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

- You have now enabled password-less SSH authorized key login for your nonroot user.

## 8. Install MySQL and run configuration script

- MySQL is a popular open-source relational database management system that can store and manage data for your Django application. To install MySQL on your Droplet, you can use the `apt` package manager:

```bash
sudo apt install mysql-server
```

- You may be asked to confirm the installation. Press **Enter** to continue.
- After the installation is complete, you need to run a configuration script that will help you secure your MySQL installation and set a password for the root user. You can do this by typing:

```bash
sudo mysql_secure_installation
```

- You will be asked a series of questions and you can follow the recommendations below:

```bash
- Would you like to setup VALIDATE PASSWORD component? N
- Please set the password for root here. <Enter a strong password and press Enter>
- Remove anonymous users? Y
- Disallow root login remotely? Y
- Remove test database and access to it? Y
- Reload privilege tables now? Y
```

- You have now installed and configured MySQL on your Droplet.

## 9. Create a dedicated MySQL user

- It is not recommended to use the root user for your Django application, as it can pose security risks and grant too much access to your database. Therefore, it is better to create a dedicated MySQL user that will only have access to the database that your application needs.
- To create a new MySQL user, you need to log in to the MySQL shell as the root user. You can do this by typing:

```bash
sudo mysql -u root -p
```

- You will be asked to enter the password that you set for the root user in the previous step. After that, you should see a prompt like this:

```bash
mysql>
```

- You are now logged in to the MySQL shell as the root user.
- To create a new MySQL user, use the `CREATE USER` statement and provide a username and a password. Replace `<your_mysql_username>` and `<your_mysql_password>`

Source: 12/19/2023
(1) Deploy a Django App on App Platform - DigitalOcean. https://docs.digitalocean.com/developer-center/deploy-a-django-app-on-app-platform/.
(2) How to Set Up a Scalable Django App with DigitalOcean Managed Databases .... https://www.digitalocean.com/community/tutorials/how-to-set-up-a-scalable-django-app-with-digitalocean-managed-databases-and-spaces.
(3) How to Set Up a Scalable Django App with DigitalOcean Managed Databases .... https://www.digitalocean.com/community/tutorials/how-to-set-up-a-scalable-django-app-with-digitalocean-managed-databases-and-spaces.
(4) Deploying a Django + MySQL App to a Digital Ocean Droplet. https://www.chadams.me/blog/2021-06-08-django-on-digital-ocean-droplet/.
(5) Web Hosting a Django web site in a droplet. | DigitalOcean. https://www.digitalocean.com/community/questions/web-hosting-a-django-web-site-in-a-droplet.
(6) Deploy Django App Digitalocean Ubuntu 20.04 Server - Tati Digital Connect. https://blog.tati.digital/2021/02/22/deploy-django-app-digitalocean-ubuntu-20-04-server/.
(7) How to deploy a Django 2.1.1 web application with MySQL on DigitalOcean .... https://medium.com/@bernardo.laing/how-to-deploy-a-django-2-1-1-web-application-with-mysql-on-digitalocean-e2d22f59264b.