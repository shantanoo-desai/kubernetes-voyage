# Kubernetes Configuration File

**Kubernetes Objects** are persistent entities to represent state of your cluster

**Kubernetes Objects** describe:

1. what containerized application is running on which Nodes
2. Resource Availability
3. Policies for Behaviour: restart, upgrades, fault-tolerance

## Object Spec and Status

Almost every k8s object include two nested object fields: `spec` and `status`

- `spec`: you have to set this when you create the object, providing a description
   of characteristics you want the resource to have (DESIRED State)
- `status`: describes the CURRENT State of object. Updated by Kubernetes system
  and components. Control Plane manages actively the actual state to desired state. Comes from **etcd**

### Describing Objects

- Describing `spec` is important in a file
- k8s API takes requests in JSON format, `kubetctl` converts YAML -> JSON


### Requird Fields in Configuration Files

- `apiVersion` - which version of Kubernetes API you are using to create this object
- `kind` - what kind of k8s object you want to create: `Pod`, `Service`, `Deployment`..
- `metadata` - data to uniquely identify object with `name`, `namespace`, `UID`
- `spec` - Desired state of your object

### Kubernetes API Docs

[k8s API Documentation](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/)


### Example Configuration

_Deployment_

```yaml
apiVersion: apps/v1 # Different Objects have different API Versions
kind: Deployment # can be `Service`, `Pod`, `ReplicaSets`....
metadata:
    name: nginx-depl # name of the object
    namespace: test-ns # name of namespace
spec:
    # describe what you want your cluster to look like below
    selector:
        matchLabels:
            app: nginx
    replicas: 2 # how many pods matching the template
    template: # Your Pod description goes below
        metadata:
            labels:
                app: nginx # for the `spec.selector`
        spec:
            containers:
                - name: nginx
                  image: nginx:1.16
                  ports:
                  - containerPorts: 8080
```

_Service_

```yaml
apiVersion: v1
kind: Service
metadata:
    name: nginx-service
spec:
    selector:
        app: nginx # the label for deployment
    ports:
        - protocol: TCP
          port: 80 # Port this particular Service is exposed
          targetPort: 8080 # Port of the pod mentioned in `containerPorts`
```

- `metadata` contains the Labels and `spec` contains the selectors for these labels
- `Deployment` `metadata.labels` is used by `Service` to connect to each other

## Best Practices

- Written in YAML/JSON file. Typically in YAML because of better user-friendly syntax
- Try writing all configurations into a single file (whenever possible)
- Try adding versioning to configuration files (for easy roll-back)
- Write **Deployment** configuration rather than _naked_ Pod configuration
    - **Deployment** handles **ReplicaSets** and provides Rolling Updates
- Write **Service** Configuration BEFORE **Deployment**

### Labels

Use `label` to add semantic attributes and also easily select for troubleshooting

```yaml
labels:
  app: myapp
  tier: frontend
  phase: test
  deployment: v1
```
[Kubernetes Example](https://github.com/kubernetes/examples/guestbook)

### Container Images

- avoid using `latest` tag in deployment configuration for mismatch errors


