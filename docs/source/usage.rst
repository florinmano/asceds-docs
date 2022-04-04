Usage
=====

.. _quickstart:

* The main body of documentation is installed by default in 
  /usr/share/doc/asceds/doc

* Each script displays its function and usage with the "-h" option.
* The asceds command lists documentation and status for the current computer.
* A paper describing the general idea and the functional structure is in
  the public directory of the web interface, by default in
  /usr/share/asceds/html/public/asceds.pdf
  or at https://<your.cert.manager>/public/asceds.pdf

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

Action Chains
------------

1. Initial setup or OS upgrades of clients

    a. managed clients, root@cert_manager ssh access:
    Client: asceds-init -n -s <cert_manager>
         Client: asceds-sans-update -l [-r]
         Cert_manager: asceds-client-setup -q -n -s -c <client>
              Cert_manager: asceds-certbot-gencert -q -e -c <client> -s <sans>
              Cert_manager: asceds-send-cert -q -c <client>
         Client: asceds-services
              Client: asceds-service-reconfig -q -s <service>


    b. managed clients, no root@cert_manager ssh access, web interface:
    Client: asceds-init [-r] -s <cert_manager>
         Client: asceds-sans-update -l [-r]
    Cert_manager web interface: asceds_cert.php
         Cert_manager: asceds-test-return -c <client>
         Cert_manager: asceds-web-propagate -q -c <client> 
              Cert_manager: asceds-client-setup -q -n -c <client>
                   Cert_manager: asceds-certbot-gencert -q -e -c <client> -s <sans>
                   Cert_manager: asceds-send-cert -q -c <client>
    Client: asceds-services
         Client: asceds-service-reconfig -q -s <service>

    c. unmanaged clients, web interface:
    Cert_manager web interface: request_cert.php
         Cert_manager: asceds-web-unmanaged -q -n -c <client>
              Cert_manager: asceds-certbot-gencert -e -q -c <client> -s <sans>
    Cert_manager web interface: host_check.php
         Cert_manager: download certificate files
    Client: reconfigure services

2. Push automatically renewed certificates to clients

    a. fully managed clients:
    Automatic renewal by certbot (systemd)
    Cert_manager crontab: asceds-cert-propagate 
         Cert_manager: asceds-propagate-certbot -q
              Cert_manager: asceds-send-cert -q -c <client>
    Client crontab: asceds-cert-refresh 
         Client: asceds-service-reconfig -q -t

    b. privately managed clients, root@cert_manager ssh access:
    Automatic renewal by certbot (systemd)
    Cert_manager crontab: asceds-cert-propagate
         Cert_manager: asceds-propagate-certbot -q
              Cert_manager: asceds-mailto-alarm
    Client: allow rw ssh access for asceds account from Cert_manager
    Cert_manager: asceds-send-cert -q -c <client>
    Client: asceds-service-reconfig -q -t

    c. privately managed clients, no root@cert_manager ssh access, web interface:
    Automatic renewal by certbot (systemd)
    Cert_manager crontab: asceds-cert-propagate
         Cert_manager: asceds-propagate-certbot -q
              Cert_manager: asceds-mailto-alarm
    Client: allow rw ssh access for asceds account from Cert_manager
    Cert_manager web interface: asceds_cert.php
         Cert_manager: asceds-test-return -c <client>
         Cert_manager: asceds-web-propagate -q -c <client> 
              Cert_manager: asceds-send-cert -q -c <client>
    Client: asceds-service-reconfig -q -t

    d. unmanaged clients, web interface:
    Cert_manager crontab: asceds-cert-propagate
         Cert_manager: asceds-propagate-certbot -q
              Cert_manager: new certificate email notification
    Cert_manager web interface: host_check.php
         Cert_manager: download certificate files
    Client: reconfigure services

