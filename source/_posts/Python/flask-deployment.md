---
title: Flask Deployment
date: 19-03-03 20:19:47
tags:
categories: Python
---
# *初始化

## Step 1 — Logging in as Root

## Step 2 — Creating a New User

用root容易产生权限问题

不能直接部署在home最好在home/子目录/项目目录否则容易产生权限问题

[username]代表用户名

Once you are logged in as **root**, we’re prepared to add the new user account that we will use to log in from now on.

This example creates a new user called **sammy**, but you should replace it with a username that you like:

```
adduser [username]
```

You will be asked a few questions, starting with the account password.

Enter a strong password and, optionally, fill in any of the additional information if you would like. This is not required and you can just hit `ENTER` in any field you wish to skip.

## Step 3 — Granting Administrative Privileges

Now, we have a new user account with regular account privileges. However, we may sometimes need to do administrative tasks.

To avoid having to log out of our normal user and log back in as the **root** account, we can set up what is known as “superuser” or **root** privileges for our normal account. This will allow our normal user to run commands with administrative privileges by putting the word `sudo` before each command.

To add these privileges to our new user, we need to add the new user to the **sudo** group. By default, on Ubuntu 18.04, users who belong to the **sudo** group are allowed to use the `sudo` command.

As **root**, run this command to add your new user to the **sudo** group (substitute the highlighted word with your new user):

```
usermod -aG sudo [username]
```

Now, when logged in as your regular user, you can type `sudo` before commands to perform actions with superuser privileges.

## Step 4 — Setting Up a Basic Firewall

会导致nginx失败这个可以放到最后面

Ubuntu 18.04 servers can use the UFW firewall to make sure only connections to certain services are allowed. We can set up a basic firewall very easily using this application.

