# NetScaler CPX with NetScaler Ingress Controller running as sidecar.

In a [Kubernetes](https://kubernetes.io/) or [OpenShift](https://www.openshift.com) cluster, you can deploy [NetScaler CPX](https://docs.netscaler.com/en-us/citrix-adc-cpx/) with NetScaler ingress controller as a [sidecar](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/). The NetScaler CPX instance is used for load balancing the North-South traffic to the microservices in your cluster. And, the sidecar NetScaler ingress controller configures the NetScaler CPX.

## TL;DR;

### For Kubernetes
  ```
  helm repo add netscaler https://netscaler.github.io/netscaler-helm-charts/

  helm install citrix-cpx-with-ingress-controller netscaler/citrix-cloud-native --set cpx.enabled=true,cpx.license.accept=yes
  ```

   To install NetScaler Provided Custom Resource Definition(CRDs) along with NetScaler Ingress Controller
   ```
   helm install citrix-cpx-with-ingress-controller netscaler/citrix-cloud-native --set cpx.enabled=true,cpx.license.accept=yes,cpx.crds.install=true
   ```

### For OpenShift

  ```
  helm repo add netscaler https://netscaler.github.io/netscaler-helm-charts/

  helm install citrix-cpx-with-ingress-controller netscaler/citrix-cloud-native --set cpx.enabled=true,cpx.license.accept=yes,cpx.openshift=true
  ```

  To install NetScaler Provided Custom Resource Definition(CRDs) along with NetScaler Ingress Controller
  ```
  helm install citrix-cpx-with-ingress-controller netscaler/citrix-cloud-native --set cpx.enabled=true,cpx.license.accept=yes,cpx.openshift=true,cpx.crds.install=true
  ```

> **Important:**
>
> The "cpx.license.accept" is a mandatory argument and should be set to "yes" to accept the terms of the NetScaler license.


## Introduction
This Helm chart deploys a NetScaler CPX with NetScaler ingress controller as a sidecar in the [Kubernetes](https://kubernetes.io/) or in the [Openshift](https://www.openshift.com) cluster using the [Helm](https://helm.sh/) package manager.

### Prerequisites

-  The [Kubernetes](https://kubernetes.io/) version should be 1.16 and above if using Kubernetes environment.
-  The [Openshift](https://www.openshift.com) version 4.8 or later if using OpenShift platform.
-  The [Helm](https://helm.sh/) version 3.x or later. You can follow instruction given [here](https://github.com/netscaler/netscaler-helm-charts/blob/master/Helm_Installation_version_3.md) to install the same.
-  You have installed [Prometheus Operator](https://github.com/coreos/prometheus-operator), if you want to view the metrics of the NetScaler CPX collected by the [metrics exporter](https://github.com/netscaler/netscaler-k8s-ingress-controller/tree/master/metrics-visualizer#visualization-of-metrics).
- Registration of NetScaler CPX in ADM: You may want to register your CPX in ADM for licensing or to obtain [servicegraph](https://docs.netscaler.com/en-us/citrix-application-delivery-management-service/application-analytics-and-management/service-graph.html). For this you will have to create a Kubernetes secret using ADM credentials and provide it while install the chart. Create a Kubernetes secret for the user name and password using the following command:

  ```
  kubectl create secret generic admlogin --from-literal=username=<adm-username> --from-literal=password=<adm-password> -n netscaler-system
  ```

## Installing the Chart
Add the NetScaler Ingress Controller helm chart repository using command:

  ```
  helm repo add netscaler https://netscaler.github.io/netscaler-helm-charts/
  ```

### For Kubernetes:
#### 1. NetScaler CPX with NetScaler Ingress Controller running as side car.
To install the chart with the release name ``` my-release```:

  ```
  helm install my-release netscaler/citrix-cloud-native --set cpx.enabled=true,cpx.license.accept=yes,cpx.ingressClass[0]=<ingressClassName>
  ```

> **Note:**
>
> By default the chart installs the recommended [RBAC](https://kubernetes.io/docs/admin/authorization/rbac/) roles and role bindings.

The command deploys NetScaler CPX with NetScaler ingress controller as a sidecar on the Kubernetes cluster with the default configuration. The [configuration](#configuration) section lists the mandatory and optional parameters that you can configure during installation.

#### 2. NetScaler CPX with NetScaler Ingress Controller and Exporter running as side car.
[Metrics exporter](https://github.com/netscaler/netscaler-k8s-ingress-controller/tree/master/metrics-visualizer#visualization-of-metrics) can be deployed as sidecar to the NetScaler CPX and collects metrics from the NetScaler CPX instance. You can then [visualize these metrics](https://docs.netscaler.com/en-us/citrix-k8s-ingress-controller/metrics/promotheus-grafana.html) using Prometheus Operator and Grafana.

> **Note:**
>
> Ensure that you have installed [Prometheus Operator](https://github.com/coreos/prometheus-operator).

Use the following command for this:

  ```
  helm install my-release netscaler/citrix-cloud-native --set cpx.enabled=true,cpx.license.accept=yes,cpx.ingressClass[0]=<ingressClassName>,cpx.exporter.required=true
  ```

### For OpenShift:
Add the name of the service account created when the chart is deployed to the privileged Security Context Constraints of OpenShift:

   ```
   oc adm policy add-scc-to-user privileged system:serviceaccount:<namespace>:<service-account-name>
   ```

#### 1. NetScaler CPX with NetScaler Ingress Controller running as side car.
To install the chart with the release name, `my-release`, use the following command:

  ```
  helm install my-release netscaler/citrix-cloud-native --set cpx.enabled=true,cpx.license.accept=yes,cpx.openshift=true
  ```

#### 2. NetScaler CPX with NetScaler Ingress Controller and Exporter running as side car.
[Metrics exporter](https://github.com/netscaler/netscaler-k8s-ingress-controller/tree/master/metrics-visualizer#visualization-of-metrics) can be deployed as sidecar to the NetScaler CPX and collects metrics from the NetScaler CPX instance. You can then [visualize these metrics](https://docs.netscaler.com/en-us/citrix-k8s-ingress-controller/metrics/promotheus-grafana.html) using Prometheus Operator and Grafana.

> **Note:**
>
> Ensure that you have installed [Prometheus Operator](https://github.com/coreos/prometheus-operator).

Use the following command for this:

  ```
  helm install my-release netscaler/citrix-cloud-native --set cpx.enabled=true,cpx.license.accept=yes,cpx.openshift=true,cpx.exporter.required=true
  ```
### Installed components

The following components are installed:

-  [NetScaler CPX](https://docs.netscaler.com/en-us/citrix-adc-cpx//netscaler-cpx.html)
-  [NetScaler ingress controller](https://github.com/netscaler/netscaler-k8s-ingress-controller) (if enabled)
-  [Exporter](https://github.com/netscaler/netscaler-adc-metrics-exporterporter) (if enabled)

### NetScaler CPX Service Annotations:

   The parameter `serviceAnnotations` can be used to annotate CPX service while installing NetScaler CPX using this helm chart.
   For example, if CPX is getting deployed in Azure and an Azure Internal Load Balancer is required before CPX then the annotation `service.beta.kubernetes.io/azure-load-balancer-internal:True` can be set in CPX service using Helm command:

   ```
   helm install my-release netscaler/citrix-cpx-with-ingress-controller --set license.accept=yes,serviceAnnotations.service\\.beta\\.kubernetes\\.io/azure-load-balancer-internal=True
   ```

   or the same can be provided in [values.yaml](https://github.com/netscaler/netscaler-helm-charts/blob/master/citrix-cpx-with-ingress-controller/values.yaml):

   ```
   license:
     accept: yes
   serviceAnnotations:
     service.beta.kubernetes.io/azure-load-balancer-internal: True
   ```

   which can be used to install NetScaler CPX using Helm command:

   ```
   helm install my-release netscaler/citrix-cpx-with-ingress-controller -f values.yaml
   ```

   To know more about service annotations supported by Kubernetes on various platforms please see [this](https://kubernetes.io/docs/concepts/services-networking/service/)

### NetScaler CPX Service Ports:

   By default, port 80 and 443 of CPX service will exposed when CPX is installed using this helm chart. If it is required to expose any other ports in CPX service then the parameter `servicePorts` can be used for it.
   For example, if port 9999 is required to be exposed then below helm command can be used for installing NetScaler CPX:

   ```
   helm install my-release netscaler/citrix-cpx-with-ingress-controller --set license.accept=yes,servicePorts[0].port=9999,servicePorts[0].protocol=TCP,servicePorts[0].name=https
   ```

   or the same can be provided in [values.yaml](https://github.com/netscaler/netscaler-helm-charts/blob/master/citrix-cpx-with-ingress-controller/values.yaml):

   ```
   license:
     accept: yes
   servicePorts:
     - port: 9090
       protocol: TCP
       name: https
   ```

   which can be used to install NetScaler using Helm command:

   ```
   helm install my-release netscaler/citrix-cpx-with-ingress-controller -f values.yaml
   ```

> **Note:** If `servicePorts` parameters is used, only ports provided in this parameter will be exposed in CPX service.
> If you want to expose default ports 80 or 443, then you will need to explicity mention these also in this parameter.

### Configuration for ServiceGraph:
   If NetScaler CPX need to send data to the NetScaler ADM to bring up the servicegraph, then the below steps can be followed to install NetScaler CPX with ingress controller. NetScaler ingress controller configures NetScaler CPX with the configuration required for servicegraph.

   1. Create secret using NetScaler Agent credentials, which will be used by NetScaler CPX to communicate with NetScaler ADM Agent:

	kubectl create secret generic admlogin --from-literal=username=<adm-agent-username> --from-literal=password=<adm-agent-password>

   2. Deploy NetScaler CPX with NetScaler ingress controller using helm command:

	helm install my-release netscaler/citrix-cloud-native --set cpx.enabled=true,cpx.license.accept=yes,cpx.analyticsConfig.required=true,cpx.analyticsConfig.timeseries.metrics.enable=true,cpx.analyticsConfig.distributedTracing.enable=true,cpx.analyticsConfig.endpoint.server=<ADM-Agent-IP>,cpx.ADMSettings.ADMIP=<ADM-Agent-IP>,cpx.ADMSettings.loginSecret=<Secret-for-ADM-Agent-credentials>

> **Note:**
> If container agent is being used here for NetScaler ADM, please provide `svcIP` of container agent in the `cpx.analyticsConfig.endpoint.server` parameter.

## NetScaler CPX DaemonSet with NetScaler Ingress Controller as sidecar for BGP Advertisement

   The previous section of deploying CPX as a Deployment  requires a Tier-1 Loadbalancer such as NetScaler VPX or cloud loadbalancers to route the traffic to CPX instances running in Kubernetes cluster, but you can also leverage  BGP network fabric in your on-prem environemnt to route the traffic to CPX instances in a Kubernetes or Openshift cluster. you need to deploy CPX with NetScaler Ingress Controller as Daemonset to advertise the ExternalIPs of the K8s services of type LoadBalancer to your BGP Fabric. NetScaler CPX establishes a BGP peering session with your network routers, and uses that peering session to advertise the IP addresses of external cluster services. If your routers have ECMP capability, the traffic is load-balanced to multiple CPX instances by the upstream router, which in turn load-balances to actual application pods. When you deploy the NetScaler CPX with this mode, NetScaler CPX adds iptables rules for each service of type LoadBalancer on Kubernetes nodes. The traffic destined to the external IP address is routed to NetScaler CPX pods. You can also set the 'ingressIP' variable to an IP Address to advertise the External IP address for Ingress resources. Refer [documentation](https://github.com/netscaler/netscaler-k8s-ingress-controller/blob/master/docs/network/bgp-enhancement.md) for complete details about BGP advertisement with CPX.

### Download the chart
You can download the chart usimg `helm pull` command.
```
helm repo add netscaler https://netscaler.github.io/netscaler-helm-charts/
helm pull citrix/citrix-cpx-with-ingress-controller
tar -zxvf citrix-cpx-with-ingress-controller-x.y.z.tgz
```

### Edit the BGP configuration in values.yaml
BGP configurations enables CPX to peer with neighbor routers for advertisting the routes for Service of Type LoadBalancer. NetScaler Ingress Controllers uses static IPs given in Service YAML or using an IPAM controller to allocate an External IP address, and same is advertisted to the neighbour router with the Gateway as Node IP. An example BGP configurations is given below.

```
# BGP configurations: local AS, remote AS and remote address is mandatory to provide.
bgpSettings:
  required: true
  bgpConfig:
  - bgpRouter:
      # Local AS number for BGP advertisement
      localAS:
      neighbor:
        # Address of the nighbor router for BGP advertisement
      - address: xx.xx.xx.xx
        # Remote AS number
        remoteAS:
        advertisementInterval: 10
        ASOriginationInterval: 10
```
If the cluster spawns across multiple networks, you can also specify the NodeSelector to give different neighbors for different Cluster Nodes as shown below.

```
bgpSettings:
  required: true
  bgpConfig:
  - nodeSelector: datacenter=ds1
    bgpRouter:
      localAS:
      neighbor:
      - address: xx.xx.xx.xx
        remoteAS:
        advertisementInterval: 10
        ASOriginationInterval: 10
  - nodeSelector: datacenter=ds2
    bgpRouter:
      localAS:
      neighbor:
      - address: yy.yy.yy.yy
        remoteAS:
        advertisementInterval: 10
        ASOriginationInterval: 10
```

### Deploy the chart
#### For Kubernetes:
#### 1. NetScaler CPX DaemonSet with NetScaler Ingress Controller running as side car for BGP Advertisement.


To install the chart with the release name ``` my-release```:

   ```
   helm install my-release ./citrix-cpx-with-ingress-controller --set license.accept=yes,cpxBgpRouter=true
   ```
If you are running NetScaler IPAM for auto allocation of IPs for Service of type LoadBalancer, you must enable the IPAM configurations in NetScaler Ingress Controller as show below:

   ```
   helm install my-release ./citrix-cpx-with-ingress-controller --set license.accept=yes,cpxBgpRouter=true,ipam=true
   ```

If you are using ingress resources, you must set the `ingressIP` to a valid IP Address which will enable the BGP route advertisement for this IP when ingress resource is deployed.

   ```
   helm install my-release ./citrix-cpx-with-ingress-controller --set license.accept=yes,cpxBgpRouter=true,ingressIP=<Ingress External IP Address>
   ```

> **Note:**
>
> By default the chart installs the recommended [RBAC](https://kubernetes.io/docs/admin/authorization/rbac/) roles and role bindings.

The command deploys NetScaler CPX Daemonset with NetScaler ingress controller as a sidecar on the Kubernetes cluster with the default configuration. The [configuration](#configuration) section lists the mandatory and optional parameters that you can configure during installation.

#### 2. NetScaler CPX with NetScaler Ingress Controller and Exporter running as side car for BGP Advertisement.
[Metrics exporter](https://github.com/netscaler/netscaler-k8s-ingress-controller/tree/master/metrics-visualizer#visualization-of-metrics) can be deployed as sidecar to the NetScaler CPX and collects metrics from the NetScaler CPX instance. You can then [visualize these metrics](https://docs.netscaler.com/en-us/citrix-k8s-ingress-controller/metrics/promotheus-grafana.html) using Prometheus Operator and Grafana.
> **Note:**
>
> Ensure that you have installed [Prometheus Operator](https://github.com/coreos/prometheus-operator).

Use the following command for this:
   ```
   helm install my-release ./citrix-cpx-with-ingress-controller --set license.accept=yes,cpxBgpRouter=true,exporter.required=true
   ```
If you are running NetScaler IPAM controller for auto allocation of IPs for Service of type LoadBalancer, you must enable the IPAM configurations in NetScaler Ingress Controller as show below:

   ```
   helm install my-release ./citrix-cpx-with-ingress-controller --set license.accept=yes,cpxBgpRouter=true,ipam=true,exporter.required=true
   ```

   If you are using ingress resources, you must set the `ingressIP` to a valid IP Address which will enable the BGP route advertisement for this IP when ingress resource is deployed.

   ```
   helm install my-release ./citrix-cpx-with-ingress-controller --set license.accept=yes,cpxBgpRouter=true,ingressIP=<Ingress external IP>, exporter.required=true
   ```

#### For OpenShift:
Add the name of the service account created when the chart is deployed to the privileged Security Context Constraints of OpenShift:

   ```
   oc adm policy add-scc-to-user privileged system:serviceaccount:<namespace>:<service-account-name>
   ```

#### 1. NetScaler CPX DaemonSet with NetScaler Ingress Controller running as side car for BGP Advertisement.
To install the chart with the release name, `my-release`, use the following command:
   ```
   helm install my-release ./citrix-cpx-with-ingress-controller --set license.accept=yes,cpxBgpRouter=true,openshift=true
   ```
If you are running NetScaler IPAM controller for auto allocation of IPs for Service of type LoadBalancer, you must enable the IPAM configurations in NetScaler Ingress Controller as show below:

   ```
   helm install my-release ./citrix-cpx-with-ingress-controller --set license.accept=yes,cpxBgpRouter=true,ipam=true,openshift=true
   ```

   If you are using ingress or Route resources, you must set the `ingressIP` to a valid IP Address which will enable the BGP route advertisement for this IP when ingress resource is deployed.

   ```
   helm install my-release ./citrix-cpx-with-ingress-controller --set license.accept=yes,cpxBgpRouter=true,ingressIP=<Ingress External IP Address>,openshift=true
   ```

#### 2. NetScaler CPX with NetScaler Ingress Controller and Exporter running as side car for BGP Advertisement.
[Metrics exporter](https://github.com/netscaler/netscaler-k8s-ingress-controller/tree/master/metrics-visualizer#visualization-of-metrics) can be deployed as sidecar to the NetScaler CPX and collects metrics from the NetScaler CPX instance. You can then [visualize these metrics](https://docs.netscaler.com/en-us/citrix-k8s-ingress-controller/metrics/promotheus-grafana.html) using Prometheus Operator and Grafana.
> **Note:**
>
> Ensure that you have installed [Prometheus Operator](https://github.com/coreos/prometheus-operator).

Use the following command for this:
   ```
   helm install my-release ./citrix-cpx-with-ingress-controller --set license.accept=yes,openshift=true,openshift=true,exporter.required=true
   ```
If you are running NetScaler IPAM controller for auto allocation of IPs for Service of type LoadBalancer, you must enable the IPAM configurations in NetScaler Ingress Controller as show below:

   ```
   helm install my-release ./citrix-cpx-with-ingress-controller --set license.accept=yes,cpxBgpRouter=true,ipam=true,openshift=true,exporter.required=true
   ```

   If you are using ingress or Route resources, you must set the `ingressIP` to a valid IP Address which will enable the BGP route advertisement for this IP when ingress resource is deployed.

   ```
   helm install my-release ./citrix-cpx-with-ingress-controller --set license.accept=yes,cpxBgpRouter=true,ingressIP=<Ingress External IP Address>,openshift=true,exporter.required=true
   ```

## CRDs configuration

CRDs can be installed/upgraded when we install/upgrade NetScaler CPX with NetScaler ingress controller using `crds.install=true` parameter in Helm. If you do not want to install CRDs, then set the option `crds.install` to `false`. By default, CRDs too get deleted if you uninstall through Helm. This means, even the CustomResource objects created by the customer will get deleted. If you want to avoid this data loss set `crds.retainOnDelete` to `true`.

> **Note:**
> Installing again may fail due to the presence of CRDs. Make sure that you back up all CustomResource objects and clean up CRDs before re-installing NetScaler CPX with NetScaler ingress controller.

There are a few examples of how to use these CRDs, which are placed in the folder: [Example-CRDs](https://github.com/netscaler/netscaler-helm-charts/tree/master/example-crds). Refer to them and install as needed, using the following command:
```kubectl create -f <crd-example.yaml>```

### Details of the supported CRDs:

#### authpolicies CRD:

Authentication policies are used to enforce access restrictions to resources hosted by an application or an API server.

NetScaler provides a Kubernetes CustomResourceDefinitions (CRDs) called the [Auth CRD](https://github.com/netscaler/netscaler-k8s-ingress-controller/tree/master/crd/auth) that you can use with the NetScaler ingress controller to define authentication policies on the ingress NetScaler.

Example file: [auth_example.yaml](https://github.com/netscaler/netscaler-helm-charts/tree/master/example-crds/auth_example.yaml)

#### continuousdeployments CRD  for canary:

Canary release is a technique to reduce the risk of introducing a new software version in production by first rolling out the change to a small subset of users. After user validation, the application is rolled out to the larger set of users. NetScaler-Integrated [Canary Deployment solution](https://github.com/netscaler/netscaler-k8s-ingress-controller/tree/master/crd/canary) stitches together all components of continuous delivery (CD) and makes canary deployment easier for the application developers.

#### httproutes and listeners CRDs for contentrouting:

[Content Routing (CR)](https://github.com/netscaler/netscaler-k8s-ingress-controller/tree/master/crd/contentrouting) is the execution of defined rules that determine the placement and configuration of network traffic between users and web applications, based on the content being sent. For example, a pattern in the URL or header fields of the request.

Example files: [HTTPRoute_crd.yaml](https://github.com/netscaler/netscaler-helm-charts/tree/master/example-crds/HTTPRoute_crd.yaml), [Listener_crd.yaml](https://github.com/netscaler/netscaler-helm-charts/tree/master/example-crds/Listener_crd.yaml)

#### ratelimits CRD:

In a Kubernetes deployment, you can [rate limit the requests](https://github.com/netscaler/netscaler-k8s-ingress-controller/tree/master/crd/ratelimit) to the resources on the back end server or services using rate limiting feature provided by the ingress NetScaler.

Example files: [ratelimit-example1.yaml](https://github.com/netscaler/netscaler-helm-charts/tree/master/example-crds/ratelimit-example1.yaml), [ratelimit-example2.yaml](https://github.com/netscaler/netscaler-helm-charts/tree/master/example-crds/ratelimit-example2.yaml)

#### vips CRD:

NetScaler provides a CustomResourceDefinitions (CRD) called [VIP](https://github.com/netscaler/netscaler-k8s-ingress-controller/tree/master/crd/vip) for asynchronous communication between the IPAM controller and NetScaler ingress controller.

The IPAM controller is provided by NetScaler for IP address management. It allocates IP address to the service from a defined IP address range. The NetScaler ingress controller configures the IP address allocated to the service as virtual IP (VIP) in NetScaler ADX VPX. And, the service is exposed using the IP address.

When a new service is created, the NetScaler ingress controller creates a CRD object for the service with an empty IP address field. The IPAM Controller listens to addition, deletion, or modification of the CRD and updates it with an IP address to the CRD. Once the CRD object is updated, the NetScaler ingress controller automatically configures NetScaler-specfic configuration in the tier-1 NetScaler VPX.

#### rewritepolicies CRD:

In kubernetes environment, to deploy specific layer 7 policies to handle scenarios such as, redirecting HTTP traffic to a specific URL, blocking a set of IP addresses to mitigate DDoS attacks, imposing HTTP to HTTPS and so on, requires you to add appropriate libraries within the microservices and manually configure the policies. Instead, you can use the [Rewrite and Responder features](https://github.com/netscaler/netscaler-k8s-ingress-controller/blob/master/crd/rewrite-policy/rewrite-responder-policies-deployment.yaml) provided by the Ingress NetScaler device to deploy these policies.

Example files: [target-url-rewrite.yaml](https://github.com/netscaler/netscaler-k8s-ingress-controller/blob/master/simplified-deployment-usecases/CRDs/rewrite.md#url-manipulation)

#### wafs CRD:

[WAF CRD](https://github.com/netscaler/netscaler-k8s-ingress-controller/blob/master/docs/crds/waf.md) can be used to configure the web application firewall policies with the NetScaler ingress controller on the NetScaler VPX, MPX, SDX, and CPX. The WAF CRD enables communication between the NetScaler ingress controller and NetScaler for enforcing web application firewall policies.

In a Kubernetes deployment, you can enforce a web application firewall policy to protect the server using the WAF CRD. For more information about web application firewall, see [Web application security](https://docs.netscaler.com/en-us/citrix-adc/current-release/application-firewall/introduction-to-citrix-web-app-firewall.html).

Example files: [wafhtmlxsssql.yaml](https://github.com/netscaler/netscaler-helm-charts/tree/master/example-crds/wafhtmlxsssql.yaml)

## NetScaler CPX servicetype LoadBalancer
NetScaler CPX can be installed with service having servicetype LoadBalancer. Following arguments can be used in the `helm install` command for the same:

```
helm install citrix-cpx-with-ingress-controller netscaler/citrix-cloud-native --set cpx.enabled=True,cpx.license.accept=yes,cpx.serviceType.loadBalancer.enabled=True
```

## NetScaler CPX servicetype NodePort
NetScaler CPX can be installed with service having servicetype Nodeport. Following arguments can be used in the `helm install` command for the same:

```
helm install citrix-cpx-with-ingress-controller netscaler/citrix-cloud-native --set cpx.enabled=True,cpx.license.accept=yes,cpx.serviceType.nodePort.enabled=True
```

Additionally, `cpx.serviceType.nodePort.httpPort` and `cpx.serviceType.nodePort.httpsPort` arguments can be used to select the nodePort for the CPX service for HTTP and HTTPS ports.

#### CORS CRD:

[CORS CRD](https://github.com/netscaler/netscaler-k8s-ingress-controller/blob/master/docs/crds/cors.md) Cross-origin resource sharing (CORS) is a mechanism allows a web application running under one domain to securely access resources in another domain. You can configure CORS policies on NetScaler using NetScaler ingress controller to allow one domain (the origin domain) to call APIs in another domain. For more information, see the [cross-origin resource sharing CRD](https://github.com/netscaler/netscaler-k8s-ingress-controller/blob/master/docs/crds/cors.md) documentation.

Example files: [cors-crd.yaml](https://github.com/netscaler/netscaler-helm-charts/tree/master/example-crds/corspolicy-example.yaml)

#### APPQOE CRD:

[APPQOE CRD](https://github.com/netscaler/netscaler-k8s-ingress-controller/blob/master/docs/crds/appqoe.md) When a NetScaler appliance receives an HTTP request and forwards it to a back-end server, sometimes there may be connection failures with the back-end server. You can configure the request-retry feature on NetScaler to forward the request to the next available server, instead of sending the reset to the client. Hence, the client saves round trip time when NetScaler initiates the same request to the next available service.
For more information, see the AppQoE support documentation. [Appqoe resource sharing CRD](https://github.com/netscaler/netscaler-k8s-ingress-controller/blob/master/docs/crds/appqoe.md) documentation.

Example files: [appqoe-crd.yaml](https://github.com/netscaler/netscaler-helm-charts/tree/master/example-crds/appqoe_example.yaml)

#### WILDCARDDNS CRD:

[WILDCARDDNS CRD](https://github.com/netscaler/netscaler-k8s-ingress-controller/blob/master/docs/crds/wildcarddns.md) Wildcard DNS domains are used to handle requests for nonexistent domains and subdomains. In a zone, use wildcard domains to redirect queries for all nonexistent domains or subdomains to a particular server, instead of creating a separate Resource Record (RR) for each domain. The most common use of a wildcard DNS domain is to create a zone that can be used to forward mail from the internet to some other mail system.
For more information, see the Wild card DNS domains support documentation. [Wildcard DNS Entry CRD](https://github.com/netscaler/netscaler-k8s-ingress-controller/blob/master/docs/crds/wildcarddns.md) documentation.

Example files: [wildcarddns-crd.yaml](https://github.com/netscaler/netscaler-helm-charts/tree/master/example-crds/wildcarddns-example.yaml)

### Tolerations

Taints are applied on cluster nodes whereas tolerations are applied on pods. Tolerations enable pods to be scheduled on node with matching taints. For more information see [Taints and Tolerations in Kubernetes](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/).

Toleration can be applied to pod running NetScaler CPX and ingress controller containers using `tolerations` argument while deploying CPX+NSIC using helm chart. This argument takes list of tolerations that user need to apply on the CPX+NSIC pods.

For example, following command can be used to apply toleration on the CPX+NSIC pod:

```
helm install my-release netscaler/citrix-cloud-native --set cpx.enabled=true,cpx.license.accept=yes,cpx.tolerations[0].key=<toleration-key>,cpx.tolerations[0].value=<toleration-value>,cpx.tolerations[0].operator=<toleration-operator>,cpx.tolerations[0].effect=<toleration-effect>
```

Here tolerations[0].key, tolerations[0].value and tolerations[0].effect are the key, value and effect that was used while tainting the node.
Effect represents what should happen to the pod if the pod don't have any matching toleration. It can have values `NoSchedule`, `NoExecute` and `PreferNoSchedule`.
Operator represents the operation to be used for key and value comparison between taint and tolerations. It can have values `Exists` and `Equal`. The default value for operator is `Equal`.

### Resource Quotas
There are various use-cases when resource quotas are configured on the Kubernetes cluster. If quota is enabled in a namespace for compute resources like cpu and memory, users must specify requests or limits for those values; otherwise, the quota system may reject pod creation. The resource quotas for the NSIC and CPX containers can be provided explicitly in the helm chart.

To set requests and limits for the NSIC container, use the variables `cpx.cic.resources.requests` and `cpx.cic.resources.limits` respectively.
Similarly, to set requests and limits for the CPX container, use the variable `cpx.resources.requests` and `cpx.resources.limits` respectively.

Below is an example of the helm command that configures

A) For NSIC container:

  CPU request for 500milli CPUs

  CPU limit at 1000m
  
  Memory request for 512M
  
  Memory limit at 1000M

B) For CPX container:
  
  CPU request for 250milli CPUs
  
  CPU limit at 500m
  
  Memory request for 256M
  
  Memory limit at 512M

```
helm install citrix-cpx-with-ingress-controller netscaler/citrix-cloud-native --set cpx.enabled=true,license.accept=yes --set cpx.cic.resources.requests.cpu=500m,cpx.cic.resources.requests.memory=512Mi,cpx.cic.resources.limits.cpu=1000m,cpx.cic.resources.limits.memory=1000Mi --set cpx.resources.limits.cpu=500m,cpx.resources.limits.memory=512Mi,cpx.resources.requests.cpu=250m,cpx.resources.requests.memory=256Mi
```

### Analytics Configuration
#### Analytics Configuration required for ADM

If NetScaler CPX needs to send data to the ADM for analytics purpose, then the below steps can be followed to install NetScaler CPX with ingress controller. NSIC configures the NetScaler CPX with the configuration required for analytics.

1. Create secret using ADM Agent credentials, which will be used by NetScaler CPX to communicate with ADM Agent:

```
kubectl create secret generic admlogin --from-literal=username=<adm-agent-username> --from-literal=password=<adm-agent-password>
```

|Note: If you have installed container based `adm-agent` using [this](https://github.com/netscaler/netscaler-helm-charts/tree/master/adm-agent) helm chart, above step is not required, you just need to tag the namespace where the CPX is being deployed with `citrix-cpx=enabled`.

2. Deploy NetScaler CPX with NSIC using helm command:

```
helm repo add netscaler https://netscaler.github.io/netscaler-helm-charts/

helm install citrix-cpx-with-ingress-controller netscaler/citrix-cloud-native --set cpx.enabled=true --set license.accept=yes,cpx.analyticsConfig.required=true,cpx.analyticsConfig.distributedTracing.enable=true,cpx.analyticsConfig.endpoint.service=<Namespace/ADM_ServiceName-logstream>,cpx.ADMSettings.ADMIP=<ADM-Agent-IP_OR_FQDN>,cpx.ADMSettings.loginSecret=<Secret-for-ADM-Agent-credentials>,cpx.analyticsConfig.transactions.enable=true,cpx.analyticsConfig.transactions.port=5557
```

|Note: For container based ADM agent, please provide the logstream service FQDN in `analyticsConfig.endpoint.service`. The `logstream` service will be running on port `5557`.

#### Analytics Configuration required for NSOE

If NetScaler CPX needs to send data to the NSOE for observability, then the below steps can be followed to install NetScaler CPX with ingress controller. NSIC configures NetScaler CPX with the configuration required.

Deploy NetScaler CPX with NSIC using helm command:

```
helm repo add netscaler https://netscaler.github.io/netscaler-helm-charts/

helm install citrix-cpx-with-ingress-controller netscaler/citrix-cloud-native --set cpx.enabled=true --set license.accept=yes,cpx.analyticsConfig.required=true,cpx.analyticsConfig.timeseries.metrics.enable=true,cpx.analyticsConfig.timeseries.port=5563,cpx.analyticsConfig.timeseries.metrics.mode=prometheus,cpx.analyticsConfig.transactions.enable=true,cpx.analyticsConfig.transactions.port=5557,cpx.analyticsConfig.distributedTracing.enable=true,cpx.analyticsConfig.endpoint.server=<NSOE_SERVICE_IP>,cpx.analyticsConfig.endpoint.service=<Namespace/NSOE_SERVICE_NAME>
```

#### Analytics Configuration required for export of metrics to Prometheus

If NetScaler CPX needs to send data to Prometheus directly without an exporter resource in between, then the below steps can be followed to install NetScaler CPX with ingress controller. NSIC configures NetScaler CPX with the configuration required.

1. Create secret to enable read-only access for a user, which will be required by NetScaler CPX to export metrics to Prometheus.

```
kubectl create secret generic prom-user --from-literal=username=<prometheus-username> --from-literal=password=<prometheus-password>
```

2. Deploy NetScaler CPX with NSIC using helm command:

```
helm repo add netscaler https://netscaler.github.io/netscaler-helm-charts/

helm install citrix-cpx-with-ingress-controller netscaler/citrix-cloud-native --set cpx.enabled=true,cpx.license.accept=yes,cpx.cic.prometheusCredentialSecret=<Secret-for-read-only-user-creation>,cpx.analyticsConfig.required=true,cpx.analyticsConfig.timeseries.metrics.enable=true,cpx.analyticsConfig.timeseries.port=5563,cpx.analyticsConfig.timeseries.metrics.mode=prometheus,cpx.analyticsConfig.timeseries.metrics.enableNativeScrape=true
```

3. To setup Prometheus in order to scrape natively from NetScaler CPX pod, a new scrape job is required to be added under scrape_configs in the prometheus [configuration](https://prometheus.io/docs/prometheus/latest/configuration/configuration/). For more details, check kubernetes_sd_config [here](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config). A sample of the Prometheus job is given below -

```
    - job_name: 'kubernetes-cpx'
      scheme: http
      metrics_path: /nitro/v1/config/systemfile
      params:
        args: ['filename:metrics_prom_ns_analytics_time_series_profile.log,filelocation:/var/nslog']
        format: ['prometheus']
      basic_auth:
        username:  # Prometheus username set in cpx.cic.prometheusCredentialSecret
        password:  # Prometheus password set in cpx.cic.prometheusCredentialSecret
      scrape_interval: 30s
      kubernetes_sd_configs:
      - role: pod
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_netscaler_prometheus_scrape]
        action: keep
        regex: true
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_netscaler_prometheus_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: kubernetes_pod_name
```

> **Note:**
>
> For more details on Prometheus integration, please refer to [this](https://docs.netscaler.com/en-us/citrix-adc/current-release/observability/prometheus-integration)

### NetScaler CPX License Provisioning
#### Bandwidth based licensing

By default, CPX runs with 20 Mbps bandwidth called as [CPX Express](https://www.netscaler.com/platform/cpx-container). However, for better performance and production deployments, customer needs licensed CPX instances. [NetScaler ADM](https://docs.netscaler.com/en-us/citrix-application-delivery-management-service/) is used to check out licenses for NetScaler CPX. For more detail on CPX licensing please refer [this](https://docs.netscaler.com/en-us/citrix-adc-cpx/current-release/cpx-licensing.html).

For provisioning licensing on NetScaler CPX, it is mandatory to provide License Server information to CPX. This can be done by setting **ADMSettings.licenseServerIP** as License Server IP. In addition to this, **ADMSettings.bandWidthLicense** needs to be set true and desired bandwidth capacity in Mbps should be set **ADMSettings.bandWidth**.
For example, to set 2Gbps as bandwidth capacity, below command can be used.

 ```
helm repo add netscaler https://netscaler.github.io/netscaler-helm-charts/

helm install citrix-cpx-with-ingress-controller netscaler/citrix-cloud-native --set cpx.enabled=true --set license.accept=yes --set cpx.ADMSettings.licenseServerIP=<LICENSESERVER_IP_OR_FQDN>,cpx.ADMSettings.bandWidthLicense=True --set cpx.ADMSettings.bandWidth=2000,cpx.ADMSettings.licenseEdition="ENTERPRISE"
```

#### vCPU based licensing

For vCPU based licensing on NetScaler CPX, set `cpx.ADMSettings.vCPULicense` as True and `cpx.ADMSettings.cpxCores` with the number of cores that can be allocated for the CPX.

```
helm repo add netscaler https://netscaler.github.io/netscaler-helm-charts/

helm install citrix-cpx-with-ingress-controller netscaler/citrix-cloud-native --set cpx.enabled=true --set license.accept=yes --set cpx.ADMSettings.licenseServerIP=<LICENSESERVER_IP_OR_FQDN>,cpx.ADMSettings.vCPULicense=True --set cpx.ADMSettings.cpxCores=4,cpx.ADMSettings.licenseEdition="ENTERPRISE"
```

### Bootup Configuration for NetScaler CPX
To add bootup config on NetScaler CPX, add commands below `cpxCommands` and `cpxShellCommands` in the values.yaml file. The commands will be executed in order.

For e.g. to add `X-FORWARDED-PROTO` header in all request packets processed by the CPX, add below commands under `cpxCommands` in the `values.yaml` file.

```
cpxCommands: |
  add rewrite action rw_act_x_forwarded_proto insert_http_header X-Forwarded-Proto "\"https\""
  add rewrite policy rw_pol_x_forwarded_proto CLIENT.SSL.IS_SSL rw_act_x_forwarded_proto
  bind rewrite global rw_pol_x_forwarded_proto 10 -type REQ_OVERRIDE
```

Commands that needs to be executed in shell of CPX should be kept under `cpxShellCommands` in the `values.yaml` file.

```
cpxShellCommands: |
  touch /etc/a.txt
  echo "this is a" > /etc/a.txt
  echo "this is the file" >> /etc/a.txt
  ls >> /etc/a.txt
```

## Configuration
The following table lists the configurable parameters of the NetScaler CPX with NetScaler ingress controller as side car chart and their default values.

| Parameters | Mandatory or Optional | Default value | Description |
| ---------- | --------------------- | ------------- | ----------- |
| cpx.enabled | Mandatory | False | Set to "True" for deploying NetScaler CPX with NetScaler Ingress Controller running as sidecar. |
| cpx.license.accept | Mandatory | no | Set `yes` to accept the NetScaler ingress controller end user license agreement. |
| cpx.imageRegistry                   | Mandatory  |  `quay.io`               |  The NetScaler CPX image registry             |  
| cpx.imageRepository                 | Mandatory  |  `citrix/citrix-k8s-cpx-ingress`              |   The NetScaler CPX image repository             | 
| cpx.imageTag                  | Mandatory  |  `13.1-49.15`               |   The NetScaler CPX image tag            | 
| cpx.pullPolicy | Mandatory | IfNotPresent | The NetScaler CPX image pull policy. |
| cpx.daemonSet | Optional | False | Set this to true if NetScaler CPX needs to be deployed as DaemonSet. |
| cpx.cic.imageRegistry                   | Mandatory  |  `quay.io`               |  The NetScaler ingress controller image registry             |  
| cpx.cic.imageRepository                 | Mandatory  |  `citrix/citrix-k8s-ingress-controller`              |   The NetScaler ingress controller image repository             | 
| cpx.cic.imageTag                  | Mandatory  |  `1.37.5`               |   The NetScaler ingress controller image tag            | 
| cpx.cic.pullPolicy | Mandatory | IfNotPresent | The NetScaler ingress controller image pull policy. |
| cpx.cic.required | Mandatory | true | NSIC to be run as sidecar with NetScaler CPX |
| cpx.cic.resources | Optional | {} |	CPU/Memory resource requests/limits for NetScaler Ingress Controller container |
| cpx.cic.rbacRole  | Optional |  false  |  To deploy NSIC with RBAC Role set rbacRole=true; by default NSIC gets installed with RBAC ClusterRole(rbacRole=false)) |
| cpx.cic.prometheusCredentialSecret  | Optional |  N/A  |  The secret key required to create read only user for native export of metrics using Prometheus. |
| cpx.imagePullSecrets | Optional | N/A | Provide list of Kubernetes secrets to be used for pulling the images from a private Docker registry or repository. For more information on how to create this secret please see [Pull an Image from a Private Registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/). |
| cpx.nameOverride | Optional | N/A | String to partially override deployment fullname template with a string (will prepend the release name) |
| cpx.fullNameOverride | Optional | N/A | String to fully override deployment fullname template with a string |
| cpx.resources | Optional | {} |	CPU/Memory resource requests/limits for NetScaler CPX container |
| cpx.nitroReadTimeout | Optional | 20 | The nitro Read timeout in seconds, defaults to 20 |
| cpx.logLevel | Optional | INFO | The loglevel to control the logs generated by NSIC. The supported loglevels are: CRITICAL, ERROR, WARNING, INFO, DEBUG and TRACE. For more information, see [Logging](https://github.com/netscaler/netscaler-k8s-ingress-controller/blob/master/docs/configure/log-levels.md).|
| cpx.jsonLog | Optional | false | Set this argument to true if log messages are required in JSON format | 
| cpx.nsConfigDnsRec | Optional | false | To enable/disable DNS address Record addition in NetScaler through Ingress |
| cpx.nsSvcLbDnsRec | Optional | false | To enable/disable DNS address Record addition in NetScaler through Type Load Balancer Service |
| cpx.nsDnsNameserver | Optional | N/A | To add DNS Nameservers in NetScaler |
| cpx.optimizeEndpointBinding | Optional | false | To enable/disable binding of backend endpoints to servicegroup in a single API-call. Recommended when endpoints(pods) per application are large in number. Applicable only for NetScaler Version >=13.0-45.7  |
| cpx.defaultSSLCertSecret | Optional | N/A | Provide Kubernetes secret name that needs to be used as a default non-SNI certificate in NetScaler. |
| cpx.nsHTTP2ServerSide | Optional | OFF | Set this argument to `ON` for enabling HTTP2 for NetScaler service group configurations. |
| cpx.cpxLicenseAggregator | Optional | N/A | IP/FQDN of the CPX License Aggregator if it is being used to license the CPX. |
| cpx.nsCookieVersion | Optional | 0 | Specify the persistence cookie version (0 or 1). |
| cpx.profileSslFrontend | Optional | N/A | Specify the frontend SSL profile. For Details see [Configuration using FRONTEND_SSL_PROFILE](https://docs.netscaler.com/en-us/citrix-k8s-ingress-controller/configure/profiles.html#global-front-end-profile-configuration-using-configmap-variables) |
| cpx.profileTcpFrontend | Optional | N/A | Specify the frontend TCP profile. For Details see [Configuration using FRONTEND_TCP_PROFILE](https://docs.netscaler.com/en-us/citrix-k8s-ingress-controller/configure/profiles.html#global-front-end-profile-configuration-using-configmap-variables) |
| cpx.profileHttpFrontend | Optional | N/A | Specify the frontend HTTP profile. For Details see [Configuration using FRONTEND_HTTP_PROFILE](https://docs.netscaler.com/en-us/citrix-k8s-ingress-controller/configure/profiles.html#global-front-end-profile-configuration-using-configmap-variables) |
| cpx.logProxy | Optional | N/A | Provide Elasticsearch or Kafka or Zipkin endpoint for NetScaler observability exporter. |
| cpx.nsProtocol | Optional | http | Protocol http or https used for the communication between NetScaler Ingress Controller and CPX |
| cpx.cpxBgpRouter | Optional | false| If set to true, this CPX is deployed as daemonset in BGP controller mode wherein BGP advertisements are done for attracting external traffic to Kubernetes clusters |
| cpx.replicaCount  | Optional  | 1 | Number of CPX-NSIC pods to be deployed. With `cpx.cpxBgpRouter : true`, replicaCount is 1 since CPX will be deployed as DaemonSet |
| cpx.nsIP | Optional | 192.168.1.2 | NSIP used by CPX for internal communication when run in Host mode, i.e when cpxBgpRouter is set to true. A /24 internal network is created in this IP range which is used for internal communications withing the network namespace. |
| cpx.nsGateway | Optional | 192.168.1.1 | Gateway used by CPX for internal communication when run in Host mode, i.e when cpxBgpRouter is set to true. If not specified, first IP in the nsIP network is used as gateway. It must be in same network as nsIP |
| cpx.bgpPort | Optional | 179 | BGP port used by CPX for BGP advertisement if cpxBgpRouter is set to true|
| cpx.ingressIP | Optional | N/A | External IP address to be used by ingress resources if not overriden by ingress.com/frontend-ip annotation in Ingress resources. This is also advertised to external routers when pxBgpRouter is set to true|
| cpx.entityPrefix | Optional | k8s | The prefix for the resources on the NetScaler CPX. |
| cpx.ingressClass | Optional | N/A | If multiple ingress load balancers are used to load balance different ingress resources. You can use this parameter to specify NetScaler ingress controller to configure NetScaler associated with specific ingress class. For more information on Ingress class, see [Ingress class support](https://docs.netscaler.com/en-us/citrix-k8s-ingress-controller/configure/ingress-classes/). For Kubernetes version >= 1.19, this will create an IngressClass object with the name specified here |
| cpx.setAsDefaultIngressClass | Optional | False | Set the IngressClass object as default ingress class. New Ingresses without an "ingressClassName" field specified will be assigned the class specified in ingressClass. Applicable only for kubernetes versions >= 1.19 |
| cpx.updateIngressStatus | Optional | False | Set this argument if you want to update ingress status of the ingress resources exposed via CPX. This is only applicable if servicetype of CPX service is LoadBalancer. |
| cpx.disableAPIServerCertVerify | Optional | False | Set this parameter to True for disabling API Server certificate verification. |
| cpx.openshift | Optional | false | Set this argument if OpenShift environment is being used. |
| cpx.disableOpenshiftRoutes | Optional | false | By default Openshift routes are processed in openshift environment, this variable can be used to disable Ingress controller processing the openshift routes. |
| cpx.routeLabels | Optional | N/A | You can use this parameter to provide the route labels selectors to be used by NetScaler Ingress Controller for routeSharding in OpenShift cluster. |
| cpx.namespaceLabels | Optional | N/A | You can use this parameter to provide the namespace labels selectors to be used by NetScaler Ingress Controller for routeSharding in OpenShift cluster. |
| cpx.sslCertManagedByAWS | Optional | False | Set this argument if SSL certs used is managed by AWS while deploying NetScaler CPX in AWS. |
| cpx.nodeSelector.key | Optional | N/A | Node label key to be used for nodeSelector option for CPX-NSIC deployment. |
| cpx.nodeSelector.value | Optional | N/A | Node label value to be used for nodeSelector option in CPX-NSIC deployment. |
| cpx.podAnnotations | Optional | N/A | Map of annotations to add to the pods. |
| cpx.affinity | Optional | N/A | Affinity labels for pod assignment. |
| cpx.tolerations | Optional | N/A | Specify the tolerations for the CPX-NSIC deployment. |
| cpx.serviceType.loadBalancer.enabled | Optional | False | Set this argument if you want servicetype of CPX service to be LoadBalancer. |
| cpx.serviceType.nodePort.enabled | Optional | False | Set this argument if you want servicetype of CPX service to be NodePort. |
| cpx.serviceType.nodePort.httpPort | Optional | N/A | Specify the HTTP nodeport to be used for NodePort CPX service. |
| cpx.serviceType.nodePort.httpsPort | Optional | N/A | Specify the HTTPS nodeport to be used for NodePort CPX service. |
| cpx.serviceAnnotations | Optional | N/A | Dictionary of annotations to be used in CPX service. Key in this dictionary is the name of the annotation and Value is the required value of that annotation. For example, [see this](#citrix-adc-cpx-service-annotations). |
| cpx.serviceSpec.externalTrafficPolicy | Optional | Cluster | Use this parameter to provide externalTrafficPolicy for CPX service of type LoadBalancer or NodePort. `serviceType.loadBalancer.enabled` or `serviceType.nodePort.enabled` should be set to `true` according to your use case for using this parameter. |
| cpx.serviceSpec.loadBalancerIP | Optional | N/A | Use this parameter to provide LoadBalancer IP to CPX service of type LoadBalancer. `serviceType.loadBalancer.enabled` should be set to `true` for using this parameter. |
| cpx.serviceSpec.loadBalancerSourceRanges | Optional | N/A | Provide the list of IP Address or range which should be allowed to access the Network Load Balancer. `serviceType.loadBalancer.enabled` should be set to `true` for using this parameter. For details, see [Network Load Balancer support on AWS](https://kubernetes.io/docs/concepts/services-networking/service/#aws-nlb-support). |
| cpx.servicePorts | Optional | N/A | List of port. Each element in this list is a dictionary that contains information about the port. For example, [see this](#citrix-adc-cpx-service-ports). |
| cpx.ADMSettings.licenseServerIP | Optional | N/A | Provide the NetScaler Application Delivery Management (ADM) IP address to license NetScaler CPX. For more information, see [Licensing]( https://docs.netscaler.com/en-us/citrix-k8s-ingress-controller/licensing/). |
| cpx.ADMSettings.licenseServerPort | Optional | 27000 | NetScaler ADM port if non-default port is used. |
| cpx.ADMSettings.ADMIP | Optional | N/A |  NetScaler Application Delivery Management (ADM) IP address. |
| cpx.ADMSettings.loginSecret | Optional | N/A | The secret key to login to the ADM. For information on how to create the secret keys, see [Prerequisites](#prerequistes). |
| cpx.ADMSettings.bandWidthLicense | Optional | False | Set to true if you want to use bandwidth based licensing for NetScaler CPX. |
| cpx.ADMSettings.bandWidth | Optional | 1000 | Desired bandwidth capacity to be set for NetScaler CPX in Mbps. |
| cpx.ADMSettings.vCPULicense | Optional | N/A | Set to true if you want to use vCPU based licensing for NetScaler CPX. |
| cpx.ADMSettings.cpxCores | Optional | 1 | Desired number of vCPU to be set for NetScaler CPX. |
| cpx.ADMSettings.licenseEdition| Optional | PLATINUM | License edition that can be Standard, Platinum and Enterprise . By default, Platinum is selected.|
| cpx.exporter.required | Optional | false | Use the argument if you want to run the [Exporter for NetScaler Stats](https://github.com/netscaler/netscaler-adc-metrics-exporterporter) along with NetScaler ingress controller to pull metrics for the NetScaler CPX|
| cpx.exporter.imageRegistry                   | Optional  |  `quay.io`               |  The Exporter for NetScaler Stats image registry             |  
| cpx.exporter.imageRepository                 | Optional  |  `citrix/citrix-adc-metrics-exporter`              |   The Exporter for NetScaler Stats image repository             | 
| cpx.exporter.imageTag                  | Optional  |  `1.4.9`               |  The Exporter for NetScaler Stats image tag|
| cpx.exporter.pullPolicy | Optional | IfNotPresent | The Exporter for NetScaler Stats image pull policy. |
| cpx.exporter.resources | Optional | {} |	CPU/Memory resource requests/limits for Metrics exporter container |
| cpx.exporter.ports.containerPort | Optional | 8888 | The Exporter for NetScaler Stats container port. |
| cpx.exporter.serviceMonitorExtraLabels | Optional |  | Extra labels for service monitor whem NetScaler-adc-metrics-exporter is enabled. | 
| cpx.analyticsConfig.required | Mandatory | false | Set this to true if you want to configure NetScaler to send metrics and transaction records to Analytics service. |
| cpx.analyticsConfig.distributedTracing.enable | Optional | false | Set this value to true to enable OpenTracing in NetScaler. |
| cpx.analyticsConfig.distributedTracing.samplingrate | Optional | 100 | Specifies the OpenTracing sampling rate in percentage. |
| cpx.analyticsConfig.endpoint.server | Optional | N/A | Set this value as the IP address or DNS address of the  analytics server. |
| cpx.analyticsConfig.endpoint.service | Optional | N/A | Set this value as the IP address or service name with namespace of the analytics service deployed in Kubernetes. Format: namespace/servicename|
| cpx.analyticsConfig.timeseries.port | Optional | 5563 | Specify the port used to expose analytics service for timeseries endpoint. |
| cpx.analyticsConfig.timeseries.metrics.enable | Optional | Set this value to true to enable sending metrics from NetScaler. |
| cpx.analyticsConfig.timeseries.metrics.mode | Optional | avro |  Specifies the mode of metric endpoint. |
| cpx.analyticsConfig.timeseries.metrics.exportFrequency | Optional | 30 |  Specifies the time interval for exporting time-series data. Possible values range from 30 to 300 seconds. |
| cpx.analyticsConfig.timeseries.metrics.schemaFile | Optional | schema.json |  Specifies the name of a schema file with the required Netscaler counters to be added and configured for metricscollector to export. A reference schema file reference_schema.json with all the supported counters is also available under the path /var/metrics_conf/. This schema file can be used as a reference to build a custom list of counters. |
| cpx.analyticsConfig.timeseries.metrics.enableNativeScrape | Optional | false |  Set this value to true for native export of metrics. |
| cpx.analyticsConfig.timeseries.auditlogs.enable | Optional | false | Set this value to true to export audit log data from NetScaler. |
| cpx.analyticsConfig.timeseries.events.enable | Optional | false | Set this value to true to export events from the NetScaler. |
| cpx.analyticsConfig.transactions.enable | Optional | false | Set this value to true to export transactions from NetScaler. |
| cpx.analyticsConfig.transactions.port | Optional | 5557 | Specify the port used to expose analytics  service for transaction endpoint. |
| cpx.crds.install | Optional | False | Unset this argument if you don't want to install CustomResourceDefinitions which are consumed by NSIC. |
| cpx.crds.retainOnDelete | Optional | false | Set this argument if you want to retain CustomResourceDefinitions even after uninstalling NSIC. This will avoid data-loss of Custom Resource Objects created before uninstallation. |
| cpx.bgpSettings.required | Optional | false | Set this argument if you want to enable BGP configurations for exposing service of Type Loadbalancer through BGP fabric|
| cpx.bgpSettings.bgpConfig | Optional| N/A| This represents BGP configurations in YAML format. For the description about individual fields, please refer the [documentation](https://github.com/netscaler/netscaler-k8s-ingress-controller/blob/master/docs/network/bgp-enhancement.md) |
| cpx.nsLbHashAlgo.required | Optional | false | Set this value to set the LB consistent hashing Algorithm |
| cpx.nsLbHashAlgo.hashFingers | Optional | 256 | Specifies the number of fingers to be used for hashing algorithm. Possible values are from 1 to 1024, Default value is 256 |
| cpx.nsLbHashAlgo.hashAlgorithm | Optional | 'default' | Specifies the supported algorithm. Supported algorithms are "default", "jarh", "prac", Default value is 'default' |
| cpx.cpxCommands| Optional | N/A | This argument accepts user-provided bootup NetScaler config that is applied as soon as the CPX is instantiated. Please note that this is not a dynamic config, and any subsequent changes to the configmap don't reflect in the CPX config unless the pod is restarted. For more info, please refer the [documentation](https://docs.netscaler.com/en-us/citrix-adc-cpx/current-release/configure-cpx-kubernetes-using-configmaps.html).  |
| cpx.cpxShellCommands| Optional | N/A | This argument accepts user-provided bootup config that is applied as soon as the CPX is instantiated. Please note that this is not a dynamic config, and any subsequent changes to the configmap don't reflect in the CPX config unless the pod is restarted. For more info, please refer the [documentation](https://docs.netscaler.com/en-us/citrix-adc-cpx/current-release/configure-cpx-kubernetes-using-configmaps.html). |

> **Note:**
>
> If NetScaler ADM related information is not provided during installation, NetScaler CPX will come up with the default license.

Alternatively, you can define a YAML file with the values for the parameters and pass the values while installing the chart.

For example:
    ```
    helm install citrix-cpx-with-ingress-controller netscaler/citrix-cloud-native -f values.yaml
    ```

> **Tip:**
>
> The [values.yaml](https://github.com/netscaler/netscaler-helm-charts/blob/master/citrix_cloud_native_values.yaml) contains the default values of the parameters.

## Uninstalling the Chart
To uninstall/delete the ```my-release``` deployment:

  ```
  helm delete my-release
  ```

## Related documentation

- [NetScaler CPX Documentation](https://docs.netscaler.com/en-us/citrix-adc-cpx//12-1/cpx-architecture-and-traffic-flow.html)
- [NetScaler ingress controller Documentation](https://docs.netscaler.com/en-us/citrix-k8s-ingress-controller/)
- [NetScaler ingress controller GitHub](https://github.com/netscaler/netscaler-k8s-ingress-controller)
- [BGP advertisement for External IPs with CPX](https://github.com/netscaler/netscaler-k8s-ingress-controller/blob/master/docs/network/bgp-enhancement.md)
