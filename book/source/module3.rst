
Keystone identity service
============================
.. toctree::
   :numbered: 




Keystone concepts
----------------- 

- ``Keystone`` : Provides an authentication and authorization service for other OpenStack services.Provides a catalog of endpoints for all OpenStack services.

- ``User`` : Digital representation of a person, system, or service who uses OpenStack cloud services.The Identity Service validates that incoming requests are made by the user who claims to be making the call. Users have a login and may be assigned tokens to access resources. Users can be directly assigned to a particular tenant and behave as if they are contained in that tenant.

- ``Credentials`` : Data that is known only by a user that proves who they are. In the Identity Service,examples are: User name and password, user name and API key, or an authentication token provided by the Identity Service.

- ``Authentication`` : The act of confirming the identity of a user. The Identity Service confirms an incoming request by validating a set of credentials supplied by the user.These credentials are initially a user name and password or a user name and API key. In response to these credentials, the Identity Service issues an authentication token to the user, which the user provides in subsequent requests.

- ``Token`` : An arbitrary bit of text that is used to access resources. Each token has a scope which describes which resources are accessible with it. A token may be revoked at any time and is valid for a finite duration.

- ``Tenant`` : A container used to group or isolate resources and/or identity objects. Depending on the service operator, a tenant may map to a customer, account, organization, or project.

- ``Service`` : An OpenStack service, such as Compute (Nova), Object Storage (Swift), or Image Service (Glance). Provides one or more endpoints through which users can access resources and perform operations.

- ``Endpoint`` : An endpoint is a network-accessible address, usually described by URL, from which services are accessed.

- ``Role`` : A role includes a set of assigned user rights and privileges for performing a specific set of operations. A user token issued by Keystone includes a list of that user’s roles. Services then determine how to interpret those roles.

- ``Domains`` : A domain defines administrative boundaries for the management of Identity entities. A domain may represent an individual, company, or operator-owned space. It is used for exposing administrative activities directly to the system users. A domain is a collection of tenants, users, and roles. Users may be given a domain's administrator role. A domain administrator may create tenants, users, and groups within a domain and assign roles to users and groups.

- ``Region`` : To provide a single identity access point for consumers.

Install Identity Service
=========================

1. Install MySQL and Ntp
-------------------------

.. code:: bash

	#apt-get install python-mysqldb mysql-server
	#apt-get install ntp

2. Edit the /etc/mysql/my.cnf
-----------------------------

[mysqld]

| bind-address = 0.0.0.0 
| default-storage-engine = innodb
| collation-server = utf8_general_ci
| init-connect = 'SET NAMES utf8'
| character-set-server = utf8

3. Restart the mysql Service
-----------------------------

.. code:: bash

	# service mysql restart

4.Install Identity Service Package
----------------------------------

.. code:: bash
	# apt-get install keystone

5.Verify your keystone is running
----------------------------------


.. code:: bash

	#service keystone status

6.Create a new MySQL database for keystone
------------------------------------------


.. code:: bash

	#mysql -u root -p

| CREATE DATABASE keystone;
| GRANT ALL ON keystone.* TO 'keystoneUser'@'%' IDENTIFIED BY 'keystonePass';
| quit;

7. Edit /etc/keystone/keystone.conf
-----------------------------------

| admin_token = ADMIN
| connection = mysql://keystoneUser:keystonePass@10.10.100.12/keystone

8. Restart the identity service then synchronize the database
--------------------------------------------------------------

.. code:: bash

	#service keystone restart
	#keystone-manage db_sync

9. Download and Execute Keystone Keystone Scripts
-------------------------------------------------

Modify the HOST_IP and HOST_IP_EXT variables before executing the scripts HOST_IP(10.10.100.12) and HOST_IP_EXT(192.168.0.73) 

.. code:: bash

	#wget https://raw.github.com/mseknibilel/OpenStack-Grizzly-Install-Guide/OVS_SingleNode/KeystoneScripts/keystone_basic.sh
	#wget https://raw.github.com/mseknibilel/OpenStack-Grizzly-Install-Guide/OVS_SingleNode/KeystoneScripts/keystone_endpoints_basic.sh


