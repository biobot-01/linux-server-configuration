# Linux Server Configuration
__IP ADDRESS:__ 35.176.196.186 __HTTP ALIAS ADDRESS:__ 35.176.196.186.xip.io __SSH Port:__ 2200
## Project Overview
> Take a baseline installation of a Linux server and prepare it to host web applications. To secure the server from a number of attack vectors, install & configure a database server, & deploy an existing web application onto it.

## __Steps to complete this project:__
### Setup the server
1. Start a new Ubuntu Linux server instance on [Amazon Lightsail][1].
1. Log in or Sign up to [AWS][2].
1. Click on __Create instance__.
1. Instance location:
    * You can change AWS Region or leave the default selected.
1. Pick your instance image:
    1. Select a platform - __Linux/Unix__
    1. Select a blueprint - __OS Only__ & __Ubuntu 18.04 LTS__
1. Choose your instance plan - Lowest tier (First month free!)
1. Identify your instance:
    * You can change the hostname or leave the default selected.
1. Click on __Create instance__.
1. The new instance will appear, then click on it.
1. Take note of the __Public IP__: 35.176.196.186 & __Username__: ubuntu  
[Learn more about instances][3]

### Configure SSH for the server
> You can click on the __Connect using SSH__ button to connect to the instance, however we will set up a private key to connect using our local machine.

Following from the previous section:
1. Scroll to the bottom, where the last sentence reads:  
`You can download your default private key from the Account page`
1. Click on __Account page__.
1. Manage your SSH keys:
    * Click on the __Download__ link of your AWS Region.
1. Save the file in your Downloads folder.
1. Open the terminal `Alt + Ctrl + T`.
1. Check you are in your home directory:  
`$ pwd`
    * This should print `/home/{your-username}`
1. Move the downloaded file `LightsailDefaultKey-{AWS Region}.pem` from `Downloads` directory into `.ssh` directory & rename it `ubuntu_motorbike_catalog.rsa`:  
`$ mv -v Downloads/LightsailDefaultKey-{AWS Region}.pem .ssh/ubuntu_motorbike_catalog.rsa`
1. Change the file permissions so only the owner can read & write:  
`$ chmod -v 600 .ssh/ubuntu_motorbike_catalog.rsa`
1. To SSH into the instance server using the __Public IP__ & __Username__:  
`$ ssh -i .ssh/ubuntu_motorbike_catalog.rsa ubuntu@35.176.196.186`
    * The terminal will output some info about the instance server & the prompt will change to `ubuntu@ip-{private-ip}:~$`, this means you are successfully connected to the [Amazon Lightsail][1] [instance server][5].

### Secure the server
Following from the previous section:
1. Update all currently installed packages on the [instance server][5] through the SSH connection in your terminal, run the following commands:  
`$ sudo apt update`  
__NOTE:__ To view the upgradable packages, run the command `$ apt list --upgradable`  
`$ sudo apt upgrade`  
__NOTE:__ In the terminal a message about restarting services during package upgrades: select _Yes_
__NOTE:__ In the terminal a message about configuring openssh-server: select _install the package maintainer's version_
__NOTE:__ In the terminal a message about grub menu: select _install the package maintainer's version_
`$ exit`
1. Reboot the [instance server][5]:
    * Click on the [instance server][5] & then click __Reboot__.
1. SSH back into the [instance server][5]:  
`$ ssh -i .ssh/ubuntu_motorbike_catalog.rsa ubuntu@35.176.196.186`
1. Change the SSH port from __22__ to __2200__:
    * Edit the `/etc/ssh/sshd_config` file `$ sudo nano /etc/ssh/sshd_config`
    * Search for `#Port 22` & change to `Port 2200`.
    * Save the file `Ctrl + O`, then press Enter.
    * Exit nano, `Ctrl + X`
    * Restart SSH service `$ sudo service ssh restart`  
