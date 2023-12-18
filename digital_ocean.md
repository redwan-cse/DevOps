Hello, this is Bing. I can help you write a full guide to deploying a Django + MySQL app to a Digital Ocean Droplet [Ubuntu]. I have searched the web for some relevant resources and I have used them to generate the guide based on your instructions. Here is the guide I created:

# Deploying a Django + MySQL App to a Digital Ocean Droplet [Ubuntu]

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
- To create a new MySQL user, use the `CREATE USER` statement and provide a username and a password of your choice:

```bash
CREATE USER '<your_mysql_username>'@'localhost' IDENTIFIED BY '<your_mysql_password>';
```

- You have now created a new MySQL user.

## 10. Grant necessary database privileges to the user you just created

- The new MySQL user needs to have the appropriate permissions to access and modify the database that your Django application will use. To grant the necessary privileges, use the `GRANT` statement and provide the database name and the username:

```bash
GRANT ALL ON <your_database_name>.* TO '<your_mysql_username>'@'localhost';
```

- This will grant the user all privileges on the database and its tables. You can also specify more granular permissions, such as `SELECT`, `INSERT`, `UPDATE`, `DELETE`, etc. For more information, see the [MySQL documentation](^1^) on privileges.
- To apply the changes, use the `FLUSH` statement:

```bash
FLUSH PRIVILEGES;
```

- You have now granted the necessary database privileges to the user you just created.

## 11. Create a new database for your application

- The next step is to create a new database that your Django application will use to store and manage data. To create a new database, use the `CREATE DATABASE` statement and provide a name for the database:

```bash
CREATE DATABASE <your_database_name>;
```

- You can use any name you want, but make sure it is descriptive and unique. For example, you can use the name of your project or application.
- You have now created a new database for your application.

## 12. Install NGINX and Supervisor

- NGINX is a high-performance web server that can serve static and media files, as well as proxy requests to your Django application. Supervisor is a process control system that can monitor and manage your Django application and ensure that it is always running.
- To install NGINX and Supervisor on your Droplet, you can use the `apt` package manager:

```bash
sudo apt install nginx supervisor
```

- You may be asked to confirm the installation. Press **Enter** to continue.
- You have now installed NGINX and Supervisor on your Droplet.

## 13. Setup a virtual environment on your Droplet to manage requirements and packages

- A virtual environment is a tool that allows you to create an isolated Python environment for your project, where you can install and manage the packages and dependencies that your project requires, without affecting the system-wide Python installation.
- To create a virtual environment on your Droplet, you can use the `virtualenv` command that you installed earlier. First, create a directory where you will store your project files and move into it:

```bash
mkdir ~/<your_project_name>
cd ~/<your_project_name>
```

- Then, create a virtual environment with the same name as your project:

```bash
virtualenv <your_project_name>
```

- This will create a directory with the same name as your project, where a local version of Python and pip will be installed. You can use this virtual environment to install and configure the Python packages that your project needs.
- To activate the virtual environment, use the `source` command and provide the path to the `activate` script:

```bash
source <your_project_name>/bin/activate
```

- Your prompt should change to indicate that you are now operating within a Python virtual environment. It will look something like this:

```bash
(<your_project_name>)user@host:~/<your_project_name>$
```

- With your virtual environment active, you can install the Python packages that your project requires. You can use the `pip` command and provide the path to the `requirements.txt` file that you created on your local machine:

```bash
pip install -r requirements.txt
```

- This will install all the packages and their dependencies that are listed in the `requirements.txt` file, such as Django, Gunicorn, dj-database-url, and psycopg2.
- You have now set up a virtual environment on your Droplet and installed the Python packages that your project needs.

## 14. Create and configure an application user for your Django application

- It is not recommended to run your Django application as the nonroot user that you created earlier, as it can pose security risks and grant too much access to your system. Therefore, it is better to create a dedicated application user that will only run your Django application and have limited permissions.
- To create a new application user, use the `adduser` command and provide a username. You can use the same name as your project or application:

```bash
sudo adduser <your_application_user>
```

- You will be asked to set a password and provide some optional information for the user. You can leave the fields blank by pressing **Enter** or fill them as you wish.
- Next, you need to add the new user to the `www-data` group, which is the default group for web servers on Ubuntu. This will allow the user to access the web server files and directories. You can do this by using the `usermod` command:

```bash
sudo usermod -aG www-data <your_application_user>
```

- You have now created a new application user for your Django application.

## 15. Configure the Python virtual environment and clone your project repo

