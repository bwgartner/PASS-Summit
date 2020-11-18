# Ensuring availability of SQL Server with Kubernetes

## References:

* Traditional high availability approach in Azure webinar series [How to Run SQL Server on Azure VMs on SUSE Linux](https://webinars.dev/microsoft/azure-webinar-series-how-run-sql-server-azure-vms-suse-linux/)
* Related, overview PASS Summit presentation - [Modernizing SQL Server Operations with Kubernetes](https://virtual.passsummit.com/fsPopupEmbed.asp?Mode=presInfo&PresentationID=798356&embedded=true) ... [presentation](../Session-Modernizing_SQL_Server_Operations_with_Kubernetes/Modernizing_SQL_Server_Operations_with_Kubernetes.pdf), [video](../Session-Modernizing_SQL_Server_Operations_with_Kubernetes/videos/Modernizing_SQL_Server_Operations_with_Kubernetes.mp4)
  - snippet of the goal for this demo ... [video](../Session-Modernizing_SQL_Server_Operations_with_Kubernetes/videos/SQLServer_K8s_failover.mp4)
* Supplementary PASS Summit 2020 demo session [SQL Server on Linux on Azure Quickstart](https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODE5M2UyY2MtZDMxZS00MTNkLWE1ZDItNzUyNDI4OWJlY2E1%40thread.v2/0?context=%7b%22Tid%22%3a%22f7a17af6-1c5c-4a36-aa8b-f5be247aa4ba%22%2c%22Oid%22%3a%227e47b779-9638-4f49-b6ba-be9fda179168%22%2c%22IsBroadcastMeeting%22%3atrue%7d)
* SUSECon 2020 HOL-1289 [Introduction to Kubernetes](https://github.com/bwgartner/suse-doc/blob/master/SUSECon/2020/HOL-1289/pdf/lab-guide.pdf) ... includes links to videos

## Assumptions:

* Have internet/intranet access
* Azure
  - Have a portal account with access to resources
* [SUSE CaaS Platform](https://www.suse.com/products/caas-platform/)
  - Have the respective [kubeconfig](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/) to access the cluster resources with and respective [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

## Preparation:

* Choice of Kubernetes clusters
  - Login to [Azure Portal](https://portal.azure.com/)
    - setup an [Azure Kubernetes Service](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal) [ AKS ] ... [video](./videos/AKS_Setup.mp4)
  - [Deploy](https://documentation.suse.com/suse-caasp/4.5/single-html/caasp-deployment/) SUSE CaaS Platform
    - on Azure VMs or other cloud service providers or
    - choice of various hypervisors or baremetal systems (on-premise)
    - NOTE: can also leverage a [deployment](https://documentation.suse.com/ses/7/) of [SUSE Enterprise Storage](https://www.suse.com/products/suse-enterprise-storage/) as the [CSI](https://kubernetes-csi.github.io/docs/) storage class for persistent volumes via [integration](https://documentation.suse.com/suse-caasp/4.5/single-html/caasp-admin/#ses-integration) with an existing cluster or in a [hyperconverged](https://documentation.suse.com/ses/7/single-html/ses-rook/#book-storage-rook) fashion

## Process:

* Setup access to the respective Kubernetes cluster instance
  - General client
    - Linux Azure VM resource (either [PAYG or BYOS](https://azure.microsoft.com/en-us/overview/linux-on-azure/suse/)) or
    - local Linux [VM](https://susedefines.suse.com/definition/jeos-just-enough-operating-system/) client ... [video](./videos/Setup_client.mp4)
      - configure necessary client-side tools
        - install [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) client
        - install [mssql-tools](https://docs.microsoft.com/en-us/sql/linux/sql-server-linux-setup-tools?view=sql-server-ver15#SLES) client
        - install [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-zypper) client
  - Connect to Azure Kubernetes Service
    - via a browser to [Azure](https://portal.azure.com/)
    - connect to respective AKS instance from client system
      - az login
      - az account set --subscription YourSubscription
      - az aks get-credentials --resource-group YourResourceGroup --name YourResourceName
  - Connect to SUSE CaaS Platform
    - obtain respective cluster and user kubeconfig file for kubectl access
  - Validate kubectl access to respective Kubernetes cluster
    - kubectl get nodes

* Setup SQL Server application deployment
  - In Azure Kubernetes Service resource ... [video](./videos/AKS_SQLServer.mp4)
    - [Deploy a SQL Server container in Kubernetes with Azure Kubernetes Services (AKS)](https://docs.microsoft.com/en-us/sql/linux/tutorial-sql-server-containers-kubernetes?view=sql-server-ver15)
      - create SA password in Kubernetes credential secret
        - kubectl create secret generic mssql --from-literal=SA_PASSWORD="MyC0m9l&xP@ssw0rd"
      - create persistent volume claim from existing storage class (for backend storage)
        - kubectl get sc
        - kubectl apply -f ./[pvc.yaml](./pvc.yaml)
      - create complete deployment of loadbalancer, replica set and leverage containerized SQL Server
        - kubectl apply -f ./[sqldeployment.yaml](./sqldeployment.yaml)
  - In SUSE CaaS Platform
    - leverage previous AKS process and YAML templates
      - create SA password in Kubernetes credential secret
      - leverage available storage class
        - kubectl get sc 
          - kubectl apply -f ./[pvc-ses.yaml](./pvc-ses.yaml) ... or you could use the [NFS](./pvc-nfs.yaml) storage class variant
      - create SQL Server deployment
        - kubectl apply -f ./[sqldeployment.yaml](./sqldeployment.yaml)

* Access SQL Server instance and test failover
  - Connect to SQL Server container pod via sqlcmd
    - through loadbalancer from Client ... [video](./videos/Client_sqlcmd.mp4)
      - sqlcmd -S IPLoadBalancer -U sa -P secretPassword
    - directly within the SQL Server pod ... [video](./videos/Pod_sqlcmd.mp4)
      - kubectl get pods (look for mssql-deployment-podNameID)
      - kubectl exec -it mssql-deployment-podNameID /bin/sh
        - sqlcmd -S localhost -U sa -P secretPassword
  - Query / modify SQL Server content via sqlcmd ... [video](./videos/Query_sqlcmd.mp4)
    - select name from sys.databases
      go
    - CREATE DATABASE myMSSQL
      go
    - select name from sys.databases
      go
    - quit
  - Emulate failure of SQL Server container and benefit from Kubernetes-based resilience ... [video](./videos/Emulate_Failover.mp4)
    - kubectl get deployments -A (find mssql-deployment or use Dashboard)
    - kubectl get pods -A (find the mssql-deployment-podNameID)
    - kubectl delete pod mssql-deployment-podNameID
    - kubectl get pods -A (find the new mssql-deployment-podNameID)
  - Validate data content persists
    - retry previous sqlcmd connect/query access, listing databases to show persistence

* Cleanup SQL Server deployment ... [video](./videos/Cleanup.mp4)
  - kubectl delete -f ./[sqldeployment.yaml](./sqldeployment.yaml)
  - kubectl delete -f "respective PVC.yaml"
  - kubectl delete secret mssql
