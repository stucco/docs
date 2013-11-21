stucco documentation
====================

## Architecture

* [Architecture (v1)](./docs/arch-v1.md)
* [List of projects](./docs/repositories.md)

## Alignment

Alignment refers to the process of merging new data into the knowledge graph. The [alignment document](./docs/alignment.md) describes the overall approach to alignment. The [metrics document](./docs/metrics.md) explains the different scores that may influence the alignment process.

## Deployment

Several ways to deploy and run stucco:

* run one rt instance on a virtual machine, described in the [dev setup](https://github.com/stucco/dev-setup) repo  
* run several rt instances in docker containers, on a [one-node openstack deployment](./docs/openstack_dev_single.md)
* run an [openstack deployment with several nodes](./docs/openstack_dev_cluster.md)