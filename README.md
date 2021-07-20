# Django and React Deployment with DigitalOcean

### First create ssh key on your local machine

This is so that later you don't need a password to log into your server

To generate an ssh key, first navigate into your .ssh folder in your home directory:

    cd ~/.ssh

Once in here, run the following command to generate an ssh key:

    ssh-keygen

Once you run this, enter the following for the name:

    id_rsa_do

-   The **do** part will signify this is for digitalocean.
-   By not having the defalt name, it will make it much easier to manage multiple ssh keys on your machine.
-   After this you will set a password for the ssh key, make sure to remember this password.
-   It's quite common to use ssh keys with GitHub as well so that you also don't need to enter a password every time you push or pull to GitHub, so by not having the default **id_rsa** name it will make it much easier to manage having GitHub ssh keys as well on your machine.

Once this command finished, you will have two files created:

    id_rsa_do
    id_rsa_do.pub

Both of these files will sit inside of ~/.ssh

Now that we have an ssh key ready, the next step is to setup a droplet on digitalocean.

-   Simply create a digitalocean account
-   Create a new project where you'll hold this private server (droplet)
-   Click **Create**, then **Droplets**
-   Select Ubuntu for the distribution
-   Here you can select a plan, you can select **Basic**
-   Then you have CPU options, you can select **Regular Intel with SSH**, then do the **$5/mo** option
-   If you have an app in production, you should get a better server to host the app
-   Then you'll select a datacenter region

### After this the ssh key we generated will come into play

Run the following command on your local machine:

    cat ~/.ssh/id_rsa_do.pub

-   Then you want to copy the output so you can use it later
-   Back in digitalocean, you'll have authentication options
-   Click **New SSH Key**, then paste in your ssh key you copied from **id_rsa_do.pub**, make sure you didn't copy from **id_rsa_do**
-   Then the last thing to do is set the hostname, add backups if you want, and then you're done
-   You can also later change the hostname in your server if you mess this up with: sudo hostnamectl set-hostname your-new-hostname
-   You can also edit the droplet name easily if you mess this up as well

Now that your server is ready, we have to log into it

### Log into your server

Copy your server's IP Address, open a terminal, and run the following:

    ssh root@SERVER_IP

If your local machine deny's you access, run the following:

    ssh-add ~/.ssh/id_rsa_do

You'll then enter the password, and after that you should be able to log into your digitalocean server

Next we want to create a new user

### Creating a user on the server

To create a user, run the following command:

    adduser your-username

In my case I use **bryan** as my username, you use whatever you want, and you'll set a password for this user.

Next you want to give this user root permissions, so run the following:

    usermod -aG sudo your-username

Next you want to grab your ssh key, if you don't have it copied, then do one of the following:

-   Go in your local terminal and copy the output from running:

    cat ~/.ssh/id_rsa_do.pub

-   Or go into digitalocean, then settings, then security, and from there you'll see your ssh keys where you can click more, then edit to see the ssh key and copy it

Once you've copied your ssh key, go back into your digitalocean server where you're logged in as the root user.

### Add ssh keys for user created on digitalocean server

As the root user, do the following to set up ssh keys on the server:

    cd /home/your-username
    mkdir .ssh
    cd .ssh
    nano authorized_keys

From there you want to do the following:

-   Paste your ssh key into this file
-   Press **ctrl-x**
-   Then press **y** to save the changes
-   Then press **enter** to exit the editor

In this authorized_keys file you can add multiple ssh keys by just having them on separate lines.

So if you have another machine you want to have access to your digitalocean server, you can just add in that ssh key in your digitalocean settings, add it into you authorized_keys, and then you're good to go.

Now run **exit** in the terminal to exit the server session

Then we want to log in with our newly created user:

### Log in with new user

    ssh your-username@SERVER_IP

Then from there you want to disable logging in with the root user by doing the following:

    sudo nano /etc/ssh/sshd_config

Then make sure the following settings are like this:

    PermitRootLogin no
    PasswordAuthentication no

