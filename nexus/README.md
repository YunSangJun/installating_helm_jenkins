# Installing Helm Nexus
Installation guide for Nexus helm chart

## Add Repo

```
$ helm repo add sonatype https://sonatype.github.io/helm3-charts/

$ helm repo update

$ helm search repo nexus
NAME                                    CHART VERSION   APP VERSION     DESCRIPTION
sonatype/nexus-repository-manager       29.2.0          3.29.2          Sonatype Nexus Repository Manager - Universal B...
```

## Create Namespace

```
kubectl create ns nexus
```

## Install

```
$ helm install \
  -n [NAMESPACE] \
  -f [FILE_NAME] \
  --set [KEY]=[VALUE] \
  [RELEASE_NAME] [CHART_NAME] --version [CHART_VERSION] \

## e.g
$ helm install \
  -n nexus \
  -f values.yaml \
  --set service.type=LoadBalancer \
  my-nexus sonatype/nexus-repository-manager --version 29.2.0
```

## Login to web console

1. Get your 'admin' user password by running:

    ```
    $ kubectl get po
    NAME                                                 READY   STATUS    RESTARTS   AGE
    my-nexus-nexus-repository-manager-66485b8cf6-r4zv5   1/1     Running   0          7m18s
    
    $ kubectl exec -it my-nexus-nexus-repository-manager-66485b8cf6-r4zv5 -- cat /nexus-data/admin.password
    a469b43b-9f32-41e8-aa86-274581b85102
    ```

2. Get the web console URL to visit by running these commands in the same shell:

    ```
    $ export SERVICE_IP=$(kubectl get svc --namespace nexus my-nexus-nexus-repository-manager --template "{{ range (index .status.loadBalancer.ingr^Cs 0) }}{{.}}{{ end }}")
    $ echo "https://$SERVICE_IP:8081"
    http://x.x.x.x:8081
    ```

3. Login with the password from step 1 and the username: admin

## Uninstall

```
$ helm uninstall -n [NAMESPACE] [RELEASE_NAME]

## e.g
$ helm uninstall -n nexus my-nexus
```
