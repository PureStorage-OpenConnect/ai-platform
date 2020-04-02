## AI Data Hub
This repo contains examples of using Pure Storage FlashBlade in an AI Data Hub. 
Modules available: 
- JupyterHub as a Service

##
# JupyterHub as a Service on FlashBlade

## Introduction
Jupyter notebooks are a popular tool for data scientists to explore datasets and experiment with model development. They enable developers to easily supplement code with analysis and visualizations. 

Rather than the historical practice of having users manage their own notebook servers, JupyterHub can be deployed by an organization to offer a centralized notebook platform. JupyterHub also enables infrastructure teams to give each user access to centralized storage for: shared datasets, scratch space, and a persistent IDE. 

In this example deployment, users are able to create new notebook servers on the fly within a Kubernetes cluster with zero-touch provisioning. IT teams are able to manage efficient use of compute and storage resources across users. 

## Installation

### Prep Steps
- Clone this repo. 

- On your local machine, ensure that your cluster config is active for kubectl. 

- On your local machine, install Helm and the Helm repos for PSO and for JupyterHub:
    - `helm repo add pure https://purestorage.github.io/helm-charts`  
    - `helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/`

- Each node in the cluster needs to have access to the datasets on FlashBlade. Mount the datasets folder directly to each cluster node at `/datasets`.

### Deploy PSO
**Customize:**

Adjust the "arrays" section of [./psovalues.yaml](https://github.com/PureStorage-OpenConnect/ai-platform/blob/master/psovalues.yaml) to include your FlashBlade specifics. Example customization: 

```arrays:
  FlashBlades:
    - MgmtEndPoint: "10.61.169.20"                           # CHANGE 
      APIToken: "T-c4925090-c9bf-4033-8537-d24ee5669135"     # CHANGE 
      NFSEndPoint: "10.61.169.30"                            # CHANGE 
```
*Further reading: PSO configuration <link>.*

**Install:**

`helm install pure-storage-driver pure/pure-csi --namespace jhub -f <your_own_dir>/psovalues.yaml`

Installing PSO creates a few storage classes in your cluster. The example values.yaml file uses the “pure-file” storage class for JupyterHub. 


### Deploy a PV for shared datasets
**Customize:**

The [./datasetpv.yaml](https://github.com/PureStorage-OpenConnect/ai-platform/blob/master/datasetpv.yaml) file is used create a Persistent Volume Claim named “shared-ai-datasets”. Adjust it to use your FlashBlade Data VIP and filesystem name.  

```nfs:
    server: 10.61.169.100      # CHANGE to your data vip 
    path: /datasets            # CHANGE to your filesystem name
```
**Install:**

`kubectl create -f datasetpv.yaml`


### Deploy JupyterHub
**Customize:**

The only change required for the [./jupvalues.yaml](https://github.com/PureStorage-OpenConnect/ai-platform/blob/master/jupvalues.yaml) file is to add a security token. Generate a random hex string:

`openssl rand -hex 32`

Copy the output and, in your jupvalues.yaml file, replace the phrase SECRET_TOKEN with your generated string:
```proxy:
  secretToken: 'SECRET_TOKEN'   # CHANGE to 32-digit secret token (use straight quotes '')
```
*Further reading: description of settings in Pure’s jupvalues.yaml <link>*

**Install:**

`helm install jhub jupyterhub/jupyterhub --namespace jhub --version 0.8.2 -f jupyterhub/values.yaml`

## Use Jupyter notebooks! 
JupyterHub is now ready for use. 

Installing JupyterHub creates a proxy service that serves traffic for end users. The public address (proxy-public) can be found via:

```
> kubectl --namespace=jhub get svc proxy-public
NAME           TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                      AGE
proxy-public   LoadBalancer   10.43.197.255.   10.61.169.60    80:30615/TCP,443:30987/TCP   5d19h
```
