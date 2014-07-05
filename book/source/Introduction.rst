Introduction
============

.. toctree::
   :numbered: 

The tutorial covers the basic overview of OpenStack and vSphere
integration. The tutorials cover Openstack compute and storage using the
vCenter Web Client and the Openstack Horizon Dashboard.There is a file
copy-paste.txt on your windows desktop that includes any strings you
need to enter. This can be useful if you are using an international
keyboard and the lab requires you to enter text that you cannot easily
type.

WHAT IS OPENSTACK?
^^^^^^^^^^^^^^^^^^

OpenStack is open source software enabling the creation of clouds on top
of a diverse set of hardware and software infrastructure technologies.

CLOUD API LAYER IN A CLOUD TECHNOLOGY STACK
                                           

.. figure:: _static/vmwarelab1.png
   :alt: import

   import
A typical cloud technology stack consists of following major components

-  Hardware Infrastructure
-  Software Infrastructure (or virtualization layer)
-  Cloud API layer that enables consumption and orchestration of
   underlying cloud infrastructure
-  Cloud Management Layer that provides governance, monitoring,
   provisioning, budgeting etc and potentially manages multiple
   underlying cloud fabrics
-  Applications running on top of cloud infrastructure

In a non-cloud datacenter model, an application owner would contact one
or more datacenter administrators, who would then deploy the application
on the application owner's behalf using software infrastructure tools
(e.g., VMware vSphere) to deploy the application workloads on top of
physical compute, network, and storage hardware.

OpenStack is a software layer that sits on top of the software
infrastructure and enables an API based consumption of infrastructure.
OpenStack enables "self-service" model in which application owners can
directly request and provision the compute, network, and storage
resources needed to deploy their application.

The primary benefits of self-service are increased agility from
applications owners getting "on demand" access to the resources they
need and reduced operating expenses by eliminated manual + repetitive
deployment tasks.

ANATOMY OF A CLOUD TECHNOLOGY STACK
                                   

.. figure:: _static/vmwarelab2.png
   :alt: import

   import
OpenStack Cloud API Layer adds following services in the cloud
technology stack.

An API layer presents abstracted compute/network/storage resources,
completely decoupled from any datacenter hardware, for user by self
service tools.

Enables self-service to compute/network/storage resources, there is a
Web GUI, CLI tools, and programmatic SDK

Provides an identity service that provides authentication and basic
control over resource consumption by managing quotas on infrastructure
resources.

The core logic of OpenStack takes requests from the API layer,
determines if the request is permitted, and routes the request to the
proper portion of the software infrastructure.

Based on the type of software infrastructure in use, OpenStack uses a
"driver" layer to translate abstract resource requests into a call to a
particular underlying technology (e.g., create a VM on VMware vSphere).

ANATOMY OF OPENSTACK
                    

.. figure:: _static/vmwarelab3.png
   :alt: import

   import
OpenStack matches this same architecture, but splits functions into
several different services. Each of these services is known by its
project code name:

-  Keystone: Identity service.
-  Horizon: Web GUI.
-  Nova: Compute service.
-  Glance: Image service.
-  Neutron: Network services (formerly called "Quantum").
-  Cinder: Block Storage service.

OpenStack services orchestrate and manage the underlying infrastructure
and expose APIs for end users to consume the resources. OpenStack's
strength is that it is a highly customizable framework, allowing those
deploying it to choose from a number of different technology components,
and even customize the code themselves.

