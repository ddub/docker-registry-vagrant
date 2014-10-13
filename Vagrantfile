# -*- mode: ruby -*-
# vi: set ft=ruby :

# Set a default IP Address for this VM
ip_address = '192.168.42.42'
# Check the contents of this file to override the default
IP_ADDRESS_FILE = '.vagrant.ip.address'
# Check validity of IP addresses
IP_OCTET = '(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)'
IP_MATCHER = /^(?:#{IP_OCTET}\.){3}#{IP_OCTET}$/

# Load the IP address config file to override the default
if File.file?(IP_ADDRESS_FILE)
  File.open(IP_ADDRESS_FILE, 'r') do |file|
    ip = file.gets.chomp
    IP_MATCHER.match(ip) && ip_address = ip
  end
end
# Allow the environment variable to override the config file and default
if ENV.key?('DOCKER_REGISTRY_IP')
  ip = ENV['DOCKER_REGISTRY_IP']
  IP_MATCHER.match(ip) && ip_address = ip
end

# Save the value for future reloads
File.open(IP_ADDRESS_FILE, 'w') { |file| file.write(ip_address) }

Vagrant.configure(2) do |config|
  config.vm.hostname = 'docker-registry'
  config.vm.box      = 'ubuntu/trusty64'
  config.vm.network :forwarded_port, guest: 5000, host: 5000,
                                     auto_correct: true
  config.vm.network :public_network, ip: ip_address
  config.vm.provision 'docker' do |d|
    d.pull_images 'registry'
    a = ' -v /vagrant/registry:/registry'\
        ' -e SETTINGS_FLAVOR=local'\
        ' -e STORAGE_PATH=/registry'\
        ' -e SEARCH_BACKEND=sqlalchemy'\
        ' -e SQLALCHEMY_INDEX_DATABASE=sqlite:////registry/docker-registry.db'\
        " -p #{ip_address}:5000:5000"
    d.run 'registry', args: a
  end
end