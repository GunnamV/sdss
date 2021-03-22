**Readme Splunk Deployment Server Service** -  Microk8s/Kubernetes cluster strategy + MetalLB


This deployment is currently tested with Ubuntu 18.04 using microk8s version 1.19 there are 2 versions 

* mdsv1 - complete working single-node deployment using hostMapped paths for app/config data
* sdssv1 - WIP/POC with NFS mappings for app/config data & HA compatibilities

**scripts** - Run in order & follow echo statements as needed
* build_microk8s_1.19.sh - build script to deploy a microk8s cluster & kubectl for a single node but with HA as an available option
* mk8.sh - enables storage, dns & metalLB addons
* -- iptables.sh  - set IPTABLES rules to allow traffic on TCP/32740 *optional depending on enviroment
*!! enable_ds.sh - enables the DS feature on all pods deployed through the CLI

* Common Yaml configurations across MDS/SDSS
* -- configmap.yaml - TCP services configmap
* -- lb.yaml - LoadBalancer service setting a open proxy address for HOST_IP:32740
* MDS
* -- mds_v1/mds.yaml - deploy full Splunk image & configure
* -- mds_v1/pv-appdata.yaml - local volume for /etc/deployment-apps directory
* -- mds_v1/pv-config.yaml - local volume for /etc/deployment-apps directory
* -- mds_v1/pvclaim-appdata.yaml - local volume for /etc/deployment-apps directory
* -- mds_v1/pvclaim-config.yaml - local volume for /etc/deployment-apps directory
* SDSS
* -- sdss_v1/sdss.yaml - deploy full Splunk image & configure
* -- sdss_v1/pv-appdata.yaml - NFS volume for /etc/deployment-apps directory
* -- sdss_v1/pv-config.yaml - NFS volume for /etc/deployment-apps directory
* -- sdss_v1/pvc-appdata.yaml - NFS volume claim for /etc/deployment-apps directory
* -- sdss_v1/pvc-config.yaml - NFS volume claim for /etc/deployment-apps directory

**MDS POC Install Overview**
The 2 scripted installs will deploy the following:
* build_microk8s_1.18.sh -- Microk8s cluster w/ storage, dns & MetalLB addons enabled
* build_mds.sh -- A TCP Configmap + MetalLB LoadBalancer Service on default port TCP/32740
    > 2 local mounts in RW/Many mode for
    >> * /opt/splunk/etc/apps/sdss_config <- serverclass.conf configurations and is mapped to /splunk-local via hostPath
    >> * /opt/splunk/etc/deployment-aps <- deployment app configuration data and is mapped to /splunk-mds via hostPath
    >> * **enable_ds.sh - enables the DS feature on all pods deployed through the CLI - UFS won't connect to the deployed replica nodes until this has been run.**

**SDSS WIP Install Overview**
The 2 scripted installs will deploy the following:
* build_microk8s_1.19.sh -- Microk8s cluster w/ storage, dns & MetalLB addons enabled
* build_sdss.sh -- A TCP Configmap + MetalLB LoadBalancer Service on default port TCP/32740
    > 2 NFS mounts in RW/Many mode for
    >> * /opt/splunk/etc/apps/sdss_config <- serverclass.conf configurations
    >> * /opt/splunk/etc/deployment-aps <- deployment app configuration data
    >> * **enable_ds.sh - enables the DS feature on all pods deployed through the CLI - UFS won't connect to the deployed replica nodes until this has been run.**
    
**Operations -- Replicas can be scaled up/down as needed** 
* kubectl scale --replicas=0 -f mds.yaml //down
* kubectl scale --replicas=6 -f mds.yaml //up
* **sh enable_ds.sh should be run after any scaling action to enable the DS per POD**

**Known Issues
HA Mode can be enabled on the microk8s cluster but there is a lingering DNS failure to resolve all conatiners across all nodes when trying running the enable_ds.sh script
>> * available workaround is to run the script across all nodes
>> * the YAML should have a lifecycle process added to run a post-script configuration and enable the server at build time
NFS mount is currently running through a HostPath configuration instead of direct mapping
>> * this appears to be related to some kind of network/proxy issue and may need an init container and/or NFS service passthru? Needs more investigation

![SDSS](SDSS.png)
