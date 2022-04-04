API
===

.. autosummary::
   :toctree: generated

**/etc/asceds/home/cert.conf**::

   # don't return certificate to client?
   # empty for fully managed clients, non-empty otherwise
   NO_RETURN=''

   # fqdn name on the certificate
   CLIENT_DOMAIN_NAME=qwes.math.cmu.edu

   # SANS list of the certificate
   CLIENT_SANS_LIST=nero.math.cmu.edu,smtp.math.cmu.edu,webmail.math.cmu.edu,sattva.math.cmu.edu

   # certificate id 
   # empty for managed clients, non-empty for unmanaged clients
   ASCEDS_CERT_ID=''

   # should contain the email address or user name of the requestor for notifications
   # empty for managed clients, non-empty for unmanaged clients
   NO_ASCEDS=''

   # client platform
   # for managed clients it should be linux (for now)
   # for unmanaged clients: unknown
   CLIENT_PLATFORM='linux'

   # client certificate encoding
   # thos should be rsa (default) or ecc
   CLIENT_ENC='rsa'