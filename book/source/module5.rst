Cinder(Block storage)
======================

.. toctree::
   :numbered: 



- ``OpenStack Block Storage (Cinder)``: provides persistent block level storage devices for use with OpenStack compute instances. The block storage system manages the creation, attaching and detaching of the block devices to servers.
cinder-api: It receives all cinder calls for processing.

- ``cinder-volumes`` : cinder-volume. Responds to requests to read from and write to the Block Storage database to maintain state, interacting with other processes through a message queue and directly upon block storage providing hardware or software.

- ``cinder-scheduler``:  Provides block storage which is used to create volume.


.. note::

	 On AMD machines create a volume-group called “cinder-volumes” while installing Ubuntu12.04 and for Intel machines create an empty partition which can later be used for creating a volume-group


1.Install cinder Packages
+++++++++++++++++++++++++

.. code:: bash

	#apt-get install cinder-api cinder-scheduler cinder-volume


2. Create Database 
++++++++++++++++++

| mysql -u root -p
| CREATE DATABASE cinder;
| GRANT ALL ON cinder.* TO 'cinderUser'@'%' IDENTIFIED BY 'cinderPass';
| quit;


3. Edit /etc/cinder/cinder.conf
++++++++++++++++++++++++++++++++

| [DEFAULT]
| glance_host = 10.10.100.73

| rootwrap_config = /etc/cinder/rootwrap.conf

.. code:: bash
	
	#api_paste_confg = /etc/cinder/api-paste.ini

| iscsi_helper = tgtadm
| volume_name_template = volume-%s
| volume_group = cinder-volumes
| verbose = True
| auth_strategy = keystone
| state_path = /var/lib/cinder
| lock_path = /var/lock/cinder
| volumes_dir = /var/lib/cinder/volumes

| [keystone_authtoken]
| auth_uri = http://10.10.100.73:5000
| auth_host = 10.10.100.73
| auth_port = 35357
| auth_protocol = http

| admin_tenant_name = service

| admin_user = cinder
| admin_password = service_pass
| sql_connection = mysql://cinderUser:cinderPass@10.10.100.73/cinder

4. Configure Cinder Volume-Group
+++++++++++++++++++++++++++++++++

| Install Package for volume Group

.. code:: bash

	#apt-get install lvm2
	#apt-get install system-config-lvm

5. create a volume-group called “cinder-volumes” using the empty partition
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

.. code:: bash

	#system--config-lvm
	#Assign the empty partition to "cinder-volumes"

6. Verify you volume Group
+++++++++++++++++++++++++++

.. code:: bash
	
	#vgdisplay

7. Synchronize Database
+++++++++++++++++++++++

.. code:: bash

	#cinder-manage db sync

8. Restart cinder* services
+++++++++++++++++++++++++++++

.. code:: bash

	#service cinder-scheduler restart
	# service cinder-api restart
	#service cinder-volume restart
	# service tgt restart
                        
