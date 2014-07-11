Neutron (Openstack Networking Service)
======================================

.. toctree::
   :numbered: 


Neutron provides networking to instance on openstack

| Edit /etc/sysctl.conf
| net.ipv4.ip_forward=1
| net.ipv4.conf.all.rp_filter=0
| net.ipv4.conf.default.rp_filter=0


.. code:: bash

	#sysctl -p

1.Install and Configure Open vSwitch
-------------------------------------

.. code:: bash

	#apt-get install -y openvswitch-switch openvswitch-datapath-dkms
2. Create bridges
-----------------
``br-int will be used for VM integration`` 

.. code:: bash

	#ovs-vsctl add-br br-int

``Add br-ex is used to access the external network`` 

.. code:: bash

	#ovs-vsctl add-br br-ex
3. This will guide you to setting up the br-ex interface. Edit eth1 in vi /etc/network/interfaces
-------------------------------------------------------------------------------------------------
.. code:: bash

	# VM internet Access 

| auto eth1 
| iface eth1 inet manual
| up ifconfig $IFACE 0.0.0.0 up
| up ip link set $IFACE promisc on
| down ip link set $IFACE promisc off
| down ifconfig $IFACE down


Add the interface to br-ex
++++++++++++++++++++++++++

.. code:: bash

	#ovs-vsctl add-port br-ex eth1

| If you want to get internet connection back, you can assign the eth1’s IP address to the br-ex in the /etc/network/interfaces

auto br-ex
++++++++++

| iface br-ex inet static
| address 192.168.0.73
| netmask 255.255.255.0
| gateway 192.168.0.1
| dns-nameservers 8.8.8.8

4. Restart the Networking
---------------------------

.. code:: bash

	#/etc/init.d/networking restart

5. Install neutron Packages
----------------------------

.. code:: bash

	#apt-get install neutron-server neutron-plugin-ml2 neutron-plugin-openvswitch-agent neutron-l3-agent neutron-dhcp-agent 

6. Create a database
---------------------

| mysql -u root -p
| CREATE DATABASE neutron;

| GRANT ALL ON neutron.* TO 'neutronUser'@'%' IDENTIFIED BY 'neutronPass';
| quit;

7. Edit /etc/neutron/neutron.conf
----------------------------------

| [DEFAULT] 
| core_plugin = neutron.plugins.ml2.plugin.Ml2Plugin
| service_plugins = router
| allow_overlapping_ips = True
| api_paste_config = /etc/neutron/api-paste.ini
| auth_strategy = keystone
| rpc_backend = neutron.openstack.common.rpc.impl_kombu
| control_exchange = neutron
| notify_nova_on_port_status_changes = True
| notify_nova_on_port_data_changes = True
| nova_url = http://10.10.100.73:8774/v2
| nova_admin_username = nova
| nova_admin_tenant_id = 2fc71d0514274d2a99242d201dae7e1e
| nova_admin_password = service_pass
| nova_admin_auth_url = http://10.10.100.73:35357/v2.0
| root_helper = sudo /usr/bin/neutron-rootwrap /etc/neutron/rootwrap.conf
| [keystone_authtoken]
| auth_uri = http://10.10.100.73:5000
| auth_host = 10.10.100.73
| auth_port = 35357
| auth_protocol = http
| admin_tenant_name = service
| admin_user = neutron
| admin_password = service_pass
| signing_dir = /var/lib/neutron/keystone-signing
| [database]
| sql_connection=mysql://neutronUser:neutronPass@10.10.100.73/neutron



8. Edit/ etc/neutron/plugins/ml2/ml2_conf.ini 
---------------------------------------------

| [ml2] 

| type_drivers = gre 
| tenant_network_types = gre 
| mechanism_drivers = openvswitch 
| [ml2_type_gre] 
| tunnel_id_ranges = 1:1000 

| [securitygroup] 
| firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver 
| enable_security_group = True 

| [ovs]
| local_ip = 10.10.100.73 
| tunnel_type = gre
| enable_tunneling = True

9.Edit /etc/neutron/l3_agent.ini
---------------------------------

| [DEFAULT] 
| verbose =  True
| interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver 
| use_namespaces = True 

10. Edit  /etc/neutron/dhcp_agent.ini
-------------------------------------

| [DEFAULT]	
| interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver 
| dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
| use_namespaces = True 

To configure the metadata agent
++++++++++++++++++++++++++++++++

The ``metadata agent`` provides configuration information such as credentials for remote access to instances.


11. Edit the /etc/neutron/metadata_agent.ini file and add the following keys to the [DEFAULT] section:
------------------------------------------------------------------------------------------------------

| [DEFAULT]
| auth_url = http://10.10.100.73:5000/v2.0
| auth_region = regionOne
| admin_tenant_name = service
| admin_user = neutron
| admin_password = service_pass
| nova_metadata_ip = 10.10.100.73
| nova_metadata_port = 8775 
| metadata_proxy_shared_secret =  helloOpenStack