This will make it so we need to use an ssh key to log in and that we can't log in with the root user

Then you want to reload the sshd service by running the following:

    sudo systemctl reload sshd

### Firewall

To view which apps have a firewall you can run:

    sudo ufw app list

You want to make sure you allow OpenSSH, so run the following

    sudo ufw allow OpenSSH

Then you can enable the firewall with:

    sudo ufw enable

Then to check the status you can run:

    sudo ufw status

### Software Setup

Next we want to make sure we have the software we need on our server

First update the packages on the system with the following:

    sudo apt update
    sudo apt upgrade

After this install Python3, PostgreSQL, and NGINX by running the following:

    sudo apt install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx curl

### Setup Database

Next we want to setup our PostgreSQL database

First we need to login with the following:

    sudo -u postgres psql

Next we want to create our database:

    CREATE DATABASE database_name;

Then we create a user:

    CREATE USER your-user WITH PASSWORD '[YOUR PASSWORD]';

Next we want to set a default encoding and transaction isolation scheme:

    ALTER ROLE your-user SET client_encoding TO 'utf8';
    ALTER ROLE your-user SET default_transaction_isolation TO 'read committed';
    ALTER ROLE your-user SET timezone TO 'UTC';

Then we want to give our user access to the database we made:

    GRANT ALL PRIVILEGES ON DATABASE database_name TO your-user;

Then from there we're done with the PostgreSQL setup and can quit out of it:

    \q

### Getting Project Onto Server

First we want to create a folder to hold python apps have on the server and create a virtual environment.

    sudo apt install python3-venv
    python3 -m venv venv
    source venv/bin/activate

Now we want to have our project on our local machine ready for production and on a GitHub repository

-   we want to have the pip dependencies of our project in a requirements.txt file to make it easy to instlal the depencendies on our server
-   you can check the dependencies of your project with: pip freeze
-   you can also bring in all those dependencies into your requirements.txt with: pip freeze > requirements.txt
-   though you can also just grab the ones you want and place them in that file

In your settings.py file, at the bottom add:

    try:
        from .local_settings import *
    except ImportError:
        pass

Then in your django project folder, you can create a local_settings.py file where you can store important info

This local_settings.py file I'd place into the .gitignore so it doesn't get pushed into the reporitory

You can have settings in here you'll use when testing things in your local machine, and on your server you can later create this and place the appropriate production settings you want.

Settings I place in the local_settings.py file are the following:

-   SECRET_KEY
-   ALLOWED_HOSTS
-   DEBUG
-   DATABASES
-   EMAIL

Then any Braintree settings, Stripe settings, PayPal settings, Google settings, Facebook settings I'd place in here:

-   BT_ENVIRONMENT
-   BT_MERCHANT_ID
-   BT_PUBLIC_KEY
-   BT_PRIVATE_KEY
-   STRIPE_PUBLIC_KEY
-   STRIPE_SECRET_KEY
-   PAYPAL_CLIENT_ID
-   PAYPAL_SECRET_ID
-   SOCIAL_AUTH_GOOGLE_OAUTH2_KEY
-   SOCIAL_AUTH_GOOGLE_OAUTH2_SECRET
-   SOCIAL_AUTH_GOOGLE_OAUTH2_SCOPE
-   SOCIAL_AUTH_GOOGLE_OAUTH2_EXTRA_DATA
-   SOCIAL_AUTH_FACEBOOK_KEY
-   SOCIAL_AUTH_FACEBOOK_SECRET
-   SOCIAL_AUTH_FACEBOOK_SCOPE
-   SOCIAL_AUTH_FACEBOOK_PROFILE_EXTRA_PARAMS

So anything you want to keep secure you'd place inside of the local_settings.py file.

### Whitenoise

We also want to make sure we setup whitenoise in our local project, this will make it so that static files work properly on our server once we set **DEBUG** to **False**