[Learn more about nano][6]
1. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), & NTP (port 123):
    * Check the status of UFW:  
    `$ sudo ufw status verbose` or `$ sudo ufw status`
    * Define default rules for allowing & denying connections:  
    `$ sudo ufw default deny incoming`  
    `$ sudo ufw default allow outgoing`
    * Allow connection on port 2200 (SSH) using TCP:  
    `$ sudo ufw allow 2200/tcp`
    * Deny connection on port 22 (SSH) using TCP:  
    `$ sudo ufw deny 22/tcp` or `$ sudo ufw deny ssh`
    * Allow connection on port 80 (HTTP) using TCP:  
    `$ sudo ufw allow 80/tcp` or `$ sudo ufw allow www`
    * Allow connection on port 123 (NTP) using UDP:  
    `$ sudo ufw allow 123/udp` or `$ sudo ufw allow ntp`
    * Check added user rules `$ sudo ufw show added`
    * Activate UFW `$ sudo ufw enable`
    * Close the SSH connection `$ exit`  
Source: [DigitalOcean][7]
1. Update firewall to match changes on [Amazon Lightsail][1] [instance server][5]:
    * On the [instance server][5], click on the three vertical dots & click on Manage.
    * Click the Networking tab.
    * Scroll down to Firewall section & click on __Add another__.
    * Update the rules to match the rules applied above.
    * Scroll back to the top & click on __Reboot__.  
__NOTE:__ From this point forward, to SSH into the [instance server][5], run the command:  
`$ ssh -i .ssh/ubuntu_motorbike_catalog.rsa -p 2200 ubuntu@35.176.196.186`
1. Test the above command.

### Create a new user __grader__ & give __grader__ access
> Allow the grader to log in to the server with sudo privileges.

1. Create a new user account named __grader__:  
`$ sudo adduser grader`
    * Enter a password for the new user. Password: `grader`
    * Press Enter for default information.
1. Give __grader__ the permission to `sudo`:
    * Open a file `sudoers.tmp` with the command `$ sudo visudo`
    * Find the comment `# User privilege specification` & under `root`, add the following `grader ALL=(ALL:ALL) ALL`
    * Save the file `Ctrl + O`, then press Enter.
    * Exit nano, `Ctrl + X`
1. Create a SSH key pair for __grader__ using the `ssh-keygen` tool:
    * Keep the current terminal open.
    * Open a new terminal `Alt + Ctrl + T`, this will be the local machine.
    * Run the command `$ ssh-keygen`, save file to `/home/{your-username}/.ssh/grader_key` & press Enter if prompted for passphrase.
    * List new files created `$ ls -a .ssh/`
    * Copy contents of file from terminal `$ cat .ssh/grader_key.pub`
    * Switch back to the other terminal with the SSH connection:
        * Login as __grader__ `$ su - grader`
        * Check if there is a `.ssh` directory `$ ls -a`
        * Create `.ssh` directory `$ mkdir .ssh`
        * Create `authorized_keys` file in `.ssh` directory `$ touch .ssh/authorized_keys`
        * Paste contents of `grader_key.pub` file into `authorized_keys` file `$ nano .ssh/authorized_keys`
        * Save the file `Ctrl + O`, then press Enter.
        * Exit nano, `Ctrl + X`
        * Change `.ssh` folder permission `chmod -v 700 .ssh/`
        * Change `authorized_keys` file permission `chmod -v 644 .ssh/authorized_keys`
        * Force key based authentication:
            * Edit the `/etc/ssh/sshd_config` file `$ sudo nano /etc/ssh/sshd_config`
            * Search for `#PasswordAuthentication yes` & change to `PasswordAuthentication no`
        * Disable remote root login:
            * Search for `#PermitRootLogin prohibit-password` & change to `PermitRootLogin no`
            * Save the file `Ctrl + O`, then press Enter.
            * Exit nano, `Ctrl + X`
        * Restart SSH service `$ sudo service ssh restart`
        * Logout as __grader__ `$ exit`
        * Close the SSH connection `$ exit`  
__NOTE:__ You can now SSH into the [instance server][5] as __grader__, run the command:  
        `$ ssh -i .ssh/grader_key -p 2200 grader@35.176.196.186`
1. Test the above command.

