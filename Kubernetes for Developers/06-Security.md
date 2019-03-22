# Security

- [Security](#security)
  - [Accessing the API](#accessing-the-api)
    - [Authentication](#authentication)
    - [Authorization](#authorization)
      - [ABAC (Attribute Based Access Control)](#abac-attribute-based-access-control)
      - [RBAC (Role Based Access Controler)](#rbac-role-based-access-controler)
    - [Admission Controller](#admission-controller)
  - [Security Contexts](#security-contexts)
  - [Pod Security Contexts](#pod-security-contexts)
  - [Network Security Polices](#network-security-polices)

## Accessing the API

To perform any action on the K8s API, we need to go throught three steps:

* Authentication
* Authorization
* Admission Controller

[More details on the officiel website about controlling access](https://kubernetes.io/docs/reference/access-authn-authz/controlling-access/)

Request on the API -> Authentication module -> if pass -> Authorization module

At the authorization module, the request will be checked against our polices (like permissions on users). 

Then, it will go throught the admission controller, that will check the actual content of the object

### Authentication

* Authentication is done with certificates/tokens/basic-auth
* Users not created by API, but managed by external system
* System accounts are used by processes to access the API

Advanced authentication mechanisms: 

* **Webhook**: verify bearer token
* Connection with an external **OpenID** provider

The authentication type is defined in the kube-apiserver at startup. 

* `--basic-auth-file`
* `--oidc-issuer-url`
* `--token-auth-file`
* `--authorization-webhook-config-file`

More than one can be used at the same time.

### Authorization

Three main authorization mdoes:

* ABAC
* RBAC
* WebHook

Can be configured as kube-apiserver startup options : `--authorization-mode=[ABAC | RBAC | WebHook | AlwaysDeny | AlwaysAllow]`

#### ABAC (Attribute Based Access Control)

First auth. model in k8s. More and more RBAC is becoming the default. 

Polices are defined through a JSON at start-up like this: `--authorization-policy-file=my_policy.json`

[Polices example](https://kubernetes.io/docs/reference/access-authn-authz/abac/#examples)

#### RBAC (Role Based Access Controler)

* Rules: operation which can act upon an API group (core/apps)
* Roles: group of rules which affect a single namespaces
* ClusterRoles: scope the entire cluster
  
  Each op can act upon one of 3 subjects:

  * User Accounts
  * Service Accounts
  * Groups

RBAC is then writing rules to allow/deny op by users,roles or group upon resources.

RBAC can be complex, but the basic flow is to create a certificate for a user. User != API object of K8s, so we require an outside authentication such as OpenSSL certificates.

After verifying the certificate against the cluster certifacate authority, we can set the creds for the user using a context.

Roles can then be used to configure an association of apiGroups, resources and the verbs allowed to them. 

A user can be bound to a role. 

Determine or create namespace -> Create certificate creds for user -> Set the creds for the user to the namespace using a context -> Create a role for the expected task set -> Bind the user to the role -> Verify the user has limited access

### Admission Controller

This is the last step. It access the content of the request made to K8s. They can modify/valide it and refuse it.

1.9.1 defines this set of controllers by default: 

`--admission-control=Initializers,NamespaceLifecycle,LimitRanger,ServiceACcount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestrictions,ResourceQuota`

## Security Contexts

a Security Context can be defined for an entire pod or per container, and is represented as additional sections in the resource manifest. Linux capabilities are set at the container level.

```yaml
# Enfore a policy that containers cannot run their process as the root user

apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  securityContext:
      runAsNonRoot: true
  containers:
  - image: nginx
    name: nginx
```

[Security Context reference](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

## Pod Security Contexts

Automate the enforcement of security context -> Define a PodSecurityPolices (PSP). 

Cluster-level rules.

```yaml

# Example of a PSP

apiVersion: extensions/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  runAsUser:
    rule: MustRunAsNonRoot
  fsGroup:
    rule: RunAsAny
```
To be enabled, the admission controller of the controller-manager to contain PodSecurityPolicy

[An example of a PSP RBAC's github repo](https://github.com/kubernetes/examples/blob/master/staging/podsecuritypolicy/rbac/README.md)

## Network Security Polices

Default: All pods can reach each other; all ingress/egress traffic is allowed. 

Network isolation can be configured an dtraffic to pods can be block, even egress traffic. 

Done by configuring a NetworkPolicy. Not all network providers support the NetworkPolicies

[Example of network-policies on github](https://github.com/ahmetb/kubernetes-network-policy-recipes)
