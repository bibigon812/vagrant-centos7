# frozen_string_literal: true

nodes = [
  {
    hostname: 'centos7',
    autostart: true,
    ram: 1024,
  }
]

domain = 'local'

Vagrant.configure('2') do |config|
  %w[vagrant-reload].each do |plugin|
    unless Vagrant.has_plugin?(plugin)
      puts "ERROR: the `#{plugin}` plugin for Vagrant is not installed."
      puts "RUN: `vagrant plugin install #{plugin}`"
      exit 1
    end
  end

  Vagrant.has_plugin?('vagrant-cachier') && config.cache.auto_detect = true

  nodes.each do |node|
    fqdn = "#{node[:hostname]}.#{domain}"
    config.vm.define node[:hostname], autostart: node.fetch(:autostart) { false } do |node_config|
      node_config.vm.box = 'centos/7'
      node_config.vm.hostname = fqdn
      node_config.ssh.insert_key = false

      node_config.vm.provider :virtualbox do |vb|
        vb.customize [
          'modifyvm', :id,
          '--name', node[:hostname],
          '--memory', node.fetch(:ram) { 512 },
          '--cpus', node.fetch(:cpu) { 1 },
          '--ioapic', 'on',
          '--nictype1', 'virtio',
        ]
      end

      node_config.vm.provision :shell, inline: <<-SCRIPT.gsub(/^\s*/, '')
        echo 'SELINUX=disabled' > /etc/selinux/config
        yum -y install epel-release
        rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
        rpm -Uvh https://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
        yum -y erase kernel-tools-libs kernel-tools
        yum -y --enablerepo=elrepo-kernel install kernel-ml kernel-ml-tools kernel-ml-tools-libs
        grub2-set-default 0
      SCRIPT

      node_config.vm.provision :reload
      node_config.vm.provision :shell, inline: <<-SCRIPT.gsub(/^\s*/, '')
        yum -y erase $(rpm -qa | grep ^kernel | grep -v $(uname -r))
        yum -y update
        yum -y --enablerepo=elrepo-kernel install kernel-ml-devel kernel-ml-headers perl gcc dkms make bzip2
        VBGA_VERSION=$(curl -s https://download.virtualbox.org/virtualbox/LATEST.TXT)
        curl -s -o vbga.iso https://download.virtualbox.org/virtualbox/${VBGA_VERSION}/VBoxGuestAdditions_${VBGA_VERSION}.iso
        mount -o loop vbga.iso /mnt
        /mnt/VBoxLinuxAdditions.run --nox11
        umount /mnt
        rm -f vbga.iso
        yum -y erase kernel-ml-devel kernel-ml-headers perl gcc dkms make bzip2
        yum -y autoremove
        rm -rf /var/cache/yum
      SCRIPT
    end
  end
end
