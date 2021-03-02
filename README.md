stackhpc.spack
=========

Install and configure the [Spack](https://spack.readthedocs.io/en/latest/index.html) source-based package manager, and optionally packages using it.

- `TODO:` provide a way to add spack-installed compilers (not openhpc-packaged ones) automatically.
- `TODO:` uninstall packages removed from the list?
- `TODO:` run spack selfchecks on first install

After running this for the first time, any users defined in `spack_users` should re-source `~/.bashrc` to make the `spack` command available and update `module avail`.

Requirements
------------

- Must be run with `become: true`.
- The defaults for `spack_system_compilers` assume OpenHPC v2 system.
- For installing on a cluster, a shared filesystem and shared `/home` is required.

Role Variables
--------------

The following variables will commonly need changing:

- `spack_root`: Optional. Path to install spack to. Default `/home/{{ ansible_user }}/spack` (note a final directory is included) - for a cluster this should be on a shared filesystem.
- `spack_version`: Optional. Git tag etc of spack version to install. Default `HEAD`.
- `spack_packages`: Optional. List of [spack specs](https://spack.readthedocs.io/en/latest/basic_usage.html#specs-dependencies) describing packages to install using spack. Default is an empty list. Note that removing packages from this list does not uninstall them.

  E.g.:
    ```yaml
    - "openmpi@3.1.6 %gcc@9.3.0 fabrics=ucx schedulers=auto"
    ```
  
  On an OpenHPC v2 system this installs a version of OpenMPI using the UCX transport layer which integrates with the (OpenHPC-installed) Slurm. Note that `%gcc@9.3.0` requires Spack to use a specific compiler, which is installed from the OpenHPC repositories and configured in spack by the defaults below. While OpenHPC does provide an openmpi+ucx package, it can be hard to integrate this with further Spack packages.
- `spack_users`: Optional: List of users to make spack available to by modifying their `.bashrc`. The default is only `{{ ansible_user }}`.

The following variables are unlikely to need changing:

- `spack_cmd`: Optional. Path to spack binary. Default is "{{ spack_root}}/bin/spack".
- `spack_system_compilers`: Optional. A list of dicts describing compiler packages to install from the system package manager - these are installed and added as spack compilers. The default is as follows and explains the keys:

  ```yaml
  - system_pkg: gnu9-compilers-ohpc # name of the system-package-manager compiler package, in this case from openhpc
    module: gnu9/9.3.0          # the module it creates
    spack_pkg: gcc@9.3.0        # what `spack compiler list` shows this compiler as
  ```
  
  For OpenHPC v2 systems the above is currently the only compiler available without additional licences (see Table 6 of OpenHPC [installation guide](https://github.com/openhpc/ohpc/releases/download/v2.0.GA/Install_guide-CentOS8-Warewulf-SLURM-2.0-aarch64.pdf), so the default is appropriate on such systems.

- `spack_module_config`: Optional. YAML for spack's `modules.yml` configuration file. The default configures `lmod` module system (as used by OpenHPC) with all compilers in `spack_system_compilers` as core.
  

Dependencies
------------

None.

Example Playbook
----------------

This should be run on any nodes from which you want to be able to build spack packages `spack`, i.e. login nodes for a cluster.

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
