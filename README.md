Ansible role: maresb.micromamba
===============================

Installs micromamba, and optionally create a root/base conda environment.

Motivation
----------

[Conda](https://docs.conda.io/projects/conda) is a very powerful Python-centric dependency management tool. Unfortunately, for environments with large numbers of dependencies, its Python-based dependency solver can take [hours](https://github.com/iterative/dvc.org/issues/2370#issuecomment-818891218) to complete.

The new [Mamba](https://github.com/mamba-org/mamba) project addresses this issue by reimplementing the dependency solver in C++, and is lightning-fast. Apart from the solver, `mamba` delegates most tasks to the otherwise dependable `conda` tool.

[Micromamba](https://github.com/mamba-org/mamba#micromamba) is a highly experimental pure C++ package manager for conda environments. Because it has no Python dependencies, it can efficiently create environments for any version of Python from a single `micromamba` binary. If none of your conda packages has a Python dependency, then Micromamba will even create a conda environment without Python!

Micromamba eliminates the need for "distributions" such as Anaconda or Miniconda. You can set up your desired environment directly.

Role Variables
--------------

```yaml
arch: linux-64
version: latest
```

For the [latest architectures and version numbers](https://api.anaconda.org/release/conda-forge/micromamba/latest), check `distributions[#].basename`, which has the format `{arch}/micromamba-{version}.tar.bz2`. Current possible values for `arch` are `linux-64`, `linux-aarch64`, `osx-64`, `osx-arm64`, `win-64`. The format of `version` is either `latest` or something like `0.15.2-0`, where `-0` denotes the build number.

---

```yaml
dest: /usr/local/bin/micromamba
```

Location of the `micromamba` executable.

---

```yaml
root_prefix: /opt/conda
```

When the root prefix is defined and does not already exist, a new root prefix will be created in this location.

---

```yaml
packages:
  - mamba
  - python=3.9
```

A list of initial conda packages to be installed when a new root prefix is created.

Example Playbooks
-----------------

```yaml
- hosts: servers
  become: yes
  roles:
      - maresb.micromamba
```

This downloads the `micromamba` executable to the default location of `/usr/local/bin/micromamba`.

---

```yaml
- hosts: servers
  become: yes
  roles:
      - maresb.micromamba
  vars:
    dest: /tmp/micromamba
    root_prefix: /opt/conda
    packages:
      - mamba
      - python=3.9
```

This downloads `micromamba` into `/tmp/micromamba` and creates a new root prefix in `/opt/conda/` with Python 3.9 and Mamba.

---

```yaml
- hosts: servers
  become: yes
  become_user: conda-user
  roles:
      - maresb.micromamba
  vars:
    root_prefix: ~/micromamba
    packages:
      - s3fs-fuse
```

This creates a new root prefix in /home/conda-user/micromamba and creates a conda environment without Python.

Subsequent Usage
----------------

In order to be used, a conda environment must first be *activated*. Activation involves altering the `PATH` and other environment variables in the active shell. This can be accomplished with the commands

```bash
eval "$(micromamba shell hook --shell=bash)"
micromamba activate --prefix=/opt/conda
```

The first command executes a sequence of commands which defines a shell hook, also named `micromamba`. (Otherwise, the `micromamba` executable would run as a *subprocess* which is unable to modify the environment of the shell.) The second command runs the shell hook to activate the environment located at `/opt/conda`.

Since micromamba is experimental, it is advisable to install `mamba` and run

```bash
/opt/conda/bin/conda init bash
/opt/conda/bin/mamba init bash
```

These commands modify `~/.bashrc` so that upon next login the environment will be activated.

License
-------

MIT

Author Information
------------------

I'm fairly new to Ansible, so probably many things can be accomplished more efficiently than what I have done here. I am grateful for any advice, and especially pull requests.