12. Edit /etc/neutron/ap-paste.ini
-----------------------------------

| [filter:authtoken]
| paste.filter_factory = keystoneclient.middleware.auth_token:filter_factory
| auth_host=10.10.100.73
| auth_uri=http://10.10.100.73:5000
| admin_user=neutron
| admin_tenant_name=service
| admin_password=service_pass


13. Restart the services
-------------------------

.. code:: bash

	#service neutron-server restart
	#service neutron-plugin-openvswitch-agent restart
	#service neutron-l3-agent restart
	#service neutron-dhcp-agent restart
	#service neutron-metadata-agent restart
	#service dnsmasq restart 

.. note::
	
	if you face error the execute command “killall dnsmasq” then “service dnsmasq restart”

14. Create public network
---------------------------

In this exercise you will create a Public Network, Public Network is a network segment which allows VM’s to access the extenal network.

The following are the steps for the exercise:

- Create Public Network
- Create Public Subnet

Create Public Network
+++++++++++++++++++++++

Use Cli to create Public Network, Subnet can be created using Cli or Dashboard

.. code:: bash

	#neutron net-create public-net --router:external=True
	root@ubuntu:/home/akrantha# neutron net-create public-net --router:external=True 


``Created a new network:``


.. code-block:: python

  +---------------------------+--------------------------------------+
  | Field                     | Value                                |
  +---------------------------+--------------------------------------+
  | admin_state_up            | True                                 |
  | id                        | 17cd7fc4-261e-4899-a921-aacfae5ed198 |
  | name                      | public-net                           |
  | provider:network_type     | gre                                  |
  | provider:physical_network |                                      |
  | provider:segmentation_id  | 3                                    |
  | router:external           | True                                 |
  | shared                    | False                                |
  | status                    | ACTIVE                               |
  | subnets                   |                                      |
  | tenant_id                 | bcf1c02fde1c4b3c853b1b7f83654854     |
  +---------------------------+--------------------------------------+

.. note::

	 ``external=True`` is set to public network.

Create subnet and allocate floating ip range for public-network
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Floating IP network (Publicly routable IP Range): This network is utilized for providing Public IP accessibility to selected cloud instances. tenant-id can be obtained by using the command ``keystone tenant-list``.



``Using Cli``

.. code:: bash

	# keystone tenant-list


.. code-block:: python

  +----------------------------------+---------+---------+
  |                id                |   name  | enabled |
  +----------------------------------+---------+---------+
  | bcf1c02fde1c4b3c853b1b7f83654854 |  admin  |   True  |
  | b0613869fcec41f5b7b58c698c78b73e | service |   True  |
  +----------------------------------+---------+---------+


``Using Cli``

.. code:: bash
	
	#neutron subnet-create --tenant-id bcf1c02fde1c4b3c853b1b7f83654854  --name public-subnet --gateway 192.168.0.1 public-net 192.168.0.0/24 --enable_dhcp False --allocation-pool start=192.168.0.75,end=192.168.0.99

Created a new subnet:


.. code-block:: python

  +------------------+--------------------------------------------------+
  | Field            | Value                                            |
  +------------------+--------------------------------------------------+
  | allocation_pools | {"start": "192.168.0.75", "end": "192.168.0.99"} |
  | cidr             | 192.168.0.0/24                                   |
  | dns_nameservers  |                                                  |
  | enable_dhcp      | False                                            |
  | gateway_ip       | 192.168.0.1                                      |
  | host_routes      |                                                  |
  | id               | a2e52d56-2e91-40dd-8c1a-2fa31c964aac             |
  | ip_version       | 4                                                |
  | name             | public-subnet                                    |
  | network_id       | 17cd7fc4-261e-4899-a921-aacfae5ed198             |
  | tenant_id        | bcf1c02fde1c4b3c853b1b7f83654854                 |
  +------------------+--------------------------------------------------+



The tenant-id will be the tenant, gateway is set to 192.168.0.1 For external network ``enable_dhcp is false``. The range of floating IP addresses is set to start from ``192.168.0.75`` to ``192.168.0.99``.



15.Create Private Network
--------------------------

In this exercise you will create a private network.

The following are the steps for the exercise:

- Create Private Network
- Create Private Subnet

Create Private Net
++++++++++++++++++

``Using Cli``


.. code:: bash

	#neutron net-create private-net

``Using Dashboard``
	
``Select Networks–>Create Network``

.. figure:: _static/4.png

Create subnet for private-network
+++++++++++++++++++++++++++++++++

``Using Cli``

.. code:: bash

	#neutron subnet-create --name private-subnet --dns-nameserver 8.8.8.8 private-net 10.0.0.0/24


``Using Dashboard``


.. figure:: _static/3.png
