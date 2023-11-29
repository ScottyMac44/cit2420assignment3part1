
# How to Serve a Sample Website Using NGINX

This tutorial will guide you through making a sample website using NGINX, on a fresh Debian 12 server running on DigitalOcean.
***

## Table of Contents

1. Create a new regular user
    1. Give the new user administrative permissions
    2. Set bash as the new user's login shell
    3. Allow the new user to access the server via SSH
2. Prevent the root user from connecting to the server via SSH
3. Install NGINX
4. Configure NGINX to serve a sample website
***

## 1. Create a New Regular User

Assuming you have already created your droplet, and connected to it, we'll begin by creating a new regular user. It's important to only ever log in as a regular user for security reasons, as stated by DigitalOcean:

> Good security practices recommend that you disable the root login over SSH to prevent unauthorized access to your Linux-based machine by any other user. Disabling root login prevents root access over SSH to your Linux-based machine, which means that no one will have unlimited privileges.

Your new user will have *almost* unlimited privileges, which helps to minimize damage if someone does gain access to your machine
To create a new user, enter the following command in your terminal (make sure you replace *user_name* with the name you want to use):
```bash
    adduser user_name
```

Next, you will be prompted to input a password for your new user (Note: What you type will be hidden for security reasons):
```
    Enter new UNIX password:
    Retype new UNIX password:
```

If successful, you'll see the message
```
    passwd: password updated successfully
```

After that, you'll be asked to fill in some information about the new user. It's ok to leave that information blank.
***

### Give the New User Administrative Permissions

Use the `usermod` command to add your new user to the **sudo** group
```bash
    usermod -aG sudo user_name
```

If you want to make sure you did this step correctly, you can change over to your new user with:
```bash
    su user_name
```
And check if you can use sudo by trying to access `/root`, which is only accessible to the root user by default:
```bash
    sudo ls /root
```

The first time you use sudo every session, you'll be asked to enter your password:
```
    [sudo] password for user_name
```
If you see the contents of `/root` after entering your password correctly, you've successfully given your user sudo privileges.
***

### Set Bash as the New User's Login Shell

To set bash as your new user's login shell, you use the command `chsh`. Type the following command into your terminal:
```bash
    chsh -s /bin/bash user_name
```

Running this command will prompt you to enter your password. Once you have entered your password correctly, your default login shell will be `/bin/bash`. (Note: You must log out and log back in to see this change)
***

## Prevent the Root User from Connecting to your Server via SSH

To prevent the root user from connecting to your server via SSH, you must first be logged in as your regular user. Remember: To connect to your server as your new regular user, use the following command:
```bash
    shh user_name@your_server_ip
```
You will then be prompted to enter your password.

If you're using an SSH key to log in, use the following command:
```bash
    ssh -i path/to/your/ssh_key user@your_server_ip
```

Next, using your preferred text editor (in this case, vim), open the sshd_config file with elevated permissions:
```bash
    sudo vim /etc/ssh/sshd_config
```

There's going to be a lot in this file, but the only section we're after is this one:
```bash
    # Authentication:
    
    #LoginGraceTime 2m
    PermitRootLogin yes
    #StrictModes yes
    #MaxAuthTries 6
    #MaxSessions 10
```

Simply change the `yes` to a `no` so it looks like:
```bash
        # Authentication:
    
    #LoginGraceTime 2m
    PermitRootLogin no
    #StrictModes yes
    #MaxAuthTries 6
    #MaxSessions 10
```

Save the file, and exit your text editor.
The next step is to restart the sshd daemon so that it reloads the configuration file you just changed. This is done by entering the following command:
```bash
    sudo systemctl restart sshd
```

Next, you can test if your server has been updated with your changes by logging out, and trying to log in as root.
```bash
    ssh root@your_server_ip
    or
    ssh -i path/to/your/ssh_key user@your_server_ip
```

If successful, you should see an error message saying `Permission denied`. In this case, an error message is a good message.
***

## Install NGINX

Now, you can finally install NGINX and get started with setting up your web server.
Firstly, it's a good idea to update apt's local package index so you'll get the most recent package version. Run the command:
```bash
    sudo apt update
```

