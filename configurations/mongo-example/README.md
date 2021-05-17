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

Configure the an External Service in the deployment file with `type` set to `LoadBalancer` and `port` on which particular port you want to
make the service reachable

```yaml
spec:
    type: LoadBalancer
    ports:
    - port: 30000
      targetPort: 8081
```

For Docker-Desktop use `localhost` or your Machine's IP Address
