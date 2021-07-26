# Interoperability between Jupyter and FSS

Interoperability between the FSS and Jupyter services
results in users having access to their FSS files also in Jupyter.
Such an integration is discussed here.

If you choose [CERNBox](../fss_cernbox.md) and [SWAN](../jupyter_swan.md),
they are integrated out of the box, and no further steps are needed.

On the other hand,
[Nextcloud](../fss_nextcloud.md) and [Vanilla Jupyter](../jupyter_vanilla.md)
can be integrated using [those extensions](https://github.com/sciencemesh/filesystem-for-jupyter).
More concretely, JupyterHub can be extended with
[the extended KubeSpawner](https://github.com/sciencemesh/filesystem-for-jupyter/tree/master/authorization/login-flow-v2)
that:

1. implements [Nextcloud's Login flow v2](https://docs.nextcloud.com/server/latest/developer_manual/client_apis/LoginFlow/index.html#login-flow-v2)
to obtain Nextcloud credentials for Jupyter,
2. for each user, runs extra container
[synchronizing user](https://github.com/sciencemesh/filesystem-for-jupyter/tree/master/sync) files with Nextcloud.

An example configuration can be found [here](https://github.com/sciencemesh/filesystem-for-jupyter/tree/master/examples/z2jh-sync).
