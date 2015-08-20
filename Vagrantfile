VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

#------------------------------------------------------------------------------
# Common settings that all VMs share

    config.vm.synced_folder "shared", "/home/vagrant/shared"
    config.vm.box = "ubuntu/trusty64"

    config.hostmanager.enabled = true
    config.hostmanager.manage_host = true
    config.hostmanager.ignore_private_ip = false
    config.hostmanager.include_offline = true

    config.trigger.after [:provision, :up, :reload] do
      system('echo "
        rdr pass on lo0 inet proto tcp from any to 127.0.0.1 port 80 -> 127.0.0.1 port 8080
        rdr pass on lo0 inet proto tcp from any to 127.0.0.1 port 443 -> 127.0.0.1 port 8443
        " | sudo pfctl -f - > /dev/null 2>&1; echo "==> Fowarding Ports: 80 -> 8080, 443 -> 8443"')
    end

    config.trigger.after [:halt, :destroy] do
      system("sudo pfctl -f /etc/pf.conf > /dev/null 2>&1; echo '==> Removing Port Forwarding'")
    end
#------------------------------------------------------------------------------

    config.vm.define "pelicansite" do |pelicansite|
        pelicansite.vm.synced_folder "shared", "/home/vagrant/shared"
        pelicansite.vm.box = "ubuntu/trusty64"
        pelicansite.vm.network :forwarded_port, guest: 8080, host: 8080

        # This is for Dropbox Lansync
        pelicansite.vm.network :forwarded_port, guest: 17500, host: 17501, auto_correct: true
        # Add a bridged network
        pelicansite.vm.network "public_network", bridge: 'en0: Wi-Fi (AirPort)'

        pelicansite.vm.hostname = 'pelicansite'
        pelicansite.hostmanager.aliases = %w(pelicansite.net www.pelicansite.net pelicansite.com)

        pelicansite.vm.provider :virtualbox do |vb|
            # Force more aggressive time updates, needed for when host machine sleeps
            vb.customize [ "guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold", 10000 ]

            # Set memory lower
            vb.memory = 1024
            vb.cpus = 2
        end

        pelicansite.vm.provision "ansible" do |ansible|
            ansible.extra_vars =  "@group_vars/vagrant"
            ansible.groups = {
                "vagrant" => ["pelicansite"]
            }
            ansible.verbose = "vv"
            ansible.playbook = "pelicansite.yml"
            ansible.vault_password_file = "vault/plain/pass.txt"
        end
    end

end
