## Ablauf SUSE CAASP Cluster Setup

### Overview

![Screenshot](CAASP_Land.png)

CreateVM From CAASP-3 Template
open Remote-Console

Create Admin Server pxe conf on svrl1repo01.hs.none.ch
``` code
#
# caasp3_template.cfg
#
fqd_hostname:           svrakubadmint01.hs.none.ch
hostname:               svrakubadmint01
ip_interface1:          10.5.45.35
mask_interface1:        23
mac_interface1:         00:50:56:9c:9a:c5
default_router:         10.5.45.254
dns:                    10.1.69.10 10.1.104.228
caasp_adm_node_ip:      svrakubadmint01.hs.none.ch
smt:                    svrl1smtregt01.hs.none.ch
host_type:              caasp3_adm
email_address:          XXXXX
suse_smt_bug:           true
```
Run create Config
``` code
create_sles_client_caasp_3.sh -f /repo/installserver/client_config/svrakubadmint01.cfg
```
> Wait for setup to be finished...
> Pont a browser to https://your-new-caasp-admin-node.hs.none.ch
> Configure settings

Install Tiller
``` code
tick
```

Overlay network settings
``` code
do not change
```
Proxy Settings --> configure 
``` code
http proxy --> http://USER:XXX@proxy.none.ch:3128
https proxy --> https://USER:XXX@proxy.none.ch:3128
no proxy  --> localhost, 127.0.0.1, .hs.none.ch
```

SUSE registry mirror
``` code
server --> svrl1smtregt01.hs.none.ch
cert:
-----BEGIN CERTIFICATE-----
XXX
-----END CERTIFICATE-----
```

Container runtime
``` code
do not change
```
> Click next

Create Master/Worker Server pxe conf on svrl1repo01.hs.none.ch
``` code
#
# caasp3_template.cfg
#
fqd_hostname:           svrakubmastert01.hs.none.ch
hostname:               svrakubmastert01
ip_interface1:          10.5.45.36
mask_interface1:        23
mac_interface1:         00:50:56:9c:19:f0
default_router:         10.5.45.254
dns:                    10.1.69.10 10.1.104.228
caasp_adm_node_ip:      svrakubadmint01.hs.none.ch
smt:                    svrl1smtregt01.hs.none.ch
host_type:              caasp3
email_address:          andreas.linden@none.ch
suse_smt_bug:           true
```
Run create Config
``` code
create_sles_client_caasp_3.sh -f /repo/installserver/client_config/svrakubadmint01.cfg
```
> Repeat for all Master/Worker nodes
> Wait for setup to be finished...


BUG FIX -->Connect to adminnode with SSH --> using user: root with none standart password
apply fix --> remove SUSE-CaaS-Platform-3.0-3.0-0 Repository from zypper on all nodes befor bootstrap (see below)

> Check Velum GUI and accept all nodes --> configure Master/worker & bootstrap

> See "Install deplyment host" for the next steps
***

