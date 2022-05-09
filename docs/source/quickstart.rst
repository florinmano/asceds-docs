Installation
============

Run

* install and configure dependencies: snap, certbot, apache2
* install asceds

    echo "deb [trusted=yes] https://cert.mcs.cmu.edu/debian ./" | sudo tee -a /etc/apt/sources.list
    sudo apt update
    sudo apt install asceds

Case 1: Certificate manager

* initialize the certificate manager: asceds-certmanager-setup -w
* configure the client (Case 2 below)
* customize the website: 

  * edit the apache2 site configuration file typically in
    /etc/apache2/sites-available/asceds.conf
  * decide on the authentication and add the right .htaccess
    (see examples in /usr/share/doc/asceds/examples/site-*)
  * edit the web php configuration in /usr/share/asceds/etc/config.php
    (see examples in /usr/share/doc/asceds/examples/php-etc)
  * enable the website in apache2: a2ensite asceds
* add authorizied domains: asceds-authorized-domains -a <domainname>
* add authorized users: asceds-web-user -a <username>

Case 2: Client

* install asceds (no need for snap or certbot)
* get info: certificate manager name, root password/sudo account/password
* asceds-init -n -s <your.cert.manager>

  -n --> generates a new certificate