3. SANs refresh (manual sequence)

    a. managed clients, root@cert_manager ssh access:
    Client: asceds-sans-update [-r] [-a <hostname>,...]  [-d <hostname>,...]
    Cert_manager: asceds-client-setup -q -n -c <client>
         Cert_manager: asceds-certbot-gencert -q -e -c <client> -s <sans>
         Cert_manager: asceds-send-cert -q -c <client>
    Client: asceds-service-reconfig -q -t

    b. managed clients, no root@cert_manager ssh access, web interface:
    Client: asceds-sans-update [-r] [-a <hostname>,...]  [-d <hostname>,...]
    Cert_manager web interface: asceds_cert.php
         Cert_manager: asceds-test-return -c <client>
         Cert_manager: asceds-web-propagate -q -c <client>
              Cert_manager: asceds-client-setup -q -n -c <client>
                   Cert_manager: asceds-certbot-gencert -q -e -c <client> -s <sans>
                   Cert_manager: asceds-send-cert -q -c <client>
    Client: asceds-service-reconfig -q -t

    c. unmanaged clients, web interface:
    Cert_manager web interface: request_cert.php
         Cert_manager: asceds-web-unmanaged -q -n -c <client>
              Cert_manager: asceds-certbot-gencert -q -e -c <client> -s <sans>
    Cert_manager web interface: host_check.php
         Cert_manager: download certificate files
    Client: reconfigure services

4. Certificate revoke

    a. managed clients, root@cert_manager ssh access:
    Cert_manager: asceds-certbot-revoke -q -e -c <client>

    b. any client, web interface:
    Cert_manager web interface: revoke_cert.php
         Cert_manager: asceds-web-unmanaged -q -r -c <client>
              Cert_manager: asceds-certbot-revoke -q -e -c <client>

5. Web interface

    Apache authenticated web interface on Cert_manager for:
    a. Unmanaged client, generate new certificate:
    Cert_manager web interface: request_cert.php
        Cert_manager: asceds-web-unmanaged -q -n -c <client>
             Cert_manager: asceds-certbot-gencert -e -q -c <client> -s <sans>
    Cert_manager web interface: host_check.php
        Cert_manager: download certificate files
    Client: reconfigure services

    b. Managed client, propagate ASCEDS data (including generate certificate):
    Cert_manager web interface: asceds_cert.php
        Cert_manager: asceds-test-return -c <client>
        Cert_manager: asceds-web-propagate -q -c <client>
             Cert_manager: asceds-client-setup -q -n -c <client>
                  Cert_manager: asceds-certbot-gencert -q -e -c <client> -s <sans>
                  Cert_manager: asceds-send-cert -q -c <client>

    c. Revoke certificate (any client):
    Cert_manager web interface: revoke_cert.php
        Cert_manager: asceds-web-unmanaged -q -r -c <client>
             Cert_manager: asceds-certbot-revoke -q -e -c <client>

Cron Jobs
------------

**certificate manager**
    asceds-cert-propagate
       asceds-propagate-certbot -q
          check if new key/cert
          copy in ~asceds/<client>/
          asceds-send-cert -q -c <client>
    asceds-web-actions
       asceds-web-unmanaged -q -n -c <client> --> newcert
       asceds-web-unmanaged -q -r -c <client> --> revoke
       asceds-web-propagate -q -c <client>   --> generate and propagate

**client**
    asceds-cert-refresh
       asceds-service-reconfig -q -t
          check if service reconfig trigger ~asceds/.asceds-reconf
          asceds-service-reconfig
          ${RM} ~asceds/.asceds-reconf


EMAIL
------------

Email capabilities (mailutils+postfix) are required for 
certificate managers and optional for clients.

Email ALARM (lib/asceds-utils: asceds-mailto-alarm):

If ${ALARM} is empty, no email will be sent.
Email messages are sent to the ALARM address (admin or ticketing system) if:
a. if expired cert 
      cert_manager: asceds-propagate-certbot --> asceds-expiration-test
      local:        asceds-service-reconfig  --> asceds-expiration-test              
b. systems with noreturn policy (ro like pi-kvm) --> asceds-propagate-certbot
c. certs cannot be moved (network or ssh access problems) --> asceds-send-cert 
d. TBI (not sure if needed) if errors --> create digest

Email messages are sent to the requestor (all from the certificate manager):
   * NO_ASCEDS in cert.conf file
   * REQUESTED_BY in web request file
if:
a. revoked certificate for unmanaged client
       asceds-web-unmanaged -r --> if running the queue (not with -c)
b. generate/renew certificate for unmanaged client
       asceds-web-unmanaged -n --> if running the queue (not with -c)
c. certificate was automatically renewed ny certbot for unmanaged client
       asceds-propagate-certbot --> for unmanaged clients
