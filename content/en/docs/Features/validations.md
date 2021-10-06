---
title: "Validations"
date: 2020-02-17T14:53:00+02:00
draft: false
weight: 5
---

## Validations Overview

Kiali performs a set of validations to the most common Istio Objects such as Destination Rules, Service Entries, and Virtual Services. Those validations are done in addition to the existing ones performed by Istio's Galley component. Most validations are done inside a single namespace only, any exceptions, such as gateways, are properly documented.

Galley validations are mostly syntactic validations based on the object syntax analysis of Istio objects while Kiali validations are mostly semantic validations between different Istio objects. Kiali validations are based on the runtime status of your service mesh, Galley validations are static ones and doesn't take into account what is configured in the mesh.

<div style="display: flex;">
 <span style="margin: 0 auto;">
  <a class="image-popup-fit-height" href="/images/documentation/features/config-validation-v1.22.0.png" title="Istio Config Validation">
   <img src="/images/documentation/features/config-validation-v1.22.0.png" style="width: 1333px;display:inline;margin: 0 auto;" />
  </a>
 </span>
</div>
</br>

Check the complete list of validations for further information.

## AuthorizationPolicy {#authorizationpolicies}

### KIA0101 - Namespace not found for this rule

AuthorizationPolicy enables access control on workloads. Each policy effects only to a group of request. For instance, all requests started from a workload on a list of namespaces.
The present validation points out those rules referencing a namespace that don't exist in the cluster.

#### Resolution

Either remove the namespace from the list, correct if there is any typo or create a new namespace.

#### Severity