### Prepare to deploy the project
1. Configure the local timezone to __UTC__:  
`$ sudo dpkg-reconfigure tzdata`
    * Select None of the above.
    * Select __UTC__.
__NOTE:__ To check the timezone `$ date +%Z`
1. Install & configure Apache to serve a Python 3 mod_wsgi application:
    * Install Apache `$ sudo apt install apache2`
    * Test Apache installation:
        * Open a Browser & enter the __Public IP__ address `http://35.176.196.186`
    * Install mod_wsgi for Python 3 `$ sudo apt install python3-setuptools python3-dev libapache2-mod-wsgi-py3`
    * Restart Apache `$ sudo service apache2 restart`
1. Install Python 3 pip & venv:  
`$ sudo apt install python3-pip python3-venv`
1. Install & configure PostgreSQL:  
`$ sudo apt install postgresql postgresql-contrib libpq-dev`
    * Do not allow remote connections `$ sudo nano /etc/postgresql/10/main/pg_hba.conf`
    * Only connections from localhost are allowed.
1. Create a new database user named __catalog__ that has limited permissions to the catalog application database:
    * Login to PostgreSQL as postgres `$ sudo -u postgres psql`
    * Create user `# CREATE USER catalog WITH PASSWORD 'catalog';`
    * Allow user to create databases `# ALTER USER catalog CREATEDB;`
    * Create the database `# CREATE DATABASE catalog WITH OWNER catalog;`
    * Connect to database `# \c catalog`
    * Only allow user __catalog__ permission to database `# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`
    * Quit PostgreSQL `# \q`
1. Install git:  
`sudo apt install git` - It was already installed

