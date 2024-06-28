# service-mesh-intro

## Cómo ejecutar el LAB:
1. Ejecutar: podman run -p 8080:8080 quay.io/dsanchor/service-mesh-intro-labs:1.1.4
- Si no funciona se puede correr con Docker: docker run -p 8080:8080 quay.io/dsanchor/service-mesh-intro-labs:1.1.4
2. Acceder a: http://localhost:8080/


## Installing Service Mesh

1. Login to your Openshift cluster as 'admin'.

2. Install the following Operators:

- Red Hat OpenShift Service Mesh

- Kiali

- Red Hat OpenShift distributed tracing platform (Jaeger)

- OpenShift Elasticsearch


3. Once all Operators are installed and ready, create a ServiceMeshControlPlane called **basic** in **'istio-system' namespace**. 


Then run the following commands:

```
oc new-project hello-world
oc apply -f ./00.installation/istio-hello-world.yaml
```

You should see the following objects created:
- A ServiceMeshMember called 'default' (reffers to 'istio-system/basic' Control Plane) 
- A Gateway called 'hello-world'
- A VirtualService 'hello-world'
- A ServiceAccount 'hello-world'
- A ConfigMap 'hello-world' with the content to be shown in index.html
- A Service 'hello-world'
- A Deployment 'hello-world'


**Extra:**
Deploy a test application (it will be sleeping endlessly) that includes curl command, just to test/add traffic from this app to hello-world app.
This app will be outside the Mesh.

```
oc new-project sleep
oc apply -f ./01.examples/00.sleep.yaml
```

The connect to sleep app, an run several curl commands:
```
oc get pod          (just to get the sleep app pod)

oc rsh pod/<sleep-pod-name>
```

Once connected, run: *curl <ISTIO_INGRESSGATEWAY-URL>/<VIRTUAL_SERVICE_PREFIX>* (/hello-world)

Example:
```
curl http://istio-ingressgateway-istio-system.apps.cluster-sjvqc.dynamic.redhatworkshops.io/hello-world

```

Then, test the curl command to svc hello-world; 
```
curl hello-world.hello-world
``` 



## Uninstalling Service Mesh

> **NOTE:** : This procedure assumes that the S.Mesh installation was performed in the recommended 'istio-system' namespace.

Reference: https://docs.openshift.com/container-platform/4.15/service_mesh/v2x/removing-ossm.html

### Removing the Service Mesh control plane using the CLI

1. Log in to the OpenShift Container Platform CLI.

2. Run this command to retrieve the name of the installed ServiceMeshControlPlane:

```
oc get smcp -n istio-system
```

3. Replace <name_of_custom_resource> with the output from the previous command, and run this command to remove the custom resource:

```
oc delete smcp -n istio-system <name_of_custom_resource>
```

4. Run the following command to delete the ServiceMeshMemberRoll resource.

```
oc delete smmr -n istio-system default
```

Also verify all ServiceMeshMember in all namespaces and delete them:

```
oc get smm --all-namespaces
```

```
oc delete smm <name_of_custom_resource> -n <namespace>
```

### Removing the Operators
Log in to the OpenShift Container Platform web console.

From the Operators → Installed Operators page, scroll or type a keyword into the Filter by name to find each Operator. Then, click the Operator name.

On the Operator Details page, select Uninstall Operator from the Actions menu. Follow the prompts to uninstall each Operator.

- Red Hat OpenShift Service Mesh
1. Delete the subscription: **oc get subscription -A** (to find all subscriptions)
```
oc delete subscription servicemeshoperator -n openshift-operators

OUTPUT: subscription.operators.coreos.com "servicemeshoperator" deleted
```

2. Delete the ClusterServiceVersion (CSV):

To find it: oc get clusterserviceversion -n openshift-operators

```
oc get clusterserviceversion -n openshift-operators


OUTPUT:
NAME                            DISPLAY                                          VERSION    REPLACES                        PHASE
elasticsearch-operator.v5.8.8   OpenShift Elasticsearch Operator                 5.8.8      elasticsearch-operator.v5.8.7   Succeeded
jaeger-operator.v1.57.0-6       Red Hat OpenShift distributed tracing platform   1.57.0-6   jaeger-operator.v1.57.0-5       Succeeded
kiali-operator.v1.73.8          Kiali Operator                                   1.73.8     kiali-operator.v1.73.7          Succeeded
servicemeshoperator.v2.5.2      Red Hat OpenShift Service Mesh                   2.5.2-0    servicemeshoperator.v2.5.1      Succeeded
```

```
oc delete clusterserviceversion servicemeshoperator.v2.5.2 -n openshift-operators

OUTPUT: clusterserviceversion.operators.coreos.com "servicemeshoperator.v2.5.2" deleted
```

The same for thes rest of Operators:

- Kiali
```
oc delete subscription kiali-ossm -n openshift-operators

OUTPUT:
subscription.operators.coreos.com "kiali-ossm" deleted


oc delete clusterserviceversion kiali-operator.v1.73.8 -n openshift-operators

OUTPUT: clusterserviceversion.operators.coreos.com "kiali-operator.v1.73.8" deleted
```

- Red Hat OpenShift distributed tracing platform (Jaeger)

> **NOTE:** : Pay attention on namespace 'openshift-distributed-tracing'

```
oc delete subscription jaeger-product -n openshift-distributed-tracing

OUTPUT: 
subscription.operators.coreos.com "jaeger-product" deleted


oc delete clusterserviceversion jaeger-operator.v1.57.0-6 -n openshift-distributed-tracing

OUTPUT: 
clusterserviceversion.operators.coreos.com "jaeger-operator.v1.57.0-6" deleted
```

- OpenShift Elasticsearch
```
oc delete subscription elasticsearch-operator -n openshift-operators-redhat

OUTPUT: 
subscription.operators.coreos.com "elasticsearch-operator" deleted

oc delete clusterserviceversion elasticsearch-operator.v5.8.8 -n openshift-operators-redhat

OUTPUT: 
clusterserviceversion.operators.coreos.com "elasticsearch-operator.v5.8.8" deleted
```

### Clean up Operator resources
1. Log in to the OpenShift Container Platform CLI as a cluster administrator.

2. Run the following commands:

```
oc delete validatingwebhookconfiguration/openshift-operators.servicemesh-resources.maistra.io

oc delete mutatingwebhookconfiguration/openshift-operators.servicemesh-resources.maistra.io

oc delete svc maistra-admission-controller -n openshift-operators

oc -n openshift-operators delete ds -lmaistra-version

oc delete clusterrole/istio-admin clusterrole/istio-cni clusterrolebinding/istio-cni clusterrole/ossm-cni clusterrolebinding/ossm-cni

oc delete clusterrole istio-view istio-edit

oc delete clusterrole jaegers.jaegertracing.io-v1-admin jaegers.jaegertracing.io-v1-crdview jaegers.jaegertracing.io-v1-edit jaegers.jaegertracing.io-v1-view

oc get crds -o name | grep '.*\.istio\.io' | xargs -r -n 1 oc delete

oc get crds -o name | grep '.*\.maistra\.io' | xargs -r -n 1 oc delete

oc get crds -o name | grep '.*\.kiali\.io' | xargs -r -n 1 oc delete

oc delete crds jaegers.jaegertracing.io

oc delete cm -n openshift-operators maistra-operator-cabundle

oc delete cm -n openshift-operators -lmaistra-version

oc delete sa -n openshift-operators -lmaistra-version
```

