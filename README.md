Vagrant APT Cache
=================

A Vagrant machine running [apt-cacher-ng](https://wiki.debian.org/AptCacherNg) for other Debian/Ubuntu VMs to proxy through, speeding up subsequent APT installs after destroying & reprovisioning, etc.


Usage
-----

0. _Optional: Change the IP and port in the `Vagrantfile` if conflicting_

1. Run `vagrant up`

2. Update VMs to use the VM as an APT proxy (replacing the IP addresses if changed in step 0):
  
    ```sh
    echo 'Acquire::http { Proxy "http://10.10.10.254:3142"; }' \
      > /etc/apt/apt.conf.d/00proxy && \
    echo 'Acquire::https { Proxy "http://10.10.10.254:3142"; }' \
      >> /etc/apt/apt.conf.d/00proxy
    ```

    _Optional: Install the [`vagrant-hostsupdater` plugin](https://github.com/agiledivider/vagrant-hostsupdater) to allow using `apt-cache.test` instead of the IP address above._

APT cache files will be stored in the same directory under an `apt-cache` subdirectory, preserving the cache after destroying and rebuilding this VM, and keeping the VM itself's disk size small.


Licence
-------

[MIT](LICENCE)

