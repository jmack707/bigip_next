# Ansible deploy BIG-IP Next

This playbook and group of roles deploys BIG-IP Next on Proxmox

## Getting Started by configuring the Ansible to access Proxmox
Follow the steps in the [Ansible Setup](/ansible-setup.md) document 

## Setup the Ansible directory
```shell
sudo mkdir /opt/ansible
chown -R labuser:labuser /opt/ansible
git clone https://github.com/jmack707/bigip_next.git /opt/ansible
```

## Modify the [inventory](/inventory) file
Change the pvenodes IP address to the address of your Proxmox host
```
[pvenodes]
172.16.1.254
```

## Modify the [pvenode/vars](/group_vars/pvenodes/vars) file
Update the highlighted variables. 

```
api_host: '{{inventory_hostname}}'
drive_storage: ***vm-storage***
drive_format: qcow2
snippets_storage: local
snippets_path: '/var/lib/vz/snippets/'
image_storage: local
linux_bridge1: ***vmbr100***
linux_bridge2: ***vmbr110***
linux_bridge3: ***vmbr120***
pve_node: **pve**

cm_image_name: ***BIG-IP-Next-CentralManager-20.2.1-0.3.25.qcow2***
cm_image_url: ***'url from myf5'***
cm_image_sum: ****'image check sum'****


next_image_name: ***BIG-IP-Next-20.2.1-2.430.2+0.0.48.qcow2***
next_image_url: ***'url from myf5'***

## Update the Ansible token in [pvenode/vars](/group_vars/pvenodes/vault)
api_user: ansible@pam
api_token_id: ansible-token
api_token_secret: ***'your token' ***
```

## VM Images in 110 [variable_files](/variable_files/vms/110.yml)
```
  - name: ***cm1***
    vmid: ***110***
    node: '{{pve_node}}'
    image_file: '{{cm_image_name}}'
    cores: 8
    memory: 16384
    ipv4mode: static
    ipv4_address: ***172.16.1.40/24***
    ipv4_gateway: ***172.16.1.1***
    state: new
```

## VM Images in 111 [variable_files](/variable_files/vms/111.yml)
```
  - name: ***next1***
    vmid: ***111***
    node: "{{pve_node}}" 
    image_file: '{{next_image_name}}'
    cores: 2
    memory: 8192
    ipv4mode: static
    ipv4_address: ***172.16.1.41/24***
    ipv4_gateway: ***172.16.1.1***
    state: new
    interface1: net1 
    if1_bridge: '{{linux_bridge1}}'
    interface2: net2
    if2_bridge: '{{linux_bridge2}}'
```

## VM Images in 112 [variable_files](/variable_files/vms/112.yml)
```  
  - name: ***next2***
    vmid: ***112***
    node: "{{pve_node}}" 
    image_file: '{{next_image_name}}'
    cores: 2
    memory: 8192
    ipv4mode: static
    ipv4_address: ***172.16.1.42/24***
    ipv4_gateway: ***172.16.1.1***
    state: new
    interface1: net1 
    if1_bridge: ***vmbr110***
    interface2: net2
    if2_bridge: ***vmbr120***
```