10. Edit quantum with neutron in both the Script Files
------------------------------------------------------

• keystone_basic.sh

| HOST_IP=10.10.100.12
| ADMIN_PASSWORD=${ADMIN_PASSWORD:-admin_pass}
| SERVICE_PASSWORD=${SERVICE_PASSWORD:-service_pass}
| export SERVICE_TOKEN="ADMIN"
| export SERVICE_ENDPOINT="http://${HOST_IP}:35357/v2.0"
| SERVICE_TENANT_NAME=${SERVICE_TENANT_NAME:-service}

| #Replace QUANTUM with NEUTRON
| NEUTRON_USER=$(get_id keystone user-create --name=neutron --pass="$SERVICE_PASSWORD" --tenant-id $SERVICE_TENANT --email=neutron@domain.com)
| keystone user-role-add --tenant-id $SERVICE_TENANT --user-id $NEUTRON_USER --role-id $ADMIN_ROLE 

• keystone_endpoints_basic.sh 

| # Host address
| HOST_IP=10.10.100.12
| EXT_HOST_IP=192.168.0.73 (Change IP according to your external IP address)

| #Create Neutron Service
| keystone service-create --name neutron --type network --description 'OpenStack Networking service'

11. Execute the Scripts
------------------------

.. code:: bash

	#chmod +x keystone_basic.sh
	#chmod +x keystone_endpoints_basic.sh
	#./keystone_basic.sh
	#./keystone_endpoints_basic.sh

Keystone Client
----------------

First of all you set env variable and create a file name creds
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

.. code:: bash

	#nano creds

| Paste the following:
| export OS_TENANT_NAME=admin
| export OS_USERNAME=admin
| export OS_PASSWORD=admin_pass
| export OS_AUTH_URL="http://192.168.0.73:5000/v2.0/"


1. Create a user
++++++++++++++++

.. code:: bash

	$ keystone user-create --name=demo --pass=service_pass --email=demo@domain.com 	

.. code-block:: python

  +----------+----------------------------------+
  | Property |              Value               |
  +----------+----------------------------------+
  |  email   |         demo@domain.com          |
  | enabled  |               True               |
  |    id    | 4945d3ea8dce49d19ba4861aec378912 |
  |   name   |               demo               |
  +----------+----------------------------------+



2. Listing Users
++++++++++++++++

.. code:: bash

	$ keystone user-list

	
.. code-block:: python

  +----------------------------------+---------+---------+--------------------+
  |                id                |   name  | enabled |       email        |
  +----------------------------------+---------+---------+--------------------+
  | f7bb44555d354f67b7997848b61d3bd7 |  admin  |   True  |  admin@domain.com  |
  | b0655e7dcdcb49ddab2b237c9f741344 |  cinder |   True  | cinder@domain.com  |
  | 4945d3ea8dce49d19ba4861aec378912 |   demo  |   True  |  demo@domain.com   |
  | abc19c2eafa440baa7f4b7504d4182e4 | enutron |   True  | neutron@domain.com |
  | 0306cbc590bf491daba6817d96ef12ab |  glance |   True  | glance@domain.com  |
  | 5f043fab09d745a2ac4db9ae1b5daf49 |   nova  |   True  |  nova@domain.com   |
  +----------------------------------+---------+---------+--------------------+




3. Create Role for demo
+++++++++++++++++++++++
.. code:: bash

	$ keystone role-create --name=demo


.. code-block:: python

  +----------+----------------------------------+
  | Property |              Value               |
  +----------+----------------------------------+
  |    id    | e3c22ad8df1a4326b6edd39ba32c6dfd |
  |   name   |               demo               |
  +----------+----------------------------------+


4. Listing role
++++++++++++++++

.. code:: bash

	$keystone role-list


.. code-block:: python

  +----------------------------------+----------------------+
  |                id                |         name         |
  +----------------------------------+----------------------+
  | 3e821785152d4841bab214cf93029932 |    KeystoneAdmin     |
  | b50d10b377854ec0b658dd34012375a4 | KeystoneServiceAdmin |
  | 98e1b74f1c0d4bc8ab829382dd3fad9a |        Member        |
  | 9fe2ff9ee4384b1894a90878d3e92bab |       _member_       |
  | 2ae08e0a30854dc08a30ed194ea4c4d8 |        admin         |
  | e3c22ad8df1a4326b6edd39ba32c6dfd |         demo         |
  +----------------------------------+----------------------+



