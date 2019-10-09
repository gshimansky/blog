---
title: "Vagrant Libvirt Plugin Build on Ubuntu"
date: 2019-02-26T09:45:10-06:00
draft: false
---
On modern Ubuntu versions vagrant plugin vagrant-plugin fails to build. It is
necessary to specify correct locations of libvirt headers and librarires.
Possible solution can be found
[here](https://github.com/hashicorp/vagrant/issues/7039#issuecomment-403653620):
```
CONFIGURE_ARGS="with-libvirt-include=/usr/include/libvirt with-libvirt-lib=/usr/lib64" vagrant plugin install vagrant-libvirt
Installing the 'vagrant-libvirt' plugin. This can take a few minutes...
Building native extensions.  This could take a while...
Fetching: fog-libvirt-0.5.0.gem (100%)
Fetching: vagrant-libvirt-0.0.43.gem (100%)
Installed the plugin 'vagrant-libvirt (0.0.43)'!
```
