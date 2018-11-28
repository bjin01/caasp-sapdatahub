# caasp-sapdatahub
This project is about a recent poc with SUSE CaasP and SAP Datahub v2.3 Deployment.
In general the deployment went well and I want to keep this knowledge for future deployment and improvements.

Remarks: 
We used nfs-provisioner as workaround although SAP recommended using SUSE Enterprise Storage as backend storage using RBD as backend for requested persistent volumes, especially for HANA containers.

For the deployment of SAP Datahub the saphost agent must be deployed on the "Installation host" which is a standalone Node, ideally a SLES12SPx system in order to prepare all the prerequisites. This step is critical for the success of deployment.






