# # -*- mode: ruby -*-
# # vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"
VAGRANT_BOX = 'ubuntu/bionic64'# Memorable name for your
VM_NAME = 'al-yw-vm'# VM User — 'vagrant' by default
VM_USER = 'vagrant'# Username 
# VM_PORT = 8080

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  #config.vm.provider "docker"
  config.vm.provider "virtualbox"
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.box = VAGRANT_BOX
  
  #config.vbguest.auto_update = true
  #config.vbguest.auto_reboot = true
  # Actual machine name
  config.vm.hostname = VM_NAME  # Set VM name in Virtualbox
  #not sure if i need this
  #config.vm.network "forwarded_port", guest: 8080, host: 7000, host_ip: "127.0.0.1", auto_correct: true
  
# If we want to stick with a vagrant solution use below for dev
  #config.vm.provision "docker", run: "once"
  # Set VM name in Virtualbox
  #config.vm.provider "virtual-box" do |v|
         #DHCP — commert this out if planning on using NAT instead
  #config.vm.network "private_network", type: "dhcp"  # # Port forwarding — uncomment this to use NAT instead of DHCP
  # config.vm.network "forwarded_port", guest: 80, host: VM_PORT  # Sync folder
  config.vm.provision "ansible_local" do |ansible|
    ansible.config_file = "/vagrant/ansible.cfg"
    ansible.extra_vars = { ansible_python_interpreter:"/usr/bin/python3" }
    ansible.galaxy_role_file = "provisioning/requirements.yml"
    ansible.galaxy_command = "ansible-galaxy install --role-file=%{role_file}"
    ansible.playbook = "playbooks/al-playbook.yml"
  end
end