d. certificate was generated/renewed through the website request 
   for managed client
       asceds-web-propagate --> if running the queue (not with -c)
       
       
Scripts
------------
*** asceds
status/info and general description
(interactive only), runs as root, ${ASCEDSBINDIR}/
asceds [-h] [-d] [-l] 
Shows a summary of current status of certificates and managed services
-h --> display usage and exit
-d --> describes in detail asceds utilities and usage
-l --> shows summary of warnings/errors from log files

*** asceds-authorized-domains
manages domains which are authorized by the CA to get keys
using this server's EABKID/EABHMACKEY
asceds-authorized-domains [-h] [-a <domainname>,...] [-d <domainname>,...]
-a <domainname>[,<domainname>,...] adds comma separated domain names
-d <hostname>[,<hostname>,...] deletes comma separated domain names
If no argument, displays current configuration and goes into interactive mode;
Deletes domain names from ${ASCEDSETCDIR}/asceds-certbot.conf
   and ${ASCEDSWEBSITE}/etc/users.php (if it exists)
Adds domain names to ${ASCEDSETCDIR}/asceds-certbot.conf
   and ${ASCEDSWEBSITE}/etc/users.php (if it exists)

*** asceds-certmanager-setup
setup the certificate manager
(interactive only), runs as root, ${ASCEDSCBDIR}/
asceds-certmanager-setup [-h] [-w]
-h --> display usage and exit
-w --> set up the web interface
Sets the ALARM email address.
Checks for certbot; if not found, requests certbot install.
Checks for mailer; if not found, requests mailer install.
Collects certbot credentials and authorized domains in asceds-certbot.conf.
Creates asceds user ssh keys.
Creates symlinks to certobot-related ASCEDS scripts.
Sets cron asceds-cert-propagate for driving asceds-propagate-certbot 
   to copy renewal certs to ~asceds/<domainname> and propagate
   them to managed clients through asceds-send-cert.
Web interface setup:
   Gets info: organization name, website url, request execution methods;
   Customizes config.php and asceds.php;
   Copies and reconfigs asceds-site-apache2.conf;
   Initializes request log file;
   Creates ssh keys for requests by ssh;
   Configures ASCEDSWEBSITE in asceds-local.conf, make sure it ends with a /;
   Sets crontab for running the request queues generated by the web interface.

*** asceds-client-setup
assist initial setup of a new client
to be run on the cert manager as root
through asceds-init on the client
(interactive only), runs as root, ${ASCEDSBINDIR}/
asceds-client-setup [-h] [-n] [-s] [-q] -c <client_name>
-h --> display usage and exit
-c <client_name> --> client to be setup; print usage if missing
-q --> no question asked; sends output to the logs
-n --> generate new certificate
-s --> clean old keys of the client in ~asceds/.ssh/known_hosts
Scp client's cert.conf to ${ASCEDSHOMEDIR}/${DOMAIN_NAME}_cert.conf
Validity checks
   DOMAIN_NAME --> CLIENT_DOMAIN_NAME
   SANS_LIST --> CLIENT_SANS_LIST
Asks if to generate new certificates; generate.
If -s, transfers cert back: asceds-send-cert -c <client_name>

*** asceds-init
initializes certificate subscription
(interactive only), runs as root, ${ASCEDSBINDIR}/
asceds-init [-h] [-n] [-r] [-s <certmanager>]
Initializes certificate subscription
-n --> generate new certificate
-s <certmanager> --> the certificate manager
-r --> set noreturn policy, client is read-only
-h --> display usage and exit
Figures out which certificate manager to use:
  if no cert manager is mentioned in asceds-local.conf or
  in the command line, ask for a name;
  validate and confirm before proceeding.
Sets ALARM address if a mailer is installed.
Generates (CLIENT_DOMAIN_NAME, CLIENT_SANS_LIST) --> ~asceds/cert.conf
Copies logrotate asceds config file.
Creates renewal crontab: asceds-cert-refresh
Gets the preconfigured authorized_keys if it doesn't exist, 
      using wget and the web interface.
Work done on root@certmanager if access available, 
      otherwise prints instructions and exits:
  If ~asceds/.ssh/authorized_keys still doesn't exist:
    Gets asceds@cert_manager's public ssh key into the local authorized_keys.
  Runs setup script: asceds-client-setup [-n] -s -c ${CLIENT_DOMAIN_NAME}