## Add users to velum
[Use this script to create additional users in velum](https://gitlab.hs.none.ch/linad/SuSE-CAAS-Plattform/tree/master/createVelumUser/createVU.sh "Create Users in Velum Script")

``` code
# Change settings in script first
./createVelumUser/createVU.sh
```
***

## FIXES
### remove SUSE-CaaS-Platform-3.0-3.0-0 Repository from zypper on all nodes befor bootstrap
Connect to admin node and run
``` code
docker exec -it $(docker ps | grep salt-master |awk '{print $1}') /bin/bash
#then run
salt -C '* and not L@ca' cmd.run 'zypper rr 3'
```
***

## Deploy Kubernetes stuff ####
### POC Cluster infos

DEPLYMENT HOST CLUSTER 1 & 2
svrl1caaspdplyt01.hs.none.ch 00:50:56:9c:d0:34 10.5.45.20

ALL NODES ##### Cluster 1
svrakubadmint01.hs.none.ch  00:50:56:9c:9a:c5 10.5.45.35
svrakubmastert01.hs.none.ch 00:50:56:9c:19:f0 10.5.45.36
svrakubmastert02.hs.none.ch 00:50:56:9c:ab:42 10.5.45.37
svrakubmastert03.hs.none.ch 00:50:56:9c:fc:23 10.5.45.38
svrakubworkert01.hs.none.ch 00:50:56:9c:cd:5b 10.5.45.34
svrakubworkert02.hs.none.ch 00:50:56:9c:af:f8 10.5.45.39
svrakubworkert03.hs.none.ch 00:50:56:9c:b5:65 10.5.45.43
svrakubworkert04.hs.none.ch 00:50:56:9c:9f:3f 10.5.45.44

ALL NODES ##### Cluster 2
svrkubadmint01.hs.none.ch  00:50:56:9c:4b:de 10.5.45.24
svrkubmastert01.hs.none.ch 00:50:56:9c:46:50 10.5.45.25
svrkubmastert02.hs.none.ch 00:50:56:9c:4a:43 10.5.45.26
svrkubmastert03.hs.none.ch 00:50:56:9c:a1:8d 10.5.45.27
svrkubworkert01.hs.none.ch 00:50:56:9c:ba:c5 10.5.45.28
svrkubworkert02.hs.none.ch 00:50:56:9c:19:cf 10.5.45.29
svrkubworkert03.hs.none.ch 00:50:56:9c:60:4d 10.5.45.30
svrkubworkert04.hs.none.ch 00:50:56:9c:f8:f2 10.5.45.31
***

### Deploy heapster
``` code
helm install caasp_deploy/heapster --name heapster-default --namespace=kube-system --set rbac.create=true
```
### Deploy metallb
[Check config in metallb/values.yaml](https://gitlab.hs.none.ch/linad/SuSE-CAAS-Plattform/blob/master/caasp_deploy/metallb/values.yaml "metallb values.yaml")

``` code
configInline:
  address-pools:
  - name: none-pool_1
    protocol: layer2
    addresses:
    - 10.5.45.49-10.5.45.54
```

Deploy with helm
``` code
helm install caasp_deploy/metallb --name metallb --namespace kube-system
##or
cd /root//caasp_deploy/metallb
helm install --name metallb --namespace kube-system .
```

## Deploy Dashboard
[Check config in kubernetes-dashboard/values.yaml](https://gitlab.hs.none.ch/linad/SuSE-CAAS-Plattform/blob/master/caasp_deploy/kubernetes-dashboard/values.yaml "kub-dashboard values.yaml")

This to use metallb configured adresses
``` code
service:
  type: LoadBalancer
  externalPort: 443

```

Deploy with helm
``` code
helm install caasp_deploy/kubernetes-dashboard --namespace kube-system --name kubernetes-dashboard
```
### Deploy nfs storage provisioner
``` code
helm install --name nfs-client-provisioner --namespace kube-system --set nfs.server=nastnbsnone.hs.none.ch --set nfs.path=/docker_caasp .
```
***

### Get Token to login to kubernetes dashboard
Run on system with kubeconfig
``` code
grep "id-token" ~/.kube/config  | awk '{print $2}'
```
***

## Docker Image Pull from Outside none
### Install docker for windows
[Docker for windows installer](https://gitlab.hs.none.ch/linad/SuSE-CAAS-Plattform/blob/master/caasp_stuff/DockerforWindowsInstaller.exe "Docker install EXE")
> all standart settings

Install Private Registry SSL Trust for Docker
[Get the SSL Cert from your Docker registry Server](http://svrl1smtregt01.hs.none.ch/smt.crt "Server cert")

Press [win] + r and run command
``` code
mmc
```
Click Datei --> Snap-In... --> Add Zertifikate (Computerkonto)(Local Computer) --> OK
Right Click on "Vertrauenswürdige Stammzertifizierungsstellen" --> Zertifikate --> Alle Aufgaben --> Importieren
Click Weiter --> Choose Downloaded Cert --> Weiter --> Bullet: Alle Zertifikate in folgendem... --> Vertrauenwürdige Stamm... --> Weiter --> Fertigstellen

> Restart docker
 
### Run on system without Proxy
``` code
# Pull Image from registry
docker pull quay.io/external_storage/nfs-client-provisioner:v3.1.0-k8s1.11

# Save it to tar
docker save <image_name> > nfs-client-provisioner.tar

# Tag it
docker tag <image_name> svrl1smtregt01.hs.none.ch/nfs-client-provisioner

# Push to local registry
docker push <image_name>
```

## Configure a deployment host for SAP-Datahub
Setup a standart csb3.1 server 
[Download the saltstate to configure a CAASP deploment host (2 Files)](https://gitlab.hs.none.ch/linad/SuSE-CAAS-Plattform/tree/master/caasp_deploy/install_deployment_host "SaltState to deploy")
> Run these comands
``` code
mkdir /srv/salt && pushd /srv/salt;
salt-call --local state.apply install_dply_host
```

If this was successfull download the kubeconfig from velum and put it here.
> /root/.kube/config

Logout and back in to let the proxy settings take effect. initialise helm
``` code
helm init --client-only
```

### Install SAPCAR and SAP-HOSTAGENT
[Download SAPCAR and SAP-HOSTAGENT and SLPlugin form here](https://launchpad.support.sap.com/ "happy searching")

> Support Packages and Patches  By Category  SAP Technology Components  SAP HOST AGENT  SAP HOST AGENT 7.21   <operating system>

Manually create the following user entrys
add this line to /etc/passwd
``` code
sapadm:x:11999:11000:SAP DB Administrator:/usr/local/sapadm:/bin/bash
```
add this line to /etc/group
``` code
sapsys:!:11000:sapadm
```
add this line to /etc/shadow
``` code
sapadm:x:17813:0:99999:7:::
```
> this is needed because the technical user sapadm is already present, not locally, but trouh LDAP
install sapcar
``` code
cp SAPCAR_1110-80000935.EXE /usr/local/bin/sapcar
```
OR use the RPM and run
``` code
rpm -i saphostagentrpm_39-20009394.rpm
```
check status
``` code
/usr/sap/hostctrl/exe/saphostexec -status
```

### Configure SSL for SAP-HOSTAGENT
``` code
#Create dir
mkdir /usr/sap/hostctrl/exe/sec

#Set rights
chown sapadm:sapsys /usr/sap/hostctrl/exe/sec

# Set path
export LD_LIBRARY_PATH=/usr/sap/hostctrl/exe
export SECUDIR=/usr/sap/hostctrl/exe/sec

#Create Cert
cd $SECUDIR
/usr/sap/hostctrl/exe/sapgenpse gen_pse -p SAPSSLS.pse -k GN-dNSName:svrl1caaspdplyt01 -k GN-dNSName:svrl1caaspdplyt01.hs.none.ch "CN=svrl1caaspdplyt01.hs.none.ch, O=none, C=CH"

#Grant Permissions for sapadm
/usr/sap/hostctrl/exe/sapgenpse seclogin -p SAPSSLS.pse -O sapadm
chmod go+r SAPSSLS.pse

#Restart the SAP-HOSTAGENT
../saphostexec -restart
```

> test the connectivity https://<hostname>.<domainname>:1129/SAPHostControl/?wsdl

### Configure Cross Origin Resource Sharing

Create this file --> /usr/sap/hostctrl/exe/config.d/http.server.settings
``` code
URL: /slplugin/deploy { 
  CORS { 
    AllowOrigin: https://apps.support.sap.com 
    AllowMethods: POST, GET, OPTIONS, PUT 
  }
} 
URL: /lmsl/slplugin 
 { 
  CORS { 
    AllowOrigin: https://apps.support.sap.com 
   } 
 } 
```

Reload the config
``` code
/usr/sap/hostctrl/exe/saphostctrl -function ReloadConfiguration
```

### Install with SLPlugin
[Go to the Maintenance Planner ]( https://apps.support.sap.com/sap/support/mp "SAP Data Hub installation in the Maintenance Planner")

### Install with parameters
Deply this secret to the kubernetes cluseter in advance
``` code
kubectl create secret docker-registry regcred --docker-server=$DOCKER_REGISTRY --docker-username=sapdh --docker-password=sapdhsapdh --docker-email=sapdh@none.ch -n $NAMESPACE
```

Set These env vars befor trying to pull 
``` code
export NAMESPACE=sap-datahub
export TILLER_NAMESPACE=kube-system
export KUBECONFIG=/root/.kube/config
export DOCKER_REGISTRY=svrl1smtregt01.hs.none.ch
export ARTIFACTORY_LOGIN_PASSWORD='XX)XXXXXXX'

export HTTP_PROXY='http://proxyuser:XXX@proxy.none.ch:3128'
export HTTPS_PROXY='https://proxyuser:XXX@proxy.none.ch:3128'
export NO_PROXY='10.5.45.20,.infra.caasp.local,.cluster.local,localhost,.hs.none.ch,127.0.0.1'

export http_proxy=${HTTP_PROXY}
export https_proxy=${HTTPS_PROXY}
export no_proxy=${NO_PROXY}

export CLUSTER_HTTPS_PROXY=${HTTP_PROXY}
export CLUSTER_HTTP_PROXY=${HTTPS_PROXY}
export CLUSTER_NO_PROXY=${NO_PROXY}
```
 run download all the needed images to the internal docker registry
``` code
./install.sh \
--accept-license \
--namespace=sap-datahub \
--tiller-namespace=sap-datahub \
--docker-registry=svrl1smtregt01.hs.none.ch:5000 \
--docker-repository-domain=$NAMESPACE \
--enable-rbac=no \
--enable-checkpoint-store=no \
--vora-admin-username=vora \
--vora-admin-password=XXXX@ \
--vora-system-password=XXXXX@ \
--cert-domain=hs.none.ch \
--interactive-security-configuration=no \
--image-pull-secret=regcred \
--sap-registry=73554900100200008830.dockersrv.repositories.sap.ondemand.com \
--sap-registry-login-type=1 \
--sap-registry-login-username=SAPTECHUSER -e=vora-cluster.components.dlog.replicationFactor=3 \
--cluster-http-proxy="${http_proxy}" \
--cluster-https-proxy="${https_proxy}" \
--cluster-no-proxy=${no_proxy} \
--pv-storage-class=nfs-client \
--vsystem-tenant=sapsdh \
--vsystem-storage-class=nfs-client \
--vsystem-dont-use-external-auth \
--dlog-storage-class=nfs-client \
--disk-storage-class=nfs-client \
--consul-storage-class=nfs-client \
--hana-storage-class=nfs-client \
--extra-arg=vora-vsystem.vSystem.nodePort=32123 \
--extra-arg=vora-dqp.components.txCoordinator.nodePort=30343 \
--extra-arg=vora-cluster.components.txCoordinator.hanawire.portNumber=32215 \
--prepare-images
```

install with this
``` code
./install.sh \
--accept-license \
--namespace=sap-datahub \
--tiller-namespace=sap-datahub \
--docker-registry=svrl1smtregt01.hs.none.ch:5000 \
--docker-repository-domain=$NAMESPACE \
--enable-rbac=no \
--enable-checkpoint-store=no \
--vora-admin-username=vora \
--vora-admin-password=XXXXX@ \
--vora-system-password=XXXXX@ \
--cert-domain=hs.none.ch \
--interactive-security-configuration=no \
--image-pull-secret=regcred \
--sap-registry=73554900100200008830.dockersrv.repositories.sap.ondemand.com \
--sap-registry-login-type=1 \
--sap-registry-login-username=SAPTECHUSER -e=vora-cluster.components.dlog.replicationFactor=3 \
--cluster-http-proxy="${http_proxy}" \
--cluster-https-proxy="${https_proxy}" \
--cluster-no-proxy=${no_proxy} \
--pv-storage-class=nfs-client \
--vsystem-tenant=sapsdh \
--vsystem-storage-class=nfs-client \
--vsystem-dont-use-external-auth \
--dlog-storage-class=nfs-client \
--disk-storage-class=nfs-client \
--consul-storage-class=nfs-client \
--hana-storage-class=nfs-client \
--extra-arg=vora-vsystem.vSystem.nodePort=32123 \
--extra-arg=vora-dqp.components.txCoordinator.nodePort=30343 \
--extra-arg=vora-cluster.components.txCoordinator.hanawire.portNumber=32215 \
--install
```

If something fails cleanup with this script
``` code
#!/bin/bash

kubectl delete namespace sap-datahub
kubectl delete storageclass vrep-sap-datahub

rm -rf /mnt/caasp_nfs_storage/archived-sap-datahub-data*
rm -rf /root/.helm

kubectl delete configmap -l STATUS=DELETED -n kube-system

 helm init -c
 kubectl cluster-info
 helm list

 kubectl apply -f /tmp/SAPDataHub-2.3.173-Foundation/clusterrolebinding.json -f /tmp/SAPDataHub-2.3.173-Foundation/clusterrolebinding_sapdatahub.json


export NAMESPACE=sap-datahub
export TILLER_NAMESPACE=kube-system
export KUBECONFIG=/root/.kube/config
export DOCKER_REGISTRY=svrl1smtregt01.hs.none.ch
export ARTIFACTORY_LOGIN_PASSWORD='XX)XCXXXX'

export HTTP_PROXY='http://proxyuser:XXX@proxy.none.ch:3128'
export HTTPS_PROXY='https://proxyuser:XXX@proxy.none.ch:3128'
export NO_PROXY='10.5.45.20,.infra.caasp.local,.cluster.local,localhost,.hs.none.ch,127.0.0.1'

export http_proxy=${HTTP_PROXY}
export https_proxy=${HTTPS_PROXY}
export no_proxy=${NO_PROXY}

export CLUSTER_HTTPS_PROXY=${HTTP_PROXY}
export CLUSTER_HTTP_PROXY=${HTTPS_PROXY}
export CLUSTER_NO_PROXY=${NO_PROXY}
```
Run this to be shoure:)
``` code
./install.sh \
--accept-license \
--namespace=sap-datahub \
--tiller-namespace=sap-datahub \
--docker-registry=svrl1smtregt01.hs.none.ch:5000 \
--docker-repository-domain=$NAMESPACE \
--enable-rbac=no \
--enable-checkpoint-store=no \
--vora-admin-username=vora \
--vora-admin-password=XXXXX@ \
--vora-system-password=XXXXX@ \
--cert-domain=hs.none.ch \
--interactive-security-configuration=no \
--image-pull-secret=regcred \
--sap-registry=73554900100200008830.dockersrv.repositories.sap.ondemand.com \
--sap-registry-login-type=1 \
--sap-registry-login-username=0000010019-sdhpoc -e=vora-cluster.components.dlog.replicationFactor=3 \
--cluster-http-proxy="${http_proxy}" \
--cluster-https-proxy="${https_proxy}" \
--cluster-no-proxy=${no_proxy} \
--pv-storage-class=nfs-client \
--vsystem-tenant=sapsdh \
--vsystem-storage-class=nfs-client \
--vsystem-dont-use-external-auth \
--dlog-storage-class=nfs-client \
--disk-storage-class=nfs-client \
--consul-storage-class=nfs-client \
--hana-storage-class=nfs-client \
--extra-arg=vora-vsystem.vSystem.nodePort=32123 \
--extra-arg=vora-dqp.components.txCoordinator.nodePort=30343 \
--extra-arg=vora-cluster.components.txCoordinator.hanawire.portNumber=32215 \
--purge
```

Expose gui trou metallb loadbalancer

``` code
kubectl -n $NAMESPACE expose service vsystem --type LoadBalancer --name=vsystem-lb
```
### Configure inhouse docker registry trust in SAP-DATAHUB

Create a file named vsystem-registry-secret.txt
``` code
username: "sapdh"
password: "XXXXXXXXXX"
```

[Login to SAP Datahub]( https://svrakubworkert01.hs.none.ch:32123/app/datahub-app-launchpad/ "SAP Data Hub WEB UI")

In SAP Data Hub System Management click the [settings button] \(Application Configuration & Secrets\) button above the search bar.
Click the Secrets tab, and then click the + (Create Secret) icon. 
For the secret name, enter vflow-registry

Browse to select and upload the secret file vsystem-registry-secret.txt that you previously created. 
Click Create Secret. 

open the configuration settings, click the [settings button] \(Application Configuration & Secrets\) button above the search bar. 
In the Configuration tab, find the following parameter: 
> Modeler: Name of the vSystem secret containing the credentials for Docker registry. 
Enter vflow-registry, which is the name of the secret that you previously created. 

[Download this cert]( http://svrl1smtregt01.hs.none.ch "SMT REG Cert")
Save it under smt.pem

[Go back to the launchpad]( https://svrakubworkert01.hs.none.ch:32123/app/datahub-app-launchpad/ "SAP Data Hub WEB UI")
Open Connection Management 
Click on Certificates and then on import
Import the self signt cert for your internal docker registry 

In SAP Data Hub System Management, restart the Modeler application

Launch the SAP Data Hub System Management application and open the Applications tab. 
To delete all running instances, select the Modeler application in the left pane, and click Delete All in the upper right. 
To create a new instance, select the Modeler application in the left pane, and click the + (Create an application) button in the upper right. 
