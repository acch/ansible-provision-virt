# ansible-provision-virt

Ansible playbook for provisioning virtual RHEL machines using libvirt (plain `virsh`).
Allows for automatic creation of KVM/QEMU guests using only [Ansible](https://www.ansible.com/) — no tool switching required.
Optimized for repeated instantiation of virtual machines: Performs Kickstart installation of a base volume for each guest, and subsequently creates an overlay volume for runnning the guest instance.

## Installation

1.  Copy the file [`_install.yml`](_install.yml) to your project directory (where all the other playbooks will be stored).

2.  Create a [GitHub access token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) by visiting <https://github.com/> → Settings → Developer settings → Personal access tokens → Generate new token ([direct link](https://github.com/settings/tokens/new)).

3.  Export your GitHub username and access token as environment variables:

    ```sh
    $ export GITHUB_USER="<username>"
    $ export GITHUB_TOKEN="<personal access token>"
    ```

    Note that your GitHub username is visible when you click your avatar in the top-right corner (Signed in as ...).

    You can (optionally) add above lines to you `~/.bashrc` to permanently store the secret, so that you don't have to re-run this step after logging out.

4.  Run the `_install.yml` playbook:

    ```sh
    $ ansible-playbook _install.yml
    ```

    This will download the latest release and extract it into a directory `provision/`. Running the `_install.yml` playbook again will update the directory to the latest release.

5.  It is recommended to exclude the `provision/` directory from your own Git repository by adding the following to `.gitignore`:

    ```sh
    # ansible-provision-virt release
    provision/
    ```

    This will ensure you're always using the latest release.

## Usage

1.  Once installed you can now import and use the provision playbook in your own playbooks. To use it you need to define a few parameters as variables:

    ```yaml
    - import_playbook: provision/main.yml
      vars:
        distro: RHEL-8.3
        guests:
          - name: test
            hypervisor: ukvm1
            interfaces:
              - device: ens3
                bridge: br0
                ip: 192.168.0.56
                netmask: 255.255.255.0
    ```

2.  The above is just a minimal example. It will create a virtual machine with 2 CPUs, 4 GB of memory and 10 GB of disk space. You can override these defaults like so:

    ```yaml
    - import_playbook: provision/main.yml
      vars:
        distro: RHEL-8.3
        guests:
          - name: test
            hypervisor: ukvm1
            cpu: 4
            memory: 8192
            disk: 20
            interfaces:
              - device: ens3
                bridge: br0
                ip: 192.168.0.56
                netmask: 255.255.255.0
                nameserver: 192.168.0.105
              - device: ens4
                bridge: br0
                ip: 9.155.118.56
                netmask: 255.255.240.0
                gateway: 9.155.112.1
              - device: ens5
                bridge: br1
                ip: 10.1.1.56
                netmask: 255.255.255.0
    ```

3.  The provision playbook will add all newly created virtual machine(s) to a group named `new_guests`. To further customize your hosts afterwards, add another play like so:

    ```yaml
    - import_playbook: provision/main.yml
      vars:
        ...

    - hosts: new_guests
      tasks:
        - ping:
        ...
    ```

### Optional variables

| Variable         | Choices / Default              | Description                                                                             |
| ---------------- | ------------------------------ | --------------------------------------------------------------------------------------- |
| `force_base`     | true \| false (Default: false) | Whether or not to (re-)install the virtual machine, even if a base-image already exists |
| `parallel`       | <integer> (Default: 3)         | Number of guests to install in parallel                                                 |
| `exclusive`      | true \| false (Default: false) | Whether or not to destroy all other virtual machines running on the hypervisor          |
| `extra_packages` | <list of strings>              | Per-guest list of additional packages to install                                        |
| `skip_install`   | true \| false (Default: false) | Whether or not to skip (re-)installation of the virtual machine (dry run)               |

## Copyright & License

Copyright Achim Christ, released under the terms of the [Apache License 2.0](LICENSE).
