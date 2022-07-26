$hostname = 'apt-cache.test'
$ip = '10.10.10.254'
$port = '3142'

Vagrant.configure('2') do |config|
  config.vm.box = 'ubuntu/focal64'
  config.vm.box_check_update = false

  config.vm.provider :virtualbox do |vb|
    vb.name = 'APT Cache'

    # Restrict resaurces
    vb.cpus = 2
    vb.memory = 512

    vb.customize ['modifyvm', :id, '--nictype1', 'virtio']
    vb.customize ['modifyvm', :id, '--nictype2', 'virtio']

    # Improve network speed (github.com/mitchellh/vagrant/issues/1807)
    vb.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
    vb.customize ['modifyvm', :id, '--natdnsproxy1', 'on']

    # Fix clock sync issues
    vb.customize ['setextradata', :id, 'VBoxInternal/Devices/VMMDev/0/Config/GetHostTimeDisabled', 0] # Enable time sync
    vb.customize ['guestproperty', 'set', :id, '/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold', 60 * 1_000] # Sync every 60 secs

    # Fix boot (https://bugs.launchpad.net/cloud-images/+bug/1829625/comments/5)
    vb.customize ['modifyvm', :id, '--uart1', '0x3F8', '4']
    vb.customize ['modifyvm', :id, '--uartmode1', 'file', File::NULL]
    vb.customize ['modifyvm', :id, '--nestedpaging', 'off']
    vb.customize ['modifyvm', :id, '--paravirtprovider', 'hyperv']    
  end

  config.vm.define :cache do |server|
    server.vm.hostname = $hostname

    server.vm.synced_folder './apt-cache', '/var/cache/apt-cacher-ng', mount_options: ['dmode=777,fmode=777']

    server.vm.network :private_network, ip: $ip

    server.vm.provision 'shell', inline: <<~SHELL
      CONFIG_FILE='/etc/apt-cacher-ng/acng.conf'
      CONFIG_ADDITION='PassThroughPattern: ^(.*):443$'
      apt-get update \\
        && apt-get upgrade -y \\
        && DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y apt-cacher-ng \\
        && (grep -qxF "$CONFIG_ADDITION" "$CONFIG_FILE" || echo "$CONFIG_ADDITION" >> "$CONFIG_FILE") \\
        && rm -f /usr/lib/apt-cacher-ng/userinfo.html \\
        && service apt-cacher-ng restart \\
        && update-rc.d apt-cacher-ng enable \\
        && ufw allow OpenSSH \\
        && ufw allow #{$port}/tcp comment apt-cacher-ng \\
        && ufw enable \\
        && ufw reload
    SHELL

    # Fix apt-cacher autostart issue
    config.vm.provision 'shell', run: 'always', inline: 'sudo sh -c "service apt-cacher-ng restart || true"'

    # Fix various inconsistencies
    server.vm.provision 'shell', name: 'Sync clock', run: 'always', inline: <<~SHELL
      nohup sudo sh -c 'service ntp stop ; ntpd -gq' &
    SHELL
    server.vm.provision 'shell', name: 'Fix macOS NFS timeout', run: 'always', inline: <<~SHELL
      nohup sh -c 'while true; do ls /var/cache/apt-cacher-ng > /dev/null; sleep 30; done' &
    SHELL
  end

  # Prevent mounting default Vagrant dir
  config.vm.synced_folder '.', '/vagrant', disabled: true

  # Disable NTP
  config.vm.provision 'shell', inline: 'timedatectl set-ntp false'
end
