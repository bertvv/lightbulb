# -*- mode: ruby -*-
# vi: set ft=ruby :

$NODES=3
$NODEMEM=256
# Overwrite host locale in ssh session
ENV["LC_ALL"] = "en_US.UTF-8"

BOOTSTRAP_SCRIPT=<<__EOF__
#!/bin/bash
# Bootstrap the Ansible Lightbulb workshop environment on a Vagrant box

#
# Bash shell settings: exit on failing commands, unbound variables
#
set -o errexit
set -o nounset
set -o pipefail

#
# Variables
#
# Color definitions
readonly reset='\e[0m'
readonly red='\e[0;31m'
readonly yellow='\e[0;33m'
readonly cyan='\e[0;36m'

readonly workshop_user=vagrant

readonly workshop_key="/home/${workshop_user}/.ssh/id_rsa"
readonly workshop_inventory="/home/${workshop_user}/hosts.ini"
readonly workshop_ansible_config="/home/${workshop_user}/.ansible.cfg"

readonly workshop_git_repo="https://github.com/ansible/lightbulb.git"

#
# Functions
#

main() {
  info "Bootstrapping Ansible on host ${HOSTNAME}."

  ensure_ansible_installed
  install_vagrant_private_key
  configure_ansible
  install_inventory
  install_lightbulb

}

# Ansible installation
ensure_ansible_installed() {
  if ! is_ansible_installed; then
    distro=$(get_linux_distribution)
    "install_ansible_${distro}" || die "Distribution ${distro} is not supported"
  fi

  info "Ansible version"
  ansible --version
}

is_ansible_installed() {
  which ansible-playbook > /dev/null 2>&1
}

# Print the Linux distribution
get_linux_distribution() {

  if [ -f '/etc/redhat-release' ]; then

    # RedHat-based distributions
    cut --fields=1 --delimiter=' ' '/etc/redhat-release' \
      | tr "[:upper:]" "[:lower:]"

  elif [ -f '/etc/lsb-release' ]; then

    # Debian-based distributions
    grep DISTRIB_ID '/etc/lsb-release' \
      | cut --delimiter='=' --fields=2 \
      | tr "[:upper:]" "[:lower:]"

  fi
}

# Install Ansible on a Fedora system.
# The version in the repos is fairly current, so we'll install that
install_ansible_fedora() {
  info "Fedora: installing Ansible from distribution repositories"
  dnf -y install git ansible
}

# Install Ansible on a CentOS system from EPEL
install_ansible_centos() {
  info "CentOS: installing Ansible from the EPEL repository"
  yum -y install epel-release
  yum -y install git ansible
}

# Install Ansible on a recent Ubuntu distribution, from the PPA
install_ansible_ubuntu() {
  info "Ubuntu: installing Ansible from PPA"
  # Remark: on older Ubuntu versions, it's python-software-properties
  apt-get -y install software-properties-common
  apt-add-repository -y ppa:ansible/ansible
  apt-get -y update
  apt-get -y install git ansible
}

# Install the Vagrant private key to SSH to the other nodes
install_vagrant_private_key() {
  curl --output "${workshop_key}" \
    https://raw.githubusercontent.com/hashicorp/vagrant/master/keys/vagrant
  chown "${workshop_user}:${workshop_user}" "${workshop_key}"
  chmod 0600 "${workshop_key}"
}

# Install an inventory file
install_inventory() {
  info "Installing inventory file ${workshop_inventory}"

  cat > "${workshop_inventory}" << _EOF_
[control]
ansible ansible_host=10.42.0.2 ansible_connection=local

[web]
node-1 ansible_host=10.42.0.6
node-2 ansible_host=10.42.0.7
node-3 ansible_host=10.42.0.8

[all:vars]
ansible_user=vagrant
_EOF_
  chown "${workshop_user}:${workshop_user}" "${workshop_inventory}"
}

