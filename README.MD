
# Proxmox VM Automation Role

This Ansible role simpifies and automates the process of spinning up VMs on Proxmox. It leverages Proxmox API and Ansible modules to streamline VM deployment.

## Inventory Configuration

Example inventory file:
```
node-example-01 ansible_host=10.0.100.1 vm_host=n1 storage=N1-NVME cpu_cores=24 memory_size_mb=65536 data_drive_fs=ext4 data_drive_size_gb=8000 data_drive_mount_path=/mnt/db data_drive_discard=on
```

^^^^^ the explanation
```yaml
node-example-01 # VM name
ansible_host=10.0.100.1 # VM FUTURE ip
vm_host=n1 # The name of the Proxmox NODE where the VM will be located
storage=N1-NVME # The name of the LOCAL STORAGE of Proxmox NODE where the VM will be located
cpu_cores=24 # VM cores
memory_size_mb=65536 # VM memory in Mb
data_drive_fs=ext4 # VM data drive FS type
data_drive_size_gb=1000 # VM data drive size
data_drive_mount_path=/mnt/db # VM data drive mountpoint
data_drive_discard=on # VM data drive discard option
```

## Variables

The `proxmox-vms-automation-vars.yml` file contains the following variables:

```yaml
api_host: '172.16.0.1' # Proxmox NODE ip address
api_user: 'root@pam' # Proxmox API user name
api_token_id: 'deploy' # Proxmox API token name
api_token_secret: 'xxxxxxxxxxx-proxmoxtoken-xxxxxxxxxxx' # Proxmox api access token, see https://www.youtube.com/watch?v=zMWqrdH8S1w for the details
vm_template_name: 'ubuntu-22.04-server-cloudimg-template' # VM template name must be located on a common NFS/iSCSI/etc. volume for all proxmox nodes (if more than one node is in the cluster)
vm_name: '{{ inventory_hostname }}' # VM name
vm_cpu_cores: '{{ cpu_cores | default(4) }}' # number of VM cores
vm_memory_size_mb: '{{ memory_size_mb | default(4096) }}' # The VM memory in Mb
vm_storage: '{{ storage | default("CLUSTER-NFS") }}' # shared storage inventory name
vm_data_drive_fs: '{{ data_drive_fs | default("xfs") }}' # FS type
vm_data_drive_size_gb: '{{ data_drive_size_gb | default(100) }}' # data drive size (default is 100Gb)
vm_data_drive_discard: '{{ data_drive_discard | default("ignore") }}' # discard option
vm_data_drive_mount_path: '{{ data_drive_mount_path | default("/mnt") }}' # data drive mount path
bridge_name: '{% if "10.0.100" in ansible_host %}vmbr100{% elif "10.0.200" in ansible_host %}vmbr200{% endif %}' # VM bridge name
#bridge_name: 10.0.100.XX -> vmbr100; 10.0.200.XX -> vmbr200; etc; the bridge must be configured and exist on the Proxmox node
vm_ip: '{{ ansible_host }}' # VM ip
vm_netmask: '24' # VM netmask
vm_gw: '{% if "10.0.100" in ansible_host %}10.0.100.254{% elif "10.0.200" in ansible_host %}10.0.200.254{% endif %}' # VM gateway
#vm_gw: 10.0.100.XX -> 10.0.100.254; 10.0.200.XX -> 10.0.200.254; etc; 
vm_dnsdomain: 'proxmox.example' # VM DNS search domain name
vm_user: 'automation' # VM tech user for future automation
vm_user_password: 'pAsSw0Rd' # VM tech user password
vm_user_ssh_key: 'ssh-ed25519 AAAAXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX' # VM tech user ssh key
nameservers: # VM nameservers
  - '1.1.1.1'
  - '8.8.8.8'
```

## Role Structure

The role is divided into three parts to handle different stages of VM creation:

### Part 1: Template Retrieval

This part retrieves information about the VM template and the specific VM.

```yaml
  - prox-vm-spinup-part1
```

### Part 2: VM Cloning and Configuration

This part ensures the template is placed on the target node and clones the template to create the VM.

```yaml
  - prox-vm-spinup-part2
```

### Part 3: Final Configuration

This part configures the cloned VM and completes the setup.

```yaml
  - prox-vm-spinup-part3
```

## Usage

1. Update the inventory file with the target node details.
2. Modify the variables in `proxmox-vms-automation-vars.yml` as needed.
3. Run the Ansible playbook:

```sh
ansible-playbook -i inventory proxmox-vms-automation.yml
```

## Spinning Up a VM
To spin up a VM, update the inventory file with the target node details.
Modify the variables in proxmox-vms-automation-vars.yml as needed.
Run the Ansible playbook:

```sh
ansible-playbook -i inventory proxmox-vms-automation.yml
```

## Authors

- [Evgenii Sudlenkov](https://github.com/sudlenkovea)
