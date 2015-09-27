Vagrant.require_version '~> 1.7'

# Check/install required vagrant plugins.
plugin_installed = false

%w(
  vagrant-parallels
).each do |name|
  next if Vagrant.has_plugin?(name)
  system "vagrant plugin install #{name}"
  plugin_installed = true
end

# The command must be re-run if a new plugin is installed.
exec "vagrant #{ARGV.join(' ')}" if plugin_installed

Vagrant.configure('2') do |config|
  config.vm.define 'docker'
  config.vm.box = 'bento/debian-8.2'
  config.vm.box_version = '2.2.1'

  config.vm.provider 'parallels' do |v|
    v.check_guest_tools = false
    v.optimize_power_consumption = false
  end

  config.vm.network 'private_network', ip: '10.109.20.70'
  config.vm.network :forwarded_port, guest: 2375, host: 2375

  # Sync the entire home folder.
  config.vm.synced_folder '.', '/vagrant', disabled: true
  config.vm.synced_folder(
    ENV['HOME'], ENV['HOME'],
    type: 'nfs',
    mount_options: %w(actimeo=2),
    bsd__nfs_options: %w(alldirs maproot=root:wheel))

  config.vm.provision :shell, inline: <<-SH

    # Install docker.
    systemctl stop docker
    systemctl disable docker
    curl -sSL https://get.docker.com | sh
    usermod -aG docker vagrant

    # Configure docker unit file to run HTTP server.
    sed -i 's/fd:\\/\\//0.0.0.0:2375/' /lib/systemd/system/docker.service

    # Reload docker.unit file.
    systemctl daemon-reload
    systemctl enable docker
    systemctl restart docker
  SH
end
