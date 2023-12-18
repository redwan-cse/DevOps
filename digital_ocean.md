A comprehensive guide for deploying a Django + MySQL app to a Digital Ocean Droplet with Ubuntu. Please note that some steps might need adjustments based on your specific project requirements.

### 1. Setup a Digital Ocean Droplet
#### a. Create a non-root user with sudo access
```bash
adduser your_username
usermod -aG sudo your_username
```

#### b. Disable root login
Edit the SSH configuration file:
```bash
sudo nano /etc/ssh/sshd_config
```
Set `PermitRootLogin` to `no`. Save and exit, then restart SSH:
```bash
sudo systemctl restart ssh
```

#### c. Enable SSH port on the firewall
```bash
sudo ufw allow 22/tcp
```

#### d. Enable UFW (firewall)
```bash
sudo ufw enable
```

#### e. Enable password-less SSH authorized key login
On your local machine, generate an SSH key pair if you haven't:
```bash
ssh-keygen -t rsa
```
Copy the public key to your Droplet:
```bash
ssh-copy-id your_username@your_droplet_ip
```

#### f. Packages update/upgrade
```bash
sudo apt update
sudo apt upgrade
```

### 2. Install MySQL and run configuration script
```bash
sudo apt install mysql-server
sudo mysql_secure_installation
```

### 3. Create a dedicated MySQL user
```bash
sudo mysql
```
```sql
CREATE USER 'your_mysql_username'@'localhost' IDENTIFIED BY 'your_password';
```

### 4. Grant necessary database privileges to the user
```sql
GRANT ALL PRIVILEGES ON *.* TO 'your_mysql_username'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### 5. Create a new database for your application
```bash
sudo mysql -u your_mysql_username -p
```
```sql
CREATE DATABASE your_database_name CHARACTER SET UTF8;
EXIT;
```

### 6. Install NGINX and Supervisor
```bash
sudo apt install nginx supervisor
sudo systemctl enable supervisor
```

### 7. Setup a virtual environment on your Droplet
```bash
sudo apt install python3-venv
mkdir ~/your_project
cd ~/your_project
python3 -m venv venv
```

### 8. Create and configure an application user
```bash
sudo useradd -s /bin/bash -m your_app_user
sudo passwd your_app_user
sudo usermod -aG sudo your_app_user
```

### 9. Configure the Python virtual environment and clone your project repo
```bash
su - your_app_user
cd ~
source ~/your_project/venv/bin/activate
git clone https://github.com/redwan-cse/pets-finder.git
```

### 10. Install Django project dependencies
```bash
cd pets-finder
pip install -r requirements.txt
```

### 11. Set the proper database connection credentials
Edit `settings.py` in your Django project and update the database settings.

### 12. Test everything
```bash
python manage.py migrate
python manage.py runserver 0.0.0.0:8000
```
Visit `http://your_droplet_ip:8000` in your browser.

### 13. Install and configure Gunicorn
```bash
pip install gunicorn
gunicorn your_project.wsgi:application --bind 0.0.0.0:8000
```

### 14. Configure Supervisor
Create a new Supervisor configuration file:
```bash
sudo nano /etc/supervisor/conf.d/your_project.conf
```
```ini
[program:your_project]
command=/path/to/venv/bin/gunicorn your_project.wsgi:application --bind 0.0.0.0:8000
directory=/path/to/your_project
autostart=true
autorestart=true
stderr_logfile=/var/log/your_project.err.log
stdout_logfile=/var/log/your_project.out.log
```

### 15. Configure NGINX
Create a new NGINX site configuration:
```bash
sudo nano /etc/nginx/sites-available/your_project
```
```nginx
server {
    listen 80;
    server_name your_domain_or_ip;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /path/to/your_project;
    }

    location / {
        include proxy_params;
        proxy_pass http://127.0.0.1:8000;
    }
}
```
Enable the site:
```bash
sudo ln -s /etc/nginx/sites-available/your_project /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl restart nginx
```

### 16. Updating the application
```bash
su - your_app_user
cd ~/your_project
git pull origin master
pip install -r requirements.txt
python manage.py migrate
sudo supervisorctl restart your_project
sudo systemctl restart nginx
```

Your Django + MySQL app should now be successfully deployed on Digital Ocean! Adjust the paths, names, and configurations according to your specific setup.


Certainly! Combining the deployment automation and webhook setup for secure and production-level configuration involves integrating the deployment script with a webhook endpoint protected by a reverse proxy (NGINX) and ensuring secure communication between GitHub and your server. Let's go step by step:

### 1. Prepare your VPS for automated deployment

#### a. Create a deployment script

Create a script on your VPS that pulls the latest changes from the GitHub repository, installs dependencies, and restarts the necessary services. For example, create a file named `deploy.sh`:

```bash
#!/bin/bash

cd /path/to/your_project
git pull origin main
source /path/to/venv/bin/activate
pip install -r requirements.txt
python manage.py migrate
sudo supervisorctl restart your_project
sudo systemctl restart nginx
```

Make the script executable:

```bash
chmod +x deploy.sh
```

#### b. Grant sudo permissions

If your deployment script requires `sudo` privileges (e.g., restarting services), allow your user to run the script without entering a password by adding the following line to the sudoers file:

```bash
sudo visudo
```

Add the following line at the end (replace `your_user` and `/path/to/deploy.sh` with your actual user and script path):

```text
your_user ALL=(ALL) NOPASSWD: /path/to/deploy.sh
```

### 2. Set up a web server and webhook for automated deployment

#### a. Install Flask (for webhook)

```bash
pip install flask
```

#### b. Create a Flask app (for webhook)

Create a file named `webhook_receiver.py`:

```python
from flask import Flask, request
import subprocess

app = Flask(__name__)

@app.route('/webhook', methods=['POST'])
def webhook():
    # Verify GitHub webhook secret here if used

    subprocess.run(['/path/to/deploy.sh'])
    return 'Update triggered successfully\n'

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=your_webhook_port)
```

Replace `/path/to/deploy.sh` with the actual path to your deployment script, and set `your_webhook_port` to a port of your choice.

### 3. Configure NGINX for the webhook

#### a. Create an NGINX configuration file for the webhook

```bash
sudo nano /etc/nginx/sites-available/webhook
```

Add the following configuration (replace placeholders):

```nginx
server {
    listen 80;
    server_name your_domain_or_ip;

    location / {
        proxy_pass http://127.0.0.1:your_webhook_port;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

#### b. Enable the site and restart NGINX

```bash
sudo ln -s /etc/nginx/sites-available/webhook /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl restart nginx
```

### 4. Set up a GitHub webhook

#### a. Open the GitHub repository settings

- Go to your GitHub repository.
- Navigate to "Settings" > "Webhooks" > "Add webhook."

#### b. Configure the webhook

- Payload URL: Set this to the endpoint on your VPS where NGINX forwards requests (e.g., `http://your_domain_or_ip/webhook`).
- Content type: Choose "application/json."
- Secret: Optionally, add a secret for additional security.
- Which events would you like to trigger this webhook? Choose "Just the push event."

#### c. Add the webhook

Click "Add webhook" to save the configuration.

### 5. Test the automated deployment

Push a change to the main branch on GitHub and check if the deployment script is triggered on your VPS.

Now, every time you push to the main branch on GitHub, the webhook will notify your VPS, and the deployment script will automatically update your application. This setup includes a reverse proxy (NGINX) for security and production-level configurations. Adjust paths and configurations as needed for your specific setup.