5. Create Tenatnt/Project for Demo user
+++++++++++++++++++++++++++++++++++++++

.. code:: bash

	$ keystone tenant-create --name=demo --description="demo Tenant"


.. code-block:: python

  +-------------+----------------------------------+
  | Property    | Value                            |
  +-------------+----------------------------------+
  | description | demo Tenant                      |
  | enabled     | True                             |
  | id          | b854a7b80a2a44a3a5b36e81afa8d1f1 |
  | name        | demo                             |
  +-------------+----------------------------------+




.. code:: bash

	$keystone tenant-list

.. code-block:: python

  +----------------------------------+---------+---------+
  | id                               | name    | enabled |
  +----------------------------------+---------+---------+
  | 5009539affec41ed8b0525a1de71d985 | admin   | True    |
  | b854a7b80a2a44a3a5b36e81afa8d1f1 | demo    | True    |
  | 3a170f8cd9604013bd451d31bdea6e30 | service | True    |
  +----------------------------------+---------+---------+




6. Add role 
++++++++++++

.. code:: bash

	$keystone user-role-add --user=demo --tenant=demo --role=demo



7. Verify the demo user role
+++++++++++++++++++++++++++++

.. code:: bash
	
	$keystone user-role-list --user=demo --tenant=demo


.. code-block:: python

  +----------------------------------+------+----------------------------------+----------------------------------+
  | id                               | name |            user_id               | tenant_id                        |
  +----------------------------------+------+----------------------------------+----------------------------------+
  | e3c22ad8df1a4326b6edd39ba32c6dfd | demo | 4945d3ea8dce49d19ba4861aec378912 | b854a7b80a2a44a3a5b36e81afa8d1f1 |
  +----------------------------------+------+----------------------------------+----------------------------------+


8. demo user_member_role, and demo tenant( opitional)
++++++++++++++++++++++++++++++++++++++++++++++++++++++++

.. code:: bash

	$ keystone user-role-add --user=demo --role=_member_ --tenant=demo

9. Create Service name demo
++++++++++++++++++++++++++++

.. code:: bash

	$ keystone service-create --name=demo --type=demo --description="OpenStack demo Service"



.. code-block:: python

  +-------------+----------------------------------+
  | Property    |            Value                 |
  +-------------+----------------------------------+
  | description |     OpenStack demo Service       |
  | id          | fe38f49562bb40a3af3e3b07ea10534d |
  | name        |             demo                 |
  | type        |             demo                 |
  +-------------+----------------------------------+












10. Listing Service
++++++++++++++++++++
 
.. code:: bash

	$keystone service-list


.. code-block:: python

  +----------------------------------+----------+----------+------------------------------+
  |                id                |   name   |   type   |         description          |
  +----------------------------------+----------+----------+------------------------------+
  | 2e9a4b5693de4078b02f8777260002c2 |  cinder  |  volume  |   OpenStack Volume Service   |
  | fe38f49562bb40a3af3e3b07ea10534d |   demo   |   demo   |    OpenStack demo Service    |
  | 1ff64d5d3ce34cae8661b24cfcf6c1de |   ec2    |   ec2    |    OpenStack EC2 service     |
  | 6b31955ef5764467b74a772cdb66e0d7 |  glance  |  image   |   OpenStack Image Service    |
  | df8f7e3aaf8b4516be2e5a3c34649c9a | keystone | identity |      OpenStack Identity      |
  | 8dee6d67a2044be68097b0f0cf4190d2 | neutron  | network  | OpenStack Networking service |
  | f569b5c7dadc460eb46281203912855b |   nova   | compute  |  OpenStack Compute Service   |
  +----------------------------------+----------+----------+------------------------------+




11. Create Endpoint for demo Service
++++++++++++++++++++++++++++++++++++++
.. code:: bash

	$ keystone endpoint-create  --region=RegionOne –service-id=fe38f49562bb40a3af3e3b07ea10534d --publicurl=http://192.168.0.12:9292 --internalurl=http://10.10.100.51:9292 -- adminurl=http://10.10.100.6:9292


