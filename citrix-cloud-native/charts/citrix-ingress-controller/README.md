# Deploy NetScaler Ingress Controller

[NetScaler](https://www.netscaler.com/) provides an Ingress Controller for NetScaler MPX (hardware), NetScaler VPX (virtualized), and [NetScaler CPX](https://docs.netscaler.com/en-us/citrix-adc-cpx//13/about.html) (containerized) for [bare metal](https://github.com/netscaler/netscaler-k8s-ingress-controller/tree/master/deployment/baremetal) and [cloud](https://github.com/netscaler/netscaler-k8s-ingress-controller/tree/master/deployment) deployments. It configures one or more NetScaler based on the Ingress resource configuration in [Kubernetes](https://kubernetes.io/) or in [OpenShift](https://www.openshift.com) cluster.

## TL;DR;

### For Kubernetes

  ```
  helm repo add netscaler https://netscaler.github.io/netscaler-helm-charts/

  helm install cic netscaler/citrix-cloud-native --set cic.enabled=true,cic.nsIP=<NSIP>,cic.adcCredentialSecret=<Secret-of-NetScaler-credentials>,cic.license.accept=yes
  ```

  To install NetScaler Provided Custom Resource Definition(CRDs) along with NetScaler Ingress Controller
  ```
  helm install cic netscaler/citrix-cloud-native --set cic.enabled=true,cic.nsIP=<NSIP>,cic.adcCredentialSecret=<Secret-of-NetScaler-credentials>,cic.license.accept=yes,cic.crds.install=true
  ```

### For OpenShift

  ```
  helm repo add netscaler https://netscaler.github.io/netscaler-helm-charts/

  helm install cic netscaler/citrix-cloud-native --set cic.enabled=true,cic.nsIP=<NSIP>,cic.adcCredentialSecret=<Secret-of-NetScaler-credentials>,cic.license.accept=yes,cic.openshift=true
  ```

  To install NetScaler Provided Custom Resource Definition(CRDs) along with NetScaler Ingress Controller
  ```
  helm install cic netscaler/citrix-cloud-native --set cic.enabled=true,cic.nsIP=<NSIP>,cic.adcCredentialSecret=<Secret-of-NetScaler-credentials>,cic.license.accept=yes,cic.openshift=true,cic.crds.install=true
  ```

> **Important:**
>
> The `cic.license.accept` argument is mandatory. Ensure that you set the value as `yes` to accept the terms and conditions of the NetScaler license.

## Introduction
This Helm chart deploys NetScaler ingress controller in the [Kubernetes](https://kubernetes.io) or in the [Openshift](https://www.openshift.com) cluster using [Helm](https://helm.sh) package manager.

### Prerequisites

-  The [Kubernetes](https://kubernetes.io/) version should be 1.16 and above if using Kubernetes environment.
-  The [Openshift](https://www.openshift.com) version 4.8 or later if using OpenShift platform.
-  The [Helm](https://helm.sh/) version 3.x or later. You can follow instruction given [here](https://github.com/netscaler/netscaler-helm-charts/blob/master/Helm_Installation_version_3.md) to install the same.
-  You determine the NS_IP IP address needed by the controller to communicate with NetScaler. The IP address might be anyone of the following depending on the type of NetScaler deployment:

   -  (Standalone appliances) NSIP - The management IP address of a standalone NetScaler appliance. For more information, see [IP Addressing in NetScaler](https://docs.netscaler.com/en-us/citrix-adc/current-release/networking/ip-addressing.html).

   -  (Appliances in High Availability mode) SNIP - The subnet IP address. For more information, see [IP Addressing in NetScaler](https://docs.netscaler.com/en-us/citrix-adc/current-release/networking/ip-addressing.html).

   -  (Appliances in Clustered mode) CLIP - The cluster management IP (CLIP) address for a clustered NetScaler deployment. For more information, see [IP addressing for a cluster](https://docs.netscaler.com/en-us/citrix-adc/current-release/clustering/cluster-overview/ip-addressing.html).

-  You have installed [Prometheus Operator](https://github.com/coreos/prometheus-operator), if you want to view the metrics of the NetScaler CPX collected by the [metrics exporter](https://github.com/netscaler/netscaler-k8s-ingress-controller/tree/master/metrics-visualizer#visualization-of-metrics).

-  The user name and password of the NetScaler VPX or MPX appliance used as the ingress device. The NetScaler appliance needs to have system user account (non-default) with certain privileges so that NetScaler ingress controller can configure the NetScaler VPX or MPX appliance. For instructions to create the system user account on NetScaler, see [Create System User Account for NSIC in NetScaler](#create-system-user-account-for-cic-in-citrix-adc).

    You can pass user name and password using Kubernetes secrets. Create a Kubernetes secret for the user name and password using the following command:

  ```
  kubectl create secret generic nslogin --from-literal=username='cic' --from-literal=password='mypassword'
  ```

#### Create system User account for NetScaler ingress controller in NetScaler

NetScaler ingress controller configures the NetScaler using a system user account of the NetScaler. The system user account should have certain privileges so that the NSIC has permission configure the following on the NetScaler:

-  Add, Delete, or View Content Switching (CS) virtual server
-  Configure CS policies and actions
-  Configure Load Balancing (LB) virtual server
-  Configure Service groups
-  Cofigure SSl certkeys
-  Configure routes
-  Configure user monitors
-  Add system file (for uploading SSL certkeys from Kubernetes)
-  Configure Virtual IP address (VIP)
-  Check the status of the NetScaler appliance


> **Note:**
>
> The system user account would have privileges based on the command policy that you define.

To create the system user account, do the following:

1.  Log on to the NetScaler appliance. Perform the following:
    1.  Use an SSH client, such as PuTTy, to open an SSH connection to the NetScaler appliance.

    2.  Log on to the appliance by using the administrator credentials.

2.  Create the system user account using the following command:

    ```
    add system user <username> <password>
    ```

    For example:

    ```
    add system user cic mypassword
    ```

3.  Create a policy to provide required permissions to the system user account. Use the following command:

    ```
      add cmdpolicy cic-policy ALLOW '^(\?!shell)(\?!sftp)(\?!scp)(\?!batch)(\?!source)(\?!.*superuser)(\?!.*nsroot)(\?!install)(\?!show\s+system\s+(user|cmdPolicy|file))(\?!(set|add|rm|create|export|kill)\s+system)(\?!(unbind|bind)\s+system\s+(user|group))(\?!diff\s+ns\s+config)(\?!(set|unset|add|rm|bind|unbind|switch)\s+ns\s+partition).*|(^install\s*(wi|wf))|(^\S+\s+system\s+file)^(\?!shell)(\?!sftp)(\?!scp)(\?!batch)(\?!source)(\?!.*superuser)(\?!.*nsroot)(\?!install)(\?!show\s+system\s+(user|cmdPolicy|file))(\?!(set|add|rm|create|export|kill)\s+system)(\?!(unbind|bind)\s+system\s+(user|group))(\?!diff\s+ns\s+config)(\?!(set|unset|add|rm|bind|unbind|switch)\s+ns\s+partition).*|(^install\s*(wi|wf))|(^\S+\s+system\s+file)'
    ```

    **Note**: The system user account would have privileges based on the command policy that you define.
    The command policy mentioned in ***step 3*** is similar to the built-in `sysAdmin` command policy with another permission to upload files.

    The command policy spec provided above have already escaped special characters for easier copy pasting into the NetScaler command line.

    For configuring the command policy from NetScaler Configuration Wizard (GUI), use the below command policy spec.

    ```
      ^(?!shell)(?!sftp)(?!scp)(?!batch)(?!source)(?!.*superuser)(?!.*nsroot)(?!install)(?!show\s+system\s+(user|cmdPolicy|file))(?!(set|add|rm|create|export|kill)\s+system)(?!(unbind|bind)\s+system\s+(user|group))(?!diff\s+ns\s+config)(?!(set|unset|add|rm|bind|unbind|switch)\s+ns\s+partition).*|(^install\s*(wi|wf))|(^\S+\s+system\s+file)^(?!shell)(?!sftp)(?!scp)(?!batch)(?!source)(?!.*superuser)(?!.*nsroot)(?!install)(?!show\s+system\s+(user|cmdPolicy|file))(?!(set|add|rm|create|export|kill)\s+system)(?!(unbind|bind)\s+system\s+(user|group))(?!diff\s+ns\s+config)(?!(set|unset|add|rm|bind|unbind|switch)\s+ns\s+partition).*|(^install\s*(wi|wf))|(^\S+\s+system\s+file)
    ```

4.  Bind the policy to the system user account using the following command:

    ```
    bind system user cic cic-policy 0
    ```

- **Important Note:** For deploying NetScaler VPX or MPX as ingress, you should establish the connectivity between NetScaler VPX or MPX and cluster nodes. This connectivity can be established by configuring routes on NetScaler as mentioned [here](https://github.com/netscaler/netscaler-k8s-ingress-controller/blob/master/docs/network/staticrouting.md) or by deploying [NetScaler Node Controller](https://github.com/netscaler/netscaler-k8s-node-controller)

## Installing the Chart
Add the NetScaler Ingress Controller helm chart repository using command:

  ```
  helm repo add netscaler https://netscaler.github.io/netscaler-helm-charts/
  ```

### For Kubernetes:
#### 1. NetScaler Ingress Controller
To install the chart with the release name, `my-release`, use the following command:

  ```
  helm install my-release netscaler/citrix-cloud-native --set cic.enabled=true,cic.nsIP=<NSIP>,cic.adcCredentialSecret=<Secret-of-NetScaler-credentials>,cic.license.accept=yes,cic.ingressClass[0]=<ingressClassName>
  ```

> **Note:**
>
> By default the chart installs the recommended [RBAC](https://kubernetes.io/docs/admin/authorization/rbac/) roles and role bindings.

The command deploys NetScaler ingress controller on Kubernetes cluster with the default configuration. The [configuration](#configuration) section lists the mandatory and optional parameters that you can configure during installation.

#### 2. NetScaler Ingress Controller with Exporter
[Metrics exporter](https://github.com/netscaler/netscaler-k8s-ingress-controller/tree/master/metrics-visualizer#visualization-of-metrics) can be deployed along with NetScaler ingress controller and collects metrics from the NetScaler instances. You can then [visualize these metrics](https://docs.netscaler.com/en-us/citrix-k8s-ingress-controller/metrics/promotheus-grafana.html) using Prometheus Operator and Grafana.

> **Note:**
> Ensure that you have installed [Prometheus Operator](https://github.com/coreos/prometheus-operator).

Use the following command for this:

  ```
  helm install my-release netscaler/citrix-cloud-native --set cic.enabled=true,cic.nsIP=<NSIP>,cic.adcCredentialSecret=<Secret-of-NetScaler-credentials>,cic.license.accept=yes,cic.ingressClass[0]=<ingressClassName>,cic.exporter.required=true
  ```

### For Openshift:
Add the name of the service account created when the chart is deployed to the privileged Security Context Constraints of OpenShift:

   ```
   oc adm policy add-scc-to-user privileged system:serviceaccount:<namespace>:<service-account-name>
   ```

#### 1. NetScaler Ingress Controller
To install the chart with the release name, `my-release`, use the following command:

  ```
  helm install my-release netscaler/citrix-cloud-native --set cic.enabled=true,cic.nsIP=<NSIP>,cic.adcCredentialSecret=<Secret-of-NetScaler-credentials>,cic.license.accept=yes,cic.openshift=true
  ```
The command deploys NetScaler ingress controller on your Openshift cluster in the default configuration. The [configuration](#configuration) section lists the mandatory and optional parameters that you can configure during installation.

#### 2. NetScaler Ingress Controller with Exporter
[Metrics exporter](https://github.com/netscaler/netscaler-k8s-ingress-controller/tree/master/metrics-visualizer#visualization-of-metrics) can be deployed along with NetScaler ingress controller and collects metrics from the NetScaler instances. You can then [visualize these metrics](https://docs.netscaler.com/en-us/citrix-k8s-ingress-controller/metrics/promotheus-grafana.html) using Prometheus Operator and Grafana.

> **Note:**
> Ensure that you have installed [Prometheus Operator](https://github.com/coreos/prometheus-operator)

Use the following command for this:

  ```
  helm install my-release netscaler/citrix-cloud-native --set cic.enabled=true,cic.nsIP=<NSIP>,cic.adcCredentialSecret=<Secret-of-NetScaler-credentials>,cic.license.accept=yes,cic.openshift=true,cic.exporter.required=true
  ```

### Installed components

The following components are installed:

-  [NetScaler ingress controller](https://github.com/netscaler/netscaler-k8s-ingress-controller)
-  [Exporter](https://github.com/netscaler/netscaler-adc-metrics-exporterporter) (if enabled)

## Configuration for ServiceGraph:
   If NetScaler VPX/MPX need to send data to the NetScaler ADM to bring up the servicegraph, then the below steps can be followed to install NetScaler ingress controller for NetScaler VPX/MPX. NetScaler ingress controller configures NetScaler VPX/MPX with the configuration required for servicegraph.

   1. Create secret using NetScaler VPX credentials, which will be used by NetScaler ingress controller for configuring NetScaler VPX/MPX:

	kubectl create secret generic nslogin --from-literal=username='cic' --from-literal=password='mypassword'

   2. Deploy NetScaler ingress controller using helm command:

	helm install my-release netscaler/citrix-cloud-native --set cic.enabled=true,cic.nsIP=<NSIP>,cic.nsVIP=<NSVIP>,cic.license.accept=yes,cic.adcCredentialSecret=<Secret-of-NetScaler-credentials>,cic.analyticsConfig.required=true,cic.analyticsConfig.timeseries.metrics.enable=true,cic.analyticsConfig.timeseries.port=5563,cic.analyticsConfig.distributedTracing.enable=true,cic.analyticsConfig.transactions.enable=true,cic.analyticsConfig.transactions.port=5557,cic.analyticsConfig.endpoint.server=<ADM-Agent-IP>

> **Note:**
> If container agent is being used here for NetScaler ADM, please provide `podIP` of container agent in the `cic.analyticsConfig.endpoint.server` parameter.

## CRDs configuration

CRDs can be installed/upgraded when we install/upgrade NetScaler ingress controller using `crds.install=true` parameter in Helm. If you do not want to install CRDs, then set the option `crds.install` to `false`. By default, CRDs too get deleted if you uninstall through Helm. This means, even the CustomResource objects created by the customer will get deleted. If you want to avoid this data loss set `crds.retainOnDelete` to `true`.

> **Note:**
> Installing again may fail due to the presence of CRDs. Make sure that you back up all CustomResource objects and clean up CRDs before re-installing NetScaler Ingress Controller.

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

#### apigateway CRD:

API Gateway CRD is used to configure gitops framework on NetScaler API gateway. This solution enables NetScaler ingress controller to generate API gateway configurations out of Open API Specification documents checked in to git repository by API developers and designers.

Example files: [api-gateway-crd-instance.yaml](https://github.com/netscaler/netscaler-helm-charts/tree/master/example-crds/api-gateway-crd-instance.yaml)

#### bots CRD:

[BOT CRD](https://github.com/netscaler/netscaler-k8s-ingress-controller/blob/master/docs/crds/bot.md) You can use Bot CRDs to configure the bot management policies with the NetScaler ingress controller on the NetScaler VPX. The Bot custom resource definition enables communication between the NetScaler ingress controller and NetScaler for enforcing bot management policies.

In a Kubernetes deployment, you can enforce bot management policy on therequests and responses from and to the server using the Bot CRDs. For more information on security vulnerabilities, see [Bot Detection](https://docs.netscaler.com/en-us/citrix-adc/current-release/bot-management/bot-detection.html).

Example files: [botallowlist.yaml](https://github.com/netscaler/netscaler-helm-charts/tree/master/example-crds/botallowlist.yaml)

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

Toleration can be applied to NetScaler ingress controller pod using `tolerations` argument while deploying NSIC using helm chart. This argument takes list of tolerations that user need to apply on the NSIC pods.

For example, following command can be used to apply toleration on the NSIC pod:

```
helm install my-release netscaler/citrix-cloud-native --set cic.enabled=true,cic.nsIP=<NSIP>,cic.license.accept=yes,cic.adcCredentialSecret=<Secret-for-NetScaler-credentials>,cic.tolerations[0].key=<toleration-key>,cic.tolerations[0].value=<toleration-value>,cic.tolerations[0].operator=<toleration-operator>,cic.tolerations[0].effect=<toleration-effect>
```

Here tolerations[0].key, tolerations[0].value and tolerations[0].effect are the key, value and effect that was used while tainting the node.
Effect represents what should happen to the pod if the pod don't have any matching toleration. It can have values `NoSchedule`, `NoExecute` and `PreferNoSchedule`.
Operator represents the operation to be used for key and value comparison between taint and tolerations. It can have values `Exists` and `Equal`. The default value for operator is `Equal`.

### Resource Quotas
There are various use-cases when resource quotas are configured on the Kubernetes cluster. If quota is enabled in a namespace for compute resources like cpu and memory, users must specify requests or limits for those values; otherwise, the quota system may reject pod creation. The resource quotas for the NSIC containers can be provided explicitly in the helm chart.

To set requests and limits for the NSIC container, use the variables `cic.resources.requests` and `cic.resources.limits` respectively.

Below is an example of the helm command that configures
- For NSIC container:
```
  CPU request for 500milli CPUs
  CPU limit at 1000m
  Memory request for 512M
  Memory limit at 1000M
```
```
helm install my-release netscaler/citrix-cloud-native --set cic.enabled=true,cic.nsIP=<NSIP>,cic.license.accept=yes,cic.adcCredentialSecret=<Secret-for-NetScaler-credentials>,cic.resources.requests.cpu=500m,cic.resources.requests.memory=512Mi --set cic.resources.limits.cpu=1000m,cic.resources.limits.memory=1000Mi
```

#### Analytics Configuration required for export of metrics to Prometheus

If NetScaler VPX needs to send data to Prometheus directly without an exporter resource in between, then the below steps can be followed to install NetScaler ingress controller for NetScaler VPX. NSIC configures NetScaler VPX with the configuration required.

1. Deploy NetScaler ingress controller using helm command:

```
helm repo add netscaler https://netscaler.github.io/netscaler-helm-charts/

helm install my-release netscaler/citrix-cloud-native --set cic.enabled=true,cic.nsIP=<NSIP>,cic.license.accept=yes,cic.adcCredentialSecret=<Secret-for-NetScaler-credentials>,cic.analyticsConfig.required=true,cic.analyticsConfig.timeseries.metrics.enable=true,cic.analyticsConfig.timeseries.port=5563,cic.analyticsConfig.timeseries.metrics.mode=prometheus,cic.analyticsConfig.timeseries.metrics.enableNativeScrape=true
```

2. For the NetScaler VPX to scrape using Prometheus, we need to create a system user with read only access. For more details on the user creation, refer [here](https://docs.netscaler.com/en-us/citrix-adc/current-release/observability/prometheus-integration#configure-read-only-prometheus-access-for-a-non-super-user)

3. To setup Prometheus in order to scrape natively from NetScaler VPX, a new scrape job is required to be added under scrape_configs in the prometheus [configuration](https://prometheus.io/docs/prometheus/latest/configuration/configuration/). A sample of the Prometheus job is given [here](https://docs.netscaler.com/en-us/citrix-adc/current-release/observability/prometheus-integration#prometheus-configuration)

> **Note:**
>
> For more details on Prometheus integration, please refer to [this](https://docs.netscaler.com/en-us/citrix-adc/current-release/observability/prometheus-integration)

### Configuration

The following table lists the mandatory and optional parameters that you can configure during installation:

| Parameters | Mandatory or Optional | Default value | Description |
| --------- | --------------------- | ------------- | ----------- |
| cic.enabled | Mandatory | False | Set to "True" for deploying NetScaler Ingress Controller for NetScaler VPX/MPX. |
| cic.license.accept | Mandatory | no | Set `yes` to accept the NSIC end user license agreement. |
| cic.imageRegistry                   | Mandatory  |  `quay.io`               |  The NetScaler ingress controller image registry             |  
| cic.imageRepository                 | Mandatory  |  `citrix/citrix-k8s-ingress-controller`              |   The NetScaler ingress controller image repository             | 
| cic.imageTag                  | Mandatory  |  `1.37.5`               |   The NetScaler ingress controller image tag            | 
| cic.pullPolicy | Mandatory | IfNotPresent | The NSIC image pull policy. |
| cic.imagePullSecrets | Optional | N/A | Provide list of Kubernetes secrets to be used for pulling the images from a private Docker registry or repository. For more information on how to create this secret please see [Pull an Image from a Private Registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/). |
| cic.nameOverride | Optional | N/A | String to partially override deployment fullname template with a string (will prepend the release name) |
| cic.fullNameOverride | Optional | N/A | String to fully override deployment fullname template with a string |
| cic.resources | Optional | {} |	CPU/Memory resource requests/limits for NetScaler Ingress Controller container |
| cic.adcCredentialSecret | Mandatory | N/A | The secret key to log on to the NetScaler VPX or MPX. For information on how to create the secret keys, see [Prerequisites](#prerequistes). |
| cic.secretStore.enabled | Optional | False | Set to "True" for deploying other Secret Provider classes  |
| cic.secretStore.username | Optional | N/A | if `cic.secretStore.enabled`, `username` of NetScaler will be fetched from the Secret Provider  |
| cic.secretStore.password | Optional | N/A | if `cic.secretStore.enabled`, `password` of NetScaler will be fetched from the Secret Provider  |
| cic.nsIP | Mandatory | N/A | The IP address of the NetScaler device. For details, see [Prerequisites](#prerequistes). |
| cic.nsVIP | Optional | N/A | The Virtual IP address on the NetScaler device. |
| cic.nsSNIPS | Optional | N/A | The list of subnet IPaddresses on the NetScaler device, which will be used to create PBR Routes instead of Static Routes [PBR support](https://github.com/netscaler/netscaler-k8s-ingress-controller/tree/master/docs/how-to/pbr.md) |
| cic.nsPort | Optional | 443 | The port used by NSIC to communicate with NetScaler. You can use port 80 for HTTP. |
| cic.nsProtocol | Optional | HTTPS | The protocol used by NSIC to communicate with NetScaler. You can also use HTTP on port 80. |
| cic.nsEnableLabel | Optional | True | Set to true for plotting Servicegraph. Ensure ``analyticsConfig` are set.  |
| cic.nitroReadTimeout | Optional | 20 | The nitro Read timeout in seconds, defaults to 20 |
| cic.logLevel | Optional | INFO | The loglevel to control the logs generated by NSIC. The supported loglevels are: CRITICAL, ERROR, WARNING, INFO, DEBUG and TRACE. For more information, see [Logging](https://github.com/netscaler/netscaler-k8s-ingress-controller/blob/master/docs/configure/log-levels.md).|
| cic.jsonLog | Optional | false | Set this argument to true if log messages are required in JSON format | 
| cic.nsConfigDnsRec | Optional | false | To enable/disable DNS address Record addition in NetScaler through Ingress |
| cic.nsSvcLbDnsRec | Optional | false | To enable/disable DNS address Record addition in NetScaler through Type Load Balancer Service |
| cic.nsDnsNameserver | Optional | N/A | To add DNS Nameservers in NetScaler |
| cic.optimizeEndpointBinding | Optional | false | To enable/disable binding of backend endpoints to servicegroup in a single API-call. Recommended when endpoints(pods) per application are large in number. Applicable only for NetScaler Version >=13.0-45.7  |
| cic.kubernetesURL | Optional | N/A | The kube-apiserver url that NSIC uses to register the events. If the value is not specified, NSIC uses the [internal kube-apiserver IP address](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#accessing-the-api-from-a-pod). |
| cic.clusterName | Optional | N/A | The unique identifier of the kubernetes cluster on which the NSIC is deployed. Used in gslb-controller deployments. |
| cic.ingressClass | Optional | N/A | If multiple ingress load balancers are used to load balance different ingress resources. You can use this parameter to specify NSIC to configure NetScaler associated with specific ingress class. For more information on Ingress class, see [Ingress class support](https://docs.netscaler.com/en-us/citrix-k8s-ingress-controller/configure/ingress-classes/). For Kubernetes version >= 1.19, this will create an IngressClass object with the name specified here |
| cic.setAsDefaultIngressClass | Optional | False | Set the IngressClass object as default ingress class. New Ingresses without an "ingressClassName" field specified will be assigned the class specified in ingressClass. Applicable only for kubernetes versions >= 1.19 |
| cic.serviceClass | Optional | N/A | By Default ingress controller configures all TypeLB Service on the NetScaler. You can use this parameter to finetune this behavior by specifing NSIC to only configure TypeLB Service with specific service class. For more information on Service class, see [Service class support]( https://docs.netscaler.com/en-us/citrix-k8s-ingress-controller/configure/service-classes/). |
| cic.nodeWatch | Optional | false | Use the argument if you want to automatically configure network route from the Ingress NetScaler VPX or MPX to the pods in the Kubernetes cluster. For more information, see [Automatically configure route on the NetScaler instance](https://docs.netscaler.com/en-us/citrix-k8s-ingress-controller/network/staticrouting/#automatically-configure-route-on-the-citrix-adc-instance). |
| cic.cncPbr | Optional | False | Use this argument to inform NSIC that NetScaler Node Controller(NSNC) is configuring Policy Based Routes(PBR) on the NetScaler. For more information, see [NSNC-PBR-SUPPORT](https://github.com/netscaler/netscaler-k8s-ingress-controller/blob/master/docs/network/pbr.md#configure-pbr-using-the-citrix-node-controller) |
| cic.defaultSSLCertSecret | Optional | N/A | Provide Kubernetes secret name that needs to be used as a default non-SNI certificate in NetScaler. |
| cic.podIPsforServiceGroupMembers | Optional | False |  By default NetScaler Ingress Controller will add NodeIP and NodePort as service group members while configuring type LoadBalancer Services and NodePort services. This variable if set to `True` will change the behaviour to add pod IP and Pod port instead of nodeIP and nodePort. Users can set this to 'True' if there is a route between NetScaler and K8s clusters internal pods either using feature-node-watch argument or using NetScaler Node Controller. |
| cic.ignoreNodeExternalIP | Optional | False | While adding NodeIP, as Service group members for type LoadBalancer services or NodePort services, NetScaler Ingress Controller has a selection criteria whereas it choose Node ExternalIP if available and Node InternalIP, if Node ExternalIP is not present. But some users may want to use Node InternalIP over Node ExternalIP even if Node ExternalIP is present. If this variable is set to `True`, then it prioritises the Node Internal IP to be used for service group members even if node ExternalIP is present |
| cic.nsHTTP2ServerSide | Optional | OFF | Set this argument to `ON` for enabling HTTP2 for NetScaler service group configurations. |
| cic.nsCookieVersion | Optional | 0 | Specify the persistence cookie version (0 or 1). |
| cic.profileSslFrontend | Optional | N/A | Specify the frontend SSL profile. For Details see [Configuration using FRONTEND_SSL_PROFILE](https://docs.netscaler.com/en-us/citrix-k8s-ingress-controller/configure/profiles.html#global-front-end-profile-configuration-using-configmap-variables) |
| cic.profileTcpFrontend | Optional | N/A | Specify the frontend TCP profile. For Details see [Configuration using FRONTEND_TCP_PROFILE](https://docs.netscaler.com/en-us/citrix-k8s-ingress-controller/configure/profiles.html#global-front-end-profile-configuration-using-configmap-variables) |
| cic.profileHttpFrontend | Optional | N/A | Specify the frontend HTTP profile. For Details see [Configuration using FRONTEND_HTTP_PROFILE](https://docs.netscaler.com/en-us/citrix-k8s-ingress-controller/configure/profiles.html#global-front-end-profile-configuration-using-configmap-variables) |
| cic.ipam | Optional | False | Set this argument if you want to use the IPAM controller to automatically allocate an IP address to the service of type LoadBalancer. |
| cic.disableAPIServerCertVerify | Optional | False | Set this parameter to True for disabling API Server certificate verification. |
| cic.logProxy | Optional | N/A | Provide Elasticsearch or Kafka or Zipkin endpoint for NetScaler observability exporter. |
| cic.entityPrefix | Optional | k8s | The prefix for the resources on the NetScaler VPX/MPX. |
| cic.updateIngressStatus | Optional | True | Set this argurment if `Status.LoadBalancer.Ingress` field of the Ingress resources managed by the NetScaler ingress controller needs to be updated with allocated IP addresses. For more information see [this](https://github.com/netscaler/netscaler-k8s-ingress-controller/blob/master/docs/configure/ingress-classes.md#updating-the-ingress-status-for-the-ingress-resources-with-the-specified-ip-address). |
| cic.routeLabels | Optional | N/A | You can use this parameter to provide the route labels selectors to be used by NetScaler Ingress Controller for routeSharding in OpenShift cluster. |
| cic.namespaceLabels | Optional | N/A | You can use this parameter to provide the namespace labels selectors to be used by NetScaler Ingress Controller for routeSharding in OpenShift cluster. |
| cic.podAnnotations | Optional | N/A | Map of annotations to add to the pods. |
| cic.affinity | Optional | N/A | Affinity labels for pod assignment. |
| cic.exporter.required | Optional | false | Use the argument, if you want to run the [Exporter for NetScaler Stats](https://github.com/netscaler/netscaler-adc-metrics-exporterporter) along with NSIC to pull metrics for the NetScaler VPX or MPX|
| cic.exporter.imageRegistry                   | Optional  |  `quay.io`               |  The Exporter for NetScaler Stats image registry             |  
| cic.exporter.imageRepository                 | Optional  |  `citrix/citrix-adc-metrics-exporter`              |   The Exporter for NetScaler Stats image repository             | 
| cic.exporter.imageTag                  | Optional  |  `1.4.9`               |  The Exporter for NetScaler Stats image tag            |
| cic.exporter.pullPolicy | Optional | IfNotPresent | The Exporter image pull policy. |
| cic.exporter.ports.containerPort | Optional | 8888 | The Exporter container port. |
| cic.exporter.resources | Optional | {} |	CPU/Memory resource requests/limits for Metrics exporter container |
| cic.exporter.extraVolumeMounts  |  Optional |  [] |  Specify the Additional VolumeMounts to be mounted in Exporter container. Specify the volumes in `cic.extraVolumes`  |
| cic.exporter.serviceMonitorExtraLabels | Optional |  | Extra labels for service monitor whem NetScaler-adc-metrics-exporter is enabled. |
| cic.openshift | Optional | false | Set this argument if OpenShift environment is being used. |
| cic.disableOpenshiftRoutes | Optional | false | By default Openshift routes are processed in openshift environment, this variable can be used to disable Ingress controller processing the openshift routes. |
| cic.nodeSelector.key | Optional | N/A | Node label key to be used for nodeSelector option in NSIC deployment. |
| cic.nodeSelector.value | Optional | N/A | Node label value to be used for nodeSelector option in NSIC deployment. |
| cic.tolerations | Optional | N/A | Specify the tolerations for the NSIC deployment. |
| cic.crds.install | Optional | False | Unset this argument if you don't want to install CustomResourceDefinitions which are consumed by NSIC. |
| cic.crds.retainOnDelete | Optional | false | Set this argument if you want to retain CustomResourceDefinitions even after uninstalling NSIC. This will avoid data-loss of Custom Resource Objects created before uninstallation. |
| cic.analyticsConfig.required | Mandatory | false | Set this to true if you want to configure NetScaler to send metrics and transaction records to analytics service. |
| cic.analyticsConfig.distributedTracing.enable | Optional | false | Set this value to true to enable OpenTracing in NetScaler. |
| cic.analyticsConfig.distributedTracing.samplingrate | Optional | 100 | Specifies the OpenTracing sampling rate in percentage. |
| cic.analyticsConfig.endpoint.server | Optional | N/A | Set this value as the IP address or DNS address of the  analytics server. |
| cic.analyticsConfig.endpoint.service | Optional | N/A | Set this value as the IP address or service name with namespace of the analytics service deployed in k8s environment. Format: namespace/servicename|
| cic.analyticsConfig.timeseries.port | Optional | 30002 | Specify the port used to expose analytics service outside cluster for timeseries endpoint. |
| cic.analyticsConfig.timeseries.metrics.enable | Optional | False | Set this value to true to enable sending metrics from NetScaler. |
| cic.analyticsConfig.timeseries.metrics.mode | Optional | avro |  Specifies the mode of metric endpoint. |
| cic.analyticsConfig.timeseries.metrics.exportFrequency | Optional | 30 |  Specifies the time interval for exporting time-series data. Possible values range from 30 to 300 seconds. |
| cic.analyticsConfig.timeseries.metrics.schemaFile | Optional | schema.json |  Specifies the name of a schema file with the required Netscaler counters to be added and configured for metricscollector to export. A reference schema file reference_schema.json with all the supported counters is also available under the path /var/metrics_conf/. This schema file can be used as a reference to build a custom list of counters. |
| cic.analyticsConfig.timeseries.metrics.enableNativeScrape | Optional | false |  Set this value to true for native export of metrics. |
| cic.analyticsConfig.timeseries.auditlogs.enable | Optional | false | Set this value to true to export audit log data from NetScaler. |
| cic.analyticsConfig.timeseries.events.enable | Optional | false | Set this value to true to export events from the NetScaler. |
| cic.analyticsConfig.transactions.enable | Optional | false | Set this value to true to export transactions from NetScaler. |
| cic.analyticsConfig.transactions.port | Optional | 30001 | Specify the port used to expose analytics service outside cluster for transaction endpoint. |
| cic.nsLbHashAlgo.required | Optional | false | Set this value to set the LB consistent hashing Algorithm |
| cic.nsLbHashAlgo.hashFingers | Optional | 256 | Specifies the number of fingers to be used for hashing algorithm. Possible values are from 1 to 1024, Default value is 256 |
| cic.nsLbHashAlgo.hashAlgorithm | Optional | 'default' | Specifies the supported algorithm. Supported algorithms are "default", "jarh", "prac", Default value is 'default' |
| cic.extraVolumeMounts  |  Optional |  [] |  Specify the Additional VolumeMounts to be mounted in NSIC container  |
| cic.extraVolumes  |  Optional |  [] |  Specify the Additional Volumes for additional volumeMounts  |
| cic.rbacRole  | Optional |  false  |  To deploy NSIC with RBAC Role set rbacRole=true; by default NSIC gets installed with RBAC ClusterRole(rbacRole=false) |

Alternatively, you can define a YAML file with the values for the parameters and pass the values while installing the chart.

For example:

  ```
  helm install my-release netscaler/citrix-cloud-native -f values.yaml
  ```

> **Tip:**
>
> The [values.yaml](https://github.com/netscaler/netscaler-helm-charts/blob/master/citrix_cloud_native_values.yaml) contains the default values of the parameters.

> **Note:**
>
> Please provide frontend-ip (VIP) in your application ingress yaml file. For more info refer [this](https://github.com/netscaler/netscaler-k8s-ingress-controller/blob/master/docs/configure/annotations.md).

## Route Addition in MPX/VPX
For seamless functioning of services deployed in the Kubernetes cluster, it is essential that Ingress NetScaler device should be able to reach the underlying overlay network over which Pods are running.
`feature-node-watch` knob of NetScaler Ingress Controller can be used for automatic route configuration on NetScaler towards the pod network. Refer [Static Route Configuration](https://github.com/netscaler/netscaler-k8s-ingress-controller/blob/master/docs/network/staticrouting.md) for further details regarding the same.
By default, `feature-node-watch` is false. It needs to be explicitly set to true if auto route configuration is required.

This can also be achieved by deploying [NetScaler Node Controller](https://github.com/netscaler/netscaler-k8s-node-controller).

If your deployment uses one single NetScaler Device to loadbalance between multiple k8s clusters, there is a possibilty of CNI subnets to overlap, causing the above mentioned static routing to fail due to route conflicts. In such deployments [Policy Based Routing(PBR)] ( https://docs.netscaler.com/en-us/citrix-adc/current-release/networking/ip-routing/configuring-policy-based-routes/configuring-policy-based-routes-pbrs-for-ipv4-traffic.html) can be used instead. This would require you to provide one or more subnet IP Addresses unique for each kubernetes cluster either via Environment variable or Configmap, see [PBR Support](https://github.com/netscaler/netscaler-k8s-ingress-controller/tree/master/docs/how-to/pbr.md)

   Use the following command to provide subnet IPAddresses(SNIPs) to configure Policy Based Routes(PBR) on the NetScaler

   ```
   helm install cic netscaler/citrix-cloud-native --set cic.enabled=true,cic.nsIP=<NSIP>,cic.adcCredentialSecret=<Secret-of-NetScaler-credentials>,cic.license.accept=yes,cic.nsSNIPS='[<NS_SNIP1>\, <NS_SNIP2>\, ...]'
   ```

   [NetScaler Node Controller](https://github.com/netscaler/netscaler-k8s-node-controller) by default also adds static routes while creating the VXLAN tunnel. To use [Policy Based Routing(PBR)] ( https://docs.netscaler.com/en-us/citrix-adc/current-release/networking/ip-routing/configuring-policy-based-routes/configuring-policy-based-routes-pbrs-for-ipv4-traffic.html) to avoid static route clash, both NetScaler Node Controller and NetScaler Ingress Controller has to work in conjunction and has to be started with specific arguments. For more details refer [NSNC-PBR-SUPPORT](https://github.com/netscaler/netscaler-k8s-ingress-controller/blob/master/docs/network/pbr.md#configure-pbr-using-the-citrix-node-controller).

   Use the following command to inform NetScaler Ingress Controller that NetScaler Node Controller is configuring Policy Based Routes(PBR) on the NetScaler

   ```
   helm install cic netscaler/citrix-cloud-native --set cic.enabled=true,cic.nsIP=<NSIP>,cic.adcCredentialSecret=<Secret-of-NetScaler-credentials>,cic.license.accept=yes,cic.clusterName=<unique-cluster-identifier>,cic.cncPbr=<True/False>
   ```

For configuring static routes manually on NetScaler VPX or MPX to reach the pods inside the cluster follow:
### For Kubernetes:
1. Obtain podCIDR using below options:
   ```
   kubectl get nodes -o yaml | grep podCIDR
   ```
   * podCIDR: 10.244.0.0/24
   * podCIDR: 10.244.1.0/24
   * podCIDR: 10.244.2.0/24

2. Log on to the NetScaler instance.

3. Add Route in Netscaler VPX/MPX
   ```
   add route <podCIDR_network> <podCIDR_netmask> <node_HostIP>
   ```
4. Ensure that Ingress MPX/VPX has a SNIP present in the host-network (i.e. network over which K8S nodes communicate with each other. Usually eth0 IP is from this network).

   Example:
   * Node1 IP = 192.0.2.1
   * podCIDR  = 10.244.1.0/24
   * add route 10.244.1.0 255.255.255.0 192.0.2.1

### For OpenShift:
1. Use the following command to get the information about host names, host IP addresses, and subnets for static route configuration.
   ```
   oc get hostsubnet
   ```

2. Log on to the NetScaler instance.

3. Add the route on the NetScaler instance using the following command.
   ```
   add route <pod_network> <podCIDR_netmask> <gateway>
   ```

4. Ensure that Ingress MPX/VPX has a SNIP present in the host-network (i.e. network over which OpenShift nodes communicate with each other. Usually eth0 IP is from this network).

    For example, if the output of the `oc get hostsubnet` is as follows:
    * oc get hostsubnet

        NAME            HOST           HOST IP        SUBNET
        os.example.com  os.example.com 192.0.2.1 10.1.1.0/24

    * The required static route is as follows:

           add route 10.1.1.0 255.255.255.0 192.0.2.1

## Uninstalling the Chart
To uninstall/delete the ```my-release``` deployment:

  ```
  helm delete my-release
  ```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Related documentation

- [NetScaler ingress controller Documentation](https://docs.netscaler.com/en-us/citrix-k8s-ingress-controller/)
- [NetScaler ingress controller GitHub](https://github.com/netscaler/netscaler-k8s-ingress-controller)
