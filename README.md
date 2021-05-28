# webmentiond

![Version: 0.1.0](https://img.shields.io/badge/Version-0.1.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: v1.0.0](https://img.shields.io/badge/AppVersion-v1.0.0-informational?style=flat-square)

An opinionated Helm chart for running
[webmentiond](https://webmentiond.org/) (A simple webmention receiver in Go) on
Kubernetes (optionally using a secure [Traefik
IngressRoute](https://doc.traefik.io/traefik/routing/providers/kubernetes-crd/)).

**Homepage:** <https://github.com/shivjm/webmentiond-helm/>

## TL;DR

```text
$ git clone https://github.com/shivjm/webmentiond-helm/
$ cd webmentiond-helm
$ helm install --atomic my-release .
```

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| Shiv Jha-Mathur | shivjm@users.noreply.github.com |  |

## Source Code

* <https://github.com/shivjm/webmentiond-helm/>
* <https://github.com/zerok/webmentiond/>

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| affinity | object | `{}` |  |
| config.allowedTargetDomains | string | `nil` | space-separated list of domains for which server accepts Webmentions (required) |
| config.auth.adminEmails | string | `nil` | space-separated list of email addresses that can be used to log in (required) |
| config.auth.secretRef | string | `"webmentiond-jwt"` | name of secret containing `secret` key (used to encrypt JWT in admin interface) |
| config.mail.disableTls | bool | `false` | whether to disable TLS when connecting to the mail server |
| config.mail.from | string | `nil` | address to put in `From:` line of emails sent by server (required) |
| config.mail.host | string  | `nil` | ail server host (required) |
| config.mail.port | string | `"587"` | port to connect to mail server on (required) |
| config.mail.secretRef | string | `"webmentiond"` | name of secret containing `user` & `password` for mail server (set to "" to disable authentication) |
| config.public.host | string | `nil` | publicly accessible hostname, e.g. webmentions.example.com (required) |
| config.public.path | string | `nil` | publicly accessible path without leading slash, e.g. `admin` |
| config.sendNotifications | bool | `false` | whether to send email notifications |
| fullnameOverride | string | `""` |  |
| image.pullPolicy | string | `"IfNotPresent"` |  |
| image.repository | string | `"zerok/webmentiond"` |  |
| image.tag | string | `""` |  |
| imagePullSecrets | list | `[]` |  |
| ingress.annotations | object | `{}` | annotations to add to all ingress resources created |
| ingress.enabled | bool | `false` | whether to create Traefik IngressRoute using hostname and path set in config.public |
| ingress.middleware | list | `[]` | custom middleware to add |
| ingress.tls.secretRef | string | `nil` | name of secret containing TLS details (required if ingress is enabled) |
| nameOverride | string | `""` |  |
| nodeSelector | object | `{}` |  |
| persistence.enabled | bool | `true` | whether to enable persistence (will use a temporary volume if not) |
| persistence.existingClaim | string | `nil` | name of existing claim to use (required if persistence is enabled) |
| podAnnotations | object | `{}` | annotations to add to all pods created |
| podSecurityContext | object | `{"fsGroup":1500,"fsGroupChangePolicy":"OnRootMismatch"}` | security context to apply to all pods created |
| prometheus.annotations | bool | `false` | whether to add annotations to scrape Prometheus metrics |
| prometheus.serviceMonitor.annotations | object | `{}` | annotations to apply to ServiceMonitor |
| prometheus.serviceMonitor.enabled | bool | `false` | whether to create ServiceMonitor from Prometheus Operator |
| prometheus.serviceMonitor.interval | string | `nil` | how frequently Prometheus should scrape service |
| prometheus.serviceMonitor.labels | object | `{}` | labels to apply to ServiceMonitor |
| prometheus.serviceMonitor.namespace | string | `""` | namespace to create ServiceMonitor in |
| prometheus.serviceMonitor.relabelings | list | `[]` | MetricRelabelConfigs to apply to samples before ingestion |
| prometheus.serviceMonitor.scrapeTimeout | string | `nil` | time after which scrape is ended |
| replicaCount | int | `1` | how many instances of webmentiond to run (see note about persistence in README) |
| resources | object | `{}` |  |
| securityContext | object | `{"capabilities":{"drop":["ALL"]},"readOnlyRootFilesystem":true,"runAsUser":1500}` | security context to apply to all containers created |
| service.enabled | bool | `true` | whether to create Service |
| service.port | int | `80` | port on which Service should listen |
| service.type | string | `"ClusterIP"` | type of Service to create |
| tolerations | list | `[]` |  |

### About `config.public`

`public.host` and `public.path` are required by the admin interface. For a server accessible at <kbd>https://www.example.com/webmentions/</kbd>,
you would use these values:

```yaml
  public:
    host: "www.example.com"
    path: "webmentions/"
```

## Persistence

webmentiond stores all its data in an SQLite file. To preserve your
data between installs & upgrades (which is usually what you want),
you’ll need to create a
[PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
and specify its name in `config.persistence.existingClaim`.

Providing different volumes to different replicas would mean seeing
different data in each, which is why only a single volume claim is
allowed. If you increase the number of replicas from 1, make sure the
volume’s [access
mode](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes)
is `ReadWriteMany` so it can be simultaneously mounted by all the
replicas. (This is not supported by some providers.)

## Security

By default, the pods drop all privileges and run as UID 1500. If your
requirements differ, you can override `podSecurityContext` and
`securityContext` as usual.
