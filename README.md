# Create PVC

```
kubectl create -f jenkins-pvc.yaml
```

# Deploy Helm Chart

```
helm install --name jenkins \
-f values.yaml \
--set Master.Cpu="1",Master.Memory="1Gi",Master.HostName="jenkins.ghama.io",Agent.Cpu="1",Agent.Memory="1Gi",Persistence.Enabled=true,Persistence.ExistingClaim="jenkins-pvc" \
stable/jenkins
```
