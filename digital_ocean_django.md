# Deploying a Django + MySQL + Nginix App to a Digital Ocean Droplet

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
apt update
apt upgrade
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
ufw allow 22
```
- You can also enable HTTP/HTTPS and MySQL later
```bash
ufw allow 80 # HTTP
ufw allow 443 # HTTPS
ufw allow 3306 # MySQL
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

- Confirm swap space has not already been allocated using the `free -h` command. You should see something like this:

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

- To make the swap file permanent, edit the `/etc/fstab` file and add the following line at the end:

```bash
/swapfile swap swap defaults 0 0
```

- To verify that the swap file is active, use the `free -h` command again. You should see something like this:

```bash
              total        used        free      shared  buff/cache   available
Mem:          969Mi       113Mi       133Mi       0.0Ki       738Mi       728Mi
Swap:         1.0Gi          0B       1.0Gi
```

### Adjust the Swappiness Setting

- The `swappiness` property controls how aggressively Ubuntu swaps data out of RAM. This setting extends from 0 to 100. Higher values tell the system to use the swap file more often. Configure a setting at the upper end of this range to ensure RAM is also available for other applications. A low `swappiness` setting means the system is less likely to use the swap mechanism. Use a setting closer to zero when application performance is important.

- Verify the current `swappiness` setting. The default value is `60`

```bash
cat /proc/sys/vm/swappiness
```

- To temporarily adjust the value of `swappiness`, use the `sysctl` utility. The following command lowers the value to `50`.

```bash
sudo sysctl vm.swappiness=50
```

- To permanently change this value, edit the `/etc/sysctl.conf` file. Add the following line to the bottom of the file.

```bash
vm.swappiness=50
```


## Step 3: Install MySQL and run the configuration script

MySQL is a popular relational database management system that we will use to store our application data. To install MySQL on your droplet, follow these steps:

- Update the package index and install MySQL by using the following commands:

```bash
sudo apt update
sudo apt install mysql-server -y
```

- Run the MySQL configuration script to secure your installation by using the following command:

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

## Step 4: Create a dedicated MySQL user

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
- To create a new MySQL user, use the `CREATE USER` statement and provide a username and a password of your choice:

```bash
CREATE USER '<your_mysql_username>'@'localhost' IDENTIFIED BY '<your_mysql_password>';
```

- You have now created a new MySQL user.

## Step 5: Grant necessary database privileges to the user you just created

- The new MySQL user needs to have the appropriate permissions to access and modify the database that your Django application will use. To grant the necessary privileges, use the `GRANT` statement and provide the database name and the username:

```bash
GRANT ALL ON <your_database_name>.* TO '<your_mysql_username>'@'localhost';
```

- This will grant the user all privileges on the database and its tables. You can also specify more granular permissions, such as `SELECT`, `INSERT`, `UPDATE`, `DELETE`, etc.
- To apply the changes, use the `FLUSH` statement:

```bash
FLUSH PRIVILEGES;
```

- You have now granted the necessary database privileges to the user you just created.

## Step 5: Create a new database for your application

- Log in to the MySQL shell as the new user by using the following command. Enter your password when prompted.

```bash
mysql -u <your_mysql_username> -p
```

- The next step is to create a new database that your Django application will use to store and manage data. To create a new database, use the `CREATE DATABASE` statement and provide a name for the database:

```bash
CREATE DATABASE <your_database_name>;
```

- You can use any name you want, but make sure it is descriptive and unique. For example, you can use the name of your project or application.

- Exit the MySQL shell by using the following command:

```bash
exit
```
- You have now created a new database for your application.


## Step 6: Setup a virtual environment on your Droplet to manage requirements and packages

A virtual environment is a tool that allows you to create isolated Python environments for different projects. This way, you can avoid conflicts between different versions of packages and dependencies. To set up a virtual environment on your droplet, follow these steps:

- Install the `python3-venv` package by using the following command:

```bash
sudo apt install python3-venv -y
```

- Create a new directory for your project by using the following command:

```bash
mkdir ~/<your_project_name>
```

- Change to the project directory by using the following command:

```bash
cd ~/<your_project_name>
```

- Create a new virtual environment called `env` by using the following command:

```bash
python3 -m venv env
```

- Activate the virtual environment by using the following command:

```bash
source env/bin/activate
```

- You should see `(env)` at the beginning of your prompt, indicating that you are in the virtual environment.

## Step 7: Configure the Python virtual environment and clone your project repo

Now that we have a virtual environment, we need to configure it and clone our project repo. To do this, follow these steps:

- Upgrade `pip`, the Python package manager, by using the following command:

```bash
pip install --upgrade pip
```

- Install `wheel`, a tool that helps with building and installing Python packages, by using the following command:

```bash
pip install wheel
```

- Clone your project repo from GitHub by using the following example command:

```bash
git clone https://github.com/redwan-cse/pets-finder.git
```

- Change to the project directory by using the following command:

```bash
cd <your_project_name>
```

- Install your Django project's dependencies by using the following command:

```bash
pip install -r requirements.txt
```

## Step 8: Set the proper database connection credentials in your Django project's settings.py file and add your IP address to allowed hosts

- The next step is to set the proper database connection credentials in your Django project's settings.py file, which is the main configuration file for your Django application. You also need to add your Droplet's IP address to the list of allowed hosts, which tells Django which hosts are allowed to serve your application.
- To edit the settings.py file, you can use a text editor such as `nano`:

```bash
nano <your_project_name>/settings.py
```

- Find the `DATABASES` setting and replace it with the following:

```python
import dj_database_url
DATABASES = {
    'default': dj_database_url.config(default='mysql://<your_mysql_username>:<your_mysql_password>@localhost:3306/<your_database_name>')
}
```

- This will use the `dj_database_url` package to parse the database URL and set the proper connection parameters for your Django application. Make sure to replace the placeholders with the actual values that you used to create your MySQL user and database.
- Find the `ALLOWED_HOSTS` setting and add your Droplet's IP address to the list, like this:

```python
ALLOWED_HOSTS = ['<your_droplet_ip>']
```

- This will tell Django which hosts are allowed to serve your application. You can also use a domain name instead of an IP address, if you have one configured for your Droplet.
- Save and close the file by pressing **Ctrl+X**, then **Y**, then **Enter**.
- You have now set the proper database connection credentials in your Django project's settings.py file and added your IP address to the allowed hosts.

## Step 9: Test everything to make sure it's working

Before we proceed to configure the web server and the deployment process, we need to test everything to make sure it's working. To do this, follow these steps:

- Run the database migrations by using the following command:

```bash
python manage.py migrate
```

- Create a superuser account by using the following command. Enter your username, email, and password when prompted.

```bash
python manage.py createsuperuser
```

- Run the development server by using the following command:

```bash
python manage.py runserver 0.0.0.0:8000
```

- Open your browser and visit `http://<your_droplet_ip>:8000`. You should see your Django project's homepage.
- Visit `http://<your_droplet_ip>:8000/admin` and log in with your superuser credentials. You should see the Django admin interface.
- Test the functionality of your application and make sure everything is working as expected.
- Stop the development server by pressing `Ctrl+C` in the terminal.

## Step 10: Configure Gunicorn as a WSGI server for your Django project

Gunicorn is a Python WSGI server that can handle concurrent requests and serve your Django project in production. To configure Gunicorn for your Django project, follow these steps:

- Install Gunicorn by using the following command:

```bash
pip install gunicorn
```

- Test Gunicorn by using the following command:

```bash
gunicorn --bind 0.0.0.0:8000 <your_project_name>.wsgi
```

- You should see something like this:

```bash
[2023-12-22 02:18:10 +0600] [1234] [INFO] Starting gunicorn 20.1.0
[2023-12-22 02:18:10 +0600] [1234] [INFO] Listening at: http://0.0.0.0:8000 (1234)
[2023-12-22 02:18:10 +0600] [1234] [INFO] Using worker: sync
[2023-12-22 02:18:10 +0600] [1235] [INFO] Booting worker with pid: 1235
```

- Open your browser and visit `http://<your_droplet_ip>:8000` again. You should see your Django project's homepage served by Gunicorn.
- Stop Gunicorn by pressing `Ctrl+C` in the terminal.

## Step 11: Create a systemd service file for Gunicorn, which will allow Ubuntu to automatically start and manage the Gunicorn process

To create a systemd service file for Gunicorn, you can follow these steps:

- Create or edit the `/etc/systemd/system/gunicorn.service` file by using a text editor such as nano or vi. For example:

```bash
sudo nano /etc/systemd/system/gunicorn.service
```

- Paste the following content in the file and save it. You may need to adjust some values according to your project settings, such as the user, group, working directory, and socket file.

```bash
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
Type=notify
User=<your_username>
Group=<your_username>
WorkingDirectory=/home/<your_username>/<your_project_name>
ExecStart=/home/<your_username>/<your_project_name>/env/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/home/<your_username>/<your_project_name>/<your_project_name>.sock \
          <your_project_name>.wsgi:application

[Install]
WantedBy=multi-user.target
```

- Reload the systemd daemon to read the new file by using the following command:

```bash
sudo systemctl daemon-reload
```

- Enable the Gunicorn service to start on boot by using the following command:

```bash
sudo systemctl enable gunicorn
```

- Start the Gunicorn service by using the following command:

```bash
sudo systemctl start gunicorn
```

- Check the status of the Gunicorn service by using the following command:

```bash
sudo systemctl status gunicorn
```

## Step 12: Configure Nginx as a reverse proxy for Gunicorn

Nginx is a high-performance web server that can act as a reverse proxy for Gunicorn and serve static files for your Django project. To configure Nginx for your Django project, follow these steps:

