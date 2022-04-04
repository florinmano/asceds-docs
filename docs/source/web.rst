
Important Notes

1. The public alias/dir is supposed to be unsecured.

2. If the website is not located at the root of the URL, the link to
   public/authorized_keys in asceds_cert.php may not work properly (test
   scenarios).

Website
=======

\**\* Login (shibboleth or local) username (determines authorized
domains for certs)

\**\* Helper files: css/cert.css ../etc/asceds.php ../etc/config.php
../etc/users.php header.php footer.php

\**\* Navigation index.php –> host_check.php [client_type]
[=independent] O-> edit_cert.php –> request_cert.php OOO> host_check.php
[=asceds] O-> asceds_cert.php –> revoke_cert.php

\**\* Pages –1. index.php: ASCEDS certificate management system
(index.php) Purpose: \* Entry point for identifying the client and for
the general presentation of the ASCEDS architecture.

Info/downloads: \* All domains available to the cert manager \* Domains
authorized to the user

Input: username

Collects (all mandatory): \* client_type: managed/unmanaged –>
asceds/independent \* hostname: text \* domain: select from domains
authorized to the user

Output (GET to index.php): client_type, hostname, domain

–2. host_check.php: ASCEDS certificate inquiry Purpose: \* Presents
information about existing certificate for the client, if any. \* Offers
links to existing certificate files for unmanaged clients.

Info/downloads: if the client has a current certificate: \* certificate
validity (effective/expiration) \* SANS list \* links to existing
certificate files for unmanaged clients

Input (GET from index.php,request_cert.php): client_type, hostname,
domain

Collects: \* client type changing acknowledgement (reguired if any
change) –> propagates to logs through request_cert.php,asceds_cert.php
ack = same (no change) tonotmanaged (from managed to unmanaged)
tomanaged (from unmanaged to managed)

Output1 (POST to edit_cert.php if client_type=independent): hostname,
domain, sans, ack, addhost=’‘, adddomain=’‘, delhost=’’

Output1 (POST to asceds_cert.php if client_type=asceds): hostname,
domain, sans, ack

Output2 (POST to revoke_cert.php): hostname, domain, sans

–3. edit_cert.php: ASCEDS new certificate edit Purpose: \* Edits the
SANS list for the new certificate

Info/downloads: \* Hostname (fqdn) \* Current SANS list, as configured
so far \* Number of hostnames in the SANS list

Input (POST from host_check.php,edit_cert.php): hostname, domain, sans,
ack, addhost, adddomain, delhost

Collects: \* Host name to add to the SANS list: text \* Domain of the
host above: select from domains authorized to the user \* Full hostname
to remove from the SANS list: select from current SANS list elements

Output1 (POST to edit_cert.php): hostname, domain, sans, ack, addhost,
adddomain, delhost

Output2 (POST to request_cert.php): hostname, domain, sans, ack

–4. request_cert.php: ASCEDS new certificate create Purpose: \* Creates
a new certificate immediately (by ssh) or after a while (by cron) –>
asceds-web-unmanaged -q -n -c –> asceds-certbot-gencert -q -e -c -s \*
Copies the certificate files to /keys// \* Updates cert.conf info if
needed

Info/downloads: \* (if by cron) When are the certificate files available
for download

Input (POST from edit_cert.php): hostname, domain, sans, ack, addhost,
adddomain, delhost

Collects: nothing

Output1 (request file .cert): REQUEST_DATE= CLIENT_DOMAIN_NAME=
REQUESTED_BY= CLIENT_SANS_LIST= ACK_CHANGE_TYPE=

Output2 (log file /var/log/asceds/request.history): date, full hostname,
username, sans, ack log: Web user requested certificate for with SANS ,
client type changed to:

–5. revoke_cert.php Purpose: \* Revokes current certificate immediately
(by ssh) or after a while (by cron) –> asceds-web-unmanaged -q -r -c –>
asceds-certbot-revoke -q -e -c

Info/downloads: \* (if by cron) Timeline for certificate revocation

Input (POST from host_check.php): hostname, domain, sans

Collects: nothing

Output1 (request file .rev): REQUEST_DATE= CLIENT_DOMAIN_NAME=
REQUESTED_BY=

Output2: full hostname, username \* log: Web user revoked certificate
for

–6. asceds_cert.php Purpose: \* Creates certificate files for managed
clients; \* propagates certificate files to the clients O->
asceds-test-return -c –> asceds-web-propagate q -c –>
asceds-client-setup -q -n -c –> asceds-certbot-gencert -q -e -c -s –>
asceds-send-cert -q -c

Info/downloads: \* (if by cron) Timeline for data propagation

Input (POST from host_check.php): hostname, domain, sans, ack

Collects: nothing

Output1 (request file .asceds): REQUEST_DATE= CLIENT_DOMAIN_NAME=
REQUESTED_BY= ACK_CHANGE_TYPE=

Output2 (log file /var/log/asceds/request.history): date, full hostname,
username, ack log: Web user propagated asceds data for

Output3 (POST to asceds_cert.php): hostname, domain, sans, ack \* loop
to fix rw acces to asceds@client

.. _section-1:

Website accounts (simple auth) login should be valid email of requestors

certadmin@localhost:q1q2q3a4 -> we’ll make it admin
certadmin2@localhost:q1q2q3a5 -> math,phys,logic
certadmin3@localhost:q1q2q3a6 -> chem certadmin4@localhost:q1q2q3a7 ->
bio,mbic,mcs certadmin5@localhost:q1q2q3a8 -> phys
certadmin6@localhost:q1q2q3a9 -> nothing

htpasswd [-c] .htpasswd certadmin?

.. _section-2:
