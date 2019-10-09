---
title: "Fix for HP CUPS driver in Ubuntu"
date: 2019-10-08T21:40:58-05:00
draft: false
---
Symptom: In /var/log/cups/error_log there is a message
`(/usr/lib/cups/filter/hpcups) crashed on signal 11.` and printer
doesn't print anything. The problem is that hpcups is buggy in Ubuntu
19.04.
A solution is to install HP printer driver from sources from
[https://developers.hp.com/hp-linux-imaging-and-printing](https://developers.hp.com/hp-linux-imaging-and-printing). There
are several problems with this however.

1. Delete all printer configurations from `/etc/cups/printers.conf`.
2. First download and run HPLIP main source hplip-3.19.8.run and a
   plugin for it hplip-3.19.8-plugin.run. Use whatever version is
   latest from HP site.
3. Try to run `./hplip-3.19.8.run` and install as much as there is
   installing.
4. In my case plugin installation failed and I couldn't go on.
5. Plugin tries to access keyserver `pgp.mit.edu` which fails to
   produce any keys. It helped me to execute the following commands:
```
gpg --keyserver pool.sks-keyservers.net --recv-keys 0x4ABA2F66DBD5A95894910E0673D770CDA59047B9
/usr/bin/gpg --homedir /home/gregory/.hplip/.gnupg --no-permission-warning --keyserver pool.sks-keyservers.net --recv-keys 0x4ABA2F66DBD5A95894910E0673D770CDA59047B9
```
6. Close hplip installation and try to run `hp-setup -g`.
7. I noticed that when listing libraries I got the same errors as in
   [this comment](https://bugs.launchpad.net/hplip/+bug/1818629/comments/26). It is necessary to fix all of the libraries that it
   complains about using symlinks.
8. Plugin installation successfully finishes. It is possible to finis
   `hp-setup` and create a new printer configuration now.
9. The problem with `hpcups` doesn't go away because HPLIP driver
   didn't modify already installed Ubuntu version. It is necessary to
   uninstall it from the system with `sudo apt-get remove printer-driver-hpcups`.
10. Then don't do `sudo apt autoremove` because this will install
   necessary packages and HPLIP installation fails. I also had to
   install `python3-pyqt4` to proceed.
11. Run `./hplip-3.19.8.run` again this time to install its own
    version of `hpcups`.
12. If plugin has to be installed again and you're getting `error:
    Python gobject/dbus may be not installed` what helped me was to
    run plugin installation manually `./hplip-3.19.8-plugin.run
    --confirm`.
13. This creates a `/tmp` directory with self extracting archive
    contents and asks whether to execute command
    `./hplip-plugin-install -v 3.19.8 -c 64`
14. Manually cd to the unpacked directory in `/tmp` and execute this
    command. Plugin installation is successful, finally!
