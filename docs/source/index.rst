Welcome to ASCEDS's documentation!
===================================

**ASCEDS** is Certificate Distribution System Developed by Florin Manolache at Carnegie Mellon University for interfacing certbot with InCommon SSL certificates, and automatically distributing them to the servers on the network.

This software assists with setting up an automatic SSL certificate distribution system for certificates which can be obtained from a provider offering an ACME interface. 
The system consists of

  - a certificate manager which can get SSL certificates for wildcard hostnames in a set of domains, through certbot;
  - a set of clients getting automatic certificate renewal or simple manual reconfiguration of SANs; services depending on certificates are reconfigured/restarted automatically upon renewal
  - a web interface for requesting certificates.

ASCEDS is released under GPLv2 or any later version.

Check out the :doc:`usage` section for further information, including
how to :ref:`installation` the project.

.. note::

   This project is under active development.

Contents
--------

.. toctree::

   usage
   api
   build
