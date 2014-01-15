Deployment
==========

This document provides instructions for deploying stucco. To set up the Vagrant development/testing VM, see https://github.com/stucco/dev-setup.

### Set up Storm

To be able to push code out to a storm cluster from your host OS, you will need to download the storm project:

    cd /usr/local
    sudo curl --silent -L https://dl.dropboxusercontent.com/s/tqdpoif32gufapo/storm-${VERSION}.tar.gz -o storm.tgz
    sudo tar xzfv storm.tgz
    sudo ln -s ../storm-${VERSION}/bin/storm bin/storm
    sudo rm -f storm.tgz
    sudo chmod +x bin/storm


### Deploy to Storm Cluster

To package a project into a jar file for deployment to a cluster, you can run `sbt assembly` and submit the jar file using `storm jar /path/to/archive.jar class.of.Topology [args...]`.

More information about submitting to a storm cluster can be found in the [storm command line client documentation](https://github.com/nathanmarz/storm/wiki/Command-line-client).


### Openstack

* run several rt instances in docker containers, on a [one-node openstack deployment](openstack_dev_single.md)
* run an [openstack deployment with several nodes](openstack_dev_cluster.md)