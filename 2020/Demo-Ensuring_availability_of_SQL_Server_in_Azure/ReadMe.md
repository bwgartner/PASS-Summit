# Ensuring availability of SQL Server in Azure

## References:

* Azure webinar series [How to Run SQL Server on Azure VMs on SUSE Linux](https://webinars.dev/microsoft/azure-webinar-series-how-run-sql-server-azure-vms-suse-linux/)
* PASS Summit presentation - [Modernizing SQL Server Operations with Kubernetes](https://virtual.passsummit.com/fsPopupEmbed.asp?Mode=presInfo&PresentationID=798356&embedded=true) ... [presentation](../Session-Modernizing_SQL_Server_Operations_with_Kubernetes/Modernizing_SQL_Server_Operations_with_Kubernetes.pdf), [video](../Session-Modernizing_SQL_Server_Operations_with_Kubernetes/videos/Modernizing_SQL_Server_Operations_with_Kubernetes.mp4)
  - snippet of the goal for this demo ... [video](../Session-Modernizing_SQL_Server_Operations_with_Kubernetes/videos/SQLServer_K8s_failover.mp4)
* related PASS Summit 2020 demo session [SQL Server on Linux on Azure Quickstart](https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODE5M2UyY2MtZDMxZS00MTNkLWE1ZDItNzUyNDI4OWJlY2E1%40thread.v2/0?context=%7b%22Tid%22%3a%22f7a17af6-1c5c-4a36-aa8b-f5be247aa4ba%22%2c%22Oid%22%3a%227e47b779-9638-4f49-b6ba-be9fda179168%22%2c%22IsBroadcastMeeting%22%3atrue%7d)
* SUSECon 2020 HOL-1289 [Introduction to Kubernetes](https://github.com/bwgartner/suse-doc/blob/master/SUSECon/2020/HOL-1289/pdf/lab-guide.pdf) ... includes links to videos

## Assumptions:

* Have internet access
* Have an Azure account

## Preparation:

* Login to Azure
  - Setup an [Azure Kubernetes Service](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal) [ AKS ] ... [video](./videos/Setup_AKS.mp4)
    - alternative : [SUSE CaaS Platform](https://www.suse.com/products/caas-platform/) [deployment](https://documentation.suse.com/suse-caasp/4.5/single-html/caasp-deployment/) on Azure VMs or baremetal systems
      - NOTE: can also be deployed on-premise or other locations

## Process:

* Access the respective Kubernetes cluster instance
  - Via a browser to [Azure](https://portal.azure.com/)
  - Setup a (or use an existing) [SUSE Linux Enterprise](https://www.suse.com/products/server/) system as
    - Linux Azure VM (either [PAYG or BYOS](https://azure.microsoft.com/en-us/overview/linux-on-azure/suse/)) or
    - Local Linux [VM](https://susedefines.suse.com/definition/jeos-just-enough-operating-system/) client ... [video](./videos/Setup_client.mp4)
      - Configure necessary client-side tools
        - Install [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) client
        - Install [mssql-tools](https://docs.microsoft.com/en-us/sql/linux/sql-server-linux-setup-tools?view=sql-server-ver15#SLES) client
        - Install [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-zypper) client

* Connect to AKS instance from client system ... [video](./videos/Connect_AKS.mp4)
  - az login
  - az account set --subscription <YourSubscription>
  - az aks get-credentials --resource-group <YourResourceGroup> --name <YourResourceName>

* Setup SQL Server application components ... [video](./videos/Setup_SQLServer.mp4)
  - [Create a persistent volume claim](https://docs.microsoft.com/en-us/azure/aks/azure-disks-dynamic-pv)
  - [Deploy a SQL Server container in Kubernetes with Azure Kubernetes Services (AKS)](https://docs.microsoft.com/en-us/sql/linux/tutorial-sql-server-containers-kubernetes?view=sql-server-ver15)

* Access SQL Server and test failover
  - Connect to SQL Server container pod via sqlcmd through loadbalancer
  - Query / modify SQL Server content via sqlcmd
  - Emulate failure of SQL Server container and benefit from Kubernetes-based resilience ... [video](./videos/Access_Failover.mp4)