- The next step is to configure the Python virtual environment that you created earlier and clone your project repo from GitHub to your Droplet. To do this, you need to log in to your Droplet as the application user that you created in the previous step. You can do this by using the `su` command and providing the username:

```bash
su - <your_application_user>
```

- You will be asked to enter the password that you set for the application user. After that, you should see a prompt like this:

```bash
<your_application_user>@<your_droplet_name>:~$
```

- You are now logged in to your Droplet as the application user.
- To configure the Python virtual environment, you need to copy the virtual environment directory that you created earlier to the home directory of the application user. You can do this by using the `cp` command and providing the source and destination paths:

```bash
cp -r /home/<your_username>/<your_project_name> ~/
```

- This will copy the entire virtual environment directory to the home directory of the application user. You can verify that the copy was successful by listing the contents of the home directory:

```bash
ls -l
```

- You should see a directory with the same name as your project, which contains the virtual environment files and directories.
- To clone your project repo from GitHub, you need to have git installed on your Droplet. You can install git by using the `apt` package manager:

```bash
sudo apt install git
```

- You may be asked to confirm the installation. Press **Enter** to continue.
- After the installation is complete, you need to clone your project repo to the home directory of the application user. You can do this by using the `git` command and providing the URL of your repo:

```bash
git clone https://github.com/<your_github_username>/<your_project_name>.git
```

- This will clone your project repo to a directory with the same name as your project. You can verify that the clone was successful by listing the contents of the home directory:

```bash
ls -l
```

- You should see a directory with the same name as your project, which contains your project files and directories.
- You have now configured the Python virtual environment and cloned your project repo to your Droplet.

## 16. Install your Django project's dependencies

- The next step is to install the Django project's dependencies that are not included in the `requirements.txt` file, such as the static and media files, the migrations, and the collectstatic command. To do this, you need to activate the virtual environment that you copied earlier and move into the project directory. You can do this by typing:

```bash
source <your_project_name>/bin/activate
cd <your_project_name>
```

- Your prompt should change to indicate that you are now operating within a Python virtual environment and inside the project directory. It will look something like this:

```bash
(<your_project_name>)<your_application_user>@<your_droplet_name>:~/<your_project_name>$
```

- With your virtual environment active and inside the project directory, you can install the Django project's dependencies by using the `python` command and providing the path to the `manage.py` file:

```bash
python manage.py migrate
python manage.py collectstatic
```

- The `migrate` command will create the necessary tables and indexes in the database that your Django application will use. The `collectstatic` command will collect all the static files that your Django application needs, such as CSS, JavaScript, and images, and copy them to a directory that the web server can access.
- You have now installed your Django project's dependencies on your Droplet.

## 17. Set the proper database connection credentials in your Django project's settings.py file and add your IP address to allowed hosts

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

## 18. Install and configure Gunicorn

- Gunicorn is a Python WSGI (Web Server Gateway Interface) server that can run your Django application and handle requests from the web server. To install Gunicorn on your Droplet, you can use the `pip` command within your virtual environment:

```bash
pip install gunicorn
```

- To configure Gunicorn, you need to create a service file that will tell Supervisor how to run and manage your Django application. You can do this by using the `nano` editor and creating a file called `gunicorn.service` in the `/etc/systemd/system/` directory:

```bash
sudo nano /etc/systemd/system/gunicorn.service
```

- Paste the following content into the file, replacing the placeholders with the actual values that you used for your project:

```ini
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=<your_application_user>
Group=www-data
WorkingDirectory=/home/<your_application_user>/<your_project_name>
Environment="PATH=/home/<your_application_user>/<your_project_name>/<your_project_name>/bin"
ExecStart=/home/<your_application_user>/<your_project_name>/<your_project_name>/bin/gunicorn --access-logfile - --workers 3 --bind unix:/home/<your_application_user>/<your_project_name>/<your_project_name>.sock <your_project_name>.wsgi:application

[Install]
WantedBy=multi-user.target
```

- This will create a systemd service that will run Gunicorn as the application user and the www-data group, with the working directory set to your project directory and the environment variable PATH set to your virtual environment. It will also specify the number of workers, the socket file, and the WSGI module for your Django application.
- Save and close the file by pressing **Ctrl+X**, then **Y**, then **Enter**.
- To start and enable the Gunicorn service, use the following commands:

```bash
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
```

- You can check the status of the service by typing:

```bash
sudo systemctl status gunicorn
```

- You should see something like this:

```bash
● gunicorn.service - gunicorn daemon
     Loaded: loaded (/etc/systemd/system/gunicorn.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2021-10-11 12:34:56 UTC; 1min 23s ago
   Main PID: 1234 (gunicorn)
      Tasks: 4 (limit: 1137)
     Memory: 34.5M
     CGroup: /system.slice/gunicorn.service
             ├─1234 /home/<your_application_user>/<your_project_name>/<your_project_name>/bin/python3 /home/<your_application_user>/<your_project_name>/<your_project_name>/bin/gunicorn --access-logfile - --workers 3 --bind unix:/home/<your_application_user>/<your_project_name>/<your_project_name>.sock <your_project_name>.wsgi:application
             ├─1235 /home/<your_application_user>/<your_project_name>/<your_project_name>/bin/python3 /home/<your_application_user>/<your_project_name>/<your_project_name>/bin/gunicorn --access-logfile - --workers 3 --bind unix:/home/<your_application_user>/<your_project_name>/<your_project_name>.sock <your_project_name>.wsgi:application
             ├─1236 /home/<your_application_user>/<your_project_name>/<your_project_name>/bin/python3 /home/<your_application_user>/<your_project_name>/<your_project_name>/bin/gunicorn --access-logfile - --workers 3 --bind unix:/home/<your_application_user>/<your_project_name>/<your_project_name>.sock <your_project_name>.wsgi:application
             └─1237 /home/<your_application_user>/<your_project_name>/<your_project_name>/bin/python3 /home/<your_application_user>/<your_project_name>/<your_project_name>/bin/gunicorn --access-logfile - --workers 3 --bind unix:/home/<your_application_user>/<your_project_name>/<your_project_name>.sock <your_project_name>.wsgi:application
```

- This means that the Gunicorn service is running and enabled on your Droplet.
- You have now installed and configured Gunicorn on your Droplet.

## 19. Configure NGINX

- NGINX is a web server that can serve static and media files, as well as proxy requests to your Django application. To configure NGINX, you need to create a server block file that will tell NGINX how to handle requests for your domain or IP address. You can do this by using the `nano` editor and creating a file called `<your_project_name>` in the `/etc/nginx/sites-available/` directory:

```bash
sudo nano /etc/nginx/sites-available/<your_project_name>
```

- Paste the following content into the file, replacing the placeholders with the actual values that you used for your project:

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
- Save and close the file by pressing **Ctrl+X**, then **Y**, then **Enter**.
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

## 20. Updating the application

- To update your Django application on your Droplet, you need to pull the latest changes from your GitHub repo and restart the Gunicorn and NGINX services. You can do this by following these steps:

- Log in to your Droplet as the application user and activate the virtual environment:

```bash
su - <your_application_user>
source <your_project_name>/bin/activate
```

- Move into the project directory and pull the latest changes from GitHub:

```bash
cd <your_project_name>
git pull
```

- Install any new dependencies that may have been added to the `requirements.txt` file:

```bash
pip install -r requirements.txt
```

- Run the migrations and collect the static files:

```bash
python manage.py migrate
python manage.py collectstatic
```

- Log out of the application user and restart the Gunicorn and NGINX services:

```bash
exit
sudo systemctl restart gunicorn
sudo systemctl restart nginx
```

- You have now updated your Django application on your Droplet.

## Conclusion

You have successfully deployed a Django + MySQL app to a Digital Ocean Droplet using Ubuntu 20.04. You have also learned how to configure and manage the database, the web server, and the application server. You have also learned how to update your application with the latest changes from GitHub. You can now access your application by visiting your Droplet's IP address in your browser. You can also use a domain name instead of an IP address, if you have one configured for your Droplet. For more information on how to do that, you can follow this [guide](^3^) on how to set up a domain name with DigitalOcean. I hope you found this guide helpful and informative. Thank you for using Bing. 😊

## Source

- https://docs.digitalocean.com/developer-center/deploy-a-django-app-on-app-platform/.
- https://medium.com/@bernardo.laing/how-to-deploy-a-django-2-1-1-web-application-with-mysql-on-digitalocean-e2d22f59264b.
- https://medium.com/@bernardo.laing/how-to-deploy-a-django-2-1-1-web-application-with-mysql-on-digitalocean-e2d22f59264b.
- https://www.chadams.me/blog/2021-06-08-django-on-digital-ocean-droplet/.
- https://blog.tati.digital/2021/02/22/deploy-django-app-digitalocean-ubuntu-20-04-server/.
- https://testdriven.io/blog/django-dokku/.
- https://www.digitalocean.com/community/questions/web-hosting-a-django-web-site-in-a-droplet.
- https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04.