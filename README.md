# Ansible role Ezeelogin

This role installs [Ezeelogin](https://www.ezeelogin.com/) standalone or cluster setups.

## Requirements

None.

## Role Variables

Variables, along with their default values, are listed in `defaults/main.yml`.

To setup an Ezeelogin cluster, make sure to define an `ezeelogin_cluster_id`.\
This is an arbitrary value to retrieve hosts of the same cluster.

## Usage

### Example Playbook

```yaml
---
- hosts: localhost
  become: true
  roles:
    - role: ansible-role-ezeelogin
```

### Server addition

Server addition through the API requires the [API to be manually enabled first](https://www.ezeelogin.com/kb/article/add-update-delete-servers-through-ezeelogin-api-257.html).\
As a result, no server addition will be performed while running this role.

It should be done through a different Ansible task/playbook:

```yaml
---
- name: Configure Ezeelogin target servers
  hosts:
    - ezeelogin_targets
  diff: True

  tasks:
    - name: Retrieve Ezeelogin standalone or master node
      set_fact:
        ezeelogin_master: "{{ groups['ezeelogin_servers']
                               | difference(groups['ezeelogin_slaves'])
                               | map('extract', hostvars)
                               | selectattr('ezeelogin_cluster_id', 'defined')
                               | selectattr('ezeelogin_cluster_id', 'equalto', ezeelogin_cluster_id)
                               | map(attribute='inventory_hostname')
                               | first }}"

    - name: Add Ezeelogin target server on master node
      import_role:
        name: ansible-role-ezeelogin
        tasks_from: add_server.yml
      delegate_to: "{{ ezeelogin_master }}"
      vars:
        ezeelogin_api_url: https://ezeelogin.local
        ezeelogin_api_secret: XXX
        server:
          name: "{{ inventory_hostname }}"
          ip_address: "{{ ansible_host }}"
          ssh_user: "{{ ansible_user | default('root') }}"
          ssh_port: "{{ ansible_port | default(22) }}"
          group: "{{ ezeelogin_server_group_name | default('unassigned') }}"
          enable_ssh: 'Y'
```

Variables that can be set while adding a server are listed in `vars/main.yml::ezelogin_server_addition_parameters`


## License

MIT

## Author Information

This role was created in 2020 by [jpiron](https://github.com/jpiron).
