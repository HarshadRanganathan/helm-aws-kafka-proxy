# helm-aws-kafka-proxy
Helm chart for installing Kafka Proxy

Uses - https://github.com/grepplabs/kafka-proxy

Table of Contents
=================

* [helm-aws-kafka-proxy](#helm-aws-kafka-proxy)
   * [Pre-Requisites](#pre-requisites)
      * [Cert Manager](#cert-manager)
      * [SASL/Plain Auth](#saslplain-auth)
         * [Vault](#vault)
   * [Install/Upgrade Chart](#installupgrade-chart)
   * [Client Connection](#client-connection)

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

### SASL/Plain Auth

`prod-values.yaml` configures the proxy to use SASL/Plain auth mechanism for the listener i.e. Clients need to authenticate with the proxy by providing username/password

Update the placeholders `${PROXY_USERNAME}` and `${PROXY_PASSWORD}` in your CI/CD script with the username and password that you want to use from your credentials management system

#### Vault

If you are using Vault to store your credentials, this could be achieved with below commands:

Read the credentials from Vault:

```bash
read proxy_username proxy_password < <(echo $(curl --header "X-Vault-Token:<token>" --header "X-Vault-Namespace:<namespace>" https://<vault_url>/<path> | jq -r '.data.data.username,.data.data.password' ))
```

Substitute the placeholders with credentials:

```bash
sed -i '' -e "s/\${PROXY_USERNAME}/$proxy_username/" -e "s/\${PROXY_PASSWORD}/$proxy_password/" stages/prod/prod-values.yaml
```

Clear the credentials from the file and env variables:

```bash
sed -i '' -e "s/$proxy_username/\${PROXY_USERNAME}/" -e "s/$proxy_password/\${PROXY_PASSWORD}/" stages/prod/prod-values.yaml
unset proxy_username
unset proxy_password
```

## Install/Upgrade Chart

Run below command to install/upgrade the chart:

```
helm upgrade -i pes-kafka-proxy . -n platform --values stages/prod/prod-values.yaml
```

## Client Connection

Sample client properties:

```
bootstrap.servers=
security.protocol=SASL_SSL
ssl.enabled.protocols=TLSv1.2
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username=${PROXY_USERNAME} password=${PROXY_PASSWORD};
```

Sample config for Spring Boot app:

```
spring:
  kafka:
    bootstrap-servers:
    admin:
      client-id:
      properties:
        security.protocol: SASL_SSL
        ssl.enabled.protocols: TLSv1.2
        sasl.mechanism: PLAIN
        sasl.jaas.config: org.apache.kafka.common.security.plain.PlainLoginModule required username=${PROXY_USERNAME} password=${PROXY_PASSWORD};
```
