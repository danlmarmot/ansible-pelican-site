# Overview

This Ansible project creates a site using the Pelican static site framework on a Ubuntu 14.04 server.  It was first
used to give Pelican a test drive on a Vagrant machine; later the scripts were adapted for deployment to a Linode
host.

Dropbox is used to sync Pelican content (written in Markdown), with cronjobs running every so often to generate the
site.  The site itself is served with nginx.

Versions of components are all current as of February 2015 and are fairly locked down.

These scripts were developed on a Mac OS X 10.10 machine, and likely will need some adjusting for other development environments.

## Vagrant usage
 
### Provisioning with Ansible

To provision a local Vagrant virtual machine for the first time use this command

    vagrant up
    
Verify everything's good by looking around on the VM with this command; you can do this fairly early when
the machine's being provisioned.  It takes anywhere from five to fifteen minutes to provision a Vagrant
instance, which is mostly dependent on the speed of your Internet network connection 
    
    vagrant ssh
    
If you've made changes to one of the Ansible roles or variables and want to apply these changes, run this: 

    vagrant provision
    
### Ansible Vault

The encrypted.yml file contains sensitive data (such as passwords) that shouldn't be in plaintext.  There are a few 
variables that are used in the provisioning scripts that are kept here.

The password file is intentionally ignored by .gitignore, you will have to create one for yourself.  To do this,
create a file at /vault/plain/pass.txt, with the content of 'none' (without the quotes).  You should change this to your
own password right away.  Consult the read me in the /vault folder for more details on how to do this.
   
### Port forwarding from the Vagrant instance

Ports 80 and 443 are forwarded from the Vagrant instance to ports 8080 and 8443.  Note this has only really been tested
on Mac OS X 10.  Once the Vagrant VM is running, visit http://localhost:8080 to see the Pelican site.

At the end of a 'vagrant provision', 'vagrant up', or 'vagrant destroy' command you may be prompted for a password to
setup or remove port forwarding.  This should be the system password.

## Linode usage

### Provisioning a new Linode with Ansible

To provision a Linode, use this command.  The Ansible playbook is the same between Vagrant and Linode, but the command 
line is quite different.

    ansible-playbook pelicansite.yml --vault-password-file vault/plain/pass.txt --extra-vars "@group_vars/linode"

## Getting Dropbox working

To be secure, don't use your primary Dropbox account.  Use a secondary Dropbox account, and share that with your primary
Dropbox user account.  This makes syncing content much faster, and protects your primary account.

Dropbox provisioning is a manual step, but you only need to do it the first time you add Dropbox.  You will need to SSH 
to SSH to the box, change to 'pelicanuser' with sudo su pelicanuser, and run the dropbox daemon with dropboxd.
This will return a URL to link the machine being provisioned to your (secondary) Dropbox account.  Copy and paste that
URL into a web browser, log into your secondary Dropbox account, and accept the link.

Pelican content and output should live in Dropbox in a directory named after the site name (in this case, pelican.foo).


## Troubleshooting Ansible

Is Ansible not working for you at all?  Using Vagrant it's pretty easy to check out your host machine's configuration.

Start by going into the pelicansite.yml file and commenting out all the roles
except the 'dumpall' role.  That section should look something like this:

      roles:
        - dumpall
    #    - bootstrap
    #    - supervisord
    
Then rerun Ansible with "vagrant provision", and you should see output like this:

    GATHERING FACTS ***************************************************************
    <127.0.0.1>
    ok: [pelicansite]
    
    TASK: [dumpall | Dump all vars] ***********************************************
    ok: [pelicansite] => {"changed": false, "checksum": "ed2095ea7bf50684ff70b07a5c874b4da742b6a1", "dest": "/tmp/ansible-dumpall.json", "gid": 0, "group": "root", "md5sum": "ee2295f1406aa72eb878c177042c4af3", "mode": "0644", "owner": "root", "size": 60936, "src": "/home/vagrant/.ansible/tmp/ansible-tmp-1440028516.15-219292919307821/source", "state": "file", "uid": 0}
    
    PLAY RECAP ********************************************************************
    pelicansite                : ok=2    changed=0    unreachable=0    failed=0