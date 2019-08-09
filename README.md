# CAPandMore : SCF deployment automation

Prerequisites:
---
- HELM client
- CF client
- kubectl client

Introduction
---
The main script is launched to deploy CAP SCF on a kubernetes cluster deployed before.
To deploy, ensure that you access a K8S cluster & prepare a PV provisionner (NFS or other)

If you are in AKS, you will need to provide more elements (revision to come) (Subscription etc)
NB: If you deployed with the https://github.com/jmlambert78/deploy-cap-aks-cluster mechanism, this deployment is compatible and will reuse envvars defined in the previous process (deploy AKS) (and especially the deploy-cap-aks-cluster/init_aks_env.sh )

Create/modify a new initxxxx.sh file (if you have not the above init_aks_env.sh for AKS)
------------------------------
```
#!/bin/bash
export AKSDEPLOYID="$PWD/CAP-080819"  #<- Create a new dir where CAPandMore will set all working files
export REGION="yourzone"
export KUBECONFIG="$AKSDEPLOYID/kubeconfig" #<- Copy your kube config file here under kubeconfig
export CF_HOME="$AKSDEPLOYID/cfconfig"      #<- will be used to store Cloudfoundry setups (to allow multi clusters)
CFEP="https://api.cf.cap2jmlzone.com"     # here is the URL of your CF deployment (in coord with the SCF-VALUES.YAML file)
echo "export CFEP=$CFEP" >>$AKSDEPLOYID/.envvar.sh  # this .envvar.sh will memorise the envvars if work is done in multisteps
cf api --skip-ssl-validation $CFEP      # this will just try to connect to the CF API endpoint
```
Source this initxxxx.sh file to get the variables ready
-------------------------------------------------------
source initxxxx.sh

Create the working directory if not existing
-------------------------------------------
mkdir $AKSDEPLOYID

Edit your SCF deployment helm chart values (scf, stratos...)
-----------
vim $AKSDEPLOYID/scf-config-values.yaml
vim $AKSDEPLOYID/ stratos-metrics-values.yaml
-> update all params, as you need

Test your configuration with kubectl
--------
kubectl get nodes # check that it is your k8S cluster

Launch the capnmore.sh script
-----
./capnmore.sh
```
>>>>>> Welcome to the CAPandMORE deployment tool <<<<<<<
NAME         STATUS   ROLES    AGE   VERSION
caasp3m141   Ready    master   2d    v1.10.11
caasp3n142   Ready    <none>   2d    v1.10.11
caasp3n143   Ready    <none>   2d    v1.10.11
 Current installed version
1) 1.3.0
2) 1.3.1
3) 1.4.0
4) 1.4.1
Please enter your choice:     <- Select the Version of CAP you want to deploy    
Current installed version 1.4.1
    1) Quit                               9) CF CreateOrgSpace                17) CF 1st mongoDB Service
    2) Review scfConfig                  10) CF 1st mysql Service             18) CF Wait for mongoDB Service
    3) Deploy UAA                        11) CF Wait for 1st Service Created  19) Deploy 2nd App Nodejs
    4) Pods UAA                          12) Deploy 1st Rails Appl            20) All localk8S
    5) Deploy SCF                        13) Deploy Stratos SCF Console       21) DELETE CAP
    6) Pods SCF                          14) Pods Stratos                     22) Upgrade Version
    7) Deploy Minibroker SB              15) Deploy Metrics
    8) CF API set                        16) Pods Metrics

Please enter your choice:
```
IF YOU ARE ON AZURE deployment :
```
>>>>>> Welcome to the CAPandMORE deployment tool <<<<<<<
NAME                       STATUS   ROLES   AGE   VERSION
aks-jmlpool19-14921831-0   Ready    agent   38h   v1.12.8
aks-jmlpool19-14921831-1   Ready    agent   38h   v1.12.8
aks-jmlpool19-14921831-2   Ready    agent   38h   v1.12.8
 Current installed version 1.4.1
 1) Quit                             11) Pods AZ OSBA                     21) Deploy Metrics
 2) Review scfConfig                 12) CF API set                       22) Pods Metrics
 3) Deploy UAA                       13) CF Add AZ SB                     23) CF 1st mongoDB Service
 4) Pods UAA                         14) CF CreateOrgSpace                24) CF Wait for mongoDB Service
 5) Deploy SCF                       15) CF 1st mysql Service             25) Deploy 2nd App Nodejs
 6) Pods SCF                         16) CF Wait for 1st Service Created  26) All Azure
 7) Deploy AZ CATALOG                17) AZ Disable SSL Mysql DBs         27) DELETE CAP
 8) Pods AZ CATALOG                  18) Deploy 1st Rails Appl            28) Upgrade Version
 9) Create AZ SB                     19) Deploy Stratos SCF Console
10) Deploy AZ OSBA                   20) Pods Stratos
Please enter your choice:
```

