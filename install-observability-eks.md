# Install Observability EKS

## Install ADOT Add-on

### Add permissions 

```
    kubectl apply -f https://amazon-eks.s3.amazonaws.com/docs/addons-otel-permissions.yaml
```


### Install cert manager

```
    kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.4/cert-manager.yaml
```

### Install Addon
    Replace with eks clustername

```
    aws eks create-addon --addon-name adot --addon-version v0.102.1-eksbuild.1 --cluster-name {clustername}
```

    Verify add-on

```
    aws eks describe-addon --addon-name adot --cluster-name {clustername} | jq .addon.status
```

    Verify operator

```
    kubectl -n opentelemetry-operator-system get pod
```

    Create an `adot-collector` namespace
```
    kubectl create ns adot-collector
```

## Collect Traces to AWS X-Ray
    This one is setting up to collect Traces on X-ray. Create IAM Role

```
    eksctl create iamserviceaccount \
    --name adot-obo-trace-xray \
    --namespace adot-collector \
    --cluster {clustername}  \
    --attach-policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy \
    --attach-policy-arn arn:aws:iam::aws:policy/AWSXrayWriteOnlyAccess \
    --approve \
    --region ${AWS_REGION}
```
    Create an `OpenTelemetryCollector CR` manifest file of type `Deployment` from `adot-obo-trace-xray-collector.yaml`. Create an OpenTelemetryCollector CR. Update namespace and others if needed.

```
    kubectl apply -f adot-obo-trace-xray-collector.yaml
```

    Create an `Instrumentation CR` manifest file, replace to each namespace which have pods deployed.

```
    kubectl apply -f adot-obo-trace-xray-instrumentation.yaml
```

    Get deployments in the list to add instrumentation

```
    kubectl get deployment --all-namespaces
```

    And apply

```
    kubectl -n app patch deployment {deployment-name}} -p '{"spec":{"template":{"metadata":{"annotations":{"instrumentation.opentelemetry.io/inject-python": "true"}}}}}'
    kubectl -n app rollout restart deployment {deployment-name}
```