Selects and configures services which depend on certificates and 
      have reconfig scripts: asceds-services
Removes the reconf trigger ${ASCEDSHOMEDIR}/.asceds-reconf

*** asceds-sans-update
change SAN values, recreate certificate, reconfigure services
(interactive only), runs as root, ${ASCEDSBINDIR}/
asceds-sans-update [-h] [-l] [-r] [-a <hostname>,...] [-d <hostname>,...]
-h --> display usage and exit
-r --> set noreturn policy, client is read-only
-l --> local mode (appropriate for asceds-init)
-a <hostname>[,<hostname>,...] adds comma separated SAN values
-d <hostname>[,<hostname>,...] deletes comma separated SAN values
If no argument, displays current configuration and goes into interactive mode;
Deletes SAN values from /etc/hosts, /etc/postfix/main.cf, /etc/ssl/openssl.cnf
Checks DNS records of SANs to be added; configures with the host IP address
Adds SAN values to /etc/hosts, /etc/postfix/main.cf, /etc/ssl/openssl.cnf
Refreshes ~asceds/cert.conf
Provides instructions to manually complete the action.

*** asceds-send-cert
send certs back to the clients
(interactive + script), runs as asceds, ${ASCEDSBINDIR}/
used by asceds-init, also when certificates are generated
(renewal, add-sans, del-sans)
asceds-send-cert [-h] [-q] -c <client_name>
-h --> display usage and exit
-q --> no question asked; sends output to the logs in ${ASCEDSLOGDIR}/
-c <client_name> --> transfer the certificates to client_name
Checks if the certificates exist.
SCP the certificates to asceds@<client_name>:certs/
Sets the trigger for service reconfiguration on <client_name>:.asceds-reconf

*** asceds-service-reconfig
copies certs in the right locations and restart services in ${ASCEDSSERVCONF}
(interactive + script), runs as root, ${ASCEDSBINDIR}/
needs to be run by trigger every time the certs are changing
    cron job <-- ~asceds/.asceds-reconf
(init, renewal, add-sans, del-sans)
asceds-service-reconfig [-q] [-t] [-h] [-s <service>]
-q --> no question asked; sends output to the logs in ${ASCEDSLOGDIR}/
-t --> reconfigure services only if trigger file .asceds-reconf exists
-s service --> reconfigures the <service> only (one service);
               in cron mode, -s options are ignored
