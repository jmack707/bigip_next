# Ansible deploy BIG-IP Next

This playbook and group of roles deploys BIG-IP Next on Proxmox

## Requirements 
The below requirements are needed on the host that executes this collection:

 - proxmoxer
 - requests

## Roles

* [user files][roles/create_user_files]
* [variables file][roles/create_variables_file]
* [create VMs][roles/create_vms]
* [download cloud_init][roles/download_cloud_init_image]
* [remove variable files][roles/remove_variable_file]
* [remove VMs][roles/remove_vms]
* [stop VMs][roles/stop_vms]
* [untar images][roles/untar_images]
