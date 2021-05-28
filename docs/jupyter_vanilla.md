# Jupyter

Jupyter is a software service for interactive computing and rich notebook editing.
See more in [the official website](https://jupyter.org/).

In Up2U, we deploy it in Kubernetes
using the [Zero to JupyterHub](https://zero-to-jupyterhub.readthedocs.io/) project.

## Software Architecture

The architecture of the Jupyter service consists of:

- JupyterHub application, run in a container (pod),
- JupyterHub's configurable HTTP proxy, run in a separate container (pod),
- a database for user metadata and sessions; by default, it is SQLite in the JupyterHub container (pod),
- single-user Jupyter notebook servers with JupyterLab user interface, each one in a separate container (pod),
- prePuller - a [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
downloading necessary Docker images to all Kubernetes nodes,
to speed up starting single-user pods. 

From the Kubernetes perspective, we have two pods for JupyterHub and the proxy
(assuming the SQLite database), and as many single-user pods as many users are currently active.

## Deployment

### Configuration

First, inspect [the default configuration file (values.yaml)](https://github.com/jupyterhub/zero-to-jupyterhub-k8s/blob/main/jupyterhub/values.yaml)
as well as [the configuration reference](https://zero-to-jupyterhub.readthedocs.io/en/stable/resources/reference.html)
to understand what you want to customize.
For testing, you can set no config values, so the service will run with default ones.

Some commonly-used config options are the following:

```
# values.yaml

proxy:
  # generate one with `openssl rand -hex 32`
  secretToken: 785e66bd19b956296fb751c96306fa23f4deaf21b999baa3353fc53d2c1695a3
  https:
    # assuming HTTPS is terminated by infrastructure provider
    enabled: false
    type: offload

hub:
  image:
    name: OUR-PROJECT/k8s-hub
    tag: v1.0
  templatePaths:
    # the custom templates added in the JupyterHub image
    - /opt/templates
  # max concurrent users
  activeServerLimit: 100
  # max users concurrently starting their containers 
  concurrentSpawnLimit: 10
  extraConfig: |-
    config = '/etc/jupyter/jupyter_notebook_config.py'
    c.Spawner.cmd = ['jupyter-labhub']

auth:
  type: custom
  custom:
    className: oauthenticator.generic.GenericOAuthenticator
    # provide configuration details depending on your SSO
    config:
      login_service: "SSO"
      client_id: *****
      client_secret: *****
      token_url: https://proxy.eduteams.org/OIDC/token
      userdata_url: https://proxy.eduteams.org/OIDC/userinfo
      userdata_method: GET
      userdata_params: {'state': 'state'}
      username_key: sub
      scope:
        - openid

prePuller:
  hook:
    enabled: true

singleuser:
  defaultUrl: "/lab"
  image:
    name: OUR-PROJECT/k8s-single-user
    tag: v1.0
  cpu:
    limit: 2
    guarantee: .5
  memory:
    limit: 2G
    guarantee: 1G
  storage:
    capacity: 10G

# disable if not auto-scaling the K8s cluster
scheduling:
  userScheduler:
    enabled: false

# destroy singleuser pods that are inactive for 30 minutes
cull:
  enabled: true
  timeout: 1800
  every: 180
```

### Theme and single-user customization

To customize theme, one need to extend the default Docker images
of JupyterHub and single-user, and reference the new images in `values.yaml`. 

Example JupyterHub image:

```
FROM jupyterhub/k8s-hub:0.9.0

# add our templates and CSS (referenced from templates)
COPY templates /opt/templates
COPY styles /usr/local/share/jupyterhub/static/css/custom
```

Read more about JupyterHub templates [here](https://jupyterhub.readthedocs.io/en/stable/reference/templates.html).

Example single-user image:

```
FROM jupyterhub/k8s-singleuser-sample:0.9.0
ENV JUPYTER_ENABLE_LAB true

# install necessary system utils
USER root
RUN apt-get update && \
    apt-get install -y \
        zip \
        unzip \
        && rm -rf /var/lib/apt/lists/*

# install theme
COPY theme /opt/theme
RUN chown -R $NB_USER: /opt/theme
USER $NB_USER
RUN jupyter labextension install /opt/theme 
```

The actual theme for this image should be prepared as [JupyterLab extension](https://jupyterlab.readthedocs.io/en/2.2.x/user/extensions.html). 

### Actual deployment

Using [helm v3](https://helm.sh/), deploy the Jupyter service:

```
helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
helm repo update
helm upgrade --cleanup-on-fail \
  --install jhub jupyterhub/jupyterhub \
  --version=0.9.0 \
  --values config.yaml
```

Find out more in [the official documentation](https://zero-to-jupyterhub.readthedocs.io/en/stable/jupyterhub/installation.html).

### Get admin rights

To grant admin rights for the first time, do something like the following:

```
kubectl exec -it hub-77d75ff89-8sj59 sqlite3 jupyterhub.sqlite "update users set admin=1 where name='{{ username|quote }}'; select name from users where admin<>0;"

# if JupyterHub does not see the change, we need to restart it:
kubectl delete pod hub-77d75ff89-8sj59
```

Next, the newly-granted admin can grant admin rights to others via UI.

## Scaling up

At the time of writing:

> JupyterHub isn't designed to support being run in parallell.
> More work needs to be done in JupyterHub itself for a fully highly available (HA)
> deployment of JupyterHub on k8s is to be possible.
> [[source]](https://github.com/jupyterhub/zero-to-jupyterhub-k8s/blob/main/jupyterhub/values.yaml)

Thus, there is no easy way to horizontally scale up either JupyterHub application or the proxy.
The only way of achieving a larger scale is to provide more resources to these pods.

Please note that users send HTTP requests to JupyterHub application only when authenticating
and when starting up (spawning) their single-user pods.
Afterwards, users send requests only to their single-user pods,
so the JupyterHub pod is not so loaded.

Note that all users' HTTP requests always go through the HTTP proxy.

## Persistent data

The critical persistent storage is Kubernetes volumes mounted to single-user pods.
They hold user files.

Another critical storage in the database behind JupyterHub.
By default, it is a SQLite file database in a volume mounted to the JupyterHub pod. 
