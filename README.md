# REG.RU provider DNS01 challenge solver


## REG.RU API documentation

API documentation https://www.reg.ru/reseller/api2doc#common

## REG.RU DNS provider allow access to API from known IP-addresses only

Access configuration https://www.reg.ru/user/account/#/settings/api/
```json
{
   "charset" : "utf-8",
   "error_code" : "ACCESS_DENIED_FROM_IP",
   "error_params" : {
      "command_name" : "zone/get_resource_records"
   },
   "error_text" : "Access to API from this IP denied",
   "messagestore" : null,
   "result" : "error"
}
```
## Usage
### Installation

```
TODO
```

Create kubernetes secret with credentials:
```
kubectl --namespace cert-manager create secret generic regru-api-creds --from-literal=login='<your-username>' --from-literal=password='<your-password>'
```
One can use any suitable way https://kubernetes.io/docs/tasks/configmap-secret/


## Creating clusterIssuer resource

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: regru-dns
spec:
  acme:
    email: username@example.com
    privateKeySecretRef:
      name: letsencrypt-private-key
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    solvers:
    - dns01:
        webhook:
          config:
            apiLoginRef:
              key: login
              name: regru-api-creds
            apiPasswordRef:
              key: password
              name: regru-api-creds
          groupName: acme.regru.ru
          solverName: regru

```
## Creating your own webhook

Webhook's themselves are deployed as Kubernetes API services, in order to allow
administrators to restrict access to webhooks with Kubernetes RBAC.

This is important, as otherwise it'd be possible for anyone with access to your
webhook to complete ACME challenge validations and obtain certificates.

To make the set up of these webhook's easier, we provide a template repository
that can be used to get started quickly.

### Creating your own repository

### Running the test suite

All DNS providers **must** run the DNS01 provider conformance testing suite,
else they will have undetermined behaviour when used with cert-manager.

**It is essential that you configure and run the test suite when creating a
DNS01 webhook.**

An example Go test file has been provided in [main_test.go](https://github.com/cert-manager/webhook-example/blob/master/main_test.go).

You can run the test suite with:

```bash
$ TEST_ZONE_NAME=example.com. make test
```

The example file has a number of areas you must fill in and replace with your
own options in order for tests to pass.