Select the Action or set of actions from the menu entries.
----

- Option "All localK8S" will launch sequentially most steps to deploy CAP on a kubernetes cluster for which you 
- Option "DELETE CAP" will delete all helm charts of SCF & remove the Namespaces as well
- Pods XXX : display the pods status in kubectl format
- Upgrade Version : will process the upgrade path from the current version to the next possible
    1.3.0 -> 1.3.1 -> 1.4.0 -> 1.4.1
- AZ options are specific to AKS
- All Azure : Will launch sequentially most steps to deploy CAP on a AKS cluster
- Minibroker is useful for local K8S to deliver local service instances (mongo, mysql, postgres, redis, mariadb)

If you want to deploy manually, step by step, you may select actions, the context is preserved from one step to another even if you exit. (just source the initxxxx.sh file before launching the capnmore.sh)

Logging of your actions : history
---
Each task launched is tracked in an historyfile : 
```
2019-08-09_08h47: Launch CAPnMore : Current selected version 1.4.1
2019-08-09_08h47: vvvvvv Current environment :
2019-08-09_08h47: PWD: /home/jmlambert/azure-cap/deploy-cap-aks-cluster-jml
>>> kubectl get nodes <<<
NAME                       STATUS   ROLES   AGE   VERSION
aks-jmlpool19-14921831-0   Ready    agent   38h   v1.12.8
aks-jmlpool19-14921831-1   Ready    agent   38h   v1.12.8
aks-jmlpool19-14921831-2   Ready    agent   38h   v1.12.8
>>> helm list <<<
NAME            REVISION        UPDATED                         STATUS          CHART                           NAMESPACE
catalog         1               Thu Aug  8 11:57:25 2019        DEPLOYED        catalog-0.2.1                   catalog
osba            1               Thu Aug  8 11:59:20 2019        DEPLOYED        open-service-broker-azure-1.8.2 osba
susecf-console  1               Thu Aug  8 16:58:34 2019        DEPLOYED        console-2.4.0                   stratos
susecf-scf      1               Thu Aug  8 11:46:33 2019        DEPLOYED        cf-2.17.1                       scf
susecf-uaa      1               Thu Aug  8 11:40:39 2019        DEPLOYED        uaa-2.17.1                      uaa
>>> kubectl get ns <<<
NAME          STATUS   AGE
catalog       Active   20h
default       Active   38h
kube-public   Active   38h
kube-system   Active   38h
osba          Active   20h
scf           Active   21h
stratos       Active   15h
uaa           Active   21h
2019-08-09_08h47: ^^^^^^ Current environment
```    
    
To Do :
----
- ~~Reorg the labels (too many ;-)
- Deploy NFS Provisionner (needed to provide PVs in the local K8S)
- Install HELM if not done in your cluster (rbac + tiller)
- Integrate the CAP Backup/Restore options
- Integrate the SCALING options for SCF/UAA
- ~~Integrate the log of all actions in a file 



