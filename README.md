# DNSPod Webhook for Cert Manager

This is a webhook solver for [DNSPod](https://www.dnspod.cn).

## Prerequisites

* [cert-manager](https://github.com/jetstack/cert-manager): *tested with 0.8.0*
    - [Installing on Kubernetes](https://docs.cert-manager.io/en/release-0.8/getting-started/install/kubernetes.html)

## Installation

```console
$ helm install --name cert-manager-webhook-dnspod ./deploy/example-webhook
```

## Issuer

1. Generate API ID and API Token from DNSPod (https://support.dnspod.cn/Kb/showarticle/tsid/227/)
2. Create secret to store the API Token
```console
$ kubectl --namespace cert-manager create secret generic \
    dnspod-credentials --from-literal=api-token='<DNSPOD_API_TOKEN>'
```

3. Grant permission for service-account to get the secret
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: cert-manager-webhook-dnspod:secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["dnspod-credentials"]
  verbs: ["get", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: cert-manager-webhook-dnspod:secret-reader
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: cert-manager-webhook-dnspod:secret-reader
subjects:
  - apiGroup: ""
    kind: ServiceAccount
    name: cert-manager-webhook-dnspod
```

4. Create a staging issuer *Optional*
```yaml
apiVersion: certmanager.k8s.io/v1alpha1
kind: Issuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # The ACME server URL
    server: https://acme-staging-v02.api.letsencrypt.org/directory

    # Email address used for ACME registration
    email: user@example.com # REPLACE THIS WITH YOUR EMAIL!!!

    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-staging

    solvers:
    - dns01:
        webhook:
          groupName: example.com # REPLACE THIS TO YOUR GROUP
          solverName: dnspod
          config:
            apiID: 12345 # REPLACE WITH API ID FROM DNSPOD!!!
            apiTokenSecretRef:
              key: api-token
              name: dnspod-credentials
```

5. Create a production issuer
```yaml
apiVersion: certmanager.k8s.io/v1alpha1
kind: Issuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory

    # Email address used for ACME registration
    email: user@example.com # REPLACE THIS WITH YOUR EMAIL!!!

    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod

    solvers:
    - dns01:
        webhook:
          groupName: example.com # REPLACE THIS TO YOUR GROUP
          solverName: dnspod
          config:
            apiID: 12345 # REPLACE WITH API ID FROM DNSPOD!!!
            apiTokenSecretRef:
              key: api-token
              name: dnspod-credentials
```

## Certificate

1. Issue a certificate
```yaml
#TODO
```

### Automatically creating Certificates for Ingress resources

See [this](https://docs.cert-manager.io/en/latest/tasks/issuing-certificates/ingress-shim.html).

## Development

All DNS providers **must** run the DNS01 provider conformance testing suite,
else they will have undetermined behaviour when used with cert-manager.

**It is essential that you configure and run the test suite when creating a
DNS01 webhook.**

An example Go test file has been provided in [main_test.go]().

Before you can run the test suite, you need to download the test binaries:

```console
$ mkdir __main__
$ wget -O- https://storage.googleapis.com/kubebuilder-tools/kubebuilder-tools-1.14.1-darwin-amd64.tar.gz | tar x -
$ mv kubebuilder __main__/hack
```

Then modify `testdata/my-custom-solver/config.json` to setup the configs.

Now you can run the test suite with:

```bash
$ TEST_ZONE_NAME=example.com go test .
```
