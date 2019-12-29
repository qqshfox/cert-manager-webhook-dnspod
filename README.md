# DNSPod Webhook for Cert Manager

This is a webhook solver for [DNSPod](https://www.dnspod.cn).

## Prerequisites

* [cert-manager](https://github.com/jetstack/cert-manager): *tested with 0.8.0*
    - [Installing on Kubernetes](https://docs.cert-manager.io/en/release-0.8/getting-started/install/kubernetes.html)

## Installation

Generate API ID and API Token from DNSPod (https://support.dnspod.cn/Kb/showarticle/tsid/227/).

```console
$ helm install --name cert-manager-webhook-dnspod ./deploy/cert-manager-webhook-dnspod \
    --set groupName=<GROUP_NAME> \
    --set secrets.apiID=<DNSPOD_API_ID>,secrets.apiToken=<DNSPOD_API_TOKEN> \
    --set clusterIssuer.enabled=true,clusterIssuer.email=<EMAIL_ADDRESS>
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
