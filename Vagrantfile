# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Check if this is Windows
  is_windows = (RbConfig::CONFIG['host_os'] =~ /mswin|mingw|cygwin/)
 
  # Look for project variables file.
  if !(File.exists?(File.dirname(__FILE__) + "/vars.project.yml"))
    raise NoVarsException
  end

  # Load the yml files
  require 'yaml'
  vars = YAML.load_file(File.dirname(__FILE__) + "/vars.global.yml")
  vars.merge!(YAML.load_file(File.dirname(__FILE__) + "/vars.project.yml"))

  # Clone project repo to `./src` if the folder doesn't exist yet, and the vars exists.
  if !(File.directory?(File.dirname(__FILE__) + "/src"))

    # If a project_repo is set, clone it.
    if vars['project_repo']
      system("git clone #{vars['project_repo']} #{File.dirname(__FILE__)}/src")
    # otherwise, create the src directory so we can share it to the guest.
    else
      system("mkdir #{File.dirname(__FILE__)}/src");
    end

    if !(File.directory?(File.dirname(__FILE__) + "/src"))
      raise NoSrcException
    end
  end

  # Config the VM
  config.vm.box = "ubuntu/trusty64"
  config.vm.hostname = vars['server_hostname']

  # Sets IP of the guest machine and allows it to connect to the internet.
  config.vm.network :private_network, ip:  vars['vansible_ip']
  config.ssh.forward_agent = true

  # Read this user's host machine's public ssh key to pass to ansible.
  if !(File.exists?("#{Dir.home}/.ssh/id_rsa.pub"))
    if !(File.exists?("#{Dir.home}/.ssh/id_dsa.pub"))
      raise NoSshKeyException
    end
  end
  if !(File.exists?("#{Dir.home}/.ssh/id_rsa.pub"))
    ssh_public_key = IO.read("#{Dir.home}/.ssh/id_dsa.pub").strip!
  else
    ssh_public_key = IO.read("#{Dir.home}/.ssh/id_rsa.pub").strip!
  end

  # If ansible is installed on the host, we can use config.vm.provision.
  # If it is not, we use shell provisioner to run
  # See https://github.com/mitchellh/vagrant/issues/2103
  has_ansible = `which ansible`.to_s.strip.length != 0
  if has_ansible
    config.vm.provision "ansible" do |ansible|
      ansible.playbook = vars['vansible_playbook']
      ansible.extra_vars = {
        ansible_ssh_user: 'vagrant',
        authorized_keys: ssh_public_key
      }
      ansible.sudo = true
    end
  else
    # If local ansible is not found, install it in the guest and run the playbook there.
    config.vm.provision "shell", path: File.dirname(__FILE__) + "/tasks/setup-ansible.sh"

    # Run ansible Provisioner via shell.
    config.vm.provision "shell",
      inline: "cd /vagrant; ansible-playbook -c local  -i 'default,' #{vars['vansible_playbook']} --extra-vars 'authorized_keys=\"#{ssh_public_key}\"'"

  end

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", vars['vansible_memory']]
  end

  # Sync project folder to guest machine.
  if not is_windows
    config.vm.synced_folder "src/#{vars['path_to_drupal']}", "/var/www/#{vars['project']}",
      :nfs => true
  else
    config.vm.synced_folder "src/#{vars['path_to_drupal']}", "/var/www/#{vars['project']}",
      owner: "www-data", group: "www-data"
  end

  # Save a local alias for this project.
  if (!File.exists?("#{Dir.home}/.drush"))
    Dir.mkdir("#{Dir.home}/.drush")
  end

  DRUSH_ALIAS_FILE = "#{Dir.home}/.drush/#{vars['project']}.vagrant.alias.drushrc.php"
  if (!File.exists?(DRUSH_ALIAS_FILE))
    # @TODO: Is is possible to load this from the ansible template?
    drush_alias = "
  <?php
  $aliases['#{vars['project']}.vagrant'] = array(
    'uri' => '#{vars['server_hostname']}',
    'root' => '/var/www/#{vars['project']}',
    'remote-host' => '#{vars['server_hostname']}',
    'remote-user' => 'vagrant',
  );
    "
    if (File.write("#{Dir.home}/.drush/#{vars['project']}.vagrant.alias.drushrc.php", drush_alias))
      # @TODO: Replace with Vagrant::UI::Basic once we know how to use it :(
      puts "Drush alias created. You may access your site with `drush @#{vars['project']}`"
    end
  end

  # Make sure files folder is writable
   system("chmod a+w #{File.dirname(__FILE__)}/src/#{vars['path_to_drupal']}/sites/default/files")

end

##
# Our Exceptions
#
class NoVarsException < Vagrant::Errors::VagrantError
  error_message('Project variables file not found. Copy vars.project.example.yml to vars.project.yml, edit to match your project, then try again.')
end

class NoSrcException < Vagrant::Errors::VagrantError
  error_message('Could not create ./src folder. Run as the owner of this folder. ')
end

class NoSshKeyException < Vagrant::Errors::VagrantError
  error_message('An ssh public key could not be found at ~/.ssh/id_rsa.pub or ~/.ssh/id_dsa.pub . Please generate one and try again.')
end

class NoSSHException < Vagrant::Errors::VagrantError
  error_message('Your SSH does not currently contain any keys (or is stopped.)')
  error_message('If you are on a Mac, add a passphrase to your SSH key by running:')
  error_message('  $ ssh-keygen -p -f ~/.ssh/id_rsa')
  error_message('  $ eval $(ssh-agent)')
  error_message('  $ ssh-add')
end
