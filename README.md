Vagrantfile
==========

This directory is primarily concerned with containing a single Vagrantfile that
can be used to reliably configure and re-configure a Vagrant environment set of
VMs.

Usage
=====

This entire repository can be checked out as your user's ~/.vagrant.d/
directory, and then the Vagrantfile will be merged with ones in your
development environment. Alternatively, if you would rather, you can download
just the Vagrantfile itself to your local ~/.vagrant.d/ folder.

For questions about using the functions herein, check the code in the
Vagrantfile itself.

With OpenStack
--------------

If you have the `vagrant-openstack-provider` plugin installed, then this Vagrantfile
will assume that you have a file at `${HOME}/.config/openstack/clouds.yaml` and attempt
to read it. This is the standard location for OpenStack client apps that use the Python
libraries to store their connection config file. Those values will be inserted into the
`vagrant-openstack-provider` variables as appropriate.

Unless otherwise specified the `default` cloud from clouds.yaml will be used. If you
want to use a different cloud set the value of the environment variable OS\_CLOUD to the
name of the cloud that should be used before invoking Vagrant.
