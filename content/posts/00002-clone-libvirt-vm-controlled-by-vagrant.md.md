---
title: "How to clone a libvirt VM controlled by Vagrant"
date: 2018-08-30T10:16:00-05:00
draft: false
---
Strangely enough, googling about this problem doesn't produce any
meaningful results. So here are steps which allow cloning a libvirt VM
controlled by Vagrant. Imagine that you have a VM named `testvm1`
which you want to clone into `testvm2`.

* Shutdown all VMs with `vagrant halt` or at least from inside of VM
  with `sudo shutdown -h 0`.
* Figure our real VM name with `virsh list --all | grep
  testvm1`. Usually Vagrant adds its own `vagrant_` prefix to VM
  names. If you don't like it, you can rename VM with `virsh
  domrename` command, Vagrant will still know it because Vagrant
  refers to VMs by UUID instead of a name.
* Clone VM image with `virt-clone --original testvm1 --name testvm2
  --auto-clone`. You can specify additinal options like new image file
  name instead of `--auto-clone`.
* Edit your Vagrantfile to produce a configuration for new `testvm2`
  VM. I have an array of VM names which I loop over like this:
```ruby
      names_array = ['testvm1', 'testvm2']
    # .....
      (0..vm_total_number - 1).each do |i|
        vm_name = names_array[i]
        config.vm.define "#{vm_name}" do |node|
          node.vm.hostname = "#{vm_name}"
    # .......
        end
      end
```
* Find new VM UUID with `virsh dumpxml testvm2 | grep uuid`.
* Copy directory `.vagrant/machines/testvm1`
  to`.vagrant/machines/testvm2`.
* Edit file `.vagrant/machines/testvm2/libvirt/id` and put new UUID
  there.
* Edit file `.vagrant/machines/testvm2/libvirt/action_provision` and
  replace old UUID with new one there as well.
* Create a new 32-digital HEX identifier, e.g. with `dd count=1
  if=/dev/random 2> /dev/null | md5sum`.
* Write this ideintfier to file `index_uuid` instead of an old one.
* Edit file `~/.vagrant.d/data/machine-index/index`. It is in JSON
  format. You need to copy a block beginning with an old contents of
  `index_uuid` and all that follows it in curly brackets `{}`. Be
  careful and make a file backup because it is very easy to break JSON
  syntax. Now in new VM block change the VM id to newly generated and
  change its host name for `"name":` tag.
* Check with `vagrant status` that you have a new VM in shutdown
  state.
* You are ready to start a VM with `vagrant up` command.
