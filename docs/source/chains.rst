chains
=============


.. _chains :

Setup of certificate manager, one per site 

| (root/sudo access, mailserver, certbot, webserver)
| Cert_manager: asceds-certmanager-setup [-s <cert_manager>]
| Cert_manager: asceds-update-siteconfig -s


1. Initial setup or OS upgrades of clients
    a. managed clients, root/sudo@cert_manager ssh access:
        |    Client: asceds-init -n [-s <cert_manager_URL>]
        |        Client: asceds-update-siteconfig -c
        |        Client: asceds-sans-update -l [-r]
        |        Cert_manager: asceds-client-setup -n -s -c <client>
        |            Cert_manager: asceds-certbot-gencert -e -c <client> -s <sans>
        |            Cert_manager: sudo -u asceds -i asceds-send-cert -c <client>
        |        Client: asceds-services
        |            Client: asceds-service-reconfig -s <service>
        |                Client: asceds-update-siteconfig -q -c

    b. managed clients, no root/sudo@cert_manager ssh access, web interface:
        |    Client: asceds-init [-s <cert_manager_URL>]
        |         Client: asceds-update-siteconfig -c
        |         Client: asceds-sans-update -l [-r]
        |    Cert_manager web interface: asceds_cert.php
        |         Cert_manager: asceds-test-return -c <client>
        |         Cert_manager: asceds-web-propagate -c <client> 
        |             Cert_manager: asceds-client-setup -n -c <client>
        |                 Cert_manager: asceds-certbot-gencert -e -c <client> -s <sans>
        |                 Cert_manager: sudo -u asceds -i asceds-send-cert -c <client>
        |    Client: asceds-services
        |        Client: asceds-service-reconfig -s <service>
        |                Client: asceds-update-siteconfig -q -c

    c. unmanaged clients, web interface:
        |    Cert_manager web interface: request_cert.php
        |        Cert_manager: asceds-web-unmanaged -n -c <client>
        |            Cert_manager: asceds-certbot-gencert -e -c <client> -s <sans>
        |    Cert_manager web interface: host_check.php
        |        Cert_manager: download certificate files
        |    Client: reconfigure services

2. Push automatically renewed certificates to clients

    a. fully managed clients:
        |    Automatic renewal by certbot (systemd)
        |    Cert_manager crontab: asceds-cert-propagate 
        |        Cert_manager: asceds-propagate-certbot 
        |        Cert_manager: sudo -u asceds -i asceds-send-cert -c <client>
        |    Client crontab: asceds-cert-refresh 
        |        Client: asceds-service-reconfig -t
        |            Client: asceds-update-siteconfig -q -c

    b. privately managed clients, root/sudo@cert_manager ssh access:
        |   Automatic renewal by certbot (systemd)
        |   Cert_manager crontab: asceds-cert-propagate
        |       Cert_manager: asceds-propagate-certbot 
        |           Cert_manager: asceds-mailto-alarm
        |   Client: allow r/w ssh access for asceds account from Cert_manager
        |   Cert_manager: sudo -u asceds -i asceds-send-cert -c <client>
        |   Client: asceds-service-reconfig -t
        |       Client: asceds-update-siteconfig -q -c

    c. privately managed clients, no root/sudo@cert_manager ssh access, web interface:
        |    Automatic renewal by certbot (systemd)
        |    Cert_manager crontab: asceds-cert-propagate
        |        Cert_manager: asceds-propagate-certbot 
        |            Cert_manager: asceds-mailto-alarm
        |    Client: allow r/w ssh access for asceds account from Cert_manager
        |    Cert_manager web interface: asceds_cert.php
        |        Cert_manager: asceds-test-return -c <client>
        |        Cert_manager: asceds-web-propagate -c <client> 
        |            Cert_manager: sudo -u asceds -i asceds-send-cert -c <client>
        |    Client: asceds-service-reconfig -t
        |        Client: asceds-update-siteconfig -q -c

    d. unmanaged clients, web interface:
        |    Cert_manager crontab: asceds-cert-propagate
        |        Cert_manager: asceds-propagate-certbot 
        |            Cert_manager: new certificate email notification
        |    Cert_manager web interface: host_check.php
        |        Cert_manager: download certificate files
        |    Client: reconfigure services

