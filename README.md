Role Name
=========

Install and configure the [Spack](https://spack.readthedocs.io/en/latest/index.html) souce-based package manager, and optionally packages using it.

- `TODO:` provide a way to add spack-installed compilers (not openhpc-packaged ones) automatically.
- `TODO:` uninstall packages removed from the list?
- `TODO:` run spack selfchecks on first install

After running this for the first time, any users defined in `spack_users` should resource `~/.bashrc` to make the `spack` command available and update `module av`.

Requirements
------------

An OpenHPC v2 cluster. v1 should also work but will need `spack_ohpc_compilers` changing.

Role Variables
--------------

The following variables will commonly need changing:

- `spack_root`: Optional. Path to install spack to. Default `/home/{{ ansible_user }}/spack`. **NB** This location should be on a cluster-shared filesystem.
- `spack_version`: Optional. Git tag etc of spack version to install. Default `HEAD`.
- `spack_packages`: Optional. List of spack specs describing packages to install using spack. Default is an empty list. Note that removing packages from this list does not uninstall them.

  E.g.:
    ```yaml
    - "openmpi@3.1.6 %gcc@9.3.0 fabrics=ucx schedulers=auto"
    ```
  In this example note that the `%gcc@9.3.0` requires a specific compiler, which is installed from OpenHPC and configured in spack by the defaults below.
- `spack_users`: Optional: List of users to make spack available to by modifying their `.bashrc`. The default is only `ansible_user`.

The following variables are unlikely to need changing:

- `spack_cmd`: Optional. Path to spack binary. Default is "{{ spack_root}}/bin/spack".
- `spack_ohpc_compilers`: Optional. A list of dicts describing openhpc compiler packages to install (see Table 6 of the OpenHPC [installation guide](https://github.com/openhpc/ohpc/releases/download/v2.0.GA/Install_guide-CentOS8-Warewulf-SLURM-2.0-aarch64.pdf)). The default is as follows and explains the keys:

  ```yaml
  - os_pkg: gnu9-compilers-ohpc # name of the OpenHPC compiler package
    module: gnu9/9.3.0          # the module it creates
    spack_pkg: gcc@9.3.0        # what `spack compiler list` shows this compiler as
  ```
  This is currently the only available compiler for OpenHPC v2 without additional licences so this should not need changing.

- `spack_module_config`: Optional. YAML for spack's `modules.yml` configuration file. The default configures `lmod` module system (as used by OpenHPC) with the compilers in `spack_ohpc_compilers` as core.
  

Dependencies
------------

None.

Example Playbook
----------------

This only needs to be run on the login node (assuming that `spack_root` is set to a location on a cluster-shared filesystem).

```yaml
- hosts: cluster_login  
  become: yes
  tasks:
    - import_role:
        name: ansible_role_spack
      vars:
        spack_root: /mnt/nfs/spack
```

License
-------

Apache 2

Author Information
------------------

Steve Brasier (steveb@stackhpc.com)
