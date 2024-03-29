# K8s-ldap-auth Helm Chart

[k8s-ldap-auth](https://github.com/vbouchaud/k8s-ldap-auth) is a kubernetes webhook token authentication plugin implementation using ldap.

**NOTICE:**
This chart uses api version 2 which is only supported by helm v3+.
This chart **only** install `k8s-ldap-auth`. You need to have access to an already configured `LDAP` server.

## TL;DR;

```console
$ helm repo add vbouchaud https://vbouchaud.github.io/chartrepo/

# generating private and public key for JWT signing
$ openssl genrsa -out key.pem 4096
$ openssl rsa -in key.pem -outform PEM -pubout -out public.pem

# fetching the default values.yml for further editions
$ helm show values vbouchaud/k8s-ldap-auth > values.yml
```

```yaml
ingress:
  enabled: true
  className: traefik
  hosts:
    - host: app.valhall.local
      paths:
        - path: /cluster/ldap

config:
  ldap:
    address: ldaps://ldap.valhall.local
    bindDn: uid=k8s-ldap-auth,ou=services,ou=valhall,o=local
    bindPassword: P@ssw0rd
    user:
      searchBase: ou=people,ou=valhall,o=local
      searchFilter: (uid=%s)
  keys:
    # insert here the private key you generated previously
    privateKeyData: |-
      -----BEGIN PRIVATE KEY-----
      MIIJQQIBADANBgkqhkiG9w0BAQEFAASCCSswggknAgEAAoICAQDU4WQez1ww4llv
      # ...
      9QJeAv5SSy/Tuj9vkaI+1cdrnNagkJLKY1BFOYi0WXuq1rsUvxQvIStxtcewYF2a
      hfnkt+Md2qZtLxjOFYN51Y3lVk3e
      -----END PRIVATE KEY-----
      
    # insert here the public key you generated previously
    publicKeyData: |-
      -----BEGIN PUBLIC KEY-----
      MIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEA1OFkHs9cMOJZb/p9VgoZ
      # ...
      neybKNLe7Ftz6ZMC1aFRjROgDD8wbfDoz9KpYHwT4HqgmMFLqUKx1BCEFKls1hT5
      UKJaCLzmZMb3pMl63/8WfoUCAwEAAQ==
      -----END PUBLIC KEY-----
```

```console
$ helm install k8s-ldap-auth vbouchaud/k8s-ldap-auth --values values.yml 
```

## Getting Started

1. Add the chart repository with `helm repo add vbouchaud https://vbouchaud.github.io/chartrepo/`
2. Configure the chart by setting the various [parameters](##parameters), either in a locally downloaded values.yaml or in the next step.
3. Install the chart with `helm install k8s-ldap-auth vbouchaud/k8s-ldap-auth` and optionally set your values with `--values values.yaml` or via `--set [parameter]=[value]`.

## Parameters

| Parameter                                     | Description                                                                                                      | Default                                |
| :-------------------------------------------: | :--------------------------------------------------------------------------------------------------------------: | :------------------------------------: |
| replicaCount                                  | Number of instances running.                                                                                     | 1                                      |
| image.repository                              | The container registry to use when pulling the image.                                                            | quay.io/vbouchaud/k8s-ldap-auth        |
| image.pullPolicy                              | The policy to ue when pulling the image.                                                                         | IfNotPresent                           |
| image.tag                                     | Overrides the image tag whose default is the chart appVersion.                                                   | nil                                    |
| imagePullSecrets                              | The k8s secret names to use for the pullSecrets.                                                                 | []                                     |
| nameOverride                                  | Provide a name in place of kube-prometheus-stack for `app:` labels.                                              | nil                                    |
| fullnameOverride                              | Provide a name to substitute for the full names of resources.                                                    | nil                                    |
| serviceAccount.create                         | Specifies whether a service account should be created.                                                           | true                                   |
| serviceAccount.annotations                    | Annotations to add to the service account.                                                                       | {}                                     |
| serviceAccount.name                           | The name of the service account to use.                                                                          | nil                                    |
| podAnnotations                                | Annotations to add to the pod template.                                                                          | {}                                     |
| podSecutiryContext                            | Security context to add to the pod.                                                                              | {}                                     |
| securityContext                               | Security context to add to the container.                                                                        | {}                                     |
| service.type                                  | Service type.                                                                                                    | ClusterIp                              |
| service.port                                  | Service port.                                                                                                    | 80                                     |
| ingress.enabled                               | Whether to create an ingress.                                                                                    | false                                  |
| ingress.className                             | Ingress classname to use.                                                                                        | nil                                    |
| ingress.annotations                           | Ingress specific annotations.                                                                                    | {}                                     |
| ingress.hosts[0].host                         | The domain to use for the ingress.                                                                               | chart-example.local                    |
| ingress.hosts[0].paths[0].path                | The path to use for the ingress.                                                                                 | /                                      |
| ingress.hosts[0].paths[0].pathType            | The path type to use for the ingress.                                                                            | ImplementationSpecific                 |
| ingress.tls                                   | The tls secrets to use for the domain.                                                                           | []                                     |
| resources                                     | Resources object for the container.                                                                              | {}                                     |
| config.ldap.address                           | The ldap host (and scheme) the server will authenticate against.                                                 | nil                                    |
| config.ldap.bindDn                            | The service account dn to do the ldap search.                                                                    | nil                                    |
| config.ldap.bindPassword                      | The service account password to do the ldap search.                                                              | nil                                    |
| config.ldap.user.searchBase                   | The dn where the ldap search will take place.                                                                    | nil                                    |
| config.ldap.user.searchFilter                 | The filter to select users.                                                                                      | nil                                    |
| config.ldap.user.searchScope                  | The scope of the search. Can take to values base object: 'base', single level: 'single' or whole subtree: 'sub'. | nil                                    |
| config.ldap.user.memberofProperty             | The property that will be used to fetch groups. Usually memberof or ismemberof.                                  | nil                                    |
| config.ldap.user.usernameProperty             | The property that will be used as username in the TokenReview.                                                   | uid                                    |
| config.ldap.user.extraProperties              | Extra user properties to fetch. Those will be stored in extra values in the UserInfo object.                     | nil                                    |
| config.keys.privateKeyData                    | The key used to sign the token.                                                                                  | nil                                    |
| config.keys.publicKeyData                     | The key to verify the token.                                                                                     | nil                                    |
| config.token.ttl                              | The TTL for newly generated tokens, in seconds.                                                                  | nil                                    |
| autoscaling.enabled                           | Whether to enable autoscaling.                                                                                   | false                                  |
| autoscaling.minReplicas                       | Minimum number of replicas.                                                                                      | 1                                      |
| autoscaling.maxReplicas                       | Maximum number of replicas.                                                                                      | 10                                     |
| autoscaling.targetCPUUtilizationPercentage    | CPU level configuration for autoscaling.                                                                         | 80                                     |
| autoscaling.targetMemoryutilizationpercentage | Memory level configuration for autoscaling.                                                                      | nil                                    |
| nodeSelector                                  | Node selector.                                                                                                   | {}                                     |
| tolerations                                   | Tolerations.                                                                                                     | []                                     |
| affinity                                      | Affinity.                                                                                                        | {}                                     |
