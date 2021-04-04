# Installing Helm Keycloak
Installation guide for Keycloak helm chart

## Add Repo

```
$ helm repo add codecentric https://codecentric.github.io/helm-charts

$ helm repo update

$ helm search repo keycloak
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
codecentric/keycloak    10.1.0          12.0.4          Open Source Identity and Access Management For ...
```

## Create Namespace

```
kubectl create ns keycloak
```

## Install

```
$ helm install \
  -n [NAMESPACE] \
  -f [FILE_NAME] \
  --set [KEY]=[VALUE] \
  [RELEASE_NAME] [CHART_NAME] \

## e.g. Using LoadBalancer
$ helm install \
  -n keycloak \
  -f values.yaml \
  --set service.type=LoadBalancer \
  --set postgresql.postgresqlPassword=my-password \
  --set-string adminPassword=my-password \
  my-keycloak codecentric/keycloak --version 10.1.0

## e.g. Using Ingress
$ kubectl create secret tls example-tls -n keycloak \
--cert /PATH/TO/CERT.crt \
--key /PATH/TO/CERT.key

$ helm install \
  -n keycloak \
  -f values.yaml \
  --set ingress.enabled=true \
  --set ingress.rules[0].host=keycloak.example.com \
  --set ingress.tls[0].hosts[0]=example.com \
  --set ingress.tls[0].secretName=example-tls \
  --set postgresql.postgresqlPassword="my-password" \
  --set-string adminPassword="my-password" \
  my-keycloak codecentric/keycloak --version 10.1.0
```

## Login to web console

1. Get your 'admin' user password by running:

    ```
    $ printf $(kubectl get secret --namespace keycloak my-keycloak-admin -o jsonpath="{.data.KEYCLOAK_PASSWORD}" | base64 --decode);echo
    my-password
    ```

2. Get the web console URL to visit by running these commands in the same shell:

    ```
    ## e.g. Using LoadBalancer
    $ export SERVICE_IP=$(kubectl get service --namespace keycloak my-keycloak-http --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
    $ echo "https://$SERVICE_IP:8443"
    https://x.x.x.x:8443

    ## e.g. Using Ingress
    $ printf $(kubectl get ing --namespace keycloak my-keycloak -o jsonpath="{.spec.rules[0].host}");echo
    keycloak.example.com
    ```

3. Login with the password from step 1 and the username: admin

## Uninstall

```
$ helm uninstall -n [NAMESPACE] [RELEASE_NAME]

## e.g
$ helm uninstall -n keycloak my-keycloak
```

