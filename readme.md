# kloud-konnect-litmuschaos-demo

This is a workshop to install LitmusChaos & execute a chaos scenario on a simple web application 

## Steps: 

1. Spin up your favourite Kubernetes cluster (ex: [killercoda](https://killercoda.com/playgrounds/scenario/kubernetes))

2.  Verify the presence of a default storage class. If there are none, set up the Rancher dynamic Local PV provisioner & storage class. And then make it the default

    ```
    kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
    kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
    ```

3. Setup the sample web-application

   ```
   kubectl apply -f https://raw.githubusercontent.com/ksatchit/kubesimplify-chaos-demo/main/podtato-head-web-app.yaml
   ```
   
   Open port access to the application and view status
## Installation steps for Litmus 3.0.0

### Mongo installation via Helm - Bitnami Mongo

```shell
helm repo add bitnami https://charts.bitnami.com/bitnami
```

#### Mongo Values

```shell
cat > mongo-values.yml <<EOL
auth:
  enabled: true
  rootPassword: "1234"
  # -- existingSecret Existing secret with MongoDB(&reg;) credentials (keys: `mongodb-passwords`, `mongodb-root-password`, `mongodb-metrics-password`, ` mongodb-replica-set-key`)
  existingSecret: ""
architecture: replicaset
replicaCount: 3
persistence:
  enabled: true
volumePermissions:
  enabled: true
metrics:
  enabled: false
  prometheusRule:
    enabled: false

# bitnami/mongodb is not yet supported on ARM.
# Using unofficial tools to build bitnami/mongodb (arm64 support)
# more info: https://github.com/ZCube/bitnami-compat
#image:
#  registry: ghcr.io/zcube
#  repository: bitnami-compat/mongodb
#  tag: 6.0.5
EOL

```

```shell
helm install my-release bitnami/mongodb --values mongo-values.yml -n litmus --create-namespace
```

### Apply the Manifest

Applying the manifest file will install all the required service account configuration and ChaosCenter in cluster scope.

```shell
kubectl apply -f https://raw.githubusercontent.com/litmuschaos/litmus/master/mkdocs/docs/3.0.0/litmus-cluster-scope-3.0.0.yaml
```
   
Construct the chaos scenario to kill the helloservice replica with a http probe to verify availability. Validate hypothesis & analyze results 

Re-run the chaos scenario with mitigation to check whether the desired resilience characteristics are achieved. 


