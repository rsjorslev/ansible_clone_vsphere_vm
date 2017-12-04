# Clone vSphere 6.5 template using Ansible

__NOTE:__ This repo is intended purely to show how Ansible can be used to drive an "IaaS" style provisioning using Templates in vSphere 6.5.x environments. It's not to say it's the best approach but it works.

## What is this for?

This is for cloning a VM from a template in vSphere environments, get an IP from phpIPAM, register the IP and FQDN in Microsoft Windows DNS and prep the provisioned VM with apps/settings using ansible.
This repo shows cloning 3 Ubuntu 16.04.02 VMs and then installing the latest docker daemon afterwards, adding SSH keys and installing zsh.

## How is this used? (At a very high level)

All interaction with Microsoft Windows DNS server happens by means of WinRM and PowerShell commands directly.
The setup is expecting an inventory entry with cred_ssp configured and WinRM correctly configured on the Windows host. The host it targets is defined as `dns_server` in `group_vars/all`

Most of the variables neede are in `group_vars/all` so please edid this file so it matches your environment.
The actual placement of the cloned VM happens in `roles/common/files/provision.yml` using the `vmware_guest` module from Ansible so config specific to the cloning can be changed in that file.

The hostname and a description (used in phpIPAM) is entered into `vm_data.yml` prior to running the playbook. To run the actual playbook here is an example:

`ansible-playbook clone_vm.yml -u runner -k`

The reason for prompting for password and specifying the user is due to the way the template is provisioned. You can change this however you see fit. For an example of automating ubuntu installs into templates in vSphere check out my [ubuntu-template](https://github.com/rsjorslev/ubuntu-template) repository

There is also an option to destroy / teardown the provisioned VM(s) including deleting the used IP from phpIPAM and the DNS entries from Microsoft Windows DNS server.
Information required to delete the VM, IP and DNS entries are stored in a YAML file which can later be referenced 

`ansible-playbook clone_vm.yml --extra-vars "teardown=true" --extra-vars "@f2626ed4-b570-5593-b6b4-0a923e71d58d.yml"`

Where the YAML file referenced in `--extra-vars` is the output from running an actual provisioning.
