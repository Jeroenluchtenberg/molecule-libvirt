**************************
Molecule libvirt-ng Plugin
**************************

.. image:: https://badge.fury.io/py/molecule-libvirt-ng.svg
   :target: https://badge.fury.io/py/molecule-libvirt-ng
   :alt: PyPI Package

.. image:: https://github.com/codanael/molecule-libvirt/actions/workflows/tox.yml/badge.svg
   :target: https://github.com/codanael/molecule-libvirt/actions/workflows/tox.yml
   :alt: CI Status

.. image:: https://img.shields.io/badge/code%20style-ruff-261230.svg
   :target: https://github.com/astral-sh/ruff
   :alt: Ruff Code Style

.. image:: https://img.shields.io/badge/license-MIT-brightgreen.svg
   :target: LICENSE
   :alt: Repository License

Molecule libvirt-ng is a `Molecule <https://molecule.readthedocs.io/>`_ driver
for provisioning test VMs via libvirt/QEMU/KVM.

This is a modernized fork of the archived
`molecule-libvirt <https://github.com/ansible-community/molecule-libvirt>`_
plugin, updated for current versions of Molecule and Ansible.

.. _requirements:

Requirements
============

* Python >= 3.10
* Molecule >= 24.2.0
* Ansible >= 2.16
* A working libvirt/QEMU/KVM setup on the host
* ``libvirt-python`` (system package or built from source with ``pkg-config`` and libvirt headers)
* ``qemu-img`` (from ``qemu-utils``/``qemu-img``, for disk resizing)

Required Ansible collections (installed automatically via ``test_requirements.yml``):

* ``community.libvirt``
* ``community.crypto``
* ``community.general``
* ``ansible.posix``
* ``ansible.utils``

.. _quickstart:

Quickstart
==========

Installation
------------

.. code-block:: bash

   pip install molecule-libvirt-ng

You also need ``libvirt-python`` which requires system libvirt headers:

.. code-block:: bash

   # Debian/Ubuntu
   sudo apt install libvirt-dev pkg-config python3-dev

   # RHEL/Fedora
   sudo dnf install libvirt-devel pkg-config python3-devel

   pip install libvirt-python

Install the required Ansible collections:

.. code-block:: bash

   ansible-galaxy collection install -r test_requirements.yml

Example
-------

Create a ``molecule/default/molecule.yml`` in your Ansible role:

.. code-block:: yaml

   ---
   driver:
     name: libvirt

   platforms:
     - name: debian12-test
       image_url: "https://cloud.debian.org/images/cloud/bookworm/latest/debian-12-genericcloud-amd64.qcow2"
       memory_size: 2
       vcpu: 2
       disk_size: "10G"
       qemu_user: "qemu"

   provisioner:
     name: ansible

   verifier:
     name: ansible

Then run:

.. code-block:: bash

   molecule test

.. _platform-parameters:

Platform Parameters
===================

Required
--------

``image_url``
   URL to a cloud image (qcow2 format). Defaults to Debian 12 genericcloud.

Optional
--------

``memory_size``
   RAM in GB. Default: **1**.

``vcpu``
   Number of virtual CPUs. Default: **1**.

``disk_size``
   Disk size (qemu-img format). Default: **15G**.

   The image is grown with ``qemu-img resize`` and the guest's root partition
   and filesystem are expanded on first boot by cloud-init's ``growpart``.
   The disk is only ever grown: a ``disk_size`` smaller than the cloud image's
   own virtual size is ignored.

``image_cache_path``
   Directory where downloaded cloud images are cached. Default:
   **~/.cache/molecule-libvirt/images** (override globally with the
   ``MOLECULE_LIBVIRT_IMAGE_CACHE`` environment variable).

``image_cache_force_update``
   Re-download the image even when it is already cached. Default: **false**.

``ssh_port``
   SSH port on the guest. Default: **22**.

``cpu_model``
   CPU model requested by the guest. Default: **qemu64**.

``arch``
   CPU architecture. Default: **x86_64**.

``timezone``
   VM timezone. Default: **America/Toronto**.

``qemu_user``
   QEMU process user for filesystem ACLs. Varies by distro:

   * RHEL/Fedora: **qemu** (default)
   * Debian/Ubuntu: **libvirt-qemu**
   * NixOS: **root**

``network_name``
   Use an existing libvirt network instead of creating a molecule-managed one.

``bridge_name``
   Use an existing bridge on a remote host. Makes the VM reachable via ARP.

``libvirt_host`` and ``libvirt_user``
   Provision VMs on a remote libvirt host via ``qemu+ssh://``.

Network Parameters
------------------

These apply when molecule manages its own network (no ``network_name`` or ``bridge_name``):

``molecule_bridge``
   Bridge interface name. Default: **molecule-br0**.

``molecule_network_cidr``
   IP range for the molecule network. Default: **10.10.10.0/24**.

.. _image-cache:

Image Cache
===========

Cloud images are downloaded once into a shared cache
(``~/.cache/molecule-libvirt/images`` by default) and copied from there into
each VM disk. ``molecule destroy`` removes the VM disks but leaves the cache
alone, so recreating an instance does not re-download the image.

The cache is keyed on the image URL, so different platforms and scenarios
share an entry when they use the same URL. To refresh an image that is
published under a moving URL (``.../latest/...``, ``.../current/...``), set
``image_cache_force_update: true`` on the platform, or simply delete the
cached file.

.. code-block:: bash

   # drop the whole cache
   rm -rf ~/.cache/molecule-libvirt/images

.. _documentation:

Documentation
=============

Read the Molecule documentation at https://molecule.readthedocs.io/.

.. _authors:

Authors
=======

Original authors:

* James Regis
* Gaetan Trellu
* Gariele Cerami
* Sorin Sbarnea

.. _license:

License
=======

The `MIT`_ License.

.. _`MIT`: https://github.com/codanael/molecule-libvirt/blob/main/LICENSE
