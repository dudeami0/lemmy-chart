# Lemmy Helm Chart

This chart is designed to make running Lemmy (https://join-lemmy.org/) easy on a kubernetes cluster.

## Example installation

To start, create the namespace you wish to install into, then create secrets for postgres credentials and pictrs keys:

```sh
kubectl create namespace lemmy
```

If `postgres.enabled` is set to `true` (default is `false`), we don't need to configure postgres credentials. Otherwise,
run the following after filling in each parameter (user, password, host, and db):

```sh
kubectl create secret -n lemmy generic my-release-lemmy-postgres-creds \
    --from-literal=user= \
    --from-literal=password= \
    --from-literal=host= \
    --from-literal=db=
```

This is an example `values.yaml` that I use to run my test instance. I assume the disk space is inadequate at the
moment, but lacking any definitive data requirements I have no better guess.

```yaml
configuration:
  host: lemmy.dudeami.win
  adminUsername: "dudeami0"
  enableTLS: true

lemmy-ui:
  config:
    externalHost: lemmy.dudeami.win:443
    https: true
    debug: false

pictrs:
  storage:
    kind: "persistentVolumeClaim"
    pvc:
      storageClassName: "ceph-filesystem"
      size: 16Gi

postgres:
  enabled: true
  storage:
    kind: "persistentVolumeClaim"
    pvc:
      storageClassName: "ceph-filesystem"
      size: 8Gi

ingress:
  enabled: true
  className: nginx
  hosts:
    - lemmy.dudeami.win
  tls:
  - secretName: lemmy-dudeami-win-cert
    hosts:
      - lemmy.dudeami.win
  annotations:
    cert-manager.io/cluster-issuer: cluster-issuer
```

## Bugs, help, etc

I'm new to lemmy so everything might not have been covered by this helm chart. I will actively help develop this
chart for various usages. I would like to eventually support horizontal scaling via the postgres operator and figuring
out a good way to horizontally scale lemmy instances.
