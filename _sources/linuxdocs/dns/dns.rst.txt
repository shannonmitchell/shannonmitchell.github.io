##############
Linux and DNS
##############

What is DNS
***********

- **Long Answer**: https://en.wikipedia.org/wiki/Domain_Name_System
- **Short Answer**: Service(s) that holds maps for domain names(www.yourcrappysite.com) to/from ip addresses(192.168.1.1).

Important Info
==============

- DNS servers are distributed and hierarchical.
- fqdn is the Fully Qualified Domain Name(www.someothersite.com)
- At the top of the tree you have the 'root' dns servers(https://en.wikipedia.org/wiki/DNS_root_zone)
- root dns servers handle queries for the last part of the fqdn(.com, .org...)
- When you buy a domain from a domain name registrar, you provide at least 2 ips for your dns servers.  
- NS or special glue records for your new domain are set up in the root servers which point your domain to the registrar's dns servers(usually).
- The registrar's dns servers are usually configure via NS or glue records to point your domain to the ip addresses provided during registration.
- Your dns servers and are now authoritative for your new domain(responsible for replying to queries for this domain).
- You can also delegate subdomains downstream of your parent domains to other dns servers using NS or glue records.
- Forwarders(upstream domain server ips) are configured on your dns server that are queried when your domain server isn't authoritative for a requested domain or subdomains.

Common Record Types
===================

- **A**: Maps domain name to an ipv4 address
- **AAAA**: Maps domain name to an ipv6 address
- **CNAME**: Maps domain name to domain name for aliases
- **MX**: Points to mail servers for a domain.  Can have multiple records with a numeric weight to determine the order that is uses by email clients when contacting email servers.
- **NS**: Used to point to authoritative name servers to deletage a subdomain.
- **SOA**: Provide authoritave domain info such as primary name server, email and some times related to dns caching.
- **PTR**: Revers map from ip address to domain name.
- **SRV**: Used by software to locate servers for a specific service.
- See https://en.wikipedia.org/wiki/List_of_DNS_record_types for more.


The Communication Path for a DNS query in Linux
***********************************************

Here we want to provide a general idea of what happens when you access a service on a linux device using a domain name.

1. Applications tie into the linux shared libnss_dss library when issuing a dns query. (man nsswitch.conf)
2. The library checks the /etc/nsswitch.conf for a 'host' config entry when a dns query is ran.

.. code-block:: sh

  grep hosts /etc/nsswitch.conf

.. code-block:: none

  hosts:          files dns

3. Queries are ran from left to right 1st match wins.  'files' is checked first and 'dns' second.
4. For the 1st check('files'), the /etc/hosts file is used for 'ip address' mappings. (man hosts)

.. code-block:: sh

  # cat /etc/hosts

.. code-block:: none

  127.0.0.1 localhost

  # The following lines are desirable for IPv6 capable hosts
  ::1 ip6-localhost ip6-loopback
  fe00::0 ip6-localnet
  ff00::0 ip6-mcastprefix
  ff02::1 ip6-allnodes
  ff02::2 ip6-allrouters
  ff02::3 ip6-allhosts

  192.168.1.1  www.somerandomentry.com

5. For the 2nd check the resolver libraries are used and the /etc/resolv.conf is checked for dns servers to query against. (man resolv.conf, man resolver, man host.conf)

.. code-block:: sh

  cat /etc/resolv.conf

.. code-block:: none

  search somedomain.net
  nameserver 192.168.1.254
  nameserver 192.168.1.253

- If fqdn isn't provided, 'search' is used for the query. (man resolv.conf)
- A query against 192.168.1.254 is ran for the domain=>ip mapping until a default timeout of 30 seconds is hit..
- If the first query times out when contacting the dns server, the second 192.168.1.253 address is queried.
- If the dns server 192.168.1.253 is authoritative for the domain queried, it returns the results from that server.
- If the domain queried is a subdomain of the domain hosted by 192.168.1.253, it may use the NS or glue records to trigger a query to the authortative servers for it. 
- If the domain queried isn't hosted by your servers, the configured forwarder addresses are used to continue the query upstream possibly all the way up to the root servers and back down to the proper authoritative server.
 

Linux DNS Tools
***************

Many tools exist, but 'dig' is usually the first goto tool for troubleshooting dns issues.

- dig:  DNS lookup utility.  You can troubleshoot most dns issues using dig. Please do a 'man dig' and take the time to just read over it and play.  Taking the time to read over this at least once will help later on.
- host: DNS lookup utility.  Can do a small subset of what dig is able to do.  Stick with dig.
- nslookup: Was the norm until dig came along.  It has an interactive mode.  See the man pages for more detail.
- arpaname: Handy for converting ip addresses to the arpa name(ip octets reversed with .in-addr.arpa suffix)
- delv: Similar to dig, but can be used for validating dnssec.



.. container:: toggle

    .. container:: header

      **Show/Hide Code**

    .. code-block:: sh

      cat /etc/hosts
