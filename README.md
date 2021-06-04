![Tag](https://img.shields.io/badge/Pathfinder-v2.0.0-brightgreen)
![TAG](https://img.shields.io/badge/Traefik-v1.7.21-blue)
[![Docker Build](https://github.com/SteveTh3Piirate/pathfinder-docker/actions/workflows/CIActions.yml/badge.svg?branch=master)](https://github.com/SteveTh3Piirate/pathfinder-docker/actions/workflows/CIActions.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Dockerfile for running [Pathfinder](https://github.com/exodus4d/pathfinder), the mapping tool for EVE Online.

# Do NOT blindly copy and paste, change the values you need to change!

Traefik 1.7 Installation

Websocket server support.

Added Event Extension Installation Guide

Added guide to change - character_set_server = latin1 to character_set_server = utf8mb4

# Traefik v1.7.21 Setup
1. ```sudo apt install -y httpd-tools```
2. ```htpasswd -nb admin secure_password```
3. Save the Output given for the Above, You'll need this in Step 7. It should look like this:
   ```admin:$apr1$kEG/8JKj$yEXj8vKO7HDvkUMI/SbOO.``` 
4. Make a Directory for Trafik and ``cd`` into it. 
5. ```nano traefik.toml```
6. Paste the above Admin Secure Password in the `users = ["admin:your_encrypted_password"]` section: Keep quotations just replace the `(Your_Encrypted_Password)`
  Replace `"your_domain"` with your domain, keep quotations. 
```
defaultEntryPoints = ["http", "https"]

[entryPoints]
  [entryPoints.dashboard]
    address = ":8080"
    [entryPoints.dashboard.auth]
      [entryPoints.dashboard.auth.basic]
        users = ["admin:your_encrypted_password"]
  [entryPoints.http]
    address = ":80"
      [entryPoints.http.redirect]
        entryPoint = "https"
  [entryPoints.https]
    address = ":443"
      [entryPoints.https.tls]

[api]
entrypoint="dashboard"

[acme]
email = "your_email@your_domain"
storage = "acme.json"
entryPoint = "https"
onHostRule = true
  [acme.httpChallenge]
  entryPoint = "http"

[docker]
domain = "your_domain"
watch = true
network = "pathfinder_default" 
  ```
7. Save the file and move onto the next step.

# Running the Traefik Container
In the Traefik Folder you created follow these steps
1. ``` docker network create pathfinder_default ```
2. ``` touch acme.json ```
3. ``` chmod 600 acme.json ```
4. Copy and past the following into a text editor beforehand and edit `(monitor.your_domain)` to your desired address. Then copy and paste back into your terminal and press `enter` to run the docker. 
 ``` 
docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v $PWD/traefik.toml:/traefik.toml \
  -v $PWD/acme.json:/acme.json \
  -p 80:80 \
  -p 443:443 \
  -l traefik.frontend.rule=Host:monitor.your_domain \
  -l traefik.port=8080 \
  --network pathfinder_default \
  --name traefik \
  traefik:1.7.21-alpine
  
  ```
  
  If all has gone well you should have a Traefik container up and running and set up ready for the next steps.


# Installation
1. Clone `docker-compose.yml` file 
```
sudo wget https://raw.githubusercontent.com/SteveTh3Piirate/pathfinder-docker/master/docker-compose.yml
```
2. Clone the example `.env` file
```
sudo wget https://raw.githubusercontent.com/SteveTh3Piirate/pathfinder-docker/master/.env
```
3. Fill out the `.env` file and start up your instance
```
sudo docker-compose up -d
```

You may need to create the databases for your MYSQL image if using a fresh compose. 
* `sudo docker-compose exec db /bin/bash`
* `mysql -uroot -p`
* `CREATE DATABASE pathfinder;`
* `CREATE DATABASE eve_universe;`
# Setup
1. Navigate to your Pathfinder page, go through setup.
2. Create the databases using the database controls in the setup page.
3. [Import static database.](#Importing-static-database)
4. Import from ESI at the Cronjob section of the setup page.
5. Build Systems data index under `Build search index` in the Administration section of the setup page.
6. At the bottom after your databases are build and filled scroll down and import static data. Since v2.0 manual imports through .zip are no longer necessary.
7. You're live!


# Securing the Setup Page
1. Exec into the pathfinder docker with ```docker exec -it pathfinder /bin/bash```
2. install apache2-utils ```apt-get install apache2-utils```
3. ```apt-get update && apt-get install nano```
4. now ```cd /etc/nginx/sites-enabled/```
5. ```nano default``` And scroll all the way to the bottom and add this next block of code to the file. 
```
location /setup {
   auth_basic "Setup Login";
   auth_basic_user_file /etc/nginx/.setup_pass;
   try_files $uri $uri/ /index.php?$query_string;
  } 
  ```
6. Now create the user/pass for the setup page with the following. Change the Username!
``` 
htpasswd -c /etc/nginx/.setup_pass (YourUsername)
```
7. Removing the Setup Warning.
```
cd /var/www/pathfinder/app/
``` 
8. Nano into pathfinder.ini `nano pathfinder.ini`
9. Scroll down until you see `SHOW_SETUP_WARNING          =   1` Simply Change the 1 to 0. 
10. Now your SetUp page is secure and you wont have a warning all the time. 
11. In future you'd get to the setup page with /setup at the end of your chosen link pathfinder.yourdomain.com/setup


# Correcting HTML_Errors Warning
1. exec into the pathfinder docker container 
```
docker exec -it pathfinder /bin/bash
```
2.
```
apt-get update && apt-get install nano
```
3. cd into the php directory with 
```
cd /etc/php/7.2/fpm/
```
4. Nano into the php.ini with
``` 
nano php.ini
```
5. ctrl+w and search for the first ```html_errors``` and turn the "on" to "off" and search for the next ```html_errors``` and do the same as prior.
6. restart fpm service with ```service php7.2-fpm restart```
7. Check on the setup page that hmtl errors is marked as a 0 instead of a 1. 

# Database character_set_server adjustment
In the docker image the dedfault character_set_server is set to latin 1, here i will tell you how to adjust it to utf8mb4.

1. exec into the docker containing the db in my case its ```docker exec -it pathinfder_db_1 /bin/bash```
2. Make sure to update packages as we will need to install nano so ```apt-get install update```
3. Next install nano ```apt-get install nano```
4. Now cd to the mysqld.conf which should be ```cd /etc/mysql/mysql.conf.d/```
5. Now ```nano mysqld.conf``` 
6. Scroll down to the bottom and add the following to lines on their own line
```
skip-character-set-client-handshake
character-set-server=utf8mb4
```

7. now ctrl+x save the buffer and then exit, now we need to restart mysql with the following
8. ```service mysql restart```
9. refresh your page and note that latin1 has now been replaced with the correct type.

# Intalling Event extension - Can Break Install. Install at Your Own Risk!
1. In the cli type ```docker exec -it pathfinder /bin/bash``` or whatever your docker host name may be.
2. update packages by ```sudo apt-get update```
3. Follow the below Text to install the EventLibrary which will install Event Extension 3.0.4 for Php 7.2
```
apt-get install php7.2-dev
```
```
apt-get install libevent-dev
```

# Install extensions
```
pecl install ev
```
```
pecl install event
```

# Create configurations
```
sudo echo 'extension=ev.so' > /etc/php/7.2/mods-available/ev.ini
```
```
sudo echo 'extension=event.so' > /etc/php/7.2/mods-available/event.ini
```

# Create symlinks
```
sudo ln -s /etc/php/7.2/mods-available/ev.ini /etc/php/7.2/fpm/conf.d/20-ev.ini
```
```
sudo ln -s /etc/php/7.2/mods-available/ev.ini /etc/php/7.2/cli/conf.d/20-ev.ini
```
```
sudo ln -s /etc/php/7.2/mods-available/event.ini /etc/php/7.2/fpm/conf.d/20-event.ini
```
```
sudo ln -s /etc/php/7.2/mods-available/event.ini /etc/php/7.2/cli/conf.d/20-event.ini
```

# Optional - libevent
##WANRING! Segmentation fault on PHP 7.2.11-1+ubuntu18.04.1+deb.sury.org+1 (cli) (built: Oct 24 2019 18:23:23) ( NTS ) ##

```
sudo echo 'extension=libevent.so' > /etc/php/7.2/mods-available/libevent.ini
```
```
sudo ln -s /etc/php/7.2/mods-available/libevent.ini /etc/php/7.2/fpm/conf.d/20-libevent.ini
```
```
sudo ln -s /etc/php/7.2/mods-available/libevent.ini /etc/php/7.2/cli/conf.d/20-libevent.ini
```

# Check modules loaded:

```
php -i | grep -i ev
```
```
php -i | grep -i event
```

# Restart FPM
```
sudo service php7.2-fpm restart
```

During Refresh of setup page you will see an error, dont panic! Refresh your Pathfinder Setup page a few times and you should see the event library installed to 3.0.4
