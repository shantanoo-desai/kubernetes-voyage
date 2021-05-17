# mongo-example

MongoDB + Mongo-Express K8s

Browser - HTTP REQUEST -> ExternalService(Mongo-Express) -> Pod(Mongo-Express) -> InternalService(MongoDB) -> Pod(MongoDB)

## Pod Description
- MongoDB 4.0
- Root Username / Password comes from configured k8s Secret called `mongodb-secret`

> Secrets should be configured before Deployments


## Source

[Techworld with Nana YouTube Series | GitLab Repo](https://gitlab.com/nanuchi/youtube-tutorial-series)

## Bring the Cluster Up

```bash
$ kubectl apply -f .
```

### Check Secrets

```bash
$ kubectl get secrets/mongodb-secret
```

### External Service

- `LoadBalancer` does not work for Windows 10 Docker-Desktop and Kubernetes
- Use `NodePort` as a `Service` type and hit the IP Address of the Machine with the Port