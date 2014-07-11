Nova(Compute Service)
=====================

.. toctree::
   :numbered: 


OpenStack Compute (Nova) is a cloud computing fabric controller (the main part of an IaaS system).

KVM
----

1. Check for hardware virtualization support
++++++++++++++++++++++++++++++++++++++++++++++

- The processors of your compute host need to support virtualization technology (VT) to use KVM. 
- Install the cpu package and use the kvm-ok command to check if your processor has VT support. 

.. code:: bash

	#apt-get install cpu-checker
	#kvm-ok

If VT is enabled, you should see something like

INFO: /dev/kvm exists

KVM acceleration can be used

Normally you would get a good response. Now, move to install kvm and configure it.

2. Install and Configure KVM
++++++++++++++++++++++++++++

.. code:: bash

	#apt-get install -y kvm libvirt-bin pm-utils

3. Edit the cgroup_device_acl in the /etc/libvirt/qemu.conf
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

cgroup_device_acl = [
   | "/dev/null", "/dev/full", "/dev/zero",
   | "/dev/random", "/dev/urandom",
   | "/dev/ptmx", "/dev/kvm", "/dev/kqemu",
   | "/dev/rtc", "/dev/hpet","/dev/net/tun"
   ]

| Delete default virtual bridge

.. code:: bash

	#virsh net-destroy default
	#virsh net-undefine default


4. Enable live migration by updating /etc/libvirt/libvirtd.conf
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

| listen_tls = 0
| listen_tcp = 1
| auth_tcp = "none"


5. Edit libvirtd_opts variable in /etc/init/libvirt-bin.conf
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++


| env libvirtd_opts="-d -l"

``Edit /etc/default/libvirt-bin``

| libvirtd_opts="-d -l"


6. Restart the libvirt service and dbus to load the new values
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

.. code:: bash

	#service libvirt-bin restart

7. Install Nova Packages
+++++++++++++++++++++++

.. code:: bash

	#apt-get install nova-api nova-cert nova-conductor nova-consoleauth nova-novncproxy nova-scheduler python-novaclient  nova-compute-kvm python-guestfs

8. Create Database
++++++++++++++++++

.. code:: bash

	# mysql -u root -p

| CREATE DATABASE nova;
| GRANT ALL ON nova.* TO 'novaUser'@'%' IDENTIFIED BY 'novaPass';
| quit;


9. Edit /etc/nova/nova.conf
++++++++++++++++++++++++++++

| [DEFAULT]
| logdir=/var/log/nova
| state_path=/var/lib/nova
| lock_path=/run/lock/nova
| verbose=True
| compute_scheduler_driver=nova.scheduler.simple.SimpleScheduler
| root_helper=sudo nova-rootwrap /etc/nova/rootwrap.conf
| instance_usage_audit=True
| instance_usage_audit_period=hour
| notify_on_state_change=vm_and_task_state
| rpc_backend = rabbit
| rabbit_host = 10.10.100.73
| rabbit_password = guest
| my_ip = 10.10.100.73
| vnc_enabled = True
| vncserver_listen = 0.0.0.0
| vncserver_proxyclient_address = 10.10.100.6
| novncproxy_base_url = http://192.168.0.73:6080/vnc_auto.html
| auth_strategy = keystone
| glance_host = 10.10.100.73
| network_api_class = nova.network.neutronv2.api.API
| neutron_url = http://10.10.100.73:9696
| neutron_auth_strategy = keystone
| neutron_admin_tenant_name = service
| neutron_admin_username = neutron
| neutron_admin_password = service_pass
| neutron_admin_auth_url = http://10.10.100.73:35357/v2.0
| linuxnet_interface_driver = nova.network.linux_net.LinuxOVSInterfaceDriver
| #firewall_driver = nova.virt.firewall.NoopFirewallDriver
| security_group_api = neutron
| service_neutron_metadata_proxy = True
| neutron_metadata_proxy_shared_secret = helloOpenStack
| notification_driver=nova.openstack.common.notifier.rpc_notifier
| metadata_host = 10.10.100.73
| metadata_listen = 127.0.0.1
| metadata_listen_port = 8775
| compute_driver=libvirt.LibvirtDriver
| libvirt_type=kvm
| volume_api_class=nova.volume.cinder.API
| osapi_volume_listen_port=5900
| [keystone_authtoken]
| auth_uri = http://10.10.100.73:5000
| auth_host = 10.10.100.73
| auth_port = 35357
| auth_protocol = http
| admin_tenant_name = service
| admin_user = nova
| admin_password = service_pass
| [database]
| sql_connection=mysql://novaUser:novaPass@10.10.100.73/nova


10. Edit the /etc/nova/nova-compute.conf
++++++++++++++++++++++++++++++++++++++++

| [DEFAULT]

.. code:: bash
	
	#compute_driver=libvirt.LibvirtDriver

| libvirt_ovs_bridge=br-int
| libvirt_vif_type=ethernet
| libvirt_use_virtio_for_bridges=True
| [libvirt]
| virt_type=kvm


11. Restart Compute services and  sync Database
+++++++++++++++++++++++++++++++++++++++++++++++

.. code:: bash

	# nova-manage db sync
	# service nova-compute restart
	# service nova-api restart
	# service nova-cert restart
	# service nova-consoleauth restart
	# service nova-scheduler restart
	# service nova-conductor restart
	# service nova-novncproxy restart


12. Check for the smiling faces on nova-* services to confirm your installation
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

.. code:: bash

	#nova-manage service list

.. code-block:: python

  Binary           Host                                 Zone             Status     State Updated_At
  nova-cert        openstackcontroller                  internal         enabled    :-)   2014-07-11 06:36:02
  nova-consoleauth openstackcontroller                  internal         enabled    :-)   2014-07-11 06:36:02
  nova-scheduler   openstackcontroller                  internal         enabled    :-)   2014-07-11 06:36:02
  nova-conductor   openstackcontroller                  internal         enabled    :-)   2014-07-11 06:35:56
  nova-compute     openstackcontroller                  nova             enabled    :-)   2014-07-11 06:35:59

