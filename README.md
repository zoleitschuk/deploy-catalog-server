# deploy-catalog-server
Deploy Catalog Server outlines the software installed, and configurations changes made in the course of deploying the [Item-Catalog](https://github.com/zoleitschuk/item-catalog) project over Amazon Lightsail.

The project can be accessed at [54.185.203.236](http://54.185.203.236/) and the server can be accessed through ssh port 2200.

## Methodology

The following section outlines the operations performed in the configuraiton of this specific Linux Server.

```
Recomendation: although not specified in these instructions, if it is your first time following along, I recomend taking snapshots of your server instance frequently. That way if anything goes wrong during your configuration, you do not have to restart your configuration from scratch.
```

### Create Server and Connect via SSH

1. Create a server instance on [Amazon Lightsail]()
2. Connect to server via ssh
`ssh -i <name-of-ssh-key> ubuntu@<publicIP>`
3. Set root user password.
`sudo passwd`

### Secure Server

5. Update all currently installed packages.
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
```
6. Change ssh port from default port 20 to port 2200.
`sudo nano /etc/ssh/sshd_config`
7. Update firewall to limit ports.
```
sudo ufw status
sudo ufw status verbose
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow www
sudo ufw allow ntp
sudo ufw allow 2200/tcp
sudo ufw deny 22
```
8. Enable firewall.
`sudo ufw enable`
9. Restart ssh service in order for port changes in sshd_config to take effect.
`sudo service ssh restart`

### Add Additional Superuser
10. Create a new user.
`sudo adduser <name-of-new-user>`
11. Give the grader permission to sudo.
`sudo visudo`
Add `<name-of-new-user> ALL=(ALL:ALL) ALL` to the file.
12. On your local machine create an ssh key pair for your new user.
`ssh-keygen`
13. View and copy the public key from your local machine.
`cat <name-of-ssh-key>.pub`
14. Log into the server as your new user and execute following.
```
mkdir .ssh
touch .ssh/authorized_keys
nano authorized_keys
```
15. Paste copied public key into authorized_keys and save.
16. Execute following:
```
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
sudo nano /etc/ssh/sshd_config
```
17. In sshd_config, change `PasswordAuthentication` to `no`.

You can now ssh into the server as your new user using:
`ssh -i <name-of-ssh-key> <name-of-new-user>@<publicIP> -p 2200`

### Prepare to Deploy Server
18. Prepare the environment to run off of Python3 (this step is only needed if your applicaiton uses Python3).
```
sudo apt-get install python3-dev

python --version
sudo rm /usr/bin/python
sudo ln -s /usr/bin/python3 /usr/bin/python
```
19. Install Apache and WSGI.
```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi-py3
sudo a2enmod wsgi
sudo service apache2 restart
```
20. Install python dependencies globally. Note: a future improvement would be to use virtualenv to store dependencies.
```
sudo apt-get install python3-pip

sudo pip3 install Flask
sudo pip3 install httplib2
sudo pip3 install sqlalchemy
sudo pip3 install oauth2client
sudo pip3 install psycopg2
```
21. Install git.
`sudo apt-get install git`
22. Create directories for Flask App.
```
sudo mkdir /var/www/catalog
sudo mkdir /var/www/catalog/catalog
cd /var/www/catalog/catalog/
```
23. Create a config file for Apache.
`sudo nano /etc/apache2/sites-available/catalog.conf`
24. Paste the following into the newly created config file and save.
```
<VirtualHost *:80>
                ServerName mywebsite.com (i.e.--><publicIP>)
                ServerAdmin admin@mywebsite.com
                WSGIScriptAlias / /var/www/catalog/catalog.wsgi
                <Directory /var/www/catalog/catalog/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/catalog/catalog/static
                <Directory /var/www/catalog/catalog/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
25. Disable the default Apache sight, enable the new one, and reload the service for changes to take effect.
```
sudo a2dissite 000-default
sudo a2ensite catalog
sudo service apache2 reload
```
26. Install postgres and set postgres user's password.
```			
sudo apt-get install postgresql
sudo nano /etc/postgresql/9.5/main/pg_hba.conf
sudo passwd postgres
```
27. Change user to postgres, and create and set up postgresql database.
```
su postgres
psql
```

In postgers, execute the following:
```
CREATE USER catalog;
ALTER USER catalog WITH PASSWORD 'catalog';
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
```

28. Switch back to new user and restart postresql for changes to take effect.
```
su <new-user-name>
sudo service postgresql restart
```
29. Configure the timezone to UTC.
```
sudo dpkg-reconfigure tzdata
```
Select UTC by first selecting No region, and then UTC.

### Deploy Server
30. Clone flask app.
`sudo git clone https://github.com/zoleitschuk/item-catalog.git`
31. If you did not clone the repo into the `/var/www/catalog/catalog` move all files from cloned directory to that location.
32. Rename the `application.py` file to '__init__.py` and change code to match:
```
if(__name__ == '__main__'):
    app.run()
```
33. Update `__init__.py`, `models.py`, and `populate_db.py` to use:
```
engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
```
34. Change the filepath of the clients secrets in `__init__.py` to use the full filepath:
```
/var/www/catalog/catalog/client_secrets.json
```
35. Run `python3 models.py` to update the catalog database to match the models defined by models.py.
36. Run `python3 populate_db.py` to populate the catalog database with some initial data.

## Built With

* [Flask](http://flask.pocoo.org/)
* [Apache](https://httpd.apache.org/docs/2.4/programs/apachectl.html)
* [Amazon Lightsail](https://aws.amazon.com/lightsail/)
* [Ubuntu](https://www.ubuntu.com/)

## Authors

* **Zachary Oleitschuk** - [zoleitschuk](https://github.com/zoleitschuk/)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details

## Acknowledgements

This project was created as part of the required course work for the [Udacity Full-Stack Nanodegree](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004).

### Resources

The following is a list of resources that I found exceptionally useful throughout the execution of this project.

* [Digitial Ocean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) - How To Deploy a Flask Application on an Ubuntu VPS
* [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu) - How To Install Linux, Apache, MySQL, PHP (LAMP) stack on Ubuntu
* [linkname](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-apache-mysql-and-python-lamp-server-without-frameworks-on-ubuntu-14-04) - How To Set Up an Apache, MySQL, and Python (LAMP) Server Without Frameworks on Ubuntu 14.04
* [Abigail Mathews](https://github.com/AbigailMathews/FSND-P5) - github repository