To setup whitenoise, in your project on your local machine run the following with your virtual environment activated:

    pip install whitenoise

Then make sure whitenoise is added to your requirements.txt

Then add the following in your INSTALLED_APPS:

    INSTALLED_APPS = [
        ...
        'whitenoise.runserver_nostatic',
        # 'django.contrib.staticfiles',
        ...
    ]

Then add the following in your MIDDLEWARE:

    MIDDLEWARE = [
        ...
        # 'django.middleware.security.SecurityMiddleware',
        'whitenoise.middleware.WhiteNoiseMiddleware',
        ...
    ]

Then add the following somewhere in your settings.py:

    STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'

That's all there is to it, from there on your local machine (not on your server) you will collect your static files, though first we need to setup our frontend production build folder

Make sure you have your apps folder structure ready for production, and your build folder ready to go.

For the build folder you want to make sure that you go into your .env file and have:

    REACT_APP_API_URL = '[YOUR_DOMAIN]'

Or in the place of YOUR_DOMAIN you can also have your server's ip address, though for a real application definitely have your domain here

Then you will build your project:

    npm run build

After this we will collect the static files to create our static folder again in our local machine (not on your server):

    python manage.py collectstatic

This will create a static folder which you push up into your server along with your project.

Next we get our production application onto GitHub, then from there we can clone our project on our server.

### Back To The Server

Now we will clone our project on our digitalocean server (make sure you're in your home directory):

    git clone https://github.com/your-github-name/your-repository.git

Or if you want to clone a specific branch you can do:

    git clone --single-branch --branch [branch-name] https://github.com/your-github-name/your-repository.git

Then you will navigate into the project folder that was just created

    cd github-project

From there while in your virtual environment, install the pip dependencies:

    pip install -r requirements.txt

Then navigate into your django project folder and create a local_settings.py file:

    cd django-project-folder
    nano local_settings.py

From there you will add the settings you need into this file.

Next we want to migrate things to our database and create a superuser, so run the following in your project folder:

    python manage.py makemigrations
    python manage.py migrate
    python manage.py createsuperuser

### Gunicorn

Next we need something to run our django app, for this we will use gunicorn.

To install it run:

    pip install gunicorn

Next deactivate your virtual environment:

    deactivate

Then open the gunicorn.socket file:

    sudo nano /etc/systemd/system/gunicorn.socket

Then place the following inside of that file:

    [Unit]
    Description=gunicorn socket

    [Socket]
    ListenStream=/run/gunicorn.sock

    [Install]
    WantedBy=sockets.target

Then open the gunicorn.service file:

    sudo nano /etc/systemd/system/gunicorn.service

Then place the following inside of that file:

    [Unit]
    Description=gunicorn daemon
    Requires=gunicorn.socket
    After=network.target

    [Service]
    User=[your-username]
    Group=www-data
    WorkingDirectory=/home/[your-username]/[your-github-project-folder]
    ExecStart=/home/[your-username]/venv/bin/gunicorn \
            --access-logfile - \
            --workers 3 \
            --bind unix:/run/gunicorn.sock \
            [your-django-project-name].wsgi:application

    [Install]
    WantedBy=multi-user.target

Next you want to start and enable the gunicorn socket:

    sudo systemctl start gunicorn.socket
    sudo systemctl enable gunicorn.socket

You can check the status of this socket with the following:

    sudo systemctl status gunicorn.socket

You can also check the existence of gunicorn.sock with the following:

    file /run/gunicorn.sock

### NGINX

First we want to create a project folder:

    sudo nano /etc/nginx/sites-available/[your-project-folder-name]

Then inside of there you want to add the following:

    server {
        listen 80;
        server_name YOUR_DOMAIN;

        location = /favicon.ico { access_log off; log_not_found off; }
        location /static/ {
            root /home/[your-username]/[your-github-project-folder];
        }
        
        location /media/ {
            root /home/[your-username]/[your-github-project-folder];    
        }

        location / {
            include proxy_params;
            proxy_pass http://unix:/run/gunicorn.sock;
        }
    }

Then here **YOUR_DOMAIN** can also be your server ip address, though in a real production app I'd recommend having your domain here.

Next we need to enable this file by linking to the sites-enabled directory:

    sudo ln -s /etc/nginx/sites-available/[your-project-folder-name] /etc/nginx/sites-enabled

Then we can test that everything is in order:

    sudo nginx -t

Then we can restart NGINX:

    sudo systemctl restart nginx

Next we want to have our firewall allow NGINX:

    sudo ufw allow 'Nginx Full'

Then we want to edit nginx.conf so we don't have problems with uploading images:

    sudo nano /etc/nginx/nginx.conf

Then add this into the http section:

    client_max_body_size 20M;

Next we want to restart NGINX:

    sudo systemctl restart nginx

### Domain

To setup your domain you want to have:

-   an A Record that points to your digitalocean server ip address
-   a CNAME record that points www to your host (@)

Then in your local_settings.py file on your server you want the following:

    ALLOWED_HOSTS = ['your-domain.com', 'www.your-domain.com']

You can also have your ip address in the allowed hosts, though in a real production app I'd suggest just having the domain.

Then we want to go back into our NGINX project folder we created and add in the domain:

    sudo nano /etc/nginx/sites-available/[your-project-folder-name]

Then you can add in the domain:

    server {
        ...
        server_name your-domain.com www.your-domain.com;
        ...
    }

Next we will restart NGINX and Gunicorn:

    sudo systemctl restart nginx
    sudo systemctl restart gunicorn

Now our app is hosted and ready to go, however we will need an SSL certificate.

### SSL Certificate Setup

Run the following:
    
    cd ~/
    sudo apt install certbot python3-certbot-nginx
    sudo service nginx stop
    sudo certbot --nginx -d your-domain.com -d www.your-domain.com

The reason for having an ssl certificate for www.your-domain.com as well even though we made a CNAME record is that on some devices when someone goes to www.your-domain.com it won't re-route them to your-domain.com like you'd expect. Most devices will, but I found some don't so we can add this ssl certificate to ensure that we don't have issues.

Then run:

    sudo fuser -k 80/tcp
    sudo systemctl restart nginx

From there you're good to go and have SSL certificates setup...well sort of...

You will have an SSL certificate, but it will expire after 90 days and you will have to renew it.

So when it's almost time to renew the SSL certificate you'd run the following:

    sudo service nginx stop
    sudo certbot renew
    sudo systemctl restart nginx

Then your SSL certificate would get renewed.

However, renewing these SSL certificates every 90 days can be annoying, and if you have multiple servers where you host websites, this can be even more annoying.

Thankfully we can have our server automate this process for us so we don't have to worry about renewing these SSL certificates manually every 90 days.

### Crontabs

We're going to setup a crontab in order to automatically renew these SSL certificates for us

We want to setup a crontab that will run as root, so to create a crontab you will run the following:

    sudo crontab -e

Make sure to do this with sudo, if you don't it will make a crontab for your user and not the root user

Then in that file at the bottom, add the following:

    SHELL=/bin/bash
    PATH=/sbin:/bin:/usr/sbin:/usr/local/bin:/usr/bin:/usr/sbin/nginx:/usr/bin/certbot:/bin/systemctl
    0 6 * * 0 service nginx stop && certbot renew -n -q && systemctl restart nginx

What this will do is every Sunday at 6:00am, it will try to renew your SSL certificates

You can also adjust this so it runs less frequently, you can go to [Crontab Guru](https://crontab.guru/) to see how this works

Then also note the time these things renew will be the time on your server

You can check the current time of your server with the following:

    timedatectl

This will eliminate confusion potentially if you're testing a crontab as the server time might be different that your current time.

Then once you make your changes to your crontab, run the following:

    sudo service cron restart

And that's it, not you have fully deployment your Django and React application, have an SSL certificate, and don't have to worry about manually renewing it!
