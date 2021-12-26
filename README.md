# helm-aws-kafka-proxy
Helm chart for installing Kafka Proxy

Uses - https://github.com/grepplabs/kafka-proxy

## Pre-Requisites

### Cert Manager

Kafka proxy uses TLS listener so we need to provide a certificate.

We use cert manager (https://github.com/HarshadRanganathan/helm-aws-cert-manager) to issue a certificate for our proxy.

`certificate.yaml` creates a CRD and asks cert manager to use Lets Encrypt ACME server to issue the certificate.

Below are the sequence of steps involved when you install the chart:

[1] External DNS creates Route53 records for each of the proxy broker instance

[2] Cert Manager creates certificate request which is verified by DNS lookup

[3] Certificate issued and stored in secret on successful verification

[4] Certificate mounted to the deployment is used by the proxy to establish TLS connections

## Install/Upgrade Chart

Run below command to install/upgrade the chart:

```
helm upgrade -i pes-kafka-proxy . -n platform --values stages/prod/prod-values.yaml
```
