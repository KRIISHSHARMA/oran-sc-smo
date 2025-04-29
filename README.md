# O-RAN-SC SMO INSTALLATION GUIDE

## Pre-requisites:
```
./dep/smo-install/scripts/layer-0/0-setup-microk8s.sh
```
- **This script installs MicroK8s v1.27, disables swap, opens up firewall, enables DNS + storage + Prometheus, and sets up kubectl to work with the MicroK8s cluster**

## Installation

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
