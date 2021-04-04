# Installing Helm Harbor
Installation guide for Harbor helm chart

## Add Repo

```
$ helm repo add harbor https://helm.goharbor.io

$ helm repo update

$ helm search repo harbor
NAME            CHART VERSION   APP VERSION     DESCRIPTION
harbor/harbor   1.6.0           2.2.0           An open source trusted cloud native registry th...
```

## Create Namespace

```
kubectl create ns harbor
```

## Install

```
$ helm install \
  -n [NAMESPACE] \
  -f [FILE_NAME] \
  --set [KEY]=[VALUE] \
  [RELEASE_NAME] [CHART_NAME] --version [CHART_VERSION]

## e.g
$ helm install \
  -n harbor \
  -f values.yaml \
  --set expose.loadBalancer.IP=x.x.x.x	\
  --set externalURL=http://x.x.x.x \
  my-harbor harbor/harbor --version 1.6.0
```

## Login to web console

1. Get your 'admin' user password by running:

    ```
    $ printf $(kubectl get secret --namespace harbor my-harbor-harbor-core -o jsonpath="{.data.HARBOR_ADMIN_PASSWORD}" | base64 --decode);echo
    Harbor12345
    ```

2. Get the web console URL to visit by running these commands in the same shell:

    ```
    $ export SERVICE_IP=$(kubectl get service --namespace harbor harbor --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
    $ echo "http://$SERVICE_IP"
    http://x.x.x.x
    ```

3. Login with the password from step 1 and the username: admin

## Uninstall 

```
$ helm uninstall -n [NAMESPACE] [RELEASE_NAME]

## e.g
$ helm uninstall -n harbor my-harbor
```
