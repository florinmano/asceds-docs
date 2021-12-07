Usage
=====

.. _installation:


Quick start
------------

Case 1: Certificate manager

* install and configure dependencies: snap, certbot, apache2
* install asceds
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

Certificates
------------

# on cert manager
.. code-block:: console
   /etc/letsencrypt/live/${DOMAIN_NAME}/ --> /etc/asceds/home/${DOMAIN_NAME}/
       for unmanaged clients --> /usr/share/asceds/cert/keys/${ASCEDS_CERT_ID}/
   privkey.pem   : the private key for the certificate
   fullchain.pem : the certificate chain used in most server software
   cert.pem      : certificate alone

# on client
.. code-block:: console
   privkey.pem --> PRVKEY=/etc/ssl/private/${CLIENT_DOMAIN_NAME}.key
   chgrp ssl-cert ${PRVKEY}
   chmod o-rwx ${PRVKEY}
   fullchain.pem --> CERTCHAIN=/etc/ssl/certs/${CLIENT_DOMAIN_NAME}.pem
   chmod a+r ${CERTCHAIN}


