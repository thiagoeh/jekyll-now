---
layout: post
title: Using linked clones in Vagrant VMs 
---
TL;DR: Making Vagrant automatically create linked clones is easy as adding this line to the VirtualBox provider configuration: `vb.linked_clone = true`

Just after being confortable creating VMs using Vagrant, I started to worry about how to keep the storage usage in an acceptable level.

My computer have a single smallish SSD, so the default of duplicating the virtual disks for each VM every time a created a new one wouldn't word for me.

If I weren't using Vagrant for managing VMs, I could simply create a template and additional linked clones VMs.

A linked clone VM work a similar way to a snapshot, where there is a reference virtual disk (from a template VM), and additional virtual disks that store just the changes.

The difference between a snapshot is that the in a linkedin clone VM the metadata (machine name, hardware configuration, etc) isn't the same. That way you can run the linked clone VM as a separate from the template, and even have multiple linked cloned VMs from the same template, running simultaneously.

The storage usage is greatly decreased, specially if you have more than one VM using the same base template. The downside is that there is always a performance penalty from the snapshot implementation at the hypervisor, and also, if you don't use a SSD for storage, from the additional "scrambling" that the linked clone VM suffer.

Instead of having a 1:1 relationship between the virtual disk seen by the VM, and some physical virtual disk file, in a linked clone (or snapshoted) VM every new write is redirected to the snapshot virtual disk, and reads from unchanged data are directed to the template disk.
That way many access patterns that come as sequential from the VM operating system are "scrambled" when hit the physical drive. This don't have a great impact for SSD's, but can drastically reduce the troughput of a spinning drive.


As my setup uses a SSD, and I'm not worried about some performance impact in the storage, I was willing to use linked clones instead the conventional clones.
Fortunately, it's possible to configure a Vagrantfile to automatically creating a template VM, and a linked clone from it when you do a `vagrant up`for the first time.
The virtual disks in the template are also set to read only at the file system level. This is a nice way to avoid accidentally starting up the 

The [official documentation example](https://www.vagrantup.com/docs/virtualbox/configuration.html#linked-clones) doesn't work directly with the default template, so here is the line that you need to add:
```
config.vm.provider "virtualbox" do |vb|
  vb.linked_clone = true
end
```
If you want to include this for every new 'Vagrantfile`, you can modify the default template at this location:

`/opt/vagrant/embedded/gems/gems/vagrant-2.0.2/templates/commands/init/Vagrantfile.erb`

Maybe you will need to change the path if you have a different release of Vagrant.