.. code-block:: python

  +-------------+----------------------------------+
  |   Property  |              Value               |
  +-------------+----------------------------------+
  |   adminurl  |     http://10.10.100.12:9292     |
  |      id     | 7616d596bb85461daa60fc5d66ed973f |
  | internalurl |     http://10.10.100.12:9292     |
  |  publicurl  |     http://192.168.0.73:9292     |
  |    region   |            RegionOne             |
  |  service_id | fe38f49562bb40a3af3e3b07ea10534d |
  +-------------+----------------------------------+



12. Listing Endpoint 
++++++++++++++++++++


.. code:: bash

	$keystone endpoint-list grep 7616d596bb85461daa60fc5d66ed973f


| 7616d596bb85461daa60fc5d66ed973f | RegionOne |          http://192.168.0.73:9292         |          http://10.10.100.12:9292         |          http://10.10.100.12:9292         | fe38f49562bb40a3af3e3b07ea10534d |

 
13. Delete endpoint
++++++++++++++++++++

.. code:: bash

	$keystone endpoint-delete 7616d596bb85461daa60fc5d66ed973f(put endpoint id)

Endpoint has been deleted.

14. Delete Service
+++++++++++++++++++

.. code:: bash

	$keystone service-delete fe38f49562bb40a3af3e3b07ea10534d (put service id) 

15. Remove role
++++++++++++++++

.. code:: bash

	$keystone user-role-remove --user=demo --role=demo --tenant=demo

16. Verify the role
++++++++++++++++++++

.. code:: bash

	$ keystone user-role-list --user=demo --tenant=demo


17. Delete Tenant
++++++++++++++++++

.. code:: bash

	$keystone tenant-delete demo


18. Verify the demo Tenant
+++++++++++++++++++++++++++

.. code:: bash
	
	$keystone tenant-list



.. code-block:: python

  +----------------------------------+---------+---------+
  |                id                |   name  | enabled |
  +----------------------------------+---------+---------+
  | 5009539affec41ed8b0525a1de71d985 |  admin  |   True  |
  | 3a170f8cd9604013bd451d31bdea6e30 | service |   True  |
  +----------------------------------+---------+---------+

19. Delete the Role
++++++++++++++++++++

.. code:: bash

	$keystone role-delete demo

20. Verify the role 
++++++++++++++++++++

.. code:: bash

	$keystone role-list



.. code-block:: python

  +----------------------------------+----------------------+
  |                id                |         name         |
  +----------------------------------+----------------------+
  | 3e821785152d4841bab214cf93029932 |    KeystoneAdmin     |
  | b50d10b377854ec0b658dd34012375a4 | KeystoneServiceAdmin |
  | 98e1b74f1c0d4bc8ab829382dd3fad9a |        Member        |
  | 9fe2ff9ee4384b1894a90878d3e92bab |       _member_       |
  | 2ae08e0a30854dc08a30ed194ea4c4d8 |        admin         |
  +----------------------------------+----------------------+


21. Delete User demo
++++++++++++++++++++++

.. code:: bash

	$keystone user-delete demo

22. Verify the demo User
+++++++++++++++++++++++++

.. code:: bash

	$keystone user-list


.. code-block:: python

  +----------------------------------+---------+---------+--------------------+
  |                id                |   name  | enabled |       email        |
  +----------------------------------+---------+---------+--------------------+
  | f7bb44555d354f67b7997848b61d3bd7 |  admin  |   True  |  admin@domain.com  |
  | b0655e7dcdcb49ddab2b237c9f741344 |  cinder |   True  | cinder@domain.com  |
  | abc19c2eafa440baa7f4b7504d4182e4 | enutron |   True  | neutron@domain.com |
  | 0306cbc590bf491daba6817d96ef12ab |  glance |   True  | glance@domain.com  |
  | 5f043fab09d745a2ac4db9ae1b5daf49 |   nova  |   True  |  nova@domain.com   |
  +----------------------------------+---------+---------+--------------------+


