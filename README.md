# ansible-role-ros

An Ansible role to set up ROS or ROS 2.

## Intro

This role automatically sets up a ROS (2) environment.
It automatically detects the Ubuntu distribution and installs the corresponding ROS LTS distribution.

## Variables

- `ros_version` (optional) whether to install ROS or ROS 2.
  Defaults to `'2'`.
- `ros_variant` (optional) Install a ROS variant as defined in [REP 131][rep131].
  Defaults to `'ros-base'`.

## Use

### From an Ansible playbook

Install the role from GitHub,

```bash
ansible-galaxy install https://github.com/artivis/ansible-role-ros.git
```

Create an example playbook `my-playbook.yml`,

```yaml
---
  - hosts: localhost
    connection: local
    roles: [ansible-role-ros]
```

and execute it,

```bash
ansible-playbook -i localhost my-playbook.yml -e 'ros_version=1'
```

### Through cloud-init

Create a [`cloud-init`][cloud-init] configuration file `my-cloud-init.yaml`,

```yaml
#cloud-config

# Import SSH key from GitHub
ssh_import_id:
  - gh:my-github-user

# Earlier version of the ansible module
# doesn't install pip automatically.
# Let's avoid any surprise
packages:
  - python3-pip

# We need an Ansible playbook to run.
write_files:
  - content: |
      ---
      - hosts: localhost
        connection: local
        roles: [ansible-role-ros]
    path: /home/ubuntu/ros-playbook.yml
    owner: ubuntu:ubuntu
    permissions: '0444'
    defer: true

# Install Ansible from pip so that we can run it as a given user.
# It also retrieve the Ansible ROS role straight from GitHub.
# Finally execute the playbook created above.
ansible:
  install_method: pip
  package_name: ansible
  run_user: ubuntu

  galaxy:
    actions:
      - ["ansible-galaxy", "install", "https://github.com/artivis/ansible-role-ros.git"]

  setup_controller:
    run_ansible:
      - playbook_dir: /home/ubuntu
        playbook_name: "ros-playbook.yml"
        extra_vars: "'ros_version=1 ros_variant=robot'"
```

#### Set up ROS in a Linux container

Launch a Linux container using [LXD][lxd] with the `cloud-init` configuration,

```bash
lxc launch ubuntu:20.04 lxc-ros-noetic --config=user.user-data="$(cat ./my-cloud-init.yaml)"
```

While we could connect to the freshly spawn container,
we better wait for `cloud-init` to finish,

```bash
lxc exec lxc-ros-noetic -- cloud-init status --wait
```

#### Set up ROS in an Ubuntu VM

Launch an Ubuntu virtual machine with [multipass] with the `cloud-init` configuration,

```bash
multipass launch focal -n vm-ros-noetic --cloud-init my-cloud-init.yaml
```

[rep131]: https://ros.org/reps/rep-0131.html#ros-variants
[cloud-init]: https://cloud-init.io
[lxd]: https://ubuntu.com/lxd
[multipass]: https://multipass.run
