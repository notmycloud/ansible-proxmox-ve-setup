# ansible-proxmox-ve-setup
Role to configure basic settings for Proxmox VE.

## Features
Disable the Enterprise Repo in favor of the Community Repo.  
Disable the Subscription Nag.  
Install the Dark Theme.

## Usage
playbook.yaml
```
---
- name: Play Name
  hosts: MyHosts
  
  roles:
    - role: notmycloud.proxmox_ve_setup
      vars:
        proxmox_ve_enterprise: # Set to true if you want to keep the enterprise repo
        proxmox_ve_lib_file: # Override the default location of the subscription nag file
```

## Support
For support, please raise an issue and provide the following items
- Sample task/playbook to replicate your issue
- Resultant file that is created.
- If modifying an existing file, please provide the unmodified version as well.
