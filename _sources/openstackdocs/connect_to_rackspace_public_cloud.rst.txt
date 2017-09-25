##################################
Connect To Rackspace Public Cloud
##################################

This is just for python-openstackclient configuration.  The older way was to 
source environment files and use separate clients for each service.  You would 
have to use something like supernova to manage all of the different 
environments.  The openstack client this built into its own config file.   

Set up your python virutal environment
**************************************

Before connecting, we will need all of the openstack client tools.  You do NOT 
want to go down the road of using the OS provided python modules or things will 
eventually break.  Its just easier to deal with a virtualenv in the long run.

Install the python virtuenv package(Ubuntu/Debian)
--------------------------------------------------

.. code-block:: sh

  apt-get update
  apt-get install python-virtualenv  

Install the python virtuenv package(Fedora)
--------------------------------------------------

.. code-block:: sh

  dnf install python-virtualenv  


Create the python virtualenv
----------------------------

.. code-block:: sh

  virtualenv ${HOME}/rs_pub_venv


Make it active
----------------

.. code-block:: sh

  . ${HOME}/rs_pub_venv/bin/activate


Install the nova client software
--------------------------------

.. code-block:: sh

  pip install python-openstackclient


Create the config file
-----------------------

I like to create a different entry for each region to keep
from having to put full --os-region-name argument on every
call.

.. code-block:: sh

  mkdir -p ${HOME}/.config/openstack

  vi ${HOME}/.config/openstack/clouds.yml

.. code-block:: sh

  clouds:
    rs-dfw:
      auth:
        auth_url: 'https://identity.api.rackspacecloud.com/v2.0/'
        project_id: rs-tenant-id
        username: rs-user
        password: rs-pass
      region_name: DFW

    rs-iad:
      auth:
        auth_url: 'https://identity.api.rackspacecloud.com/v2.0/'
        project_id: rs-tenant-id
        username: rs-user
        password: rs-pass
      region_name: IAD

    rs-ord:
      auth:
        auth_url: 'https://identity.api.rackspacecloud.com/v2.0/'
        project_id: rs-tenant-id
        username: rs-user
        password: rs-pass
      region_name: ORD

    rs-lon:
      auth:
        auth_url: 'https://identity.api.rackspacecloud.com/v2.0/'
        project_id: rs-tenant-id
        username: rs-user
        jpassword: rs-pass
      region_name: LON

    rs-syd:
      auth:
        auth_url: 'https://identity.api.rackspacecloud.com/v2.0/'
        project_id: rs-tenant-id
        username: rs-user
        password: rs-pass
      region_name: SYD

    rs-hkg:
      auth:
        auth_url: 'https://identity.api.rackspacecloud.com/v2.0/'
        project_id: rs-tenant-id
        username: rs-user
        password: rs-pass
      region_name: HKG



Test it out
-----------

.. code-block:: sh
  
  openstack --os-cloud rs-dfw catalog list
  openstack --os-cloud rs-dfw server list
  openstack --os-cloud rs-dfw flavor list
  openstack --os-cloud rs-dfw image list


Add your key
------------

.. code-block:: sh

  openstack --os-cloud rs-dfw keypair list
  ssh-keygen  # To generate a local ssh keypair if needed 
  openstack --os-cloud rs-dfw keypair create --public-key ${HOME}/.ssh/id_rsa.pub mytestkey


Boot and Log into a server
----------------------------

.. code-block:: sh

  openstack --os-cloud rs-dfw server create --flavor "512MB Standard Instance" --image "Fedora 26 (PVHVM)" --key-name mytestkey test01
  openstack --os-cloud rs-dfw server list
  openstack --os-cloud rs-dfw server show test01

  ssh root@<public_ip_from_previous_command>
  ctrl-d

  openstack --os-cloud rs-dfw server delete test01



All the other things
--------------------

You can use help to get all of the available/required settings and arguments.
A command may have several layers and the openstack help will give help for
each.

.. code-block:: sh

  openstack help
  openstack help command
  openstack help command subcommand


Future Use
----------

Remember to source your virtualenv activate file before running any openstack 
commands.

.. code-block:: sh

  . ${HOME}/rs_pub_venv/bin/activate


