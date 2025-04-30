# Pre-requisites:
```
./dep/smo-install/scripts/layer-0/0-setup-microk8s.sh
```
- **This script installs MicroK8s v1.27, disables swap, opens up firewall, enables DNS + storage + Prometheus, and sets up kubectl to work with the MicroK8s cluster**

# Installation

### Clone the repository using the command below (make sure to use recursive flag)

```
git clone --recursive "https://gerrit.o-ran-sc.org/r/it/dep"
```
### setup chartmuseum and helm

```
./dep/smo-install/scripts/layer-0/0-setup-charts-museum.sh
```
- **This script sets up and runs a local instance of ChartMuseum**

```
./dep/smo-install/scripts/layer-0/0-setup-helm3.sh
```

- **This script continues setting up the Helm + ChartMuseum environment by installing Helm CLI, the helm-push plugin, and adding a local ChartMuseum instance as a Helm repo**

### Build Charts

```
./dep/smo-install/scripts/layer-1/1-build-all-charts.sh
```

- **This script is a wrapper script that calls following scripts :**
  1. ```../sub-scripts/build-onap.sh``` : Builds the ONAP components using make, after installing necessary Helm plugins
  2. ```../sub-scripts/build-oran.sh``` : Builds the O-RAN SMO components using Makefiles
  3. ```../sub-scripts/build-tests.sh``` : Builds the test suite for the O-RAN SMO

-----------------------------------------------------
### ERROR : 
```
Pushing actn-simulator-1.0.0.tgz to local...
2025-04-29T20:27:04.438Z	ERROR	[9] Request served	{"path": "/api/charts", "comment": "", "clientIP": "127.0.0.1", "method": "POST", "statusCode": 500, "latency": "2.07079ms", "reqID": "8d3db3f7-b6fb-4f14-b69f-cc71f0ad0ca7"}
Error: 500: open /home/ubuntu/dep/chartstorage/actn-simulator-1.0.0.tgz: permission denied
```
### Solution : Change Ownership for chartstorage
```
sudo chown -R $USER:$USER /home/ubuntu/dep/chartstorage
```
-----------------------------------------------------

### SMO Deployment
```
./dep/smo-install/scripts/layer-2/2-install-oran.sh
```
- **This script:
    Sets up environment and paths.
    Deploys Helm charts for ONAP, Non-RT RIC, and SMO using specific override YAMLs.
    Verifies the deployment by printing pod and namespace statuses.**

- Each of sub-scripts installs a part of the SMO stack:
  - **../sub-scripts/install-onap.sh**: Deploys ONAP (Open Network Automation Platform)
  - **../sub-scripts/install-nonrtric.sh**: Deploys ORAN's Non-Real-Time RIC
  - **../sub-scripts/install-smo.sh**: Deploys SMO-specific components
 
- Watch Pod Status During Deployment
  - ```sudo watch kubectl get pods -n onap```
  - ```sudo watch kubectl get pods -n nonrtric```
  - ```sudo watch kubectl get pods -n smo```

-----------------------------------------------------
### ERROR : PersistentVolume Binding Issue: VolumeMismatch due to storageClassName
- While deploying the nonrtric Helm chart, the `data-oran-nonrtric-postgresql-0` PersistentVolumeClaim (PVC) may remain in a Pending state with the following warning:

```
 sudo kubectl describe pvc data-oran-nonrtric-postgresql-0 -n nonrtric
Name:          data-oran-nonrtric-postgresql-0
Namespace:     nonrtric
StorageClass:  microk8s-hostpath
Status:        Pending
Volume:        kongpv
Labels:        app.kubernetes.io/managed-by=Helm
Annotations:   meta.helm.sh/release-name: oran-nonrtric
               meta.helm.sh/release-namespace: nonrtric
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      0
Access Modes:  
VolumeMode:    Filesystem
Used By:       <none>
Events:
  Type     Reason          Age   From                         Message
  ----     ------          ----  ----                         -------
  Warning  VolumeMismatch  5s    persistentvolume-controller  Cannot bind to requested volume "kongpv": storageClassName does not match

```
- This occurs when the PV kongpv is created without specifying the correct storageClassName, or with a value that doesn't match the PVC's requested storageClassName.

- Default Kongpv.yaml file
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: kongpv
  labels:
    type: local
spec:
  capacity:
    storage: "{{ .Values.kongpv.persistence.size }}"
  accessModes:
    - "{{ .Values.kongpv.persistence.accessMode }}"
  hostPath:
    path: "{{ .Values.kongpv.persistence.path }}"
  persistentVolumeReclaimPolicy: "{{ .Values.kongpv.persistence.volumeReclaimPolicy }}"
```

### Solution : 

-----------------------------------------------------

