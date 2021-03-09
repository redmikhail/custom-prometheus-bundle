# custom-prometheus-bundle

Bundle manifests for prometheus-operator version 0.45.0 and newer. 

Prometheus-operator upstream community ( https://github.com/prometheus-operator ) doesn't maintain artifacts needed for Operator bundle or packagemanifest or provide tools and capability to generate them.
OLM packages for community operators that include prometheus operator (https://github.com/operator-framework/community-operators) were maintained separately with latest release 0.37.0. 

This repository includes artrifacts that are needed to create OLM bundle for prometheus-operator version 0.45.0 (https://github.com/prometheus-operator/prometheus-operator/tree/release-0.45). 

## Generating OLM bundle for prometheus-operator

### Required tools
#

- docker or podman
- opm utility (https://github.com/operator-framework/operator-registry/releases/tag/v1.16.1)
- *Optional: operator-sdk (https://github.com/operator-framework/operator-sdk/releases/tag/v1.2.0)*
#
### Steps


- Clone git repository
- Make changes to CRD's or clusterversion if required 
- Validate package 

```
operator-sdk bundle validate ./bundle/0.45.0 --verbose
```
- Create OLM bundle image 
```
cd ./bundle/0.45.0
docker build -t quay.io/<your_quay_repository>/prometheus-operator-bundle:v0.45.0 -f Dockerfile .
docker push quay.io/<quay repository>/prometheus-operator-bundle:v0.45.0
```

## Generating index for OLM bundle 

```
opm index add --build-tool docker --bundles quay.io/<your_quay_repository>/prometheus-operator-bundle:v0.45.0 --tag quay.io/<your_quay_repository>/custom-prometheus-index:1.0.0
docker push quay.io/<your_quay_repository>/custom-prometheus-index:1.0.0
```
## Deploying OLM bundle to namespace in Openshift cluster (v4.6 and newer)

- create new or switch to existing openshift project  
```
oc project <project>
```
- Create new catalog source 
```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: custom-prometheus-operator-manifests
  namespace: prometheus-operator
spec:
  sourceType: grpc
  image: quay.io/<quay repository>/custom-prometheus-index:1.0.0
```
or use `./config/catalosource/catalogsource.yaml` included with this repository 

- Create CatalogSource object 
```
oc apply -f ./config/catalosource/catalogsource.yaml
```
- In Openshift Console in OperatorHub you should have new Provider Type `custom-prometheus-operator-manifests` . Select it , search for Prometheus and install operator