-h --> display usage and exit
Checks if the certificate in ~asceds/certs/ is not expired;
Reads parameters (CLIENT_DOMAIN_NAME, CLIENT_SANS_LIST) <-- ~asceds/cert.conf ;
Copies cert/key from ~asceds/certs/ --> /etc/ssl ; fix permissions;
Reconfigures/restarts services through ${ASCEDSSERVCONF}/*.sh ;
     to avoid sourcing: rename so it doesn't match *.sh;
     available service templates: ${ASCEDSSERVDIR}/*.sh.proto;
Removes the reconfig trigger file ${ASCEDSHOMEDIR}/.asceds-reconf

*** asceds-services
configure services handled by ASCEDS
(interactive only), runs as root, ${ASCEDSBINDIR}/
asceds-services [-h] [-a <service>] [-d <service>]
Shows available services to be updated by
      scripts in ${ASCEDSSERVDIR}/*.sh.proto, 
      and active service scripts ${ASCEDSSERVCONF}/*.sh
With no argument, offers a simple interface to activate/deactivate services
      by typing in a space separated list.
-h --> display usage and exit
-d <service> --> deactivate one service (before activating)
                 multiple [-d <service>] are allowed
-a <service> --> activate or refresh one service, 
                 multiple [-a <service>] are allowed,
                 each activated service is reconfigured through 
                      asceds-service-reconfig -q -s <service> 

*** asceds-web-user
Adds/removes/resets web users
asceds-web-user [-h] [-p] [-a <username>] [-d <username>] [-m <username>]
-h --> display usage and exit
-p --> use simple auth password file ${ASCEDSWEBSITE}/cert/.htpasswd
-a <username> --> adds username to the website;
-d <username> --> deletes user from the website;
-m <username> --> modifies (deletes, re-adds) user; 
Options -a/-d/-m are mutually exclusive.
Wildcard ALL gives user authority over all available domains.
Sets/removes list of authorized domains in ${ASCEDSWEBSITE}/etc/users.php
Sets/removes user password for simple auth in ${ASCEDSWEBSITE}/cert/.htpasswd
   (if it exists or -p)
If file ${ASCEDSWEBSITE}/cert/.htpasswd exists, -p is forced.
<Username> must be valid email address used for sending notifications,
   or a mail alias is set.


*** asceds-certbot-gencert
generates new certs using certbot
(interactive + script), runs as root, ${ASCEDSCBDIR}/
asceds-certbot-gencert [-h] [-e] [-d] [-q] -c <client_name> [-s <SAN1>,<SAN2>,...]
Generates new certs using certbot. Options:
-h --> display usage and exit
-e --> execute the certbot command (just echo the command by default)
-q --> no question asked; sends output to the logs in ${ASCEDSLOGDIR}/
-d --> dry-run (off by default)
-c <client_name> --> name of the computer requesting a certificate
-s <SAN1>,... --> comma-separated alternate names of the computer requesting 
                   a certificate not including domain_name
Figures out certbot's options based on the command line input.
Checks if cert manager is authorized to generate requested certificates.
Generates new certificates using certbot
   Certs/key are generated by certbot and go in /etc/letsencrypt/live/<cert_name>/;
   Checks if fresh certificates were generated.
   Certs/key are copied to ${ASCEDSHOMEDIR}/<client_name>/ and chown asceds:asceds;
   If client is unmanaged, copies the cert files in the web dir and chown www-data.

*** asceds-certbot-revoke
revoke certs
(interactive + script), runs as root, ${ASCEDSCBDIR}/
asceds-certbot-revoke [-h] [-e] [-q] -c <client_name>
-h --> display usage and exit
-q --> no question asked; sends output to the logs in ${ASCEDSLOGDIR}/
-c <client_name> --> cert for <client_name> will be revoked (one per use)
-e --> execute certbot revoke (just echo the command by default)
Checks if certbot is installed and certificates exist.
Displays status of the client and of the certificate files.
Displays certbot command to be executed, or it executes it (if -e).
Removes certificate files from ~asceds/<client_name>/
If client is unmanaged, removes the cert files from the web dir.

*** asceds-propagate-certbot
detect and propagate new certificates based on automatic renewals by certbot
(script by cron, interactive), runs as root, ${ASCEDSCBDIR}/
asceds-propagate-certbot [-h] [-q] [-l] [-r] [-c <client>]
-h --> display usage and exit
-q --> no question asked; sends output to the logs in ${ASCEDSLOGDIR}/
-l --> keep the certificate local, don't trigger asceds-send-cert
-r --> ignore noreturn policy and try to send the certificate
       (mutually exclusive with -l; needs -c)
-c <client> --> propagate only for <client> 
Looks for renewed certs;
Copies renewd certs to ~asceds/<client>/; chown asceds:asceds
If client is unmanaged:
   Copies the cert files in the web dir for download.
   Sends email to requestor for downloading the renewed cert files.
If client is managed:
   Creates transfer certs flag ~asceds/<client>/.newcerts.
   If fully managed client and no -l option, 
      or if privately managed client with -r option:
         Sends cert files to the client through asceds-send-cert.
   If privately managed client, sends email to ALARM announcing new cert.
Tries to re-send certs to fully managed clients if .newcerts is present.


*** asceds-test-return
Tests read/write scp access to asceds@client
asceds-test-return [-h] -c <client_name>
-h --> display usage and exit
-c <client_name> --> client name to test 
Outputs: success = read/write successful 
         readonly = read-only access 
         noaccess = no ssh access at all

*** asceds-web-unmanaged 
Performs actions requested through the web interface for unmanaged clients 
asceds-web-unmanaged [-h] [-q] [-n] [-r] [-c <client_name>]
-h --> display usage and exit
-q --> no question asked; sends output to the logs in ${ASCEDSLOGDIR}/
-n --> new certificate actions only
-r --> revoke actions only
-c <client_name> --> act only for <client_name>
If both types of requests (new certificate and revoke) are found,
revoke requests are performed first.
Revoke: requests in ${ASCEDSWEBDIR}/cert_queue/*.rev:
   read .rev file;
   log: date Web user <username> revoked certificate for <hostname>;
   asceds-certbot-revoke -q -e -c <hostname>;
   remove .rev file;
Newcert: requests in ${ASCEDSWEBDIR}/cert_queue/*.cert:
   read .cert file;
   create/adjust ~asceds/<hostname>_cert.conf;
        populate populate ASCEDS_CERT_ID if empty;
   log: date Web user <username> requested certificate for <hostname>
        with SANs <sans>, client type changed to: <ack>;
   asceds-certbot-gencert -e -q -c <hostname> -s <sans>;
   remove .cert file.

*** asceds-web-propagate
Propagates ASCEDS data requested through the web interface for managed clients
asceds-web-propagate [-h] [-q] [-c <client>]
-h --> display usage and exit
-q --> no question asked; sends output to the logs in ${ASCEDSLOGDIR}/
-c <client> --> propagate only <client> 
Propagate: requests in ${ASCEDSWEBDIR}/cert_queue/*.asceds
If run the queue (by cron), sends notification by email to requestors.
If cert.conf exists and ASCEDS_CERT_ID not empty, 
   removes web certificate files.
If ~asceds/<hostname>/.newcerts is present, just sends the certificate files
   through asceds-send-cert -c <client> (for atomic operation); 
Else, performs a full setup (generates new certificate and sends it)
   through asceds-client-setup -n -c  <client>.
Removes request file.


Triggers
------------


Action triggers:

Transfer certs flag:
asceds@<certmanager>:~asceds/<hostname>/.newcerts
* signals when new certs are copied from /etc/letsencrypt/live/<hostname>/
  into ~asceds/<hostname>/ for managed clients.
* created by: asceds-propagate-certbot, asceds-certbot-gencert.
* deleted by: asceds-send-cert if the transfer was successful.
* used by: asceds-propagate-certbot to send certificates to clients;
           asceds-web-propagate: to decide if a new certificate is generated;
               it allows the web interface to propagate automatic renewals or
               to generate and propagate new certificates for managed clients. 

Reconfigure service flag:
asceds@<client>:~asceds/.asceds-reconf
* signals when new certs were copied successfully from <certmanager>
  and the services need to be reconfigured with the new certificates.
* created by: asceds-send-cert if the transfer was successful.
* deleted by: asceds-init, asceds-service-reconfig with no -s option.
* used by: asceds-service-reconfig to reconfigure services.


Client Type Change
------------
What happens during client type transformations:
client types have sense only if the web interface is used;
otherwise only managed clients can be handled;
so client type conversion should happen as close as possible 
to the web code.

revoke:
  managed -> unmanaged: all certificate files are deleted
     asceds-web-unmanaged -r -> asceds-certbot-revoke
  unmanaged -> managed: all certificate files are deleted
     asceds-web-unmanaged -r -> asceds-certbot-revoke

generate (for unmanaged):
  managed -> unmanaged: request_cert.php -> 
     asceds-web-unmanaged -n -> populate ASCEDS_CERT_ID if empty

propagate (for managed):
  unmanaged -> managed: asceds_cert.php -> 
     asceds-web-propagate -> removes web certificate files if
                             ASCEDS_CERT_ID not empty


Default Directories
------------

ASCEDSHOME="/usr/lib/asceds"
ASCEDSWEBDIR="/usr/share/asceds"
ASCEDSETCDIR="/etc/asceds"
ASCEDSDOCDIR="/usr/share/doc/asceds/doc"
ASCEDSLOGDIR="/var/log/asceds"
ASCEDSWEBHISTFILE="${ASCEDSLOGDIR}/request.history"
ASCEDSHOMEDIR="${ASCEDSETCDIR}/home"
ASCEDSBINDIR="${ASCEDSHOME}/bin"
ASCEDSCONFDIR="${ASCEDSHOME}/etc"
ASCEDSLIBDIR="${ASCEDSHOME}/lib"
ASCEDSCBDIR="${ASCEDSHOME}/certbot"
website shell scripts="${ASCEDSHOME}/website"
ASCEDSSERVDIR="${ASCEDSHOME}/bin/service.d"
ASCEDSSERVCONF="${ASCEDSETCDIR}/service.d"



