# Nextcloud

Nextcloud is a server application allowing to storing, sharing, online editing
of documents and any other data one would like to backup or share with other
people. It provides easy, intuitive user interface and allows to ease share
data with other Nextcloud users as well as anonymous. 

Nextcloud is constantly being improved obtaining frequent updates. 

## Software Architecture

Nextcloud is designed to store data for multiple users.

It can utilize a
variety of storage devices to fulfil that purpose, including common disk and
Amazon S3 object storage.

Additionally it utilizes common relational database
(like SQLite, PostgreSQL or MariaDB) in order to store users and files
metadata.

In order to speed up the application, Redis storage can be utilized. 

## Deployment

Kubernetes provides great mechanisms for facilitating and speeding up
the deployment of many applications. Nextcloud is among them. Nextcloud's
programmers provide Helm chart designed to easily deploy Nextcloud in
Kubernetes platforms. The whole documentation is presented
[here](https://github.com/nextcloud/helm/tree/master/charts/nextcloud).

Summarizing the installation:

1. Install helm
2. Add Nextcloud repo:
    ```
    helm repo add nextcloud https://nextcloud.github.io/helm/
    ```
3. Install Nextcloud by running:
    ```
    helm install --name my-release -f values.yaml nextcloud/nextcloud
    ```
4. An example `values.yaml` file might look like this:
```yaml
## Official nextcloud image version
## ref: https://hub.docker.com/r/library/nextcloud/tags/
##
image:
  repository: nextcloud
  tag: 19.0.10-apache
  pullPolicy: IfNotPresent

nameOverride: ""
fullnameOverride: ""

# Number of replicas to be deployed
replicaCount: 1

## Allowing use of ingress controllers
## ref: https://kubernetes.io/docs/concepts/services-networking/ingress/
##
ingress:
  enabled: true
  annotations: 
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
  labels: {}


lifecycle: {}

nextcloud:
  host: drive2.test.up2u.eu
  username: admin
  password: "TODOsecretadminpassword"
  existingSecret:
    enabled: false
  update: 0
  datadir: /var/www/html/data
  persistence:
    subPath:
  mail:
    enabled: false
  phpConfigs: {}
  defaultConfigs:
    .htaccess: true
    redis.config.php: true
    apache-pretty-urls.config.php: true
    apcu.config.php: true
    apps.config.php: true
    autoconfig.php: true
    smtp.config.php: true
  configs: {}

  ## Strategy used to replace old pods
  ## IMPORTANT: use with care, it is suggested to leave as that for upgrade purposes
  ## ref: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#strategy
  strategy:
    type: Recreate
    # type: RollingUpdate
    # rollingUpdate:
    #   maxSurge: 1
    #   maxUnavailable: 0

## MariaDB chart configuration
##
mariadb:
  ## Whether to deploy a mariadb server to satisfy the applications database requirements. To use an external database set this to false and configure the externalDatabase parameters
  enabled: true

  db:
    name: nextcloud
    user: nextcloud
    password: "TODOsecretdbpassword"

  replication:
    enabled: false

  ## Enable persistence using Persistent Volume Claims
  ## ref: http://kubernetes.io/docs/user-guide/persistent-volumes/
  ##
  master:
    persistence:
      enabled: true
      storageClass: "local-path"
      accessMode: ReadWriteOnce
      size: 2Gi

redis:
  enabled: true
  usePassword: true
  password: 'TODOsecretpassword'

##it still belong to redis:
global:
  storageClass: "local-path"


service:
  type: ClusterIP
  port: 8080
  loadBalancerIP: nil
  nodePort: nil

## Enable persistence using Persistent Volume Claims
## ref: http://kubernetes.io/docs/user-guide/persistent-volumes/
##
persistence:
  enabled: true
  annotations: {}
  storageClass: "local-path"

  accessMode: ReadWriteOnce
  size: 8Gi

## Liveness and readiness probe values
## Ref: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes
##
livenessProbe:
  enabled: true
  initialDelaySeconds: 30
  periodSeconds: 30
  timeoutSeconds: 5
  failureThreshold: 3
  successThreshold: 1
readinessProbe:
  enabled: true
  initialDelaySeconds: 0
  periodSeconds: 30
  timeoutSeconds: 5
  failureThreshold: 3
  successThreshold: 1
startupProbe:
  enabled: true
  initialDelaySeconds: 10
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 180
  successThreshold: 1
```
Lookup for "TODO" within the proposed configuration above, to change the passwords.

## Integration with SSO

In order to integrate with external SSO, one need to install [social
login](https://apps.nextcloud.com/apps/sociallogin) application. Defining the
used authorization protocol is fairly easy and it can be done under the "Social login"
configuration. It is important to choose proper settings on the top of the
page, since they influence how the logging procedure is done.

The OpenID Connect Scope to be configured will usually be: `openid profile email`

Redirect URL to be registered in SSO will be `https://nextcloud.example.com/apps/sociallogin/custom_oidc/sso`,
assuming `sso` is the "Internal name" given to the identity provider in the "Social login" configuration.

## Scaling up

Unfortunatelly there is no easy way to scale up nextcloud. There are, however,
some guidlines, like [fine-tuning the installation
itself](https://docs.nextcloud.com/server/stable/admin_manual/installation/server_tuning.html),
or guides on how to deploy Nexctcloud in higher scale:
[[1]](https://indico.cern.ch/event/663264/contributions/2818170/attachments/1592445/2520694/An_insiders_look_into_scaling_Nextcloud_-_Matthias_Wobben.pdf)
and [[2]](https://faun.pub/nextcloud-scale-out-using-kubernetes-93c9cac9e493).
Nevertheless, each of them require fine-tuining and adjusting to the newest
version of Nextcloud.

## Notes

For backuping, [Velero](https://velero.io/) does a great job.

Basic styling can be done via Theming application. Official
[documentation](https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/theming.html)
covers the provided functionality.

