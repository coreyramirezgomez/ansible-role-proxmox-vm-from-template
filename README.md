Proxmox VM from Template
=========

Ansible task sequence to create Proxmox Virtual Machine from an existing template.
Features:
  - cloud-init user-data file deployment.
  - Add/update cloud-init drive.
  - Update networking (qm), storage (qm), cpu (proxmoxer), memory (proxmoxer) on resulatant VM.

Requirements
------------

- Full privilege API Credentials to a proxmox host
- Privileges to install package requirements on proxmox host, see vars/main.yml.
- Privileges to install pip package requirements on proxmox host, see vars/main.yml.

Role Variables
--------------

See defaults/main.yml

Dependencies
------------

Sort of assumes you have used my other role: coreyramirezgomez.proxmox_vm_templater - but not entirely required, this should work with any template.

Playbook - proxmox_vm_from_template_meta_host: false
----------------
    - hosts: proxmox_hosts
      roles:
        - role: coreyramirezgomez.proxmox_vm_from_template
          proxmox_vm_from_template_meta_host: false
          proxmox_vm_from_template_state: "present"
          proxmox_vm_from_template_startup: true 
          proxmox_vm_from_template_format: "raw"
          ### Required
          proxmox_vm_from_template_api_token_id: "TOKEN_ID"
          proxmox_vm_from_template_api_token_secret: "MUCH_SECRET"
          proxmox_vm_from_template_api_user: "root@pam"
          proxmox_vm_from_template_root_user_password: "root-password"
          proxmox_vm_from_template_target_host: "proxmox.example.com"
          proxmox_vm_from_template_target_node: "proxmox"
          proxmox_vm_from_template_dest_id: "100"
          proxmox_vm_from_template_dest_name: "vm-name"
          proxmox_vm_from_template_src_id: "101"
          proxmox_vm_from_template_src_name: "template-name"
          proxmox_vm_from_template_storage: "local-lvm"
          ### Optional
          Update the VM with new specs after clone:
          proxmox_vm_from_template_update:
            cores: 4
            sockets: 1
            memory: 4096
          proxmox_vm_from_template_main_disk_size: "20G"
          proxmox_vm_from_template_disks:
            "scsi1":
            size: "32"
            storage: "local-lvm"
          proxmox_vm_from_template_networks:
            "net0":
            model: "virtio"
            bridge: "vmbr0"
            firewall: "0"
            macaddr: "70:55:7F:83:2D:B1"
            tag: "10"
          proxmox_vm_from_template_ci_user_data_file_path: "/var/lib/vz/snippets/"
          proxmox_vm_from_template_ci_user_data_file_name: "{{ inventory_hostname_short }}-user-data"
          proxmox_vm_from_template_ci_user_data_volume: "local-lvm"
          proxmox_vm_from_template_ci_user_data_src_file: "files/user-data.txt"


Playbook - proxmox_vm_from_template_meta_host: true
----------------
  1. In your inventory, define the new VM:

    ...
    [proxmox_vms]
    vm-name
    ...

  2. In your inventory, create a host_vars for the new VM:

    inventories/default
    |-- host_vars
    |   |-- vm-name
    |   |   |-- coreyramirezgomez.proxmox_vm_from_template.yml

  3. Place all the configuration for the new VM into the file coreyramirezgomez.proxmox_vm_from_template.yml:

    ---
    proxmox_vm_from_template_meta_host: false
    proxmox_vm_from_template_state: "present"
    proxmox_vm_from_template_startup: true 
    proxmox_vm_from_template_format: "raw"
    ### Required
    proxmox_vm_from_template_api_token_id: "TOKEN_ID"
    proxmox_vm_from_template_api_token_secret: "MUCH_SECRET"
    proxmox_vm_from_template_api_user: "root@pam"
    proxmox_vm_from_template_root_user_password: "root-password"
    proxmox_vm_from_template_target_host: "proxmox.example.com"
    proxmox_vm_from_template_target_node: "proxmox"
    proxmox_vm_from_template_dest_id: "100"
    proxmox_vm_from_template_dest_name: "vm-name"
    proxmox_vm_from_template_src_id: "101"
    proxmox_vm_from_template_src_name: "template-name"
    proxmox_vm_from_template_storage: "local-lvm"
    ### Optional
    Update the VM with new specs after clone:
    proxmox_vm_from_template_update:
      cores: 4
      sockets: 1
      memory: 4096
    proxmox_vm_from_template_main_disk_size: "20G"
    proxmox_vm_from_template_disks:
      "scsi1":
      size: "32"
      storage: "local-lvm"
    proxmox_vm_from_template_networks:
      "net0":
      model: "virtio"
      bridge: "vmbr0"
      firewall: "0"
      macaddr: "70:55:7F:83:2D:B1"
      tag: "10"
    proxmox_vm_from_template_ci_user_data_file_path: "/var/lib/vz/snippets/"
    proxmox_vm_from_template_ci_user_data_file_name: "{{ inventory_hostname_short }}-user-data"
    proxmox_vm_from_template_ci_user_data_volume: "local-lvm"
    proxmox_vm_from_template_ci_user_data_src_file: "files/user-data.txt"

  4. Define a playbook to deploy with the role:

    - hosts: proxmox_hosts
      become: true
      gather_facts: false # This is required otherwise ansible will try to establish connection to the VM, but we haven't even created it yet. So skip over it.
      roles:
        - role: coreyramirezgomez.proxmox_vm_from_template