<i class="fas fa-exclamation-triangle"></i> Warning

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/801.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/blob/master/business/checkers/authorization/namespace_method_checker.go)
- [Istio documentation](https://istio.io/docs/reference/config/security/authorization-policy)
- [Definition of a source](https://istio.io/docs/reference/config/security/authorization-policy/#Source)


### KIA0102 - Only HTTP methods and fully-qualified gRPC names are allowed

An AuthorizationPolicy has an Operation field where is defined the oprations allowed for a request. In the method field are listed all the allowed methods that request can have.
This validation appears when a problem is found in there. The only methods accepted are: either HTTP valid methods or fully-qualified names of gRPC service in the form of "/package.service/method"

#### Resolution

Either change or remove the violating method. It has to be either a HTTP valid method or a fully-qualified names of a gRPC service in the form of "/package.service/method"

#### Severity

<i class="fas fa-exclamation-triangle"></i> Warning

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/802.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/blob/v1.25/business/checkers/authorization/service_binding_checker.go)
- [Istio documentation](https://istio.io/v1.3/docs/reference/config/authorization/istio.rbac.v1alpha1/#ServiceRoleBinding)


### KIA0104 - This host has no matching entry in the service registry

AuthorizationPolicy enables access control on workloads. Each policy effects only to a group of request going to a specific destination. For instance, allow all the request going to `details` host.

The present validation points out those rules referencing a host that don't exist in the authorization policy namespace. Kiali considers services and service entries.
Those hosts that refers to hosts outside of the object namespace will be presented with an unknow error.

#### Resolution

Either remove the host from the list, correct if there is any typo or deploy a new service or service entry.

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/804.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/blob/master/business/checkers/authorization/no_host_checker.go)
- [AuthorizationPolicy documentation](https://istio.io/docs/reference/config/security/authorization-policy)
- [Definition of the operations field](https://istio.io/docs/reference/config/security/authorization-policy/#Operation)
- [Service association requirement](https://istio.io/docs/ops/deployment/requirements)


## Destination rules {#destinationrules}

### KIA0201 - More than one DestinationRules for the same host subset combination

Istio applies traffic rules for services after the routing has happened. These can include different settings such as connection pooling, circuit breakers, load balancing, and detection. Istio can define the same rules for all services under a host or different rules for different versions of the service.

This validation warning could be a result of duplicate definition of the same subsets as well as from rules that apply to all subsets. Also, a combination of one Destination Rule (DR) applying to all subsets and another defining behavior for only some subsets triggers this validation warning.

Istio silently ignores the duplicate subsets and merge these destination rules without letting the user know. Only the first seen rule (by Istio) per subset is used and information from multiple definitions is not merged. While the routing might work correctly, this is most likely a configuration error. It may lead to a undesired behavior if one of the offending rules is removed or modified and that is probably not the intention of the deployer of this service. Also, if the two offending destination rules have different policies for traffic routing the wrong one might be used.

#### Resolution

Either merge the settings to a single DR or split the subsets in such a way that they do not interleave. This ensures that the routing behavior stays consistent.

#### Severity

<i class="fas fa-exclamation-triangle"></i> Warning

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/001.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/tree/master/business/checkers/destinationrules/multi_match_checker.go)
- [Destination rule documentation](https://istio.io/docs/reference/config/networking/destination-rule/#DestinationRule)
- [Istio source code for merging](https://github.com/istio/istio/blob/0e9cecab053aab744a7c3a731aacb07fd794d5f9/pilot/pkg/model/push_context.go#L879)
- [Istio documentation: Split large virtual services and destination rules into multiple resources](https://istio.io/docs/ops/best-practices/traffic-management/#split-virtual-services)


### KIA0202 - This host has no matching entry in the service registry (service, workload or service entries)

Istio applies traffic rules for services after the routing has happened. These can include different settings such as connection pooling, circuit breakers, load balancing, and detection. Istio can define the same rules for all services under a host or different rules for different versions of the service. The host must a service that is defined in the platform's service registry or as a ServiceEntry. Short names are extended to include '.namespace.cluster' using the namespace of the destination rule, not the service itself. FQDN is evaluated as is. It is recommended to use the FQDN to prevent any confusion.

If the host is not found, Istio ignores the defined rules.

#### Resolution

Correct the host to point to a correct service, in this namespace or with FQDN to other namespaces, or deploy the missing service to the mesh.

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/002.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/tree/master/business/checkers/destinationrules/no_dest_checker.go)
- [Destination rule documentation](https://istio.io/docs/reference/config/networking/destination-rule/#DestinationRule)



### KIA0203 - This subset’s labels are not found in any matching host

Istio applies traffic rules for services after the routing has happened. These can include different settings such as connection pooling, circuit breakers, load balancing, and detection. Istio can define the same rules for all services under a host or different rules for different versions of the service. The host must a service that is defined in the platform's service registry or as a ServiceEntry. Short names are extended to include '.namespace.cluster' using the namespace of the destination rule, not the service itself. FQDN is evaluated as is. It is recommended to use the FQDN to prevent any confusion.

Subsets can override the global settings defined in the DR for a host.

If the host is not found, Istio ignores the defined rules.

If the not found subset is not referenced in any Virtual Service, the severity of this error is changed to Info.

#### Resolution

Correct the host to point to a correct service, in this namespace or with FQDN to other namespaces, or deploy the missing service to the mesh. If the hostname is equal to the one used otherwise in the DR, consider removing the duplicate host resolution.

Also, verify that the labels are correctly matching a workload with the intended service.

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/003.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/tree/master/business/checkers/destinationrules/no_dest_checker.go)
- [Destination rule documentation](https://istio.io/docs/reference/config/networking/destination-rule/#DestinationRule)



### KIA0204 - mTLS settings of a non-local Destination Rule are overridden

Istio allows you to define DestinationRule at three different levels: mesh, namespace and service level. A mesh may have multiple DRs. In case of having two DestinationRules on the first one is at a lower level than the second one, the first one overrides the TLS values of the second one.

This validation appears only when autoMtls is disabled.

#### Resolution

This validation aims to warn Kiali users that they may be disabling/enabling mTLS from the higher DestinationRule.
Merging the TLS settings to one of the DestinationRules is the only way to fix this validation. However, this is a valid scenario so it might be impossible to remove this warning.

#### Severity

<i class="fas fa-exclamation-triangle"></i> Warning

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/005.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/tree/master/business/checkers/destinationrules/traffic_policy_checker.go)



### KIA0205 - PeerAuthentication enabling mTLS at mesh level is missing

Istio has the ability to define mTLS communications at mesh level. In order to do that, Istio needs one DestinationRule and one PeerAuthentication. The DestinationRule configures all the clients of the mesh to use mTLS protocol on their connections. The PeerAuthentication defines what authentication methods that can be accepted on the workload of the whole mesh.
If the PeerAuthentication is not found or doesn't exist and the mesh-wide DestinationRule is on ISTIO_MUTUAL mode, all the communication returns 500 errors.

This validation appears only when autoMtls is disabled.

#### Resolution
Add a PeerAuthentication within the `istio-system` namespace without specifying targets but setting peers mtls mode to STRICT or PERMISSIVE. The PeerAuthentication should be like [this](/files/validation_examples/401-PeerAuth.yaml).

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/004.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/tree/master/business/checkers/destinationrules/meshwide_mtls_checker.go)
- [Globally enabling Istio mutual TLS](https://istio.io/docs/tasks/security/authn-policy/#globally-enabling-istio-mutual-tls-in-strict-mode)


### KIA0206 - PeerAuthentication enabling namespace-wide mTLS is missing

Istio has the ability to define mTLS communications at namespace level. In order to do that, Istio needs both a DestinationRule and a PeerAuthentication targeting all the clients/workloads of the specific namespace. The PeerAuthentication allows mTLS authentication method for all the workloads within a namespace. The DestinationRule defines all the clients within the namespace to start communications in mTLS mode.
If the PeerAuthentication is not found and the DestinationRule is on STRICT mode in that namespace but there is the DestinationRule enabling mTLS, all the communications within that namespace returns 500 errors.

This validation appears only when autoMtls is disabled.

#### Resolution
A PeerAuthentication enabling mTLS method is needed for the workloads in the namespace. Otherwise all the clients start mTLS connections that those workloads won't be ready to manage.
Add a PeerAuthentication without specifying targets but setting mTLS mode to STRICT or PERMISSIVE in the same namespace as the DestinationRule.

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/006.yaml" %}}
```

#### See Also
- [Validator source code](https://github.com/kiali/kiali/blob/master/business/checkers/destinationrules/namespacewide_mtls_checker.go)
- [Enabling Istio mutual TLS per namespace](https://istio.io/docs/tasks/security/authn-policy/#enable-mutual-tls-per-namespace-or-workload)


### KIA0207 - PeerAuthentication with TLS strict mode found, it should be permissive

Istio needs both a DestinationRule and PeerAuthentication to enable mTLS communications. The PeerAuthentication configures the authentication method accepted for all the targeted workloads. The DestinationRule defines which is the authentication method that the clients of specific workloads has to start communications with.

#### Resolution

Kiali has found that there is a DestinationRule sending traffic without mTLS authentication method. There are two different ways to fix this situation. You can either change the PeerAuthentication applying to PERMISSIVE mode or change the DestinationRule to start communications using mTLS.

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/007.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/tree/master/business/checkers/destinationrules/disabled_namespacewide_mtls_checker.go)
- [Enabling Istio mutual TLS per namespace](https://istio.io/docs/tasks/security/authentication/authn-policy/#enable-mutual-tls-per-namespace-or-workload)



### KIA0208 - PeerAuthentication enabling mTLS found, permissive mode needed

Istio needs both a DestinationRule and PeerAuthentication to enable mTLS communications. The PeerAuthentication configures the authentication method accepted for all the targeted workloads. The DestinationRule defines which is the authentication method that the clients of specific workloads has to start communications with.

Kiali found a DestinationRule starting communications without TLS but there was a PeerAuthentication allowing all services in the mesh to accept *only* requests in mTLS.

#### Resolution
There are two ways to fix this situation. You can either change the PeerAuthentication to enable PERMISSIVE mode to all the workloads in the mesh or change the DestinatonRule to enable mTLS instead of disabling it (change the mode to ISTIO_MUTUAL).

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/008.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/tree/master/business/checkers/destinationrules/disabled_namespacewide_mtls_checker.go)
- [Globally enabling Istio mutual TLS](https://istio.io/docs/tasks/security/authentication/authn-policy/#globally-enabling-istio-mutual-tls-in-strict-mode)


### KIA0209 - DestinationRule Subset has not labels

A DestinationRule subset without labels may miss the destination endpoint linked with a specific workload.

#### Resolution
Validate that a subset is properly configured.

#### Severity

<i class="fas fa-exclamation-triangle"></i> Warning

#### See Also

- [DestinationRule Subset](https://istio.io/latest/docs/reference/config/networking/destination-rule/#Subset)


## Gateways {#gateways}

### KIA0301 - More than one Gateway for the same host port combination

Gateway creates a proxy that forwards the inbound traffic for the exposed ports. If two different gateways expose the same ports for the same host, this creates ambiguity inside Istio as either of these gateways could handle the traffic. This is most likely a configuration error. This check is done across all namespaces the user has access to.

There is one exception: when both gateways points to a different ingress. Then the ambiguity doesn't exist and, in consequence, no validation is shown. Kiali considers that two gateways points to the same ingress if they share the exact same selector.

#### Resolution

Remove the duplicate gateway entries or merge the two gateway definitions into a single one.

#### Severity

<i class="fas fa-exclamation-triangle"></i> Warning

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/201.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/tree/master/business/checkers/gateways/multi_match_checker.go)
- [Istio Gateway documentation](https://istio.io/docs/reference/config/networking/gateway/)


### KIA0302 - No matching workload found for gateway selector in this namespace

This validation checks the current namespace for matching workloads as this is recommended, and potentially in the future required, by the Istio. Excluded from this check are the default "istio-ingressgateway" and "istio-egressgateway" workloads which are included in Istio by default.

Although your traffic might be correctly routed to a workload in other namespace, this is not a guaranteed behavior and thus a warning is flagged in such cases also.

#### Resolution

Deploy the missing workload or fix the selector to target a correct location.

#### Severity

<i class="fas fa-exclamation-triangle"></i> Warning

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/202.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/blob/master/business/checkers/gateways/selector_checker.go)
- [Istio documentation for gateways](https://istio.io/docs/reference/config/networking/v1alpha3/gateway/#Gateway)

## Mesh Policies {#meshpolicies}

### KIA0401 - Mesh-wide Destination Rule enabling mTLS is missing

Istio has the ability to define mTLS communications at mesh level. In order to do that, Istio needs one DestinationRule and one PeerAuthentication. The DestinationRule configures all the clients of the mesh to use mTLS protocol on their connections. The PeerAuthentication defines what authentication methods can be accepted on the workload of the whole mesh.
If the DestinationRule is not found or doesn't exist and the PeerAuthentication is on STRICT mode, all the communication returns 500 errors.

This validation appears only when autoMtls is disabled.

#### Resolution
Add a DestinationRule with "*.cluster" host and ISTIO_MUTUAL as tls trafficPolicy mode. The DestinationRule should be like [this](https://github.com/kiali/kiali.io/blob/master/data/files/validation_examples/004.yaml).

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/401.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/tree/v1.17/business/checkers/meshpolicies/mesh_mtls_checker.go)
- [Globally enabling Istio mutual TLS](https://istio.io/docs/tasks/security/authn-policy/#globally-enabling-istio-mutual-tls-in-strict-mode)


## PeerAuthentication {#peerauthentication}

### KIA0501 - Destination Rule enabling namespace-wide mTLS is missing

Istio has the ability to define mTLS communications at namespace level. In order to do that, Istio needs one DestinationRule and one PeerAuthentication. The DestinationRule configures all the clients of the namespace to use mTLS protocol on their connections. The PeerAuthentication defines what authentication methods can be accepted on a specific group of workloads. PeerAuthentications without target field specified will target all the workloads within its namespace.
If the DestinationRule is not found or doesn't exist in the namespace and the namespace-wide PeerAuthentication is on STRICT mode, all the communication will return 500 errors.

This validation appears only when autoMtls is disabled.

#### Resolution
Add a DestinationRule with "*.namespace.svc.cluster.local" host and ISTIO_MUTUAL as tls trafficPolicy mode. The DestinationRule should be like [this](https://github.com/kiali/kiali.io/blob/master/data/files/validation_examples/006.yaml).

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/301.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/tree/v1.17/business/checkers/policies/namespace_mtls_checker.go)
- [Enabling Istio mutual TLS namespace-wide](https://istio.io/docs/tasks/security/authentication/authn-policy/#enable-mutual-tls-per-namespace-or-workload)



### KIA0505 - Destination Rule disabling namespace-wide mTLS is missing

PeerAuthentication objects are used to define the authentication methods that a set of workloads can accept: Mutual, Istio Mutual, Simple or Disabled.

This validation warns the scenario where there is one PeerAuthentication at namespace level with `DISABLE` mode but there is DestinationRule at namespace or mesh level enabling mTLS. With this scenario, all the traffic flowing between the services in that namespace will fail.

#### Resolution

You can either change the namespace/mesh-wide Destination Rule to `DISABLE` mode or change the current PeerAuthentication to allow mTLS (mode `STRICT` or `PERMISSIVE`).

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/305.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/blob/master/business/checkers/peerauthentications/disabled_namespacewide_checker.go)
- [PeerAuthentication reference](https://istio.io/docs/reference/config/security/peer_authentication)



### KIA0506 - Destination Rule disabling mesh-wide mTLS is missing

PeerAuthentication objects are used to define the authentication methods that a set of workloads can accept: Mutual, Istio Mutual, Simple or Disabled.

This validation warns the scenario where there is one PeerAuthentication at mesh level with `DISABLE` mode but there is DestinationRule at mesh level enabling mTLS. With this scenario, all the traffic flowing between the services in that namespace will fail.

#### Resolution

You can either change the mesh-wide Destination Rule to `DISABLE` mode or change the current PeerAuthentication to allow mTLS (mode `STRICT` or `PERMISSIVE`).

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/306.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/blob/master/business/checkers/peerauthentications/disabled_meshwide_checker.go)
- [PeerAuthentication reference](https://istio.io/docs/reference/config/security/peer_authentication)


## Ports {#ports}

### KIA0601 - Port name must follow <protocol>[-suffix] form

Istio requires the service ports to follow the naming form of 'protocol-suffix' where the '-suffix' part is optional. If the naming does not match this form (or is undefined), Istio treats all the traffic TCP instead of the defined protocol in the definition. Dash is a required character between protocol and suffix. For example, 'http2foo' is not valid, while 'http2-foo' is (for http2 protocol).

#### Resolution

Rename the service port name field to follow the form and the traffic flows correctly.

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/701.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/blob/master/business/checkers/services/port_mapping_checker.go)
- [Istio documentation port naming convention](https://istio.io/docs/ops/deployment/requirements)


## Services {#services}

### KIA0701 - Deployment exposing same port as Service not found

Service definition has a combination of labels and port definitions that are not matching to any workloads. This means the deployment will be unsuccessful and no traffic can flow between these two resources. The port is read from the Service 'TargetPort' definition first and if undefined, the 'Port' field is used as Kubernetes defaults the 'TargetPort' to 'Port'. If the 'TargetPort' is using a integer, the port numbers are compared and if the 'TargetPort' is a string, the deployment's portName is used for comparison.

#### Resolution

Fix the port definitions in the workload or in the service definition to ensure they match.

#### Severity

<i class="fas fa-exclamation-triangle"></i> Warning

#### Example

Invalid example with port definitions unmatched:

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/702.yaml" %}}
```

Valid example using targetPort definition matching:

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/703.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/blob/master/business/checkers/services/port_mapping_checker.go)
- [Kubernetes services](https://kubernetes.io/docs/concepts/services-networking/service/#defining-a-service)


## ServiceMesh Policies {#servicemeshpolicies}

### KIA0801 - Mesh-wide Destination Rule enabling mTLS is missing

Maistra has the ability to define mTLS communications at mesh level. In order to do that, Maistra needs one DestinationRule and one ServiceMeshPolicy. The DestinationRule configures all the clients of the mesh to use mTLS protocol on their connections. The ServiceMeshPolicy defines what authentication methods can be accepted on the workload of the whole mesh.
If the DestinationRule is not found or doesn't exist and the ServiceMeshPolicy is on STRICT mode, all the communication returns 500 errors.

#### Resolution
Add a DestinationRule named as default with "*.cluster" host and ISTIO_MUTUAL as tls trafficPolicy mode. The DestinationRule should be like [this](https://github.com/kiali/kiali.io/blob/master/data/files/validation_examples/004.yaml).

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/402.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/tree/v1.17/business/checkers/meshpolicies/mesh_mtls_checker.go)
- [Globally enabling Istio mutual TLS](https://istio.io/docs/tasks/security/authn-policy/#globally-enabling-istio-mutual-tls-in-strict-mode)


## ServiceRoles and ServiceRoleBindings {#serviceroles}

### KIA0901 - Unable to find all the defined services

Services can be listed with an exact match, prefix match as well as suffix match. Using "*" refers to all the services in this namespace. This error indicates the services list is pointing to a service that can not be found from this namespace.

#### Resolution

Deploy the missing services or fix the services list to point to correct services.

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/501.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/blob/v1.25/business/checkers/authorization/service_checker.go)
- [Istio documentation](https://istio.io/v1.3/docs/reference/config/authorization/istio.rbac.v1alpha1/#AccessRule)


### KIA0902 - ServiceRole can only point to current namespace

Services can be listed with an exact match, prefix match as well as suffix match. Using "*" refers to all the services in this namespace. Although FQDN can be used, the namespace of the services must be the same as ServiceRole's deployment.

#### Resolution

If the services in question are located in another namespace, deploy this ServiceRole to that namespace, or deploy the services to this namespace.

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/502.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/blob/v1.25/business/checkers/authorization/service_checker.go)
- [Istio documentation](https://istio.io/v1.3/docs/reference/config/authorization/istio.rbac.v1alpha1/#AccessRule)



### KIA0903 - ServiceRole does not exists in this namespace

ServiceRoleBinding assigns a ServiceRole to subjects. As such, it must refer to a ServiceRole object and this object can only reside in the same namespace. Cross-namespace refers are not valid.

#### Resolution

Deploy the missing ServiceRole to the same namespace.

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/601.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/blob/v1.25/business/checkers/authorization/service_binding_checker.go)
- [Istio documentation](https://istio.io/v1.3/docs/reference/config/authorization/istio.rbac.v1alpha1/#ServiceRoleBinding)


## Sidecars {#sidecars}

### KIA1003 - Invalid host format. 'namespace/dnsName' format expected

The Sidecar resources are used for configuring the sidecar proxies in the service mesh. IstioEgressListener specifies the properties of an outbound traffic listener on the sidecar proxy attached to a workload instance.

The hosts list is the list of hosts that will be exposed to the workload. Each host in the list must have the `namespace/dnsName` format.


#### Resolution

Make sure the host has the `namespace/dnsName` format. See more info in the documentation link right below.

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/903.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/tree/master/business/checkers/sidecars/egress_listener_checker.go)
- [Sidecar EgressListener documentation](https://istio.io/docs/reference/config/networking/sidecar/#IstioEgressListener)



### KIA1004 - This host has no matching entry in the service registry

The Sidecar resources are used for configuring the sidecar proxies in the service mesh. IstioEgressListener specifies the properties of an outbound traffic listener on the sidecar proxy attached to a workload instance.

In the hosts field, there is the list of hosts exposed to the workload. Each host in the list have the `namespace/dnsName` format where both namespace and dnsName may have non-obvious values. `namespace` may be either `.`, `~`, `*` or an actual namespace name. `dnsName` has to be a FQDN representing a service, virtual service or a service entry. This FQDN may use the wildcard character.

See more information about the syntax of both `namespace` and `dnsName` into [istio documentation](https://istio.io/docs/reference/config/networking/sidecar/#IstioEgressListener).


#### Resolution

Make sure there is a service, virtual service or service entry matching with the host.

#### Severity

<i class="fas fa-exclamation-triangle"></i> Warning

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/904.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/tree/master/business/checkers/sidecars/egress_listener_checker.go)
- [Sidecar EgressListener documentation](https://istio.io/docs/reference/config/networking/sidecar/#IstioEgressListener)



### KIA1006 - Global default sidecar should not have workloadSelector

The Sidecar resources are used for configuring the sidecar proxies in the service mesh. By default, all the sidecars are configured with the default sidecar instance specified in the control plane namespace (usually istio-system). In case there are sidecar resources in the namespaces where your applications are, this default sidecar resource won't be considered. The sidecar in your namespace will be applied.

Having `workloadSelector` in your global default sidecar won't make any effect in the other sidecars living outside of the control plane namespace.

#### Resolution

Make sure you don't have the `workloadSelector` in this global sidecar resource. In case you need specific settings for specific workloads, move those settings to the sidecar resources in your application namespaces.

#### Severity

<i class="fas fa-exclamation-triangle"></i> Warning

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/906.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/tree/master/business/checkers/sidecars/global_checker.go)
- [Sidecar documentation: second warning](https://istio.io/docs/reference/config/networking/sidecar)


## VirtualServices {#virtualservices}

### KIA1101 - DestinationWeight on route doesn't have a valid service (host not found)

VirtualService routes matching requests to a service inside your mesh. Routing can also match a subset of traffic to a certain version of it for example. Any service inside the mesh must be targeted by its name, the IP address are only allowed for hosts defined through a Gateway. Host must be in a short name or FQDN format. Short name will evaluate to VS' namespace, regardless of where the actual service might be placed.

If the host is not found, Istio ignores the defined rules. However, if a subset with a Destination Rule is not found it affects all the subsets and all the routings. As such, care must be taken that the Destination rule is available before deploying the Virtual Service.

#### Resolution

Correct the host to point to a correct service (in this namespace or with FQDN to other namespaces), deploy the missing service to the mesh or remove the configuration linking to that non-existing service.

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/102.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/tree/master/business/checkers/virtualservices/no_host_checker.go)
- [Destination rule documentation](https://istio.io/docs/reference/config/networking/destination-rule/#DestinationRule)



### KIA1102 - VirtualService is pointing to a non-existent gateway

By default, VirtualService routes apply to sidecars inside the mesh. The gateway field allows to override that default and if anything is defined, the VS applies to those selected. 'mesh' is a reserved gateway name and means all the sidecars in the mesh. If one wishes to apply the VS to gateways as well as the sidecars, the 'mesh' keyword must be used as one of the gateways. Incorrect gateways mean that the VS is not applied correctly.

#### Resolution

Fix the possible gateway field to target all necessary gateways or remove the field if the default 'mesh' is enough.

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/101.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/tree/master/business/checkers/virtualservices/no_gateway_checker.go)



### KIA1103 - VirtualService doesn't define any route protocol

VirtualService is a defined set of rules for routing certain type of traffic to target destinations with rules. At least one, 'tcp', 'http' or 'tls' must be defined.

#### Resolution

This appears to be a configuration error. Fix the definition.

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/103.yaml" %}}
```

#### See Also

- [Istio validation for route types](https://github.com/istio/istio/blob/0e9cecab053aab744a7c3a731aacb07fd794d5f9/pilot/pkg/model/validation.go#L1628)
- [Validator source code](https://github.com/kiali/kiali/blob/master/business/checkers/virtualservices/no_host_checker.go)



### KIA1104 - The weight is assumed to be 100 because there is only one route destination

Istio assumes the weight to be 100 when there is only one [HTTPRouteDestination](https://istio.io/docs/reference/config/networking/virtual-service/#HTTPRouteDestination) or [RouteDestination](https://istio.io/docs/reference/config/networking/virtual-service/#RouteDestination). The warning is present because there is one route with a weight less than 100.

#### Resolution

Either remove the weight field or you might want to add another RouteDestination with an specific weight.

#### Severity

<i class="fas fa-exclamation-triangle"></i> Warning

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/106.yaml" %}}
```

#### See Also
- [Istio documentation about HTTP Route Destination struct](https://istio.io/docs/reference/config/networking/virtual-service/#HTTPRouteDestination)
- [Istio documentation about Route Destination struct](https://istio.io/docs/reference/config/networking/virtual-service/#RouteDestination)
- [Validator source code](https://github.com/kiali/kiali/blob/master/business/checkers/virtualservices/route_checker.go)




### KIA1105 - This subset is already referenced in another route destination

Istio allows you to apply rules over the traffic targetting to a specific service. In order to achieve that, it is necessary to add those rules into either _http_, _tcp_ or _tls_ fields in a VirtualService. In each field it is possible to specify rules for redirection or forwarding traffic. Those rules are the _RouteDestination_ and _HTTPRouteDestination_ structs. Each structs defines where the traffic is shifted to using the _subset_ field.
This warning message refers to the fact of referencing one subset more than one time within the same route. Galley, Istio module in charge of configuration validation, allows the subset duplicity. However, the mesh it might become broken when there are different duplicates. Also, the presented warning might help spoting a typo.

#### Resolution

Make sure there is only one reference to the same subset for each RouteDestination. Either [HTTPRouteDestination](https://istio.io/docs/reference/config/networking/virtual-service/#HTTPRouteDestination) or [RouteDestination](https://istio.io/docs/reference/config/networking/virtual-service/#RouteDestination).

#### Severity

<i class="fas fa-exclamation-triangle"></i> Warning

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/111.yaml" %}}
```

#### See Also
- [Istio documentation about HTTP Route Destination struct](https://istio.io/docs/reference/config/networking/virtual-service/#HTTPRouteDestination)
- [Istio documentation about Route Destination struct](https://istio.io/docs/reference/config/networking/virtual-service/#RouteDestination)
- [Validator source code](https://github.com/kiali/kiali/blob/master/business/checkers/virtualservices/route_checker.go)



### KIA1106 - More than one Virtual Service for same host

A VirtualService defines a set of traffic routing rules to apply when a host is addressed. Each routing rule defines matching criteria for traffic of a specific protocol. If the traffic is matched, then it is sent to a named destination service (or subset/version of it) defined in the registry.

#### Resolution

This is a valid configuration only if two VirtualServices share the same host but are bound to a different gateways, sidecars do not accept this behavior. There are several caveats when using this method and defining the same parts in multiple Virtual Service definitions is not recommended. While Istio will merge the configuration, it does not guarantee any ordering for cross-resource merging and only the first seen configuration is applied (rest ignored). As recommended, each VS definition should have a 'catch-all' situation, but this can only be defined in a definition affecting the same host.

#### Severity

<i class="fas fa-exclamation-triangle"></i> Warning

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/104.yaml" %}}
```

#### See Also

- [Istio documentation: Split large virtual services and destination rules into multiple resources](https://istio.io/docs/ops/best-practices/traffic-management/#split-virtual-services)
- [Validator source code](https://github.com/kiali/kiali/blob/master/business/checkers/virtualservices/single_host_checker.go)



### KIA1107 - Subset not found

VirtualService routes matching requests to a service inside your mesh. Routing can also match a subset of traffic to a certain version of it for example. The subsets referred in a VirtualService have to be defined in one DestinationRule.

If one route in the VirtualService points to a subset that doesn't exist Istio won't be able to send traffic to a service.

#### Resolution
Fix the routes that points to a non existing subsets. It might be fixing a typo in the subset's name or defining the missing subset in a DestinationRule.

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/105.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/tree/master/business/checkers/virtualservices/subset_presence_checker.go)



### KIA1108 - Preferred nomenclature: <gateway namespace>/<gateway name>

A virtual service may include a list of gateways which the defined routes should be applied to. Gateways in other namespaces may be referred to by <gateway namespace>/<gateway name>; specifying a gateway with no namespace qualifier is the same as specifying the VirtualService’s namespace.

#### Resolution
Move the nomenclature of the gateways into the supported Istio form: <gateway namespace>/<gateway name>

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/112.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/tree/master/business/checkers/virtualservices/no_gateway_checker.go)


## Generic {#generic}

### KIA0001 - Unable to verify the validity, cross-namespace validation is not supported for this field

In certain cases, Kiali is unable to validate the field since it spans another namespace to which the validator is not capable of looking at. In such cases, Kiali will mark this field with a grey icon indicating that the fields correctness could not be verified. This does not necessarily mean there is an error, but that the user should be careful and do the validation manually.

### KIA0002 - More than one selector-less object in the same namespace

This validation refers to the usage of the `selector`. Selector-less Istio objects are those objects that don't have the `selector` field specified. Therefore, objects that apply to all the workloads of a namespace (or whole mesh if the namespace is the same as the control plane namespace).

This validation warns you that you have two different objects living in the same namespace. This may leave an non-deterministic or unexpected behavior on the workloads of the namespace.

#### Resolution

The natural solution is to merge both objects. In case there are different behaviors you want to apply, consider to define the `selector` field targeting a specific set of workloads.

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/302.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/tree/master/business/checkers/common/multi_match_selector_checker.go)


### KIA0003 - More than one object applied to the same workload

This validation refers to the usage of the `selector`. In this field are defined the labels of the workloads that this object will be applied to. It might be one or more workloads in the same namespace.

This validation warns the scenario where there are two different objects applying to the same workload(s). This may leave an undeterministic or unexpected behavior on the workloads of the namespace.

#### Resolution

There isn't a standard solution for that. It is a good practice not to have multiple rules of the same kind applying to the same workloads. Otherwise you would end up having interferences between objects and having troubles when debugging.
The first approach would be to merge both objects into one if possible. The second approach would be to reorganize the objects of the same kind in a way that each one only applies to a different set of workloads. Applying no change into the objects is also an option although not desiderable.

#### Severity

<i class="fas fa-times-circle"></i> Error

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/303.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/tree/master/business/checkers/common/workload_selector_checker.go)



### KIA0004 - No matching workload found for the selector in this namespace

This validation warns the scenario where there are not workloads matching with the `selector` labels. In other terms, this object doesn't have any implication into the mesh.

#### Resolution

There are three scenarios: either change the labels to match an existing workload (useful with typos), deploy a workload that match with those labels or safely remove this object.

#### Severity

<i class="fas fa-exclamation-triangle"></i> Warning

#### Example

```yaml
{{% readfile file="/themes/docsy/static/files/validation_examples/304.yaml" %}}
```

#### See Also

- [Validator source code](https://github.com/kiali/kiali/tree/master/business/checkers/common/workload_selector_checker.go)