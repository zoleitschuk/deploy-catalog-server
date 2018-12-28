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
23. Clone flask app into /var/www/catalog/catalog/
``
24. Create a config file for Apache.
`sudo nano /etc/apache2/sites-available/catalog.conf`
25. Paste the following into the newly created config file and save.
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
26. Disable the default Apache sight, enable the new one, and reload the service for changes to take effect.
```
sudo a2dissite 000-default
sudo a2ensite catalog
sudo service apache2 reload
```
27. Install postgres and set postgres user's password.
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

### Deploy Server



## Built With

* [Knockout](https://knockoutjs.com/) - JavaScript Framework for MVVC management.
* [Google Maps](https://developers.google.com/maps/documentation/) - Googl Maps API
* [FourSquare](https://developer.foursquare.com/docs/api) - 3rd Party API for providing additional location data
* [Bootstrap](https://getbootstrap.com/docs/4.0/getting-started/introduction/) - Styling Framework
* [jQuery](http://api.jquery.com/jQuery/) - JavaScript Framework

## Authors

* **Zachary Oleitschuk** - [zoleitschuk](https://github.com/zoleitschuk/)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details

## Acknowledgements

This project was created as part of the required course work for the [Udacity Full-Stack Nanodegree](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004).
