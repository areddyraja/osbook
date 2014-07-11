Glance (Image Service)
======================



.. toctree::
   :numbered: 


The Image Service includes the following components:
-----------------------------------------------------





The OpenStack Image Service enables users to discover, register, and retrieve virtual machine images. Also known as the glance project, the Image Service offers aRESTAPI that enables you to query virtual machine image metadata and retrieve an actual image. You can store virtual machine images made available through the Image Service in a variety of locations from simple file systems to object-storage systems like OpenStack Object Storage.

The Image Service includes the following components:
-----------------------------------------------------

1. ``glance-api``: Accepts Image API calls for image discovery, retrieval, and storage.
2. ``glance-registry``: Stores, processes, and retrieves metadata about images. Metadata includes items such as size and type.

3. ``Database``: Stores image metadata.

4. ``Storage repository for image files``: The Image Service supports a variety of repositories including normal file systems, Object Storage, RADOS block devices, and amzon s3


Installation Image service
---------------------------


Install Packages
++++++++++++++++

1. Install the Image Service on the controller node
`````````````````````````````````````````````````````
.. code:: bash

        # apt-get install glance python-glanceclient




2. Create a new MySQL database for Glance:
``````````````````````````````````````````

| mysql -u root -p
| CREATE DATABASE glance;
| GRANT ALL ON glance.* TO 'glanceUser'@'%' IDENTIFIED BY 'glancePass';
| quit;


3. Edit /etc/glance/glance-api.conf 
`````````````````````````````````````

| [keystone_authtoken]
| auth_uri = http://10.10.100.73:5000
| auth_host = 10.10.100.73
| auth_port = 35357
| auth_protocol = http
| admin_tenant_name = service
| admin_user = glance
| admin_password = service_pass

| [database]
| sql_connection = mysql://glanceUser:glancePass@10.10.100.73/glance

| paste_deploy]
| flavor = keystone


4. Edit /etc/glance/glance-registry.conf
````````````````````````````````````````

| [keystone_authtoken]
| auth_uri = http://10.10.100.73:5000
| auth_host = 10.10.100.73
| auth_port = 35359
| auth_protocol = http
| admin_tenant_name = service
| admin_user = glance
| admin_password = service_pass

| [database]
| sql_connection = mysql://glanceUser:glancePass@10.10.100.6/glance

| [paste_deploy]
| flavor = keystone




5. Restart the glance-api and glance-registry services
```````````````````````````````````````````````````````
.. code:: bash

        #service glance-api restart; service glance-registry restart


6. Synchronize the glance database:
```````````````````````````````````
.. code:: bash

        #glance-manage db_sync


7. Restart the services again to take into account the new modifications
``````````````````````````````````````````````````````````````````````````

.. code:: bash

        #service glance-registry restart; service glance-api restart


8. To test Glance, upload the cirros cloud image directly from the internet
```````````````````````````````````````````````````````````````````````````
.. code:: bash

        #glance image-create --name cirros_x86_64 --is-public true --container-format bare --disk-format qcow2 --location  wget http://cloudhyd.com/openstack/images/images.html+download/cirros-0.3.2-i386-disk.img


9 . List Images
```````````````````````
.. code:: bash

        #glance image-list


.. code-block:: python

  +--------------------------------------+---------------+-------------+------------------+---------+--------+
  | ID                                   | Name          | Disk Format | Container Format | Size    | Status |
  +--------------------------------------+---------------+-------------+------------------+---------+--------+
  | b0246790-96d3-43d2-be42-db7461e2c925 | cirros_x86_64 | qcow2       | bare             | 9761280 | active |
  +--------------------------------------+---------------+-------------+------------------+---------+--------+


10. Download Image form Glance
`````````````````````````````````

.. code:: bash

        $glance image-download --file cirros --progress  b0246790-96d3-43d2-be42-db7461e2c925



11.How to see Image  
````````````````````

.. code:: bash

        $glance --os-image-api-version=2 image-show b0246790-96d3-43d2-be42-db7461e2c925 