3. SANs refresh (manual sequence)

    a. managed clients, root/sudo@cert_manager ssh access:
        |    Client: asceds-sans-update [-r] [-a <hostname>,...]  [-d <hostname>,...]
        |    Cert_manager: asceds-client-setup -n -c <client>
        |        Cert_manager: asceds-certbot-gencert -e -c <client> -s <sans>
        |        Cert_manager: sudo -u asceds -i asceds-send-cert -c <client>
        |    Client: asceds-service-reconfig -t
        |        Client: asceds-update-siteconfig -q -c

    b. managed clients, no root/sudo@cert_manager ssh access, web interface:
        |    Client: asceds-sans-update [-r] [-a <hostname>,...]  [-d <hostname>,...]
        |    Cert_manager web interface: asceds_cert.php
        |        Cert_manager: asceds-test-return -c <client>
        |        Cert_manager: asceds-web-propagate -c <client>
        |            Cert_manager: asceds-client-setup -n -c <client>
        |                Cert_manager: asceds-certbot-gencert -e -c <client> -s <sans>
        |                Cert_manager: sudo -u asceds -i asceds-send-cert -c <client>
        |    Client: asceds-service-reconfig -t
        |        Client: asceds-update-siteconfig -q -c

    c. unmanaged clients, web interface:
        |    Cert_manager web interface: request_cert.php
        |        Cert_manager: asceds-web-unmanaged -n -c <client>
        |            Cert_manager: asceds-certbot-gencert -e -c <client> -s <sans>
        |    Cert_manager web interface: host_check.php
        |        Cert_manager: download certificate files
        |    Client: reconfigure services

4. Certificate revoke

    a. managed clients, root/sudo@cert_manager ssh access:
        |    Cert_manager: asceds-certbot-revoke -e -c <client>

    b. any client, web interface:
        |    Cert_manager web interface: revoke_cert.php
        |        Cert_manager: asceds-web-unmanaged -r -c <client>
        |            Cert_manager: asceds-certbot-revoke -e -c <client>

5. Site config file management

    a. create: 
        |    Cert_manager: asceds-certmanager-setup [-s <certmanager_cname>]
        |        Cert_manager: asceds-update-siteconfig -s -> 
        |                        asceds-utils/asceds-build-siteconf

    b. update:
        |    Cert_manager: asceds-update-siteconfig -s -> 
        |                    asceds-utils/asceds-build-siteconf

    c. initialize on managed clients:
        |    Client: asceds-init [-s <certmanager>]
        |        Client: asceds-update-siteconfig -c ->
        |                    asceds-utils/asceds-parse-siteconf

    d. automatic propagate trough http on fully managed clients:
        |    Cert_manager crontab: asceds-cert-propagate
        |        Certmanager: asceds-propagate-certbot
        |            Cert_manager: sudo -u asceds -i asceds-send-cert -c <client>
        |    Client crontab: asceds-cert-refresh 
        |        Client: asceds-service-reconfig -t
        |            Client: asceds-update-siteconfig -q -c

    e. manual propagate on privately managed clients:
        |    Client: allow r/w ssh access for asceds account from Cert_manager
        |    Client: asceds-update-siteconfig -c

6. Web interface

    Apache authenticated web interface on Cert_manager for:
        a. Unmanaged client, generate new certificate:
            |    Cert_manager web interface: request_cert.php
            |        Cert_manager: asceds-web-unmanaged -n -c <client>
            |            Cert_manager: asceds-certbot-gencert -e -c <client> -s <sans>
            |    Cert_manager web interface: host_check.php
            |        Cert_manager: download certificate files
            |    Client: reconfigure services

        b. Managed client, propagate ASCEDS data (including generate certificate):
            |    Cert_manager web interface: asceds_cert.php
            |        Cert_manager: asceds-test-return -c <client>
            |        Cert_manager: asceds-web-propagate -c <client>
            |            Cert_manager: asceds-client-setup -n -c <client>
            |                Cert_manager: asceds-certbot-gencert -e -c <client> -s <sans>
            |                Cert_manager: sudo -u asceds -i asceds-send-cert -c <client>

        c. Revoke certificate (any client):
            |    Cert_manager web interface: revoke_cert.php
            |        Cert_manager: asceds-web-unmanaged -r -c <client>
            |            Cert_manager: asceds-certbot-revoke -e -c <client>
