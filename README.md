# Proxmox-setup

An [Ansible](https://www.ansible.com) playbooks to setup a [Proxmox](https://proxmox.com) virtualisation environment as home server. This setup includes configuration changes so that LXC containers can be used to host [K3S](https://k3s.io/) or [Docker](docker.com).

>** Beware** Do not run some script from a crazy guy on the internet. Do Read it! 
In this case it will not work out of the box anyway.

## Requirements

1. A Proxmox server with ssh access. Initial ssh access with a root password is fine, our playbook will install your key
3. Python 
2. Ansible on your local machine
3. Optionally Mitogen Ansible ssh accelaration.
4. A few Ansible collections and modules.

## Bootstrapping Ansible

Install Python, pip and Ansible on your local machine.

For Debian machines: 

```bash
apt install python3
apt install python3-pip
```

For Mac use [HomeBrew](https://brew.sh/). For other Linux distributions use the package manager for your distribution.
Make sure that you install Python 3 and the related pip for Python 3.

Cross platform:

```bash
pip install ansible-core
pip install ansible
```

I use pip to install Ansible instead of my package manager since the Ansible version of that is often a bit older.

After that you can maintain it with A playbook for your laptop such as my [win-dev](https://github.com/nicenemo/win-dev-playbook) playbook for Debian based machines.

## Mitogen ssh acceleration

I use mitogen ssh acceleration in my playbooks.

Install with:

```bash
pip install -U git+https://github.com/dw/mitogen.git
```
If you do not want to use that you can comment out the configuration lines in [ansible.cfg](ansible.cfg)

```ini
[defaults]
...
# strategy_plugins = ~/.local/lib/python3.9/site-packages/ansible_mitogen
# strategy = mitogen_linear
...
```

##  your proxmox hosts in the Ansible Inventory

I use a YAML based inventory in [hosts.yml](hosts.yml).
You probably want to change my Proxmox hostname `aegean` to match that of your local Proxmox server(s).

My host names are in DNS. You can use ip addresses in the inventory file `hosts.yml` , `/etc/hosts` file  and/or an ssh configuration in `~/.ssh/config` to match host names to IP addresses too.

## Ansible vault password

I have my Ansible vault password in `~/.vault_password.txt` Create your own.
With `source ./set_ansible_vault_password.sh` you set the `ANSIBLE_VAULT_PASSWORD_FILE`. 
You could add that to your `.bashrc` or `.zshrc`. 

Keep this password in another place away from your playbook repository, e.g. in your password manager.

## API password

We use an API password to create and remove containers later.
> Proxmox has the option to use API keys instead, I did not get that to work yet.

```YAML
---
proxmox_api_password_encrypted: 'replace by root password'
```

Please do replace my no _not so secret_ password string with your Proxmox server's root password. 

Encrypt using your own vault key with:

```bash
ansible-vault encrypt ./group_vars/all/vault
```
## Root user ssh key

In the `hosts.yml` inventory set the `root_user_ssh_key` for your Proxmox servers.

We use the [Ansible Authorized keys](https://docs.ansible.com/ansible/latest/collections/ansible/posix/authorized_key_module.html) module to set the key. See the examples for other options than Github ssh keys.


## User Creation and ssh login

> This will change in the future as I will change my [Debian base](https://github.com/nicenemo/nicenemo.debianbase) role to become more flexible.

In the [authorized_keys](authorized_keys) folder you can publish your public keys.
Create a file per username. Remove my public keys.

The users will be created based on the filenames. Their key files will be copied to give them ssh access and they will get `sudo` rights.

## ZFS pools and Samba shares

I use [Open ZFS](https://openzfs.org/wiki/Main_Page) to store my personal data and the data of my containers.
ZFS's builtin [Samba](https://www.samba.org) is used to create the most barebone NAS functionality.

For working Samba server functionality the Linux Samba server is installed. On Linux this cooperates with the ZFS Samba support.

>TODO: SMB user/password creation is still a manual step.

In the `zfs_pools` variable in [group_vars/proxmox_servers](group_vars/proxmox_servers) I define my ZFS pools.

>TODO: The pools are NOT created by Ansible that is still a manual step.

I have two pools:

* `tank`, hard-drive based to store data.
* `zones`, for LXC containers and virtual machines.

> I migrated from [Illumos](https://illumos.org/) based [SmartOS])(https://www.smartos.org) in 2019. The `zones` pool came from SmartOS. The `tank` name for data is often used as a pool for data.

Per pool a list of top level datasets is defined. 
Per dataset, state the owner, and whether the pool is exposed via [Samba](https://www.samba.org) file sharing. 

Since I only have one Proxmox server I can use loop mount to make datasets in `tank` available to the container. There is no need to use SMB or NFS mounts for now. 

>TODO: Automate loop mount creation.
>For [K3S][https://k3s.io/] I can expand the `zfs_pools` variable to also have a boolean for enabling NFS later.

# Initial run

The initial run of the playbook should be with using the `root` user ssh password.

```bash
./proxmox_system_setup.yml -k
```
Since your key is copied into the root user's `/root/.ssh/authorized_keys` file, following runs can be run without the `-k`flag.

```bash
./proxmox_system_setup.yml
```

>TODO: disable root ssh login

## Create LXC container for K3S

With the ./create_elk_container.yml

## Destroy K3S container

## Provision K3S container
