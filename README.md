# ansilear
## Basic steps for LUN clone using 
1. Create dynamic inventory file using bash script
   1. Log in to storage controller using SSH
   2. take input `igroup` name
   3. list all LUNs mapped to igroup. run command `lun show -m -g <igroup_name>
   4. pass this list of luns as inventory to ansible playbook
2. take this inventoy and creat play book
3. task to run for each item to check if this lun needs to be increased or not. 
4. if the input is `yes` next ask for how much is the lun size.
5. before increasing lun, volume should be increased.
6. volume increment steps
    1. first volume name is to be extracted from the lun name.
    2. use `split` to identify volume name
    3. now increase volume by 1.5 times of lun size
    4. ``` use size property to increase
    - name: Modify volume with snapshot auto delete options
  na_ontap_volume:
    state: present
    name: vol_auto_delete
    snapshot_auto_delete:
      state: "on"
      commitment: try
      defer_delete: scheduled
      target_free_space: 30
      destroy_list: lun_clone,vol_clone
      delete_order: newest_first
    aggregate_name: "{{ aggr }}"
    vserver: "{{ vserver }}"
    hostname: "{{ netapp_hostname }}"
    username: "{{ netapp_username }}"
    password: "{{ netapp_password }}"
    https: False

7. Once LUN increase completed now write task to increase lun size command
```
    - name: Resize LUN
      netapp.ontap.na_ontap_lun:
        state: present
        name: ansibleLUN
        force_resize: true
        flexvol_name: ansibleVolume
        vserver: ansibleVServer
        size: 5
        size_unit: gb
        hostname: "{{ netapp_hostname }}"
        username: "{{ netapp_username }}"
        password: "{{ netapp_password }}"
 ```
### Snapmirror playbook for data migrations
1. create dynamic inventory using bash
  1. Log in to storage controller via SSH
  2. take input server name `igroup`
  3. list all LUNs mapped to `igroup`
  4. pass the list of luns as inventory to ansible playbook
2. take this dynamic inventory pass to playbook
3. create a task to create snapmirrors relation ships for data migrations
```
  - name: Create SnapMirror
  na_ontap_snapmirror:
    state: present
    source_volume: test_src
    destination_volume: test_dest
    source_vserver: ansible_src
    destination_vserver: ansible_dest
    schedule: hourly
    policy: MirrorAllSnapshots
    hostname: "{{ netapp_hostname }}"
    username: "{{ netapp_username }}"
    password: "{{ netapp_password }}"
``` 
