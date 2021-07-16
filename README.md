# Django and React Deployment with Digitalocean Ubuntu Server

## First create ssh key on your local machine

### This is so that later you don't need a password to log into your server

To generate an ssh key, first navigate into your .ssh folder in your home directory:

    cd ~/.ssh

Once in here, run the following command to generate an ssh key:

    ssh-keygen

Once you run this, enter the following for the name:

    id_rsa_do

-   The do part will signify this is for digitalocean.
-   By not having the defalt name, it will make it much easier to manage multiple ssh keys on your machine.
-   After this set you will set a password for the ssh key, make sure to remember this password.
-   It's quite common to use ssh keys with GitHub as well so that you also don't need to enter a password every time you push or pull to GitHub.
-   Once this command finished, you will have two files created:

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

-   Then you want to copy the output
-   Back in digitalocean, you'll have authentication options
-   Click **New SSH Key**, then paste in your ssh key you copied from **id_rsa_do.pub**, make sure you didn't copy from **id_rsa_do**
-   Then the last thing to do is set the hostname, add backups if you want, and then you're done
-   You can also later change the hostname in your server if you mess this up with: sudo hostnamectl set-hostname your-new-hostname
-   You can also edit the droplet name easily if you mess this up as well

Now that your server is ready, now we have to log into it

### Log into your server

Copy your server's IP Address, open a terminal, and run the following:

    ssh root@SERVER_IP

The first time you do this, your ssh key won't work and you'll have to run the following:

    ssh-add ~/.ssh/id_rsa_do

You'll then enter the password, and after that you should be able to log into your digitalocean server

Next we want to create a new user

### Creating a user on the server

To create a user, run the following command:

    adduser your-username

In my case I use **bryan** as my username, you use whatever you want, and you'll set a password for this user.

Next you want to give this user root permissions, so run the following:

    usermod -aG sudo your-username

Next you want to grab your ssh key, to do this do one of the following:

-   In your local terminal run: cat ~/.ssh/id_rsa_do.pub
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


