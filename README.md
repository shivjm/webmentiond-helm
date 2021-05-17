# webmentiond-helm
An opinionated Helm chart for running
[webmentiond](https://webmentiond.org/) on Kubernetes (optionally
using a secure [Traefik
IngressRoute](https://doc.traefik.io/traefik/routing/providers/kubernetes-crd/)).

## TL;DR

```text
$ git clone https://github.com/shivjm/webmentiond-helm/
$ cd webmentiond-helm
$ helm install --atomic my-release .
```

## Parameters

See the comments in [values.yaml](values.yaml).

## Persistence

webmentiond stores all its data in an SQLite file. To persist your
data between installs & upgrades (which is usually what you want),
you’ll need to create a
[PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
and specify its name in `config.persistence.existingClaim`.

Providing different volumes to different replicas would mean seeing
different data in each, which is why only a single volume claim is
used. If you increase the number of replicas from 1, make sure the
volume’s [access
mode](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes)
is `ReadWriteMany` so that it can be simultaneously mounted by all the
replicas. (This is not supported by some providers.)

## Security

By default, the pods drop all privileges and run as UID 1500. If your
requirements differ, you can override `podSecurityContext` and
`securityContext` as usual.
