# SWAN

SWAN (Service for Web based ANalysis) is a platform to perform interactive data analysis in the cloud.

Find more information in [the official website](https://swan.web.cern.ch/).

## Software Architecture

SWAN works on top of CERNBox and EOS.
The common architecture is discussed in [the CERNBox part](fss_cernbox.md).  

## Deployment

In Up2U, it is deployed in a Kubernetes (K8s) cluster,
by applying provided definitions (templates) of K8s objects.
These definitions as well as step by step deployment instructions
are available [here](https://github.com/sciencebox/kuboxed).

First, deploy CERNBox and EOS,
as discussed in [the CERNBox part](fss_cernbox.md).

Then, deploying SWAN is as simple as applying K8s objects
from `SWAN.yaml`.

## Integration with SSO

Example configuration for integrating with SSO via SAML
is available [here](https://github.com/up2university/SWAN-customizations).

Such a configuration can be provided to `SWAN.yaml`
by setting the following environmental variables for
the `jupyterhub` container within the `swan` K8s deployment:  

```
- name: CUSTOMIZATION_REPO
  value: "https://github.com/up2university/SWAN-customizations.git"
- name: CUSTOMIZATION_COMMIT
  value: "master"
- name: CUSTOMIZATION_SCRIPT
  value: ""
```
