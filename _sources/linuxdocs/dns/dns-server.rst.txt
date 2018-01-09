########################
Setting up a DNS Servers
########################

********
Overview
********

We are going to be setting up 2 dns servers(dns.parentdomain.local and 
dns.subdomain.parentdomain.local).  This is to get a better idea of how
dns servers interact with one another.  We will also be using both fedora 
and ubuntu to see the environmental differences.
 

****************************
Create two cloud dns servers
****************************

- Follow steps to set up your workstation for rackspace public cloud: :doc:`/openstackdocs/connect_to_rackspace_public_cloud`
- Create both a Fedora and Ubuntu box.

.. code-block:: sh

  . ${HOME}/rs_pub_venv/bin/activate

  # Create a new keypair
  openstack --os-cloud rs-iad keypair create dnsservers | tee ${HOME}/.ssh/dnsservers
  chmod 400 ${HOME}/.ssh/dnsservers

  # Create the fedora server making it the parentdomain dns server.
  openstack --os-cloud rs-iad server create --flavor "512MB Standard Instance" --image "Fedora 26 (PVHVM)" --key-name dnsservers dns.parentdomain.local

  # Create the ubuntu server making it the subdomain.parentdomain dns server.
  openstack --os-cloud rs-iad server create --flavor "512MB Standard Instance" --image "Ubuntu 16.04 LTS (Xenial Xerus) (PVHVM)" --key-name dnsservers dns.subdomain.parentdomain.local

  # Make sure they are up and running
  openstack --os-cloud rs-iad server list

.. code-block:: none

  +--------------------------------------+----------------------------------+--------+-----------------------------------------------------------------------------------+-----------------------------------------+-------------------------+
  | ID                                   | Name                             | Status | Networks                                                                          | Image                                   | Flavor                  |
  +--------------------------------------+----------------------------------+--------+-----------------------------------------------------------------------------------+-----------------------------------------+-------------------------+
  | 88fe2882-8e61-47b4-8e95-9d9e76b03baa | dns.subdomain.parentdomain.local | BUILD  | private=10.176.3.131; public=162.209.96.125, 2001:4802:7800:1:be76:4eff:fe20:431a | Ubuntu 16.04 LTS (Xenial Xerus) (PVHVM) | 512MB Standard Instance |
  | 81508e38-e875-4b51-b46a-ea3b4d01344b | dns.parentdomain.local           | ACTIVE | private=10.176.3.90; public=162.209.96.43, 2001:4802:7800:1:be76:4eff:fe20:1aea   | Fedora 26 (PVHVM)                       | 512MB Standard Instance |
  +--------------------------------------+----------------------------------+--------+-----------------------------------------------------------------------------------+-----------------------------------------+-------------------------+

************************************
Get your workstation public address
************************************

.. code-block:: sh

  curl icanhazip.com -4

.. code-block:: none

  108.224.169.87

  


***********************************************
Set up the dns server on dns.parentdomain.local
***********************************************

Connect using the ssh key created earlier
=========================================

.. code-block:: sh

  ssh -i ${HOME}/.ssh/dnsservers <ip address of dns1.parentdomain.local>


Install the needed packages
===========================

.. code-block:: sh

  dnf -y install bind bind-utils  


Configure bind/named
====================

.. code-block:: sh

  vi /etc/named.conf

.. literalinclude:: parentdomain-named.conf


Configure the zone file
========================

.. code-block:: sh

  mkdir /var/named/zones
  chcon -t named_zone_t /var/named/zones
  vi /var/named/zones/db.parentdomain.local

.. literalinclude:: parentdomain-zonefile


Configure the reverse record zone file
======================================

.. code-block:: sh

  vi /var/named/zones/db.209.162

.. literalinclude:: parentdomain-reverse-zonefile


Start and test named
====================

.. code-block:: sh

  systemctl start named.service
  systemctl status named.service

  dig @162.209.96.43 dns.parentdomain.com


*********
Clean Up
*********

.. code-block:: sh

  . ${HOME}/rs_pub_venv/bin/activate
  openstack --os-cloud rs-iad server delete dns.parentdomain.local
  openstack --os-cloud rs-iad server delete dns.subdomain.parentdomain.local