**Note:** If your servers are running on DigitalOcean, you can optionally use [DigitalOcean Cloud Firewalls](https://www.digitalocean.com/community/tutorials/an-introduction-to-digitalocean-cloud-firewalls) instead of the UFW firewall. We recommend using only one firewall at a time to avoid conflicting rules that may be difficult to debug.

Different applications can register their profiles with UFW upon installation. These profiles allow UFW to manage these applications by name. OpenSSH, the service allowing us to connect to our server now, has a profile registered with UFW.

You can see this by typing:

```
ufw app list
OutputAvailable applications:
  OpenSSH
```

We need to make sure that the firewall allows SSH connections so that we can log back in next time. We can allow these connections by typing:

```
ufw allow OpenSSH
```

Afterwards, we can enable the firewall by typing:

```
ufw enable
```

Type “`y`” and press `ENTER` to proceed. You can see that SSH connections are still allowed by typing:

```
ufw status
OutputStatus: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
```

As **the firewall is currently blocking all connections except for SSH**, if you install and configure additional services, you will need to adjust the firewall settings to allow acceptable traffic in. You can learn some common UFW operations in [this guide](https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands).



Now that we have a regular user for daily use, we need to make sure we can SSH into the account directly.

**Note:** Until verifying that you can log in and use `sudo` with your new user, we recommend staying logged in as **root**. This way, if you have problems, you can troubleshoot and make any necessary changes as **root**. If you are using a DigitalOcean Droplet and experience problems with your **root** SSH connection, you can [log into the Droplet using the DigitalOcean Console](https://www.digitalocean.com/community/tutorials/how-to-use-the-digitalocean-console-to-access-your-droplet).

The process for configuring SSH access for your new user depends on whether your server’s **root** account uses a password or SSH keys for authentication.

### If the Root Account Uses Password Authentication

我直接用finalshell

If you logged in to your **root** account *using a password*, then password authentication is enabled for SSH. You can SSH to your new user account by opening up a new terminal session and using SSH with your new username:

```
ssh sammy@your_server_ip
```

After entering your regular user’s password, you will be logged in. Remember, if you need to run a command with administrative privileges, type `sudo` before it like this:

```
sudo command_to_run
```

You will be prompted for your regular user password when using `sudo` for the first time each session (and periodically afterwards).

To enhance your server’s security, **we strongly recommend setting up SSH keys instead of using password authentication**. Follow our guide on [setting up SSH keys on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1804) to learn how to configure key-based authentication.

### If the Root Account Uses SSH Key Authentication

If you logged in to your **root** account *using SSH keys*, then password authentication is *disabled* for SSH. You will need to add a copy of your local public key to the new user’s `~/.ssh/authorized_keys` file to log in successfully.

Since your public key is already in the **root** account’s `~/.ssh/authorized_keys` file on the server, we can copy that file and directory structure to our new user account in our existing session.

The simplest way to copy the files with the correct ownership and permissions is with the `rsync` command. This will copy the **root** user’s `.ssh` directory, preserve the permissions, and modify the file owners, all in a single command. Make sure to change the highlighted portions of the command below to match your regular user’s name:

**Note:** The `rsync` command treats sources and destinations that end with a trailing slash differently than those without a trailing slash. When using `rsync` below, be sure that the source directory (`~/.ssh`) **does not** include a trailing slash (check to make sure you are not using `~/.ssh/`).If you accidentally add a trailing slash to the command, `rsync` will copy the *contents* of the **root** account’s `~/.ssh` directory to the `sudo` user’s home directory instead of copying the entire `~/.ssh` directory structure. The files will be in the wrong location and SSH will not be able to find and use them.

```
rsync --archive --chown=sammy:sammy ~/.ssh /home/sammy
```

Now, open up a new terminal session and using SSH with your new username:

```
ssh sammy@your_server_ip
```

You should be logged in to the new user account without using a password. Remember, if you need to run a command with administrative privileges, type `sudo` before it like this:

```
sudo command_to_run
```

You will be prompted for your regular user password when using `sudo` for the first time each session (and periodically afterwards).

# **Ubuntu安装软件

### 方法一

## Step 1 — Installing the Components from the Ubuntu Repositories

Our first step will be to install all of the pieces we need from the Ubuntu repositories. This includes `pip`, the Python package manager, which will manage our Python components. We will also get the Python development files necessary to build some of the Gunicorn components.

First, let’s update the local package index and install the packages that will allow us to build our Python environment. These will include `python3-pip`, along with a few more packages and development tools necessary for a robust programming environment:

```
sudo apt update
sudo apt install python3-pip python3-dev build-essential libssl-dev libffi-dev python3-setuptools
sudo apt install python3-venv
sudo apt-get install nginx
```

With these packages in place, let’s move on to creating a virtual environment for our project.

```

```

拉代码

```
sudo apt-get install git-core

git clone https://github.com/cansijyun/ncov-globe.git
```

有虚拟环境则激活

```

pip3 install -r requirements.txt
sudo apt install libcairo2-dev pkg-config python3-dev
```

没有的话就从虚拟环境开始///没有应用要从创建应用开始，进入·项目目录

```
python3 -m venv myvenv
source myvenv/bin/activate

pip3 install Flask flask_cors requests
```

If you followed the initial server setup guide, you should have a UFW firewall enabled. To test the application, you need to allow access to port `5000`:（如果没开启防火墙就不用）

```
sudo ufw allow 5000
```

Now you can test your Flask app by typing:

```
python3 myproject.py
```

You will see output like the following, including a helpful warning reminding you not to use this server setup in production:

```
Output* Serving Flask app "myproject" (lazy loading)
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```

Visit your server’s IP address followed by `:5000` in your web browser:

```
http://your_server_ip:5000
```

### Creating the WSGI Entry Point

Next, let’s create a file that will serve as the entry point for our application. This will tell our Gunicorn server how to interact with the application.

Let’s call the file `wsgi.py`:

```
nano ~/myproject/wsgi.py
```

In this file, let’s import the Flask instance from our application and then run it:

~/myproject/wsgi.py

```python
from myproject import app

if __name__ == "__main__":
    app.run()
```

Save and close the file when you are finished.



## Step 4 — Configuring Gunicorn

```
pip install gunicorn
```

Your application is now written with an entry point established. We can now move on to configuring Gunicorn.

Before moving on, we should check that Gunicorn can serve the application correctly.

We can do this by simply passing it the name of our entry point. This is constructed as the name of the module (minus the `.py` extension), plus the name of the callable within the application. In our case, this is `wsgi:app`.

We’ll also specify the interface and port to bind to so that the application will be started on a publicly available interface:

```
cd ~/myproject
gunicorn --bind 0.0.0.0:5000 wsgi:app
```

You should see output like the following:

```
Output[2018-07-13 19:35:13 +0000] [28217] [INFO] Starting gunicorn 19.9.0
[2018-07-13 19:35:13 +0000] [28217] [INFO] Listening at: http://0.0.0.0:5000 (28217)
[2018-07-13 19:35:13 +0000] [28217] [INFO] Using worker: sync
[2018-07-13 19:35:13 +0000] [28220] [INFO] Booting worker with pid: 28220
```

Visit your server’s IP address with `:5000` appended to the end in your web browser again:

```
http://your_server_ip:5000
```

You should see your application’s output:

When you have confirmed that it’s functioning properly, press `CTRL-C` in your terminal window.

We’re now done with our virtual environment, so we can deactivate it:

```
deactivate
```

Any Python commands will now use the system’s Python environment again.

Next, let’s create the systemd service unit file. Creating a systemd unit file will allow Ubuntu’s init system to automatically start Gunicorn and serve the Flask application whenever the server boots.

Create a unit file ending in `.service` within the `/etc/systemd/system` directory to begin:

```
sudo nano /etc/systemd/system/myproject.service
```

Inside, we’ll start with the `[Unit]` section, which is used to specify metadata and dependencies. Let’s put a description of our service here and tell the init system to only start this after the networking target has been reached:

/etc/systemd/system/myproject.service

```
[Unit]
Description=Gunicorn instance to serve myproject
After=network.target
```

Next, let’s open up the `[Service]` section. This will specify the user and group that we want the process to run under. Let’s give our regular user account ownership of the process since it owns all of the relevant files. 

Let’s also give group ownership to the `www-data` group so that Nginx can communicate easily with the Gunicorn processes. Remember to replace the username here with your username:

/etc/systemd/system/myproject.service

```
[Unit]
Description=Gunicorn instance to serve myproject
After=network.target

[Service]
User=sammy
Group=www-data
```

Next, let’s map out the working directory and set the `PATH` environmental variable so that the init system knows that the executables for the process are located within our virtual environment. Let’s also specify the command to start the service. This command will do the following:

- Start 3 worker processes (though you should adjust this as necessary)
- Create and bind to a Unix socket file, `myproject.sock`, within our project directory. We’ll set an umask value of `007` so that the socket file is created giving access to the owner and group, while restricting other access
- Specify the WSGI entry point file name, along with the Python callable within that file (`wsgi:app`)

Systemd requires that we give the full path to the Gunicorn executable, which is installed within our virtual environment.

Remember to replace the username and project paths with your own information:

/etc/systemd/system/myproject.service

```
[Unit]
Description=Gunicorn instance to serve myproject
After=network.target

[Service]
User=sammy
Group=www-data
WorkingDirectory=/home/sammy/myproject
Environment="PATH=/home/sammy/myproject/myprojectenv/bin"
ExecStart=/home/sammy/myproject/myprojectenv/bin/gunicorn --workers 3 --bind unix:myproject.sock -m 007 wsgi:app
```

Finally, let’s add an `[Install]` section. This will tell systemd what to link this service to if we enable it to start at boot. We want this service to start when the regular multi-user system is up and running:

/etc/systemd/system/myproject.service

```
[Unit]
Description=Gunicorn instance to serve myproject
After=network.target

[Service]
User=sammy
Group=www-data
WorkingDirectory=/home/sammy/myproject
Environment="PATH=/home/sammy/myproject/myprojectenv/bin"
ExecStart=/home/sammy/myproject/myprojectenv/bin/gunicorn --workers 3 --bind unix:myproject.sock -m 007 wsgi:app

[Install]
WantedBy=multi-user.target
```

With that, our systemd service file is complete. Save and close it now.

We can now start the Gunicorn service we created and enable it so that it starts at boot:

```
sudo systemctl start myproject
sudo systemctl enable myproject
```

Let’s check the status:

```
sudo systemctl status myproject
```

You should see output like this:

```
Output● myproject.service - Gunicorn instance to serve myproject
   Loaded: loaded (/etc/systemd/system/myproject.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2018-07-13 14:28:39 UTC; 46s ago
 Main PID: 28232 (gunicorn)
    Tasks: 4 (limit: 1153)
   CGroup: /system.slice/myproject.service
           ├─28232 /home/sammy/myproject/myprojectenv/bin/python3.6 /home/sammy/myproject/myprojectenv/bin/gunicorn --workers 3 --bind unix:myproject.sock -m 007
           ├─28250 /home/sammy/myproject/myprojectenv/bin/python3.6 /home/sammy/myproject/myprojectenv/bin/gunicorn --workers 3 --bind unix:myproject.sock -m 007
           ├─28251 /home/sammy/myproject/myprojectenv/bin/python3.6 /home/sammy/myproject/myprojectenv/bin/gunicorn --workers 3 --bind unix:myproject.sock -m 007
           └─28252 /home/sammy/myproject/myprojectenv/bin/python3.6 /home/sammy/myproject/myprojectenv/bin/gunicorn --workers 3 --bind unix:myproject.sock -m 007
```

If you see any errors, be sure to resolve them before continuing with the tutorial.



## Step 5 — Configuring Nginx to Proxy Requests

Our Gunicorn application server should now be up and running, waiting for requests on the socket file in the project directory. Let’s now configure Nginx to pass web requests to that socket by making some small additions to its configuration file.

Begin by creating a new server block configuration file in Nginx’s `sites-available` directory. Let’s call this `myproject` to keep in line with the rest of the guide:

```
sudo nano /etc/nginx/sites-available/myproject
```

Open up a server block and tell Nginx to listen on the default port `80`. Let’s also tell it to use this block for requests for our server’s domain name:

/etc/nginx/sites-available/myproject

```
server {
    listen 80;
    server_name your_domain www.your_domain;
}
```

Next, let’s add a location block that matches every request. Within this block, we’ll include the `proxy_params` file that specifies some general proxying parameters that need to be set. We’ll then pass the requests to the socket we defined using the `proxy_pass` directive:

/etc/nginx/sites-available/myproject

```
server {
    listen 80;
    server_name your_domain www.your_domain;

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/sammy/myproject/myproject.sock;
    }
}
```

Save and close the file when you’re finished.

To enable the Nginx server block configuration you’ve just created, link the file to the `sites-enabled` directory:

```
sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled
```

With the file in that directory, you can test for syntax errors:

```
sudo nginx -t
```

If this returns without indicating any issues, restart the Nginx process to read the new configuration:

```
sudo systemctl restart nginx
```

Finally, let’s adjust the firewall again. We no longer need access through port `5000`, so we can remove that rule. We can then allow full access to the Nginx server:

```
sudo ufw delete allow 5000
sudo ufw allow 'Nginx Full'
```

You should now be able to navigate to your server’s domain name in your web browser:

```
http://your_domain
```

You should see your application’s output:

![Flask sample app](https://assets.digitalocean.com/articles/nginx_uwsgi_wsgi_1404/test_app.png)

If you encounter any errors, trying checking the following:

- `sudo less /var/log/nginx/error.log`: checks the Nginx error logs.
- `sudo less /var/log/nginx/access.log`: checks the Nginx access logs.
- `sudo journalctl -u nginx`: checks the Nginx process logs.
- `sudo journalctl -u myproject`: checks your Flask app’s Gunicorn logs.



To ensure that traffic to your server remains secure, let’s get an SSL certificate for your domain. There are multiple ways to do this, including getting a free certificate from [Let’s Encrypt](https://letsencrypt.org/), [generating a self-signed certificate](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu-18-04), or [buying one from another provider](https://www.digitalocean.com/community/tutorials/how-to-install-an-ssl-certificate-from-a-commercial-certificate-authority) and configuring Nginx to use it by following Steps 2 through 6 of  [How to Create a Self-signed SSL Certificate for Nginx in Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu-18-04#step-2-–-configuring-nginx-to-use-ssl). We will go with option one for the sake of expediency.

First, add the Certbot Ubuntu repository:

```
sudo add-apt-repository ppa:certbot/certbot
```

You’ll need to press `ENTER` to accept.

Install Certbot’s Nginx package with `apt`:

```
sudo apt install python-certbot-nginx
```

Certbot provides a variety of ways to obtain SSL certificates through plugins. The Nginx plugin will take care of reconfiguring Nginx and reloading the config whenever necessary. To use this plugin, type the following:

```
sudo certbot --nginx -d your_domain -d www.your_domain
```

This runs `certbot` with the `--nginx` plugin, using `-d` to specify the names we’d like the certificate to be valid for.

If this is your first time running `certbot`, you will be prompted to enter an email address and agree to the terms of service. After doing so, `certbot` will communicate with the Let’s Encrypt server, then run a challenge to verify that you control the domain you’re requesting a certificate for.

If that’s successful, `certbot` will ask how you’d like to configure your HTTPS settings:

```
OutputPlease choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
-------------------------------------------------------------------------------
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
-------------------------------------------------------------------------------
Select the appropriate number [1-2] then [enter] (press 'c' to cancel):
```

Select your choice then hit `ENTER`. The configuration will be updated, and Nginx will reload to pick up the new settings. `certbot` will wrap up with a message telling you the process was successful and where your certificates are stored:

```
OutputIMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/your_domain/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/your_domain/privkey.pem
   Your cert will expire on 2018-07-23. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

If you followed the Nginx installation instructions in the prerequisites, you will no longer need the redundant HTTP profile allowance:

```
sudo ufw delete allow 'Nginx HTTP'
```

To verify the configuration, navigate once again to your domain, using `https://`:

```
https://your_domain
```

You should see your application output once again, along with your browser’s security indicator, which should indicate that the site is secured.



### 方法二

假定你是在腾迅云或者阿里云购买了VPS，那么直接执行以下指令吧，其它的不多解释了，无非就是准备一下 python 环境。

```
$ sudo apt-get update
$ sudo apt-get install python-dev python-pip python-virtualenv
```

然后安装 nginx

```
$ sudo apt-get install nginx
```





在 `/var/www` 目录下建立一个 `myflask` 的文件夹(你的项目目录)，然后用 `chmod` 改一下权限

```
$ sudo mkdir /var/www/myflask
$ sudo chmod 777 /var/www/myflask
```

> 注：当然你可以使用 nginx 的默认网站目录 `/usr/share/nginx/html`

然后用 `scp` 指令直接将本机上的 flask 项目传到服务器:

```
$ scp -r myflask root@www.mydomain.com:/var/www/myflask
```

域名就改成地址或者你的服务器正在使用的域名，我这里是用 `root` 用户进入的，你得按你的服务器的用户来修改。两大云的默认根用户是：

- 腾迅 ：ubuntu
- 阿里 ：root





### **激活虚拟环境



### 安装requirement



## Gunicorn

[Gunicorn](http://www.gunicorn.org/) 绿色独角兽 是一个Python WSGI UNIX的HTTP服务器。这是一个pre-fork worker的模型，从Ruby的独角兽（[Unicorn](http://www.oschina.net/p/unicorn) ）项目移植。该Gunicorn服务器大致与各种Web框架兼容，只需非常简单的执行，轻量级的资源消耗，以及相当迅速。

我曾经Google 过 Gunicorn 与 uwsgi ，都说uwsgi 的性能要比 gunicorn 高，所以最终结果就杯具了。不过，现在回过头来看这只 “独角兽”还为时不晚吧。

### 安装 Gunicorn

Gunicorn 应该装在你的 virtualenv 环境下，关于 virtualenv 就不多说了，如果没用过那就赶快脑补吧。安装前记得激活 venv

```shell
(venv) $ pip install gunicorn
```

### 运行 Gunicorn

```shell
(venv) $ gunicorn -w 4 -b 0:0:0:0:5000 wsgi:app
```

That's all！ 它的安装就这么简单。不过这里得作一个解释。就是最后的那个参数 `wsgi:application` 这个是程序入口，我得写个小小的范例来说明一下：

新建一个 `wsgi.py` 的文件, 注意，这里和 Flask 项目中常用的 `manage.py` 引导脚本是没有半点毛关系的。（这是我笨，以前一直没分清被uwsgi搞糊涂了）

```python
# wsgi.py

from flask import Flask

def create_app():
  # 这个工厂方法可以从你的原有的 `__init__.py` 或者其它地方引入。
  app = Flask(__name__)
  return app

application = create_app()

if __name__ == '__main__':
	application.run()
```

好了，这个 `wsgi:application` 参数就很好理解了， 分两部：`wsgi` 就是引导用的 python 文件名称（不包括后缀/模块名）`application` 就是 Flask 实例的名称。这样 gunicorn 就会找到具体要 host 哪一个 flask 实例了。

从这里开始就可以体现 gunicorn 的好了，我们根本不用配什么配置文件的，一个指令就可以将它起动。

### 将 Gunicorn 作为服务运行

#### 方法一（这里的myproject指代入口python文件名）

When you have confirmed that it’s functioning properly, press `CTRL-C` in your terminal window.

We’re now done with our virtual environment, so we can deactivate it:

```
deactivate
```

Any Python commands will now use the system’s Python environment again.

Next, let’s create the systemd service unit file. Creating a systemd unit file will allow Ubuntu’s init system to automatically start Gunicorn and serve the Flask application whenever the server boots.

Create a unit file ending in `.service` within the `/etc/systemd/system` directory to begin:

```
sudo nano /etc/systemd/system/myproject.service
```

Inside, we’ll start with the `[Unit]` section, which is used to specify metadata and dependencies.

 Let’s put a description of our service here and tell the init system to only start this **after the networking target** has been reached:

/etc/systemd/system/myproject.service

```
[Unit]
Description=Gunicorn instance to serve myproject
After=network.target
```

Next, let’s open up the `[Service]` section. This will specify the user and group that we want the process to run under. Let’s give our regular user account ownership of the process since it owns all of the relevant files.

Let’s also give group ownership to the `www-data` group so that Nginx can communicate easily with the Gunicorn processes. Remember to replace the username here with your username:

/etc/systemd/system/myproject.service

```
[Unit]
Description=Gunicorn instance to serve myproject
After=network.target

[Service]
User=sammy
Group=www-data
```

Next, let’s map out the working directory and set the `PATH` environmental variable so that the init system knows that the executables for the process are located within our virtual environment. Let’s also specify the command to start the service. This command will do the following:

- Start 3 worker processes (though you should adjust this as necessary)
- Create and bind to a Unix socket file, `myproject.sock`, within our project directory. We’ll set an umask value of `007` so that the socket file is created giving access to the owner and group, while restricting other access
- Specify the WSGI entry point file name, along with the Python callable within that file (`wsgi:app`)

Systemd requires that we give the full path to the Gunicorn executable, which is installed within our virtual environment.

Remember to replace the username and project paths with your own information:

/etc/systemd/system/myproject.service

```
[Unit]
Description=Gunicorn instance to serve myproject
After=network.target

[Service]
User=sammy
Group=www-data
WorkingDirectory=/home/sammy/myproject
Environment="PATH=/home/sammy/myproject/myprojectenv/bin"
ExecStart=/home/sammy/myproject/myprojectenv/bin/gunicorn --workers 3 --bind unix:myproject.sock -m 007 wsgi:app
```

如果没有用virtualenv就不用Environment

myproject.sock是给nginx绑定时使用

可以事先这样测试

```
exec gunicorn --workers 3 --bind unix:myproject.sock -m 007 wsgi:app
```



Finally, let’s add an `[Install]` section. This will tell systemd what to link this service to if we enable it to start at boot. We want this service to start when the regular multi-user system is up and running:

/etc/systemd/system/myproject.service

```
[Unit]
Description=Gunicorn instance to serve myproject
After=network.target

[Service]
User=sammy
Group=www-data
WorkingDirectory=/home/sammy/myproject
Environment="PATH=/home/sammy/myproject/myprojectenv/bin"
ExecStart=/home/sammy/myproject/myprojectenv/bin/gunicorn --workers 3 --bind unix:myproject.sock -m 007 wsgi:app

[Install]
WantedBy=multi-user.target
```

With that, our systemd service file is complete. Save and close it now.

We can now start the Gunicorn service we created and enable it so that it starts at boot:

```
sudo systemctl start myproject
sudo systemctl enable myproject
```

Let’s check the status:

```
sudo systemctl status myproject
```

You should see output like this:

```
Output● myproject.service - Gunicorn instance to serve myproject
   Loaded: loaded (/etc/systemd/system/myproject.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2018-07-13 14:28:39 UTC; 46s ago
 Main PID: 28232 (gunicorn)
    Tasks: 4 (limit: 1153)
   CGroup: /system.slice/myproject.service
           ├─28232 /home/sammy/myproject/myprojectenv/bin/python3.6 /home/sammy/myproject/myprojectenv/bin/gunicorn --workers 3 --bind unix:myproject.sock -m 007
           ├─28250 /home/sammy/myproject/myprojectenv/bin/python3.6 /home/sammy/myproject/myprojectenv/bin/gunicorn --workers 3 --bind unix:myproject.sock -m 007
           ├─28251 /home/sammy/myproject/myprojectenv/bin/python3.6 /home/sammy/myproject/myprojectenv/bin/gunicorn --workers 3 --bind unix:myproject.sock -m 007
           └─28252 /home/sammy/myproject/myprojectenv/bin/python3.6 /home/sammy/myproject/myprojectenv/bin/gunicorn --workers 3 --bind unix:myproject.sock -m 007
```

If you see any errors, be sure to resolve them before continuing with the tutorial.

#### 方法二

这就是最后一步了，我们在此将采用 UpStart 配置Flask程序作为服务程序在Linux起动时运行。首先建立起动配置文件:

```shell
sudo nano /etc/init/myflask.conf
```

然后加入如下配置

```
description "The myflask service"

start on runlevel [2345]
stop on runlevel [!2345]


respawn
setuid root
setgid www-data

env PATH= /var/www/myflask/venv/bin
chdir /var/www/myflask/

exec gunicorn -w 4 -b 127.0.0.1:8080 wsgi:application
```

OK 大功告成，启动 myflask 服务

```shell
sudo service myflask start
```

这里有一点必须补充的，请留意在 `myflask.conf` 中的

```
env PATH= /var/www/myflask/venv/bin
chdir /var/www/myflask/
```

这里所指向的地址就是你的项目路径和 virtualenv 的路径

### Nginx 的配置

关于 Nginx 我也就不详细讲了，我们就直奔主题，杀入 Nginx 的默认配置文件

```shell
sudo nano /etc/nginx/sites-available/default
```

暴力修改成为以下的内容

> 建议先备份一下 `default` 文件
> `sudo cp /etc/nginx/site-avalidable/default /etc/nginx/site-avalidable/default.bak`

Next, let’s add a location block that matches every request. Within this block, we’ll include the `proxy_params` file that specifies some general proxying parameters that need to be set. We’ll then pass the requests to the socket we defined using the `proxy_pass` directive:

/etc/nginx/sites-available/myproject

```nginx
server {
    listen 5000;
    server_name pyact.com;

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/ncov-world/back/flask/myproject.sock;
    }
}
```

​        proxy_pass 

```
http://unix:+sock文件所在地就是之前service的workingDirectory;
```



记得完成 nginx 需要重新起动 nginx 服务喔！

```shell
sudo service nginx restart
```



如果出502问题，查看日志

```
sudo tail -30 /var/log/nginx/error.log
```





## 小结

这个部署过程感觉比我之前所介绍的 uwsgi 那种简单很多吧。这里给一点小 Tips 如果你用 Fabric 来完成这个部署过程的话那么就是纯自动化部署了喔，值得尝试的。



分类: [Flask](https://www.cnblogs.com/Ray-liang/category/752618.html)