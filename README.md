# Create PVC

```
kubectl create -f jenkins-pvc.yaml
```

# Deploy Helm Chart

```
helm install --name jenkins \
-f values.yaml \
--set Master.AdminPassword=jenkins,Master.Cpu="1",Master.Memory="1Gi",Master.HostName="jenkins.ghama.io",Agent.Cpu="1",Agent.Memory="1Gi",Persistence.Enabled=true,Persistence.ExistingClaim="jenkins-pvc" \
stable/jenkins
```

# Create Jenkins Pipeline Job

1. Cofigure Parameter

```
GIT_URL=https://github.com/YunSangJun/jenkins-test.git
GIT_BRANCH=master
DOCKER_REGISTRY=DOCKER_REGISTRY_URL:DOCKER_REGISTRY_PORT
DOCKER_REGISTRY_NAMESPACE=prod
DOCKER_ID=admin
DOCKER_PW=admin
DOCKER_IMAGE_NAME=IMAGE_NAME
K8S_CONF_PATH=./k8s/prd
DOCKER_IMAGE_VERSION=1.0
```

2. Make Pipeline Script

```
podTemplate(label: 'jenkins-release-sj-jenkins-slave', containers: [
    containerTemplate(name: 'docker', image: 'docker', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.8.0', command: 'cat', ttyEnabled: true)
  ],
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
    persistentVolumeClaim(mountPath: '/home/jenkins/.m2', claimName: 'jenkins-pvc')
  ]) {
    node('jenkins-release-sj-jenkins-slave') {
        stage('Build') {
          git branch: env.GIT_BRANCH, url: env.GIT_URL
          sh './mvnw clean install'
        }
 
        stage('Image build') {
          container('docker') {
            sh 'docker build  . -t ' + env.DOCKER_REGISTRY+"/"+env.DOCKER_REGISTRY_NAMESPACE+"/"+env.DOCKER_IMAGE_NAME+":"+env.DOCKER_IMAGE_VERSION
            sh 'docker login ' + env.DOCKER_REGISTRY + ' -u ' + env.DOCKER_ID + ' -p ' + env.DOCKER_PW
            sh 'docker push ' + env.DOCKER_REGISTRY+"/"+env.DOCKER_REGISTRY_NAMESPACE+"/"+env.DOCKER_IMAGE_NAME+":"+env.DOCKER_IMAGE_VERSION
          }
        }
         
        stage('Deploy') {
            container('kubectl') {
                sh "kubectl apply -n " + env.DOCKER_REGISTRY_NAMESPACE + " -f " + env.K8S_CONF_PATH
            }
        }
  }
}
```
