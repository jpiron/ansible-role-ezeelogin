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

### Server addition/deletion

Server addition/deletion through the API requires the [API to be manually enabled first](https://www.ezeelogin.com/kb/article/add-update-delete-servers-through-ezeelogin-api-257.html).\
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
      delegate_to: "{{ ezeelogin_master }}"
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

    - name: Remove Ezeelogin target server on master node
      import_role:
        name: ansible-role-ezeelogin
        tasks_from: add_server.yml
      delegate_to: "{{ ezeelogin_master }}"
      vars:
        ezeelogin_api_url: https://ezeelogin.local
        ezeelogin_api_secret: XXX
        server:
          name: foobar
          state: absent
```

Variables that can be set while adding a server are listed in `vars/main.yml::ezelogin_server_addition_parameters`

### ACLs

This role provides a taskset `acls.yml` to manage Ezeelogin ACLs. \
Currently, only user->server and user->server group ACLs are managed.

As the user and server group creations aren't managed by this roles yet, the ACLs aren't created when
running this role.

It should be done through a different Ansible task/playbook:

```yaml
---
- name: Configure Ezeelogin ACLs
  hosts:
    - ezeelogin_servers
  diff: True

  tasks:
    - name: Configure Ezeelogin ACLs
      import_role:
        name: ansible-role-ezeelogin
        tasks_from: acls.yml
      vars:
        ezlogin_database_table_prefix: foobarbaz
        ezeelogin_acls:
          - username: username_01
            ezeelogin_server_groups:
              - name: prod
                state: absent
              - name: test
                state: present
            ezeelogin_servers:
              - name: server_01
                state: absent
              - name: server_01
                state: present
```

## License

MIT

## Author Information

This role was created in 2020 by [jpiron](https://github.com/jpiron).
