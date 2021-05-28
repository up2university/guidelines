# Single Sign-On

Keycloak is designed to deal with users authentication process as an Identity and Access Management module.

It takes care of every element for authenticating user with prepared login form for different applications as Single-Sign On, storing information about users and theirs attributes as well as the provided logout operation as Single-Sign Off for all integrated applications.
From other side it allows integrating user federations (using built in support for connect to existing LDAP or Active Directory servers) or to implement social login and can also authenticate users with existing OpenID Connect or SAML 2.0 Identity Providers.

## Software Architecture

The Keycloak implementation of Single Sign-On module works as a service which uses separate database for storing executive data.

It comes with its own embedded Java-based relational database called H2 however it uses two layered technologies to persist its relational data using JDBC as bottom layer and Hibernate JPA as a top layer.
As a bottom layer Keycloak is able to use one of these vendor-specific database drivers as a JDBC connection layer.

Further in this document, the PostgreSQL database will be configured as an example.

## Deployment

The deployment of full working SSO module consist of the sugested Keycloak module and database service for data persistance. 

### Requirements

The system requirements of the Keycloak module are well described in the documentation [Server installation - System Requirements](https://www.keycloak.org/docs/latest/server_installation/#system-requirements).

### Themes 

Up2U themes for SSO should be located in the `themes` folder. 
In order to extend please follow [Keycloak themes configuration](https://www.keycloak.org/docs/latest/server_development/#_themes).

### Extensions

If there is a need of adding user defined extensions for Keycloak their may be located in the `extensions` folder and consists of set of classes and descriptors adding new functionality.
Read more in [Extending the Server](https://www.keycloak.org/docs/latest/server_development/#_extensions).

### Building Keycloak image

#### Configure ssl for internal communication 

Needed:

* SSO domain name
* Certificate key and keystore

Follow the section of the Keycloak documentation for appropriate configuration:
[Enabling SSL/HTTPS for the Keycloak Server](https://www.keycloak.org/docs/latest/server_installation/#_setting_up_ssl).

#### Configure other options

Needed:

* `certificate.pem` (from the step above)
* SSO domain name
* Username for the SSO administrator
* Password for the SSO administrator
* Password for the certificate key and keystore
* RSA keypair and certificate (encoded in PEM format)
* optional: OIDC Client Id and Client Secret obtained from Google
* optional: OIDC Client Id and Client Secret obtained from Facebook
* optional: OIDC Client Id and Client Secret obtained from Github
* optional: reCAPTCHA site key and secret obtained from Google

Populate `pom.xml:/project/profiles/profile/properties` for the selected target environment profile
(match with `/project/profiles/profile/id`) with the above configuration details.

#### Keycloak configuration files

Those configuration files are used for define Keycloak server behaviour in runtime.
This is the place for pointing appropriate keystore file, set password for it,
define the default theme used in user interface, or set cache time values for themes, and many more. 

- `standalone.xml` - see [Keycloak Standalone Mode](https://www.keycloak.org/docs/latest/server_installation/#standalone-configuration)
- `standalone-ha.xml` - see [Keycloak Standalone Clustered Mode](https://www.keycloak.org/docs/latest/server_installation/#_standalone-ha-mode)

#### Build Docker image

Secrets should be located in `secrets` folder containing:

```
certificate.pem
keycloak.jks
```

Example figure of the Dockerfile:

```
FROM jboss/keycloak:13.0.0

RUN echo "Building custom Kecycloak SSO module."

USER root

# Add keystore
ADD secrets/keycloak.jks /opt/jboss/keycloak/standalone/configuration/keycloak.jks
RUN chown jboss:root /opt/jboss/keycloak/standalone/configuration/keycloak.jks
RUN chmod 660 /opt/jboss/keycloak/standalone/configuration/keycloak.jks

# Add Keycloak configuration
ADD configuration/standalone-ha.xml /opt/jboss/keycloak/standalone/configuration/standalone-ha.xml
RUN chown jboss:root /opt/jboss/keycloak/standalone/configuration/standalone-ha.xml
RUN chmod 660 /opt/jboss/keycloak/standalone/configuration/standalone-ha.xml
ADD configuration/standalone.xml /opt/jboss/keycloak/standalone/configuration/standalone.xml
RUN chown jboss:root /opt/jboss/keycloak/standalone/configuration/standalone.xml
RUN chmod 660 /opt/jboss/keycloak/standalone/configuration/standalone.xml

# Install find
RUN microdnf install findutils

# Add Keycloak themes
ADD themes /opt/jboss/keycloak/themes/
RUN find /opt/jboss/keycloak/themes/ -type d -exec chmod 755 {} \;
RUN chown -R jboss:root /opt/jboss/keycloak/themes/

USER 1000

EXPOSE 8443 8080

ENTRYPOINT [ "/opt/jboss/tools/docker-entrypoint.sh" ]
```

Following environment variables with secrets should be set:

```
REGISTRY_DOMAIN
IMAGE_TAG
```

Example of build custom image script (the project name and image name are examples):

```shell
#!/bin/sh
docker build --tag=${REGISTRY_DOMAIN}/up2u/keycloak-sso:${IMAGE_TAG} .
```

#### Push Docker image to the image registry

The following steps applies to the public image registry.
If you push the image to some other registry, please consult its documentation.

Needed:

* Image registry domain name
* Image tag (check logs from the *Build project and docker image* step)

To perform and publish docker image built in the previous step it is enough to proceed (the project name and image name as previously defined):
```shell
docker push ${REGISTRY_DOMAIN}/up2u/keycloak-sso:${IMAGE_TAG}
```

### Deployment parameters for helm charts

Example of `values.yaml` for parametrizing of the Keycloak helm chart:

```yaml
## Official keycloak image version
## ref: https://hub.docker.com/r/jboss/keycloak/tags/
## to use the custom user built image based on official one 
## please modify repository parameter (with project and image name) and tag settings
##
image:
  repository: docker.io/jboss/keycloak
  tag: 13.0.1
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

keycloak:
  host: sso.test.up2u.eu
  username: admin
  password: "TODOsecretadminpassword"
  persistence:
    subPath:
  mail:
    enabled: false

  ## Strategy used to replace old pods
  ## IMPORTANT: use with care, it is suggested to leave as that for upgrade purposes
  ## ref: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#strategy
  strategy:
    type: Recreate
    # type: RollingUpdate
    # rollingUpdate:
    #   maxSurge: 1
    #   maxUnavailable: 0

## PostgresDB chart configuration
##
postgresql:
  # Disable PostgreSQL dependency
  resources:
    requests:
      memory: "0"
      cpu: "0"
  persistence:
    enabled: true
    storageClass: "local-path"
    accessMode: ReadWriteOnce
    size: 2Gi

  replication:
    enabled: false

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
  size: 2Gi

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

extraEnv: |
  - name: DB_VENDOR
    value: postgres
  - name: DB_ADDR
    value: up2u-postgresql.up2u-sso.svc.cluster.local
  - name: DB_PORT
    value: "5432"
  - name: DB_DATABASE
    value: <DATABASE_NAME>
  - name: DB_USER
    value: <DATABASE_USER>
  - name: DB_PASSWORD
    value: <DATABASE_PASSWORD>
```

Please refer to the `{REGISTRY_DOMAIN}/up2u/keycloak-sso:{$IMAGE_TAG}` image,
if it was created, or use the default one.

### Deployment procedure

Sequence:
1. Login to the K8s.
2. Perform `helm install` operation:
   `helm install --name up2u-test-sso codecentric/keycloak -n up2u-sso -f values.yaml`
3. Login to the keycloak administrator console 
(`https://<SSO_DOMAIN>/auth/admin/master/console/#/realms/master`)
and finish setup by setting themes, internationalization options, and hardening master realm.
4. It is strongly recommended enabling 2FA for the administrator account.

## Scaling up

The best possibilities are described in the Keycloak documentation
[Server installation - Clustering](https://www.keycloak.org/docs/latest/server_installation/#_clustering).

It is also worth to consider using official Keycloak Operator for Kubernetes
[Keycloak Operator on Kubernetes](https://www.keycloak.org/getting-started/getting-started-operator-kubernetes). 

## Notes

The backup procedure should be performed by exporting database content.
 
It is also possible to perform logical exports from the Keycloak user interface,
e.g. by exporting realms or clients configuration.
Those exports cause loosing any private passwords, secrets, etc.
So, it is important to keep in mind that storing those are at the discretion of the administrators.