- Install Nginx by using the following command:

```bash
sudo apt install nginx -y
```

- Create a new Nginx configuration file for your Django project by using the following command:

```bash
sudo nano /etc/nginx/sites-available/<your_project_name>
```

- Paste the following content in the file and save it. Replace `<your_droplet_ip>` with your droplet's IP address and `<your_domain>` with your domain name.

```nginx
server {
    listen 80;
    server_name <your_droplet_ip>;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/<your_application_user>/<your_project_name>;
    }

    location /media/ {
        root /home/<your_application_user>/<your_project_name>;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/<your_application_user>/<your_project_name>/<your_project_name>.sock;
    }
}
```
- This will create a server block that will listen on port 80 and respond to requests for your Droplet's IP address. It will also serve the favicon, static, and media files from your project directory, and proxy all other requests to the Gunicorn socket file.
- To enable the server block, you need to create a symbolic link from the file you created to the `/etc/nginx/sites-enabled/` directory:

```bash
sudo ln -s /etc/nginx/sites-available/<your_project_name> /etc/nginx/sites-enabled
```

- To test the NGINX configuration for syntax errors, use the following command:

```bash
sudo nginx -t
```

- You should see something like this:

```bash
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

- To apply the changes, restart the NGINX service:

```bash
sudo systemctl restart nginx
```

- You have now configured NGINX on your Droplet.

- Open your browser and visit `http://<your_domain>` or `http://<your_droplet_ip>`. You should see your Django project's homepage served by Nginx and Gunicorn.


## Step 13: Set up your git repository on your droplet and add a post-receive hook to automate deployment

To automate the deployment process, we will use a git repository on our droplet and a post-receive hook that will run some commands whenever we push our code from our local machine. To set up your git repository on your droplet and add a post-receive hook, follow these steps:

- Change to the home directory by using the following command:

```bash
cd ~
```

- Create a new directory for your git repository by using the following command:

```bash
mkdir <your_project_name>.git
```

- Change to the git directory by using the following command:

```bash
cd <your_project_name>.git
```

- Initialize a bare git repository by using the following command:

```bash
git init --bare
```

- Create a new file called `post-receive` in the `hooks` directory by using the following command:

```bash
nano hooks/post-receive
```

- Paste the following content in the file and save it. This script will activate the virtual environment, pull the latest code from the git repository, install the dependencies, run the database migrations, collect the static files, and restart Gunicorn and Nginx.

```bash
#!/bin/bash

# Activate the virtual environment
source /home/<your_username>/<your_project_name>/env/bin/activate

# Pull the latest code from the git repository
git --work-tree=/home/<your_username>/<your_project_name> --git-dir=/home/<your_username>/<your_project_name>.git checkout -f

# Install the dependencies
pip install -r /home/<your_username>/<your_project_name>/requirements.txt

# Run the database migrations
python /home/<your_username>/<your_project_name>/manage.py migrate

# Collect the static files
python /home/<your_username>/<your_project_name>/manage.py collectstatic --noinput

# Restart Gunicorn and Nginx
sudo systemctl restart gunicorn
sudo systemctl restart nginx
```

- Make the file executable by using the following command:

```bash
chmod +x hooks/post-receive
```

## Step 14: Push your code from your local machine to your git repo and test automated deployment on droplet

Now that we have set up our git repository and our post-receive hook on our droplet, we can push our code from our local machine and test the automated deployment process. To do this, follow these steps:

- On your local machine, change to your project directory by using the following command:

```bash
cd ~/<your_project_name>
```

- Add your droplet's git repository as a remote by using the following command. Replace `<your_droplet_ip>` with your droplet's IP address.

```bash
git remote add production <your_username>@<your_droplet_ip>:<your_project_name>.git
```

- Push your code to the production remote by using the following command:

```bash
git push production main
```

- You should see something like this:

```bash
Counting objects: 10, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (10/10), done.
Writing objects: 100% (10/10), 1.23 KiB | 0 bytes/s, done.
Total 10 (delta 8), reused 0 (delta 0)
remote: Activating the virtual environment
remote: Pulling the latest code from the git repository
remote: Installing the dependencies
remote: Running the database migrations
remote: Collecting the static files
remote: Restarting Gunicorn and Nginx
To <your_username>@<your_droplet_ip>:pets-finder.git
   1234567..89abcdef  main -> main
```

- This means that the post-receive hook has run successfully and your code has been deployed to your droplet.
- Open your browser and visit `http://<your_domain>` or `http://<your_droplet_ip>` again. You should see your Django project's homepage with the latest changes.

## Conclusion

Congratulations! You have successfully deployed your Django + MySQL + Nginix app to a Digital Ocean droplet. You have also learned how to use a git repository and a post-receive hook to automate the deployment process. You can now update your code on your local machine and push it to your droplet with ease. You have also followed some best practices to secure your droplet and optimize your application performance. You can now enjoy your Django web application and share it with the world. 🎉