# This will generate an ansible.cfg file from a here document
configure_ansible() {
  info "Installing Ansible configuration file ${workshop_ansible_config}"

  cat > "${workshop_ansible_config}" << _EOF_
# config file for ansible -- http://ansible.com/
# ==============================================

[defaults]

inventory = ./hosts.ini
forks = 50
host_key_checking = False
retry_files_enabled = False
no_target_syslog = False

[ssh_connection]
scp_if_ssh = True
_EOF_
}

# Install the lightbulb repository
install_lightbulb() {
  if [ ! -d "/home/${workshop_user}/lightbulb" ]; then
    info "Installing the Lightbulb Git repository for user ${workshop_user}"
    su --command="git clone ${workshop_git_repo}" - "${workshop_user}"
  else
    info "Lightbulb already installed"
  fi
}

# Usage: info [ARG]...
#
# Prints all arguments on the standard output stream
info() {
  printf "${yellow}>>> %s${reset}\n" "${*}"
}

# Usage: debug [ARG]...
#
# Prints all arguments on the standard output stream
debug() {
  printf "${cyan}### %s${reset}\n" "${*}"
}

# Usage: error [ARG]...
#
# Prints all arguments on the standard error stream
error() {
  printf "${red}!!! %s${reset}\n" "${*}" 1>&2
}

# Usage: die MESSAGE
# Prints the specified error message and exits with an error status
die() {
  error "${*}"
  exit 1
}

main "${@}"
__EOF__

# All Vagrant configuration is done here.
Vagrant.configure("2") do |cluster|
  # The most common configuration options are documented and commented below.
  # For more refer to https://www.vagrantup.com/docs/vagrantfile/

  # Every Vagrant virtual environment requires a box to build off of.

  # The ordering of these 2 lines expresses a preference for a hypervisor
  cluster.vm.provider "virtualbox"
  cluster.vm.provider "libvirt"
  cluster.vm.provider "vmware_fusion"

  # Avoid using the Virtualbox guest additions
  cluster.vm.synced_folder ".", "/vagrant", disabled: true
  if Vagrant.has_plugin?("vagrant-vbguest")
    cluster.vbguest.auto_update = false
  end

  # For convenience, testing and instruction, all you need is 'vagrant up'
  # Every vagrant box comes with a user 'vagrant' with password 'vagrant'
  # Every vagrant box has the root password 'vagrant'

  # This vagrant box is downloaded from https://vagrantcloud.com/centos/7
  # Other variants https://app.vagrantup.com/boxes/search
  cluster.vm.box = "centos/7"
  cluster.ssh.insert_key = false
  # Don't install your own key (you might not have it)
  # Use this: $HOME/.vagrant.d/insecure_private_key

  # host to run ansible and tower
  cluster.vm.define "ansible", primary: true do |config|
    config.vm.hostname = "ansible"
    config.vm.network :private_network, ip: "10.42.0.2"
    config.ssh.forward_agent = true
    config.vm.provider :virtualbox do |vb, override|
      vb.customize [
        "modifyvm", :id,
        "--name", "ansible",
        "--memory", "2048",
        "--cpus", 1
      ]
      vb.customize ['modifyvm', :id, '--groups', '/lightbulb_workshop']
    end
    config.vm.provision 'shell', inline: BOOTSTRAP_SCRIPT
  end

  # hosts to run ansible-core
  (1..$NODES).each do |i|
    cluster.vm.define "node-#{i}" do |node|
      node.vm.hostname = "node-#{i}"
      node.vm.box = "centos/7"
      node.vm.network :private_network, ip: "10.42.0.#{i+5}"
      node.vm.provider :virtualbox do |vb, override|
        vb.customize [
          "modifyvm", :id,
          "--name", "node-#{i}",
          "--memory", "#$NODEMEM",
          "--cpus", 1
        ]
        vb.customize ['modifyvm', :id, '--groups', '/lightbulb_workshop']
      end
    end
  end
end