Next, you'll install NGINX by running sudo apt again:
```bash
    sudo apt install nginx
```

Once it's done installing, the NGINX service should automatically start. You can verify this by running the following command:
```bash
    sudo systemctl status nginx
```

You should see a lot of information show up in your terminal. What you want to see is
```
    Loaded: loaded (/lib/systemd/system/nginx.service; enabled; preset: enabled)
    Active: active (running) since Tue 2023-11-28 22:21:31 UTC; 1h 50min ago
```
Of course, the time shown for `active (running) since` will be different for you.
***

## Configure NGINX to Serve a Sample Website

By default, NGINX will serve a simple html document so you can make sure it's working properly. You can view this document by switching to your host machine's terminal and running:
```bash
    curl your_server_ip
```

You should see this html document appear in your terminal:
```html
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
    html { color-scheme: light dark; }
    body { width: 35em; margin: 0 auto;
    font-family: Tahoma, Verdana, Arial, sans-serif; }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>

    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>
```

Now, here's how to configure NGINX to display your own html document:
The default directory for web servers is `/var/www`, so use
```bash
    cd /var/www
```
You're going to want to make a new directory for your website. In this case, let's call it `my-website`. You need elevated permissions to change stuff in this directory, so make sure you use sudo when making the directory and file:
```bash
    sudo mkdir my-website
```

Inside that directory, make a new file `index.html`:
```bash
    sudo vim index.html
```

Copy and paste the following text into your new file (or feel free to use your own html document!):
```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>My Website</title>
        <style>
            body {
                display: flex;
                align-items: center;
                justify-content: center;
                height: 100vh;
                margin: 0;
            }
            h1 {
                text-align: center;
            }
        </style>
    </head>
    <body>
        <h1>Hello, World</h1>
    </body>
    </html>
```

Save the html file and exit your text editor.
Next, you need to make a new server block file to specify what website your server should serve. Make a new file in `/etc/nginx/sites-available`. Let's call it `my-website.conf`:
```bash
    sudo vim /etc/nginx/sites-available/my-website.conf
```

The following text is cut down from the default server block configuration file. These are just the necessary parts without any commenting (though, it's a good idea to read the comments in the default file to understand what's going on here!):
```bash
    server {
        listen 80 default_server;
        listen [::]:80 default_server;
        
        root /var/www/my-website;
        
        index index.html index.htm index.nginx-debian.html;
        
        server_name _;
        
        location / {
            # First attempt to serve request as file, then
            # as directory, then fall back to displaying a 404.
            try_files $uri $uri/ =404;
        }
    }
```

This is an oversimplification of the code in this file. Like I said, it's a good idea to read the comments in the default server block file.
This code firstly states the port the server is listening for connection on, then it specifies the root **directory** for your website (not the index file!). After that, the possible index files are specified. Next, your domain name is specified (the "_" is a wildcard, so this represents any domain). Finally, `location` is used to define what location block to use to handle requests.
Once you've written that code in to your server block configuration file, you can save it and close your text editor.

You must now create a symbolic link from `/etc/nginx/sites-available` to `/etc/nginx/sites-enabled` to enable this website. Change directory into `/etc/nginx/sites-enabled`:
```bash
    cd /etc/nginx/sites-enabled
```

To make the sybolic link, use the following command:
```bash
    sudo ln /etc/nginx/sites-available
```

If you tried testing your website now, you would get an error  you need to delete the symbolic link to NGINX's default server block configuration file:
```bash
    sudo unlink default
```

To apply your changes, you need to restart the NGINX service:
```bash
    sudo systemctl restart nginx
```

Now, you can test if your website is working using:
```bash
    sudo nginx -t
```

If you didn't see any error messages in the previous command, you can try running the command from before on your home machine:
```bash
    curl your_server_ip
```

If you see your html document appear in the terminal, **congradulations!** You've successfuly set up an NGINX web server. You can even go to your web browser and type your server IP, you'll see your beautiful html document in all it's glory.