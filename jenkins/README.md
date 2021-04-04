# Installing Helm Jenkins

## 1.Add Repo

```
$ helm repo add jenkins https://charts.jenkins.io

$ helm repo update

$ helm search repo jenkins
NAME            CHART VERSION   APP VERSION     DESCRIPTION
jenkins/jenkins 3.2.3           lts             Open source source 
```

## 2.Create Namespace

```
kubectl create ns jenkins
```

## 3.Install Jenkins

```
$ helm install \
  -n [NAMESPACE] \
  -f [FILE_NAME] \
  --set [KEY]=[VALUE] \
  [RELEASE_NAME] [CHART_NAME] --version [CHART_VERSION]

## e.g. Using LoadBalancer
$ helm install \
  -n jenkins \
  -f values.yaml \
  --set controller.adminPassword=my-password \
  --set controller.serviceType=LoadBalancer \
  my-jenkins jenkins/jenkins --version 3.2.3

## e.g. Using Ingress
$ kubectl create secret tls example-tls -n jenkins \
--cert /PATH/TO/CERT.crt \
--key /PATH/TO/CERT.key

$ helm install \
  -n jenkins \
  -f values.yaml \
  --set ingress.enabled=true \
  --set ingress.hostName=jenkins.example.com \
  --set ingress.tls[0].hosts[0]=example.com \
  --set ingress.tls[0].secretName=example-tls \
  --set controller.adminPassword=my-password \
  my-jenkins jenkins/jenkins --version 3.2.3
```

## (Optional) 4.Deploy Clusterrolebinding
To enable Jenkins to access all Namespaces, run the following command.

```
kubectl create -f cluster-role-binding.yaml -n jenkins
```

## 5.Login to Jenkins web console

1. Get your 'admin' user password by running:

    ```
    $ printf $(kubectl get secret --namespace jenkins my-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
    my-password
    ```

2. Get the Jenkins URL to visit by running these commands in the same shell:

    NOTE: It may take a few minutes for the LoadBalancer IP to be available.
          You can watch the status of by running 'kubectl get svc --namespace jenkins -w my-jenkins'
  
    ```
    ## e.g. Using LoadBalancer
    $ export SERVICE_IP=$(kubectl get svc --namespace jenkins my-jenkins --template "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
    $ echo http://$SERVICE_IP:8080/login
    http://x.x.x.x:8080/login

    ## e.g. Using Ingress
    $ printf $(kubectl get ing --namespace jenkins my-jenkins -o jsonpath="{.spec.rules[0].host}");echo
    jenkins.example.com
    ```

3. Login with the password from step 1 and the username: admin

## 6.Test

1. New Item > Enter pipeline name > Select "Pipeline" > Click "OK" button

2. Click "Configure" menu > Copy & Paste the following script to "Pipeline" section > Click "Save" button

    NOTE: To run this pipeline "cluster-admin" role is required. Do "Deploy Clusterrolebinding" section first.

    ```
    pipeline {
        agent {
            kubernetes {
                yaml '''
                  apiVersion: v1
                  kind: Pod
                  spec:
                    containers:
                    - name: kubectl
                      image: lachlanevenson/k8s-kubectl
                      command:
                      - sleep
                      args:
                      - infinity
                '''
            }
        }
        stages {
            stage('Main') {
                steps {
                    container('kubectl') {
                        sh '''
                            kubectl get po --all-namespaces
                        '''
                    }
                }
            }
        }
    }
    ```

3. Click "Build Now" menu > Build History > Click build number(e.g: #1) > Click "Console Output" menu

    You are able to see result such as the following.

    ```
    ...
    Running on kubectl-1-8664g-4s114-z7nwk in /home/jenkins/agent/workspace/kubectl
    ...
    [Pipeline] sh
    + kubectl get po --all-namespaces
    NAMESPACE     NAME                                                        READY   STATUS    RESTARTS   AGE
    jenkins       kubectl-1-8664g-4s114-z7nwk                                 2/2     Running   0          12s
    jenkins       my-jenkins-557598f569-fqlnb                                 2/2     Running   0          17m
    ...
    [Pipeline] End of Pipeline
    Finished: SUCCESS
    ```

## 7.Uninstall Jenkins

```
$ helm uninstall -n [NAMESPACE] [RELEASE_NAME]

## e.g
$ helm uninstall -n jenkins my-jenkins
```