### Deploy the Item Catalog project
The project to be deployed will be [Motorbike Catalog][8]
1. Clone & setup the __Item Catalog__ project from the Github repository:
    * Move to Apache root folder `$ cd /var/www/`
    * Clone project repo `$ sudo git clone https://github.com/biobot-01/motorbike-catalog.git`
    * Change ownership of project folder `$ sudo chown -vR grader:grader motorbike-catalog`
    * Rename `motorbike-catalog` directory `$ sudo mv -v motorbike-catalog/ MotorbikeCatalog`
    * Move into `MotorbikeCatalog` directory `$ cd MotorbikeCatalog`
    * Rename `catalog` directory `$ mv -v catalog/ MotorbikeCatalog`
    * Move `database_setup.py` file into inner `MotorbikeCatalog` `$ mv -v database_setup.py MotorbikeCatalog/`
    * Move to the inner `MotorbikeCatalog` directory `$ cd MotorbikeCatalog/`
    * Rename `app.py` to `__init__.py` `$ mv -v app.py __init__.py`
    * Edit `__init__.py`, `models.py`, `database_setup.py` & change `engine = create_engine('sqlite:///motorbike_catalog.db')` to `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
    * Edit `database_setup.py` & change `from catalog.models` to `from MotorbikeCatalog.models`
    * Edit `__init__.py` & change
    ```
    from models import ...

    app.secret_key = 'super_secret_key'
    app.debug = True
    app.run(host='127.0.0.1', port=8000, threaded=False)
    ```
    to
    ```
    from MotorbikeCatalog.models import ...

    app.run()
    ```
    * Edit `__init__.py` & change all file paths to absolute paths.
1. Configure __wsgi__ file:
    * Return to outer `MotorbikeCatalog` directory `cd ../`
    * Create a new file `app.wsgi` `touch app.wsgi`
    * Edit `app.wsgi` & add
    ```python
    #!/usr/bin/env python3

    import sys
    import site
    import logging

    python_home = '/var/www/MotorbikeCatalog/MotorbikeCatalog/venv/flask'
    # Calculate path to site-packages directory
    python_version = '.'.join(map(str, sys.version_info[:2]))
    site_packages = python_home + '/lib/python{}/site-packages'.format(python_version)
    # Add the site-packages directory
    site.addsitedir(site_packages)

    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, '/var/www/MotorbikeCatalog')

    from secrets import token_hex

    from MotorbikeCatalog import app as application

    application.secret_key = token_hex(32)

    ```
1. Setup Python virtual environment with python3-venv:
    * Move to the inner `MotorbikeCatalog` directory `$ cd MotorbikeCatalog/`
    * Create folder `$ mkdir venv`
    * Create venv `$ python3 -m venv venv/flask` & `$ python3 -m venv venv/python`
    * Activate `venv/flask` with `$ source venv/flask/bin/activate`
    * Update pip `$ pip install -U pip`
    * Install requirements `$ pip install -r requirements.txt`
    * Install psycopg2 `$ pip install psycopg2`
    * Create the database schema `$ python models.py`
    * Deactivate venv `$ deactivate`
1. Set it up in the server so that it functions correctly when visiting the server's IP address in a browser:
`$ cd /etc/apache2/`
    * Configure virtual host:
        * Copy vhost file `$ sudo cp -v sites-available/000-default.conf sites-available/001-motorbikecatalog.conf`
        * Edit `001-motorbikecatalog.conf` `$ sudo nano sites-available/001-motorbikecatalog.conf` & add
        ```
        <VirtualHost *:80>
          ServerName 35.176.196.186
          ServerAlias 35.176.196.186.xip.io

          ServerAdmin webmaster@localhost
          DocumentRoot /var/www/MotorbikeCatalog

          ErrorLog ${APACHE_LOG_DIR}/error.log
          CustomLog ${APACHE_LOG_DIR}/access.log combined

          WSGIDaemonProcess MotorbikeCatalog python-home=/var/www/MotorbikeCatalog/MotorbikeCatalog/venv/python user=grader group=grader threads=5

          WSGIScriptAlias / /var/www/MotorbikeCatalog/app.wsgi

          <Directory /var/www/MotorbikeCatalog/MotorbikeCatalog>
            WSGIProcessGroup MotorbikeCatalog
            WSGIApplicationGroup %{GLOBAL}
            Require all granted
          </Directory>

          Alias /static /var/www/MotorbikeCatalog/MotorbikeCatalog/static

          <Directory /var/www/MotorbikeCatalog/MotorbikeCatalog/static>
            Require all granted
          </Directory>
        </VirtualHost>
        ```
        * Disable default vhost `$ sudo a2dissite 000-default.conf`
        * Enable motorbikecatalog vhost `$ sudo a2ensite 001-motorbikecatalog.conf`
        * Restart Apache seerver `$ sudo service apache2 restart`  
[Source: DigitalOcean][9]
1. Make sure that the `.git` directory is not publicly accessible via the browser:
    * Change to `apache2` directory `$ cd /etc/apache2/`
    * Edit `security.conf` file `$ sudo nano conf-available/security.conf` & change `ServerTokens OS` to `ServerTokens Prod`, `ServerSignature on` to `ServerSignature off` & add
    ```
    <DirectoryMatch "/\.git">
      Require all denied
    </DirectoryMatch>
    ```
    * Restart Apache `$ sudo service apache2 restart`

[Thanks to Daniel A (jungleBadger) for pointing me to his project][10]

[1]: https://lightsail.aws.amazon.com "Amazon Lightsail"
[2]: https://aws.amazon.com/ "Amazon Web Services (AWS)"
[3]: https://lightsail.aws.amazon.com/ls/docs/en_us/articles/understanding-instances-virtual-private-servers-in-amazon-lightsail "Virtual Private Servers in Amazon Lightsail"
[4]: https://help.ubuntu.com/lts/serverguide/openssh-server.html "OpenSSH - Remote Administration"
[5]: https://lightsail.aws.amazon.com/ls/webapp/home/instances "Amazon Lightsail Instances"
[6]: https://www.tecmint.com/learn-nano-text-editor-in-linux/ "Nano guide"
[7]: https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server "Setup & Configure UFW"
[8]: https://github.com/biobot-01/motorbike-catalog "Udacity Full Stack Web Developer Nanodegree Item Catalog Project"
[9]: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps "Deploy flask app on ubuntu vps"
[10]: https://github.com/jungleBadger/-nanodegree-linux-server "Linux Server Configuration"
