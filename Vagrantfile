# -*- mode: ruby -*-
# vi: set ft=ruby :

#############################
# Codeup Server Setup
#############################

box           = 'ubuntu/trusty64'
vmware_box    = 'puppetlabs/ubuntu-14.04-64-nocm'
parallels_box = 'parallels/ubuntu-14.04'
hostname      = 'codeup-vm'
domain        = 'codeup.dev'
ip            = '192.168.77.77'
ram           = '512'
num_cpus      = '1'

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.define hostname do |v|
    v.vm.box = box

    v.vm.hostname = hostname
    v.vm.network :private_network, ip: ip

    v.vm.synced_folder "./", "/vagrant",
      id:   "vagrant-root",
      type: "nfs",
      mount_options: ["nolock,vers=3,tcp,noatime,actimeo=1"]

    # # VirtualBox's builtin shared folder technology
    # # Slower than NFS, but does not require admin password
    # v.vm.synced_folder "./", "/vagrant", id: "vagrant-root",
    #     owner: "vagrant",
    #     group: "www-data",
    #     mount_options: ["dmode=775,fmode=664"]

    v.vm.provider :virtualbox do |vb|
      vb.name = hostname

      vb.memory = ram
      vb.cpus   = num_cpus

      vb.customize ["modifyvm",             :id, "--natdnshostresolver1", "on"]
      vb.customize ["setextradata",         :id, "--VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
      vb.customize ["guestproperty", "set", :id, "--timesync-threshold",  "1000"]
    end

    # Configuration options for the VMware Fusion provider.
    v.vm.provider :vmware_fusion do |v, override|
      v.vmx["memsize"]  = ram
      v.vmx["numvcpus"] = num_cpus

      override.vm.box = vmware_box
    end

    # Configuration options for the Parallels provider.
    v.vm.provider :parallels do |v, override|
      v.update_guest_tools = true
      v.customize ["set", :id, "--longer-battery-life", "off"]

      v.memory = ram
      v.cpus   = num_cpus

      override.vm.box = parallels_box
    end

    v.vm.provision :ansible do |ansible|
      ansible.playbook   = "ansible/playbooks/vagrant/init.yml"
      ansible.extra_vars = { domain: domain }
    end
  end

  # Plugin specific options. Helpful for development but most likely not necessary for class
  if Vagrant.has_plugin? "vagrant-dns"
    config.dns.tld      = "dev"
    config.dns.patterns = [/^.*\.dev$/]
  end

  if Vagrant.has_plugin? "vagrant-cachier"
    config.cache.scope = :box

    config.cache.enable :apt
    config.cache.enable :apt_lists
    config.cache.enable :apt_cacher
    config.cache.enable :composer
    config.cache.enable :bower
    config.cache.enable :npm
    config.cache.enable :gem
    # config.cache.enable :pip

    config.cache.enable :generic, {
      "rbenv" => { cache_dir: "/usr/local/rbenv/cache" }
    }

    config.cache.synced_folder_opts = {
      type: :nfs,
      # The nolock option can be useful for an NFSv3 client that wants to avoid the
      # NLM sideband protocol. Without this option, apt-get might hang if it tries
      # to lock files needed for /var/cache/* operations. All of this can be avoided
      # by using NFSv4 everywhere. Please note that the tcp option is not the default.
      mount_options: ['rw', 'vers=3', 'nolock']
    }
  end
end
