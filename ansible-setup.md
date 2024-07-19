
# How to configure Ansible to automate Proxmox 


## On the Ansible host

Install Ansible, the instructions below are for Ubuntu
```shell
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```


Create a SSH-Key for passwordless access 
```shell
ssh-keygen -t ecdsa -b 521
```

Copy the SSH-Key to the Proxmox Node
```shell
ssh-copy-id root@172.16.1.8
```

Create an Inventory file for Ansible with the Proxmox Node
  If you used the IP address for ssh copy use IP address in the inventory file
```shell
cat > inventory << EOF
[pvenodes]
172.16.1.8
EOF
```

Test ansible connectivity, you should a green output stating ping pong
```shell
ansible pvenodes -i inventory -m ping --user=root
```

The next setup creates a user called ansbile, transfers the ssh public, and install sudo
  Replace /home/vagrant with your home directory
```shell  
cat > pve_onboard.yml << EOF
- hosts: pvenodes
  tasks:

  - name: install sudo package
    apt:
      name: sudo
      update_cache: yes
      cache_valid_time: 3600
      state: latest

  - name: create Ansible user
    user:
      name: ansible
      shell: '/bin/bash'

  - name: add Ansible ssh key
    authorized_key:
      user: ansible
      key: "{{ lookup('file', '/home/vagrant/.ssh/id_ecdsa.pub') }}"

  - name: add ansible to sudoers
    copy:
      src: sudoer_ansible
      dest: /etc/sudoers.d/ansible
      owner: root
      group: root
      mode: 0440
EOF
```

Creating the sudoer file for the ansible user
```shell
mkdir files
cat > files/sudoer_ansible << EOF
ansible ALL=(ALL) NOPASSWD: ALL
EOF
```

Run the Ansible play we just created.
  Prior to running this play ensure that you enable your Proxmox subscription or moved to the free repository 
  https://pve.proxmox.com/wiki/Package_Repositories

```shell
ansible-playbook pve_onboard.yml -i inventory --user=root
```
Test to the see the newly created ansbile user can secure shell into the Proxmox via Ansible
```shell
ansible pvenodes -m ping -i inventory --user=ansible 
```

## On the Proxmox Node create the ansible user and token
  Save the SSH token, Proxmox will not provide it again. We will need it later.
```shell
pveum user add ansible@pam
pveum acl modify / --role Administrator --user 'ansible@pam' 
pveum user token add ansible@pam ansible-token --privsep false
pvesm set local --content images,rootdir,vztmpl,backup,iso,snippets

```

## Back to the Ansible node 
  Create a playbook install Proxmoxer, a wrapper around the APIS for Proxmox
  https://proxmoxer.github.io/docs/2.0/

```shell
cat > pve_install_proxmoxer.yml << EOF
- hosts: pvenodes
  become: true
  tasks:

  - name: update repository cache
    apt:
      update_cache: yes
      cache_valid_time: 3600

  - name: install proxmoxer
    apt:
      name:
      - python3-proxmoxer
      state: latest
EOF
```

Run the playbook to install Proxmoxer
```shell
ansible-playbook pve_install_proxmoxer.yml -i inventory --user=ansible
```


Create a playbook to install minimal VM
  Update the hosts, API token, api_host, and node
```shell
cat > pve_create_vm.yml << EOF
- hosts: 172.16.1.8
  become: false
  gather_facts: false
  tasks:

  - name: Create new vm with minimal options
    vars:
      ansible_python_interpreter: /usr/bin/python3
    proxmox_kvm:
      api_user: ansible@pam
      api_token_id: ansible-token
      api_token_secret: << your previuosly saved token>>
      api_host: 172.16.1.8
      node: node2
      name: vmtest
EOF
```
Run the playbook
```shell
ansible-playbook pve_create_vm.yml -i inventory --user=ansible
```
