Virtual Machines
================

.. contents:: Table of Contents

libvirt
-------

"libvirt" provides a framework and API for accessing and controlling
different virtualization hypervisors. This Root Pages' guide assumes
that libvirt is used for managing Quick Emulator (QEMU) virtual
machines. [1]

VNC
~~~

Any virtual machine can be accessed remotely via a VNC GUI. Shutdown the virtual machine with ``virsh shutdown`` and then run ``virsh edit ${VM}``.

Examples:

Automatically assign a VNC port number (starting at 5900/TCP) and listen on every IP address.

.. code-block:: xml

   <domain>
       <devices>
           <graphics type='vnc' port='-1' autoport='yes' listen='0.0.0.0'/>
       </devices>
   </domain>

Assign a static port number, listen only on localhost, and password protect the VNC console. The password will be stored in plaintext on the file system.

.. code-block:: xml

   <domain>
       <devices>
           <graphics type='vnc' port='5987' autoport='no' listen='127.0.0.1' passwd='securepasswordhere'/>
       </devices>
   </domain>

[50]

Hardware Virtualization
-----------------------

Hardware virtualization speeds up and further isolates virtualized environments. Most newer CPUs support this. There is "Intel VT (Virtualization Technology)" and "AMD SVM (Secure Virtual Machine)" for x86 processors. Hardware virtualization must be supported by both the motherboard and processor. It should also be enabled in the BIOS. [2]

Intel has three subtypes of virtualization:

-  VT-x = Basic hardware virtualization and host separation support.
-  VT-d = I/O pass-through support.
-  VT-c = Improved network I/O pass-through support.

[3]

AMD has two subtypes of virtualization:

-  AMD-V = Basic hardware virtualization and host separation support.
-  AMD-Vi = I/O pass-through support.

Check for Intel or AMD virtualization support. If a result is found, then virtualization is supported by the processor but may still need to be enabled via the motherboard BIOS.

.. code-block:: sh

    $ grep -m 1 --color vmx /proc/cpuinfo # Intel

.. code-block:: sh

    $ grep -m 1 --color svm /proc/cpuinfo # AMD

Verify the exact subtype of virtualization:

.. code-block:: sh

    $ lscpu | grep ^Virtualization # Intel or AMD

KVM
~~~

The "Kernel-based Virtual Machine (KVM)" is the default kernel module
for handling hardware virtualization in Linux since the 2.6.20 kernel.
[4] It is used to accelerate the QEMU hypervisor. [5]

Fedora installation:

-  Install KVM and Libvirt. Add non-privileged users to the "libvirt" group to be able to manage virtual machines through ``qemu:///system``. By default, users can only manage them through ``qemu:///session`` which has limited configuration options.

.. code-block:: sh

    $ sudo dnf -y install qemu-kvm libvirt
    $ sudo systemctl enable --now libvirt
    $ sudo groupadd libvirt
    $ sudo usermod -a -G libvirt $USER

Performance Tuning
^^^^^^^^^^^^^^^^^^

Processor
'''''''''

Configuration details for virtual machines can be modified to provide
better performance. For processors, it is recommended to use the same
CPU settings so that all of it's features are available to the guest.
[6]

QEMU:

.. code-block:: sh

    $ sudo qemu -cpu host ...

libvirt:

.. code-block:: sh

    $ sudo virsh edit <VIRTUAL_MACHINE>
    <cpu mode='host-passthrough'/>

Memory
''''''

Enable Huge Pages and disable Transparent Hugepages (THP) on the hypervisor for better memory performance in virtual machines.

View current Huge Pages allocation. The total should be "0" if it is disabled. The default size is 2048 KB on Fedora.

.. code-block:: sh

    $ grep -i hugepages /proc/meminfo
    AnonHugePages:         0 kB
    ShmemHugePages:        0 kB
    HugePages_Total:       0
    HugePages_Free:        0
    HugePages_Rsvd:        0
    HugePages_Surp:        0
    Hugepagesize:       2048 kB

Calculate the optimal Huge Pages total based on the amount of RAM that will be allocated to virtual machines. For example, if 24GB of RAM will be allocated to virtual machines then the Huge Pages total should be set to ``12288``.

::

    <AMOUNT_OF_RAM_FOR_VMS_IN_KB> / <HUGEPAGES_SIZE> = <HUGEPAGES_TOTAL>

Enable Huge Pages by setting the total in sysctl.

.. code-block:: sh

    $ sudo vim /etc/sysctl.conf
    vm.nr_hugepages = <HUGEPAGES_TOTAL>
    $ sudo sysctl -p
    $ sudo mkdir /hugepages
    $ sudo vim /etc/fstab
    hugetlbfs    /hugepages    hugetlbfs    defaults    0 0

Huge Pages must be configured to be used by the virtualization software. The hypervisor isolates and reserves the Huge Pages RAM and will otherwise make the memory unusable by other resources.

libvirt:

.. code-block:: xml

    <domain type='kvm'>
        <memoryBacking>
            <hugepages/>
        </memoryBacking>
    </domain>

Disable THP using GRUB.

File: ``/etc/default/grub``

.. code-block:: sh

    GRUB_CMDLINE_LINUX="<EXISTING_OPTIONS> transparent_hugepage=never"

Rebuild the GRUB configuration.

-  Debian:

   .. code-block:: sh

      $ sudo update-grub

-  Fedora UEFI:

   .. code-block:: sh

       $ sudo grub2-mkconfig -o /boot/efi/EFI/<OPERATING_SYSTEM>/grub.cfg

-  Fedora BIOS:

   .. code-block:: sh

       $ sudo grub2-mkconfig -o /boot/grub2/grub.cfg

Alternatively, THP can be manually disabled. Note that if the GRUB method is used, it will set "enabled" to "never" on boot which means "defrag" does not need to be set to "never" since it is not in use. This manual method should be used on systems that will not be rebooted.

.. code-block:: sh

    $ echo never | sudo tee /sys/kernel/mm/transparent_hugepage/enabled
    $ echo never | sudo tee /sys/kernel/mm/transparent_hugepage/defrag

In Fedora, services such as ktune and tuned will, by default, force THP to be enabled. Profiles can be modified in ``/usr/lib/tuned/`` on Fedora or in ``/etc/tune-profiles/`` on <= RHEL 7.

Increase the security limits in Fedora to allow the maximum valuable of RAM (in kilobytes) for a virtual machine that can be used with Huge Pages.

File: /etc/security/limits.d/90-mem.conf

::

    soft memlock 25165824
    hard memlock 25165824

Reboot the server and verify that the new settings have taken affect.

.. code-block:: sh

    $ grep -i huge /proc/meminfo
    AnonHugePages:         0 kB
    ShmemHugePages:        0 kB
    HugePages_Total:    8192
    HugePages_Free:        0
    HugePages_Rsvd:        0
    HugePages_Surp:        0
    Hugepagesize:       2048 kB
    Hugetlb:        16777216 kB

[33]

Network
'''''''

The network driver that provides the best performance is "virtio." Some
guests may not support this feature and require additional drivers.

QEMU:

.. code-block:: sh

    $ sudo qemu -net nic,model=virtio ...

libvirt:

.. code-block:: sh

    $ sudo virsh edit <VIRTUAL_MACHINE>
    <interface type='network'>
      ...
      <model type='virtio' />
    </interface>****

Using a tap device (that will be assigned to an existing interface) or a
bridge will speed up network connections.

QEMU:

.. code-block:: sh

    ... -net tap,ifname=<NETWORK_DEVICE> ...

.. code-block:: sh

    ... -net bridge,br=<NETWORK_BRIDGE_DEVICE> ...

libvirt:

.. code-block:: sh

    $ sudo virsh edit <VIRTUAL_MACHINE>
        <interface type='bridge'>
    ...
          <source bridge='<BRIDGE_DEVICE>'/>
          <model type='virtio'/>
        </interface>

Storage
'''''''

**virtio**

Raw disk partitions have the greatest speeds with the "virtio" driver, cache disabled, and the I/O mode set to "native." If a sparsely allocated storage device is used for the virtual machine (such as a thin-provisioned QCOW2 image) then the I/O mode of "threads" is preferred. This is because with "native" some writes may be temporarily blocked as the sparsely allocated storage needs to first grow before committing the write. [20]

QEMU:

-  Block:

   .. code-block:: sh

       $ sudo qemu -drive file=<PATH_TO_STORAGE_DEVICE>,cache=none,aio=threads,if=virtio ...

-  CDROM:

   .. code-block:: sh

      $ sudo qemu -cdrom <PATH_TO_CDROM>

libvirt:

-  Block:

   .. code-block:: xml

      <disk type='block' device='disk'>
            <driver name='qemu' type='raw' cache='none'/>
            <source dev='/dev/sr0'/>
            <target dev='vdb' bus='virtio'/>
      </disk>

-  CDROM:

   .. code-block:: xml

      <disk type='block' device='cdrom'>
        <driver name='qemu' type='raw'/>
        <source dev='/dev/sr0'/>
        <target dev='hdc' bus='ide'/>
        <readonly/>
      </disk>

Virsh:

-  Block:

   .. code-block:: sh

      $ virsh attach-disk <VM_NAME> --source <SOURCE_BLOCK_DEVICE> --target <DESTINATION_BLOCK_DEVICE> --cache none --persistent

-  CDROM:

   .. code-block:: sh

      $ virsh attach-disk <VM_NAME> /dev/sr0 vdb --config --type cdrom --mode readonly

[6][7][51]

**QCOW2**

When using the QCOW2 image format, create the image using metadata
preallocation or else there could be up to a x5 performance penalty. [8]

.. code-block:: sh

    $ qemu-img create -f qcow2 -o size=<SIZE>G,preallocation=metadata <NEW_IMAGE_NAME>

If using a file system with copy-on-write capabilities, either (1) disable copy-on-write functionality of the QCOW2 when creating the file or (2) prevent the QCOW2 file from being part of the copy-on-write for the underlying file system.

1. Create a QCOW2 file without copy-on-write.

   .. code-block:: sh

      $ qemu-img create -f qcow2 -o size=<SIZE>G,preallocation=metadata,nocow=on <NEW_IMAGE_NAME>

2. Or prevent the file system from using its copy-on-write functionality for the QCOW2 file or directory where the QCOW2 files are stored.

   .. code-block:: sh

      $ chattr +C <FILE_OR_DIRECTORY>

PCI
'''

If possible, PCI pass-through provides the best performance as there is
no virtualization overhead. The "GPU Pass-through" section expands upon this.

QEMU:

.. code-block:: sh

    $ sudo qemu -net none -device vfio-pci,host=<PCI_DEVICE_ADDRESS> ...

Networking
^^^^^^^^^^

Different models of virtual network interface cards (NICs) are available for the purposes of compatibility with the virtualized operating system. This can be set using the follow syntax:

.. code-block:: sh

   $ sudo qemu -net nic,model=<MODEL>

.. code-block:: sh

   $ sudo virt-install --network network=default,model=<MODEL>

Supported virtual device models [47]:

-  e1000 = The default NIC. It emulates a 1 Gbps Intel NIC.
-  virtio = High-performance device for operating systems with the driver available. Most Linux distributions has this driver available by default.
-  rtl8139 = An old NIC for older operating systems. It emulates a 100 Mbps Realtek 8139 card.
-  vmxnet3 = Use for VMware virtual machines and the VMware ESXi hypervisor. It emulates a virtual VMware NSXi NIC.

Nested Virtualization
^^^^^^^^^^^^^^^^^^^^^

KVM supports nested virtualization. This allows a virtual machine full
access to the processor to run another virtual machine in itself. This
is disabled by default.

Verify that the computers' processor supports nested hardware virtualization. [11] If a result is found, then virtualization is supported by the processor but may still need to be enabled via the motherboard BIOS.

-  Intel:

   .. code-block:: sh

       $ grep -m 1 --color vmx /proc/cpuinfo

-  AMD:

   .. code-block:: sh

       $ grep -m 1 --color svm /proc/cpuinfo

Newer processors support APICv which allow direct hardware calls to go straight to the motherboard's APIC. This can provide up to a 10% increase in performance for the processor and storage. [18] Verify if this is supported on the processor before trying to enable it in the relevant kernel driver. [19]

.. code-block:: sh

    $ dmesg | grep x2apic
    [    0.062174] x2apic enabled

Option #1 - Modprobe

-  Intel

File: /etc/modprobe.d/nested_virtualization.conf

   ::

       options kvm-intel nested=1
       options kvm-intel enable_apicv=1

   .. code-block:: sh

       $ sudo modprobe -r kvm-intel
       $ sudo modprobe kvm-intel

-  AMD

File: /etc/modprobe.d/nested_virtualization.conf

   ::

       options kvm-amd nested=1
       options kvm-amd enable_apicv=1

   .. code-block:: sh

       $ sudo modprobe -r kvm-amd
       $ sudo modprobe kvm-amd

Option #2 - GRUB2

Append this option to the already existing "GRUB\_CMDLINE\_LINUX"
options.

-  Intel

File: /etc/default/grub

   ::

       GRUB_CMDLINE_LINUX="kvm-intel.nested=1"

-  AMD

File: /etc/default/grub

   ::

       GRUB_CMDLINE_LINUX="kvm-amd.nested=1"

-  Then rebuild the GRUB 2 configuration.

  -  UEFI:

    .. code-block:: sh

        $ sudo grub2-mkconfig -o /boot/efi/EFI/<OPERATING_SYSTEM>/grub.cfg

  -  BIOS:

     .. code-block:: sh

         $ sudo grub2-mkconfig -o /boot/grub2/grub.cfg

[9]

Edit the virtual machine's XML configuration to change the CPU mode to
be "host-passthrough."

.. code-block:: sh

    $ sudo virsh edit <VIRTUAL_MACHINE>
    <cpu mode='host-passthrough'/>

[10]

Reboot the virtual machine and verify that the hypervisor and the
virtual machine both report the same capabilities and processor
information.

.. code-block:: sh

    $ sudo virsh capabilities

Finally verify that, in the virtual machine, it has full hardware
virtualization support.

.. code-block:: sh

    $ sudo virt-host-validate

OR

-  Intel:

   .. code-block:: sh

       $ cat /sys/module/kvm_intel/parameters/nested
       Y

-  AMD:

   .. code-block:: sh

       $ cat /sys/module/kvm_amd/parameters/nested
       Y

[11]

GPU Pass-through
^^^^^^^^^^^^^^^^

GPU pass-through provides a virtual machine guest with full access to a graphics card. It is required to have two video cards, one for host/hypervisor and one for the guest. [12] Hardware virtualization via VT-d (Intel) or SVM (AMD) is also required along with input-output memory management unit (IOMMU) support. Those settings can be enabled in the BIOS/UEFI on supported motherboards. Components of a motherboard are separated into different IOMMU groups. For GPU pass-through to work, every device in the IOMMU group has to be disabled on the host with a stub kernel driver and passed through to the guest. For the best results, it is recommended to use a motherboard that isolates each connector for the graphics card, usually a PCI slot, into it's own IOMMU group. The QEMU settings for the guest should be configured to use "SeaBIOS" for older cards or "OVMF" for newer cards that support UEFI. [36]

Enable IOMMU on the hypervisor via the bootloader's kernel options. This will provide a static ID to each hardware device. The "vfio-pci" kernel module also needs to start on boot.

File: ``/etc/default/grub``

-  Intel:

   ::

      GRUB_CMDLINE_LINUX="<EXISTING_OPTIONS> intel_iommu=on"

-  AMD:

   ::

      GRUB_CMDLINE_LINUX="<EXISTING_OPTIONS> amd_iommu=on"

For the GRUB bootloader, rebuild the configuration.

-  Fedora UEFI:

   .. code-block:: sh

      $ sudo grub2-mkconfig -o /boot/efi/EFI/<OPERATING_SYSTEM>/grub.cfg

-  Fedora BIOS:

   .. code-block:: sh

      $ sudo grub2-mkconfig -o /boot/grub2/grub.cfg

-  Find the IOMMU number for the graphics card. This should be the last alphanumeric set at the end of the line for the graphics card. The format should look similar to `XXXX:XXXX`. Add it to the options for the "vfio-pci" kernel module. This will bind a stub kernel driver to the device so that Linux does not use it.

Syntax:

.. code-block:: sh

   $ sudo lspci -k -nn -v | less
   $ sudo vim /etc/modprobe.d/vfio.conf
   options vfio-pci ids=XXXX:XXXX,YYYY:YYYY,ZZZZ:ZZZZ

Example:

.. code-block:: sh

   $ sudo lspci -k -nn -v | less
   09:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Hawaii XT / Grenada XT [Radeon R9 290X/390X] [1002:67b0] (prog-if 00 [VGA controller])
   09:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Hawaii HDMI Audio [Radeon R9 290/290X / 390/390X] [1002:aac8]
   $ sudo vim /etc/modprobe.d/vfio.conf
   options vfio-pci ids=1002:67b0,1002:aac8

-  For older AMD cards only, install the `vendor-reset kernel module <https://github.com/gnif/vendor-reset>`__ to guarantee that the AMD graphics card will not turn off when no kernel driver is bound to it. Ensure the ``vendor-reset`` module is loaded early on in the boot cycle.

   .. code-block:: sh

      $ git clone https://github.com/gnif/vendor-reset.git
      $ cd vendor-reset
      $ dkms install .
      $ echo "vendor-reset" | sudo tee -a /etc/modules

-  Rebuild the initramfs to include the VFIO related drivers.

   -  Debian:

      .. code-block:: sh

         $ echo -e "vfio\nvfio_iommu_type1\nvfio_pci" | sudo tee -a /etc/initramfs-tools/modules
         $ sudo update-initramfs -u -k all

   -  Fedora:

      .. code-block:: sh

         $ echo 'add_drivers+="vfio vfio_iommu_type1 vfio_pci"' > /etc/dracut.conf.d/vfio.conf
         $ sudo dracut --force

-  Reboot the hypervisor operating system.

[34][35]

Nvidia cards initialized in the guest with a driver version >= 337.88 can detect if the operating system is being virtualized. This can lead to a "Code: 43" error being returned by the driver and the graphics card not working. A work-a-round for this is to set a random "vendor\_id" to a alphanumeric 12 character value and forcing KVM's emulation to be hidden. This does not affect ATI/AMD graphics cards.

libvirt:

.. code-block:: sh

   $ sudo virsh edit <VIRTUAL_MACHINE_NAME>

.. code-block:: xml

   <features>
       <hyperv>
           <vendor_id state='on' value='123456abcdef'/>
       </hyperv>
       <kvm>
           <hidden state='on'/>
       </kvm>
   </features>

[13]

Xen
~~~

Xen is a free and open source software hypervisor under the GNU General
Public License (GPL). It was originally designed to be a competitor of
VMWare. It is currently owned by Citrix and offers a paid support
package for it's virtual machine hypervisor/manager XenServer. [14] By
itself it can be used as a basic hypervisor, similar to QEMU. It can
also be used with QEMU to provide accelerated hardware virtualization.

Nested Virtualization
^^^^^^^^^^^^^^^^^^^^^

Since Xen 4.4, experimental support was added for nested virtualization.
A few settings need to be added to the Xen virtual machine's file,
typically located in the "/etc/xen/" directory. Turn "nestedhvm" on for
nested virtualization support. The "hap" feature also needs to be
enabled for faster performance. Lastly, the CPU's ID needs to be
modified to hide the original virtualization ID.

::

        nestedhvm=1
        hap=1
        cpuid = ['0x1:ecx=0xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx']

[15]

Orchestration
-------------

Virtual machine provisioning can be automated through the use of
different tools.

Manual
~~~~~~

Instead of installing operating systems from scratch, a pre-built cloud virtual machine image can be used and customized for use in a non-cloud environment.

-  Find and download cloud images from `here <https://docs.openstack.org/image-guide/obtain-images.html>`__.
-  Set the root password and uninstall cloud-init: ``$ virt-customize --root-password password:<PASSWORD> --uninstall cloud-init -a <VM_IMAGE>``
-  Reset the machine-id: ``$ virt-sysprep --operations machine-id -a <VM_IMAGE>``
-  Increase the QCOW2 image size: ``$ qemu-img resize <VM_IMAGE> <SIZE>G``
-  Create a new QCOW2 image for resizing the partition: ``$ qemu-img create -f qcow2 <VM_IMAGE_NEW> <SIZE>G``
-  Resize the partition: ``$ virt-resize --expand /dev/sda1 <VM_IMAGE> <VM_IMAGE_NEW>``
-  Delete the original cloud image: ``$ rm <VM_IMAGE>``
-  Rename the new resized QCOW2 image: ``$ mv <VM_IMAGE_NEW> <VM_IMAGE>``

Anaconda
~~~~~~~~

Anaconda is an installer for the RHEL and Fedora operating systems.

Kickstart File
^^^^^^^^^^^^^^

A Kickstart file defines all of the steps necessary to install the operating system.

Common commands:

-  **authconfig** = Configure authentication using options specified in the ``authconfig`` manual.
-  autopart = Automatically create partitions.
-  **bootloader** = Define how the bootloader should be installed.
-  clearpart = Delete existing partitions.

    -  --type <TYPE> = Using one of these partition schemes: partition (partition only, no formatting), plain (normal partitions that are not Btrfs or LVM), btrfs, lvm, or thinp (thin-provisioned logical volumes).

-  {cmdline|graphical|text} = The display mode for the installer.

   -  cmdline = Non-interactive text installer.
   -  graphical = The graphical installer will be displayed.
   -  text = An interactive text installer that will prompt for missing options.

-  **eula --accept** = Automatically accept the end-user license agreement (EULA).
-  firewall = Configure the firewall.

    -  --enable
    -  --disable
    -  --port = Specify the ports to open.

-  %include = Include another file this Kickstart file.
-  **install** = Start the installer.
-  **keyboard** = Configure the keyboard layout.
-  **lang** = The primary language to use.
-  mount = Manually specify a partition to mount.
-  network = Configure the network settings.
-  %packages = A list of packages, separated by a newline, to be installed. End the list of packages by using ``%end``.
-  partition = Manually create partitions.

   -  UEFI devices need a dedicated partition for storing the EFI information. [16]

      -  part /boot/efi --fstype vfat --size=256 --ondisk=sda

-  raid = Create a software RAID.
-  repo --name="<REPO_NAME>" --baseurl="<REPO_URL>" = Add a repository.
-  **rootpw** = Change the root password.
-  selinux = Change the SELinux settings.

    -  --permissive
    -  --enforcing
    -  --disabled

-  services = Manage systemd services.

    -  --enabled=<SERVICE1>,<SERVICE2>,SERVICE3> = Enable these services.

-  sshkey = Add a SSH key to a specified user.
-  **timezone** = Configure the timezone.
-  url = Do a network installation using the specified URL to the operating system's repository.
-  user = Configure a new user.
-  vnc = Configure a VNC for remote graphical installations.
-  zerombr = Erase the partition table.

[37][38]

An example of a basic Kickstart file can be found here: https://marclop.svbtle.com/creating-an-automated-centos-7-install-via-kickstart-file.

Terraform
~~~~~~~~~

Terraform provides infrastructure automation.

Find and download the latest version of Terraform from `here <https://www.terraform.io/downloads.html>`__.

.. code-block:: sh

   $ cd ~/.local/bin/
   $ TERRAFORM_VERSION=0.12.28
   $ curl -LO https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip
   $ unzip terraform_${TERRAFORM_VERSION}_linux_amd64.zip
   $ terraform --version
   Terraform v0.12.28

Optionally install tab completion support for bash and zsh.

.. code-block:: sh

   $ terraform -install-autocomplete

[42]

Modules
^^^^^^^

A Terraform Module consists of at least a single ``main.tf`` file that defines the ``provider`` (plugin) to use and what ``resources`` to apply. In addition, ``variables.tf`` can be used to define related variables used by ``main.tf`` and a ``outputs.tf`` file can be used to define what outputs to save (such as generated SSH keys). [44]

Providers
^^^^^^^^^

Common cloud providers:

-  AWS
-  Azure
-  Cloud-init
-  DigitalOcean
-  Google Cloud Platform
-  Helm
-  Kubernetes
-  OpenStack
-  Packet
-  VMWare Cloud
-  Vultr

Database providers:

-  InfluxDB
-  MongoDB Atlas
-  MySQL
-  PostgreSQL

DNS providers:

-  DNS
-  DNSimple
-  DNSMadeEasy
-  PowerDNS
-  UltraDNS

Git providers:

-  Bitbucket
-  GitHub
-  GitLab

Logging and monitoring:

-  Auth0
-  Circonus
-  Datadog
-  Dyn
-  Grafana
-  Icinga2
-  LaunchDarkly
-  Librato
-  Logentries
-  LogicMonitor
-  New Relic
-  OpsGenie
-  PagerDuty
-  Runscope
-  SignalFx
-  StatusCake
-  Sumo Logic
-  Wavefront

Common miscellaneous providers:

-  Chef
-  Cobbler
-  Docker
-  HTTP
-  Local
-  Rundeck
-  RabbitMQ
-  Time
-  Terraform
-  TLS
-  Vault

[43]

OpenStack
'''''''''

Authentication via an existing `clouds.yaml <https://docs.openstack.org/python-openstackclient/train/configuration/index.html>`__:

::

   provider "openstack" {
      cloud = "<CLOUD>"
   }

Authentication via Terraform configuration for Keystone v3:

::

   provider "openstack" {
      project_name = "<PROJECT>"
      project_domain_name = "<PROJECT_DOMAIN_NAME>"
      user_name = "<USER>"
      user_domain_name = "<USER_DOMAIN_NAME>"
      password = "<PASSWORD>"
      auth_url = "https://<CLOUD_HOSTNAME>:5000/v3"
      region = "<REGION>"
   }

Common resources:

-  openstack_blockstorage_volume_v3
-  openstack_compute_flavor_v2
-  openstack_compute_floatingip_associate_v2
-  openstack_compute_instance_v2
-  openstack_compute_keypair_v2
-  openstack_compute_secgroup_v2
-  openstack_compute_volume_attach_v2
-  openstack_identity_project_v3
-  openstack_identity_role_v3
-  opentsack_identity_role_assignment_v3
-  openstack_identity_user_v3
-  openstack_images_image_v2
-  openstack_networking_floatingip_v2
-  openstack_networking_network_v2
-  openstack_networking_router_v2
-  openstack_networking_subnet_v2
-  openstack_lb_loadbalancer_v2
-  openstack_lb_listener_v2
-  openstack_lb_pool_v2
-  openstack_lb_member_v2
-  openstack_fw_firewall_v1
-  openstack_fw_policy_v1
-  openstack_fw_rule_v1
-  openstack_objectstorage_container_v1
-  openstack_objectstorage_object_v1
-  openstack_objectstorage_tempurl_v1
-  openstack_sharedfilesystem_securityservice_v2
-  openstack_sharedfilesystem_sharenetwork_v2
-  openstack_sharedfilesystem_share_v2
-  openstack_sharedfilesystem_access_v2

[45]

Vagrant
~~~~~~~

Vagrant is programmed in Ruby to help automate virtual machine (VM)
deployment. It uses a single file called "Vagrantfile" to describe the
virtual machines to create. By default, Vagrant will use VirtualBox as
the hypervisor but other technologies can be used.

-  Officially supported hypervisor providers [21]:

   -  docker
   -  hyperv
   -  virtualbox
   -  vmware\_desktop
   -  vmware\_fusion

-  Unofficial hypervisor providers [22]:

   -  aws
   -  azure
   -  google
   -  libvirt (KVM or Xen)
   -  lxc
   -  managed-servers (physical bare metal servers)
   -  parallels
   -  vsphere

Most unofficial hypervisor providers can be automatically installed as a
plugin from the command line.

.. code-block:: sh

    $ vagrant plugin install vagrant-<HYPERVISOR>

Vagrantfiles can be downloaded from `here <https://app.vagrantup.com/boxes/search>`__ based on the virtual machine box name.

Syntax:

.. code-block:: sh

    $ vagrant init <PROJECT>/<VM_NAME>

Example:

.. code-block:: sh

    $ vagrant init centos/7

Deploy VMs using a Vagrantfile:

.. code-block:: sh

    $ vagrant up

OR

.. code-block:: sh

    $ vagrant up --provider <HYPERVISOR>

Destroy VMs using a Vagrant file:

.. code-block:: sh

    $ vagrant destroy

The default username and password should be ``vagrant``.

This guide can be followed for creating custom Vagrant boxes:
https://www.vagrantup.com/docs/boxes/base.html.

Boxes (Images)
^^^^^^^^^^^^^^

Usage
'''''

Common Vagrant boxes to use with ``vagrant init``:

-  Arch Linux

   -  archlinux/archlinux

-  Debian

   -  debian/buster64 (Debian 10)
   -  ubuntu/focal64 (Ubuntu 20.04)

-  Fedora

   -  centos/8
   -  fedora/33-cloud-base

-  openSUSE

   -  opensuse/openSUSE-15.2-x86_64
   -  opensuse/openSUSE-Tumbleweed-x86_64

Creation
''''''''

Custom Vagrant boxes can be created from scratch and used.

-  Virtual machine setup (for an automated setup, use the `ansible_role_vagrant_box <https://github.com/ekultails/ansible_role_vagrant_box>`__ project):

   -  Create a ``vagrant`` user with password-less sudo access.

      .. code-block:: sh

         $ sudo useradd vagrant
         $ echo 'vagrant ALL=(ALL) NOPASSWD:ALL' | sudo tee /etc/sudoers.d/vagrant
         $ sudo chmod 0440 /etc/sudoers.d/vagrant

   -  Install and enable the SSH service.

      .. code-block:: sh

         # Debian
         $ sudo apt-get install openssh-server

      .. code-block:: SH

         # Fedora
         $ sudo dnf install openssh-server

   -  Add the Vagrant SSH public key.

      .. code-block:: sh

         $ sudo mkdir /home/vagrant/.ssh/
         $ sudo chmod 0700 /home/vagrant/.ssh/
         $ curl https://raw.githubusercontent.com/hashicorp/vagrant/master/keys/vagrant.pub | sudo tee -a /home/vagrant/.ssh/authorized_keys
         $ sudo chmod 0600 /home/vagrant/.ssh/authorized_keys
         $ sudo chown -R vagrant.vagrant /home/vagrant/.ssh

   -  Disable SSH password authentication.

      .. code-block:: sh

         $ sudo vi /etc/ssh/sshd_config
         PasswordAuthentication no
         PubKeyAuthentication yes

   -  Enable the SSH service.

      .. code-block:: sh

         # Debian
         $ sudo systemctl enable ssh

      .. code-block:: sh

         # Fedora
         $ sudo systemctl enable sshd

   -  Shutdown the virtual machine.

      .. code-block:: sh

         $ sudo shutdown now

-  Hypervisor steps:

   -  Create a ``metadata.json`` file with information about the virtual machine.

      ::

         {
             "provider"     : "libvirt",
             "format"       : "qcow2",
             "virtual_size" : <SIZE_IN_GB>
         }

   -  Rename the virtual machine to be ``box.img``.

      .. code-block:: sh

         $ mv <VM_IMAGE>.qcow2 box.img

   -  Create the tarball for the Vagrant-compatible box.

      .. code-block:: sh

         $ tar -c -z -f <BOX_NAME>.box ./metadata.json ./box.img

   -  Import the new box.

      .. code-block:: sh

         $ vagrant box add --name <BOX_NAME> <BOX_NAME>.box

   -  Test the new box.

      .. code-block:: sh

         $ vagrant init <BOX_NAME>
         $ vagrant up --provider=libvirt

[46]

Vagrantfile
^^^^^^^^^^^

A default Vagrantfile can be created to start customizing with.

.. code-block:: sh

    $ vagrant init

All of the settings should be defined within the ``Vagrant.configure()``
block.

.. code-block:: ruby

    Vagrant.configure("2") do |config|
        #Define VM settings here.
    end

Define the virtual machine template to use. This will be downloaded, by
default (if the ``box_url`` is not changed) from the HashiCorp website.

-  box = Required. The name of the virtual machine to download. A list
   of official virtual machines can be found at
   ``https://atlas.hashicorp.com/boxes/search``.
-  box\_version = The version of the virtual machine to use.
-  box\_url = The URL to the virtual machine details.

Example:

.. code-block:: ruby

    Vagrant.configure("2") do |config|
      config.vm.box = "ubuntu/xenial64"
      config.vm.box_version = "v20170508.0.0"
      config.vm.box_url = "https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-amd64-vagrant.box"
    end

[23]

Resource Allocation
'''''''''''''''''''

Defining the amount of resources a virtual machine has access to is different for each back-end provider. The default primary disk space is normally 40GB.

.. code-block:: ruby

   config.vm.provider "<PROVIDER>" do |vm_provider|
     vm_provider.<KEY> = <VALUE>
   end

Provider specific options:

-  libvirt [25]

   -  cpu_mode (string) = The CPU mode to use.
   -  cpus (string) = The number of vCPU cores to allocate.
   -  memory (string) = The size, in MiB, of RAM to allocate.
   -  storage (dictionary of strings) = Create additional disks.
   -  volume_cache (string) = The disk cache mode to use.

-  virtualbox [17]

   -  cpus (string) = The number of vCPU cores to allocate.
   -  customize (list of strings) = Run custom commands after the virtual machine has been created.
   -  gui (boolean) = Launch the VirtualBox GUI console.
   -  linked_clone (boolean) = Use a thin provisioned virtual machine image.
   -  memory (string) = The size, in MiB, of RAM to allocate.

-  vmware_desktop (VMware Fusion and VMware Workstation) [48]

   -  gui (boolean) = Launch the VirtualBox GUI console.
   -  memsize (string) = The size, in MiB, of RAM to allocate.
   -  numvcpus (string) = The number of vCPU cores to allocate.

The ``vmware_desktop`` provider requries a license from Vagrant. It can be used on two different computers. A new license is required when there is a new major version of the provider plugin. [49]

Networks
''''''''

Networks are either ``private`` or ``public``. ``private`` networks use
host-only networking and use network address translation (NAT) to
communicate out to the Internet. Virtual machines (VMs) can communicate
with each other but they cannot be reached from the outside world. Port
forwarding can also be configured to allow access to specific ports from
the hypervisor node. ``public`` networks allow a virtual machine to
attach to a bridge device for full connectivity with the external
network. This section covers VirtualBox networks since it is the default
virtualization provider.

With a ``private`` network, the IP address can either be a random
address assigned by DHCP or a static IP that is defined.

.. code-block:: ruby

    Vagrant.configure("2") do |config|
      config.vm.network "private_network", type: "dhcp"
    end

.. code-block:: ruby

    Vagrant.configure("2") do |config|
      config.vm.network "private_network", ip: "<IP4_OR_IP6_ADDRESS>", netmask: "<SUBNET_MASK>"
    end

The same rules apply to ``public`` networks except it uses the external
DHCP server on the network (if it exists).

.. code-block:: ruby

    Vagrant.configure("2") do |config|
      config.vm.network "public_network", use_dhcp_assigned_default_route: true
    end

When a ``public`` network is defined and no interface is given, the
end-user is prompted to pick a physical network interface device to
bridge onto for public network access. This bridge device can also be
specified manually.

.. code-block:: ruby

    Vagrant.configure("2") do |config|
      config.vm.network "public_network", bridge: "eth0: First NIC"
    end

In this example, port 2222 on the localhost (127.0.0.1) of the
hypervisor will forward to port 22 of the VM.

.. code-block:: ruby

    ...
        config.vm.network "forwarded_port", id: "ssh", guest: 22, host: 2222
    ...

[24]

libvirt
&&&&&&&

The options and syntax for public networks with the "libvirt" provider
are slightly different.

Options:

-  dev = The bridge device name.
-  mode = The libvirt mode to use. Default: ``bridge``.
-  type = The libvirt interface type. This is normally set to
   ``bridge``.
-  network\_name = The name of a network to use.
-  portgroup = The libvirt portgroup to use.
-  ovs = Instead of using a Linux bridge, use Open vSwitch instead.
   Default: ``false``.
-  trust\_guest\_rx\_filters = Enable the ``trustGuestRxFilters``
   setting. Default: ``false``.

Example:

.. code-block:: ruby

    config.vm.define "controller" do |controller|
        controller.vm.network "public_network", ip: "10.0.0.205", dev: "br0", mode: "bridge", type: "bridge"
    end

[25]

Boxes for libvirt are cached by Vagrant at: ``~/.local/share/libvirt/images/``.

Provisioning
''''''''''''

After a virtual machine (VM) has been created, additional commands can
be run to configure the guest VMs. This is referred to as
"provisioning."

-  Provisioners [26]:

   -  `ansible <https://www.vagrantup.com/docs/provisioning/ansible_intro.html>`__
      = Run a Ansible Playbook from the hypervisor node.
   -  ansible\_local = Run a Ansible Playbook from within the VM.
   -  cfengine = Use CFEngine to configure the VM.
   -  chef\_solo = Run a Chef Cookbook from inside the VM using
      ``chef-solo``.
   -  chef\_zero = Run a Chef Cookbook, but use ``chef-zero`` to emulate
      a Chef server inside of the VM.
   -  chef\_client = Use a remote Chef server to run a Cookbook inside
      the VM.
   -  chef\_apply = Run a Chef recipe with ``chef-apply``.
   -  docker = Install and configure docker inside of the VM.
   -  file = Copy files from the hypervisor to the VM. Note that the
      directory that the ``Vagrantfile`` is in will be mounted as the
      directory ``/vagrant/`` inside of the VM.
   -  puppet = Run single Puppet manifests with ``puppet apply``.
   -  puppet\_server = Run a Puppet manifest inside of the VM using an
      external Puppet server.
   -  salt = Run Salt states inside of the VM.
   -  shell = Run CLI shell commands.

Multiple Machines
'''''''''''''''''

A ``Vagrantfile`` can specify more than one virtual machine.

The recommended way to provision multiple VMs is to statically define
each individual VM to create as shown here. [27]

.. code-block:: ruby

    Vagrant.configure("2") do |config|

      config.vm.define "web" do |web|
        web.vm.box = "nginx"
      end

      config.vm.define "php" do |php|
        php.vm.box = "phpfpm"
      end

      config.vm.define "db" do |db|
        db.vm.box = "mariadb"
      end

    end

However, it is possible to use Ruby to dynamically define and create
VMs. This will work for creating the VMs but using the ``vagrant``
command to manage the VMs will not work properly [28]:

.. code-block:: ruby

    servers=[
      {
        :hostname => "web",
        :ip => "10.0.0.10",
        :box => "xenial",
        :ram => 1024,
        :cpu => 2
      },
      {
        :hostname => "db",
        :ip => "10.10.10.11",
        :box => "saucy",
        :ram => xenial,
        :cpu => 4
      }
    ]

    Vagrant.configure(2) do |config|
        servers.each do |machine|
            config.vm.define machine[:hostname] do |node|
                node.vm.box = machine[:box]
                node.vm.hostname = machine[:hostname]
                node.vm.network "private_network", ip: machine[:ip]
                node.vm.provider "virtualbox" do |vb|
                    vb.customize ["modifyvm", :id, "--memory", machine[:ram]]
                end
            end
        end
    end


GUI
---

There are many programs for managing virtualization from a graphical user interface (GUI).

Common GUIs:

-  oVirt
-  virt-manager
-  XenServer

oVirt
~~~~~

Supported operating systems: RHEL/CentOS 7

oVirt is an open-source API and GUI front-end for KVM virtualization similar to VMWare ESXi and XenServer. It is the open source upstream version of Red Hat Virtualization (RHV). It supports using network storage from NFS, Gluster, iSCSI, and other solutions.

oVirt has three components [39]:

-  oVirt Engine = The node that controls oVirt operations and monitoring.
-  Hypervisor nodes = The nodes where the virtual machines run.
-  Storage nodes = Where the operating system images and volumes of created virtual machines.

Install
^^^^^^^

Quick
'''''

All-in-One (AIO)

Minimum requirements:

-  One 1Gb network interface
-  Hardware virtualization
-  60GB free disk space in /var/tmp/ or a custom directory
-  Two fully qualified domain names (FQDNs) setup

  -  One for the oVirt Engine (that is not in use) and one already set for the hypervisor

Install the stable, development, or the master repository. [32]

-  Stable:

   .. code-block:: sh

       $ sudo yum install http://resources.ovirt.org/pub/yum-repo/ovirt-release42.rpm

-  Development:

   .. code-block:: sh

       $ sudo yum install http://resources.ovirt.org/pub/yum-repo/ovirt-release42.rpm
       $ sudo yum install http://resources.ovirt.org/pub/yum-repo/ovirt-release42-snapshot.rpm

-  Master:

   .. code-block:: sh

       $ sudo yum install http://resources.ovirt.org/pub/yum-repo/ovirt-release-master.rpm

Install the oVirt Engine dependencies.

.. code-block:: sh

    $ sudo yum install ovirt-hosted-engine-setup ovirt-engine-appliance

Setup NFS. The user "vdsm" needs full access to a NFS exported directory. The group "kvm" should have readable and executable permissions to run virtual machines from there. [31]

.. code-block:: sh

    $ sudo mkdir -p /exports/data
    $ sudo chmod 0755 /exports/data
    $ sudo vim /etc/exports
    /exports/data      *(rw)
    $ sudo systemctl restart nfs
    $ sudo groupadd kvm -g 36
    $ sudo useradd vdsm -u 36 -g 36
    $ sudo chown -R vdsm:kvm /exports/data

Run the manual Engine setup. This will prompt the end-user for different configuration options.

.. code-block:: sh

    $ sudo hosted-engine --deploy

Configure the Engine virtual machine to use static IP addressing. Enter in the address that is setup for the Engine's fully qualified domain name.

::

    How should the engine VM network be configured (DHCP, Static)[DHCP]? Static
    Please enter the IP address to be used for the engine VM []: <ENGINE_IP_ADDRESS>
    The engine VM will be configured to use <ENGINE_IP_ADDRESS>/24
    Please provide a comma-separated list (max 3) of IP addresses of domain name servers for the engine VM
    Engine VM DNS (leave it empty to skip) [127.0.0.1]: <OPTIONAL_DNS_SERVER>

If no DNS server is being used to resolve domain names, configure oVirt to use local resolution on the hypervisor and oVirt Engine via ``/etc/hosts``.

::

    Add lines for the appliance itself and for this host to /etc/hosts on the engine VM?
    Note: ensuring that this host could resolve the engine VM hostname is still up to you
    (Yes, No)[No] Yes

Define the oVirt Engine's hostname. This needs to already exist and be resolvable at least by ``/etc/hosts`` if the above option is set to ``Yes``.

::

    Please provide the FQDN for the engine you would like to use.
    This needs to match the FQDN that you will use for the engine installation within the VM.
    Note: This will be the FQDN of the VM you are now going to create,
    it should not point to the base host or to any other existing machine.
    Engine FQDN:  []: <OVIRT_ENGINE_HOSTNAME>

Specify the NFS mount options. For avoiding DNS issues, the NFS server's IP address can be used instead of the hostname.

::

    Please specify the storage you would like to use (glusterfs, iscsi, fc, nfs)[nfs]: nfs
    Please specify the nfs version you would like to use (auto, v3, v4, v4_1)[auto]: v4_1
    Please specify the full shared storage connection path to use (example: host:/path): <NFS_HOSTNAME>:/exports/data

[40]

Once the installation is complete, log into the oVirt Engine web portal at ``https://<OVIRT_ENGINE_HOSTNAME>``. Use the admin@internal account with the password that was configured during the setup. Accessing the web portal using the IP address may not work and result in this error: ``"The redirection URI for client is not registered"``. The fully qualified domain name has to be used for the link. [41]

If tasks, such as uploading an image, get stuck in the "Paused by System" state then the certificate authority (CA) needs to be imported into the end-user's web browser. Download it from the oVirt Engine by going to: ``https://<OVIRT_ENGINE_HOSTNAME>/ovirt-engine/services/pki-resource?resource=ca-certificate&format=X509-PEM-CA``. [29]

Hooks
^^^^^

Hooks can be installed on the oVirt Engine to provide additional features. After they are installed, both the ``ovirt-engine`` and ``vdsmd`` services need to be restarted.

oVirt Engine:

.. code-block:: sh

    $ sudo systemctl restart ovirt-engine

Hypervisors:

.. code-block:: sh

    $ sudo systemctl restart vdsmd

MAC Spoofing
''''''''''''

Allowing MAC spoofing on a virtual network interface card (vNIC) is required for some services such as Ironic from the OpenStack suite of software.

Install the hook and define the required virtual machine property.

.. code-block:: sh

    $ sudo yum install -y vdsm-hook-macspoof
    $ sudo engine-config -s "UserDefinedVMProperties=macspoof=(true|false)"

This will add an option to virtual machines to allow MAC spoofing. By default, it will still not be allowed.

[30]

Nested Virtualization
'''''''''''''''''''''

Install the hook.

.. code-block:: sh

    $ sudo yum install vdsm-hook-nestedvt

Nested virtualization also requires MAC spoofing to be enabled.

[30]

VMware vSphere
~~~~~~~~~~~~~~

VMware vSphere is a collection of VMware virtualization products including ESXi hypervisors, vSphere, and vCenter Server Add-on products include NSX-T, vROps, vSAN, and more. VMware Cloud Foundation = VMware vSphere with most of the add-ons included.

Terminology:

-  ESXi hypervisor = Previously Linux based, now a proprietary UNIX-like operating system. This is the base operating system and hypervisor software suite that is installed onto a node.
-  vSphere = Has two meanings. (1) The entire collection of VMware virtualization products or (2) a management dashboard for a single region of ESXi hypervisors.
-  vCenter Server = Manange and operate vSphere infrastructure such as clusters, NSX-T, DRS, vSANs, and more.
-  vSAN = Storage from each ESXi hypervisor can be pooled together in as a virtual storage area network (vSAN) device. This is a hyperconverged infrastructure.
-  vSphere cluster = A group of two or more ESXi hypervisors that typically share a common vSAN back-end.
-  NSX-T = A fork of Open vSwitch. Used for virtual networking across nodes.
-  VSS = vSphere Standard Switch. A virtual switch that is manually managed across a cluster. Each ESXi hypervisor requires a VSS to be created if VDS is not being used. This is provided for free in VMware vSphere.
-  VDS = vSphere Distributed Switch. A virtual switch that is automatically managed across a cluster by NSX-T.
-  vSwitch = A virtual switch that is either a VSS or VDS..
-  Port group = A virtual VLAN interface on a vSwitch. It can be a single VLAN or have various trunked VLANs.
-  Content library = Local virtual machines templates/images.
-  vROps = vRealize Operations. An observability tool for vSphere.
-  DRS = Distributed Resource Scheduler. Used to manage and monitor virtual machines across a vSphere cluster.
-  Predictive DRS = Requires vROps. This can predict when to reallocate virtual machines to different hypervisors based on load and usage. Moving virtual machines will happen automatically.

Troubleshooting
---------------

Errors
~~~~~~

**"Error starting domain: Requested operation is not valid: network '<LIBVIRT_NETWORK>' is not active"** when starting a libvirt virtual machine.

-  View the status of all libvirt networks: ``sudo virsh net-list --all``.
-  Start the network: ``sudo virsh net-start <LIBVIRT_NETWORK>``
-  Optionally, enable the network to start automatically when the ``libvirtd`` service starts: ``sudo virsh net-autostart <LIBVIRT_NETWORK>``

History
-------

-  `Latest <https://github.com/ekultails/rootpages/commits/master/src/virtualization/virtual_machines.rst>`__
-  `< 2019.04.01 (Virtualization) <https://github.com/ekultails/rootpages/commits/master/src/administration/virtualization.rst>`__
-  `< 2019.01.01 (Virtualization) <https://github.com/ekultails/rootpages/commits/master/src/virtualization.rst>`__
-  `< 2018.01.01 (Virtualization) <https://github.com/ekultails/rootpages/commits/master/markdown/virtualization.md>`__

Bibliography
------------

1. "libvirt Introduction." libvirt VIRTUALIZATION API. Accessed December 22, 2017. https://libvirt.org/index.html
2. "Linux: Find Out If CPU Support Intel VT and AMD-V Virtualization Support." February 11, 2015. nixCraft. Accessed December 18, 2016. https://www.cyberciti.biz/faq/linux-xen-vmware-kvm-intel-vt-amd-v-support/
3. "Intel VT (Virtualization Technology) Definition." TechTarget. October, 2009. Accessed December 18, 2016. http://searchservervirtualization.techtarget.com/definition/Intel-VT
4. "Kernel Virtual Machine." KVM. Accessed December 18, 2016. http://www.linux-kvm.org/page/Main\_Page
5. "KVM vs QEMU vs Libvirt." The Geeky Way. February 14, 2014. Accessed December 22, 2017. http://thegeekyway.com/kvm-vs-qemu-vs-libvirt/
6. "Tuning KVM." KVM. Accessed January 7, 2016. http://www.linux-kvm.org/page/Tuning\_KVM
7. "Virtio." libvirt Wiki. October 3, 2013. Accessed January 7, 2016. https://wiki.libvirt.org/page/Virtio
8. "KVM I/O slowness on RHEL 6." March 11, 2011. Accessed August 30, 2017. http://www.ilsistemista.net/index.php/virtualization/11-kvm-io-slowness-on-rhel-6.html
9. "How to Enable Nested KVM." Rhys Oxenhams' Cloud Technology Blog. June 26, 2012. Accessed December 1, 2017. http://www.rdoxenham.com/?p=275
10. "Configure DevStack with KVM-based Nested Virtualization." December 18, 2016. Accessed December 18, 2016. http://docs.openstack.org/developer/devstack/guides/devstack-with-nested-kvm.html
11. "How to enable nested virtualization in KVM." Fedora Project Wiki. June 19, 2015. Accessed August 30, 2017. https://fedoraproject.org/wiki/How\_to\_enable\_nested\_virtualization\_in\_KVM
12. "GPU Passthrough with KVM and Debian Linux." scottlinux.com Linux Blog. August 28, 2016. Accessed December 18, 2016. https://scottlinux.com/2016/08/28/gpu-passthrough-with-kvm-and-debian-linux/
13. "PCI passthrough via OVMF." Arch Linux Wiki. December 18, 2016. Accessed December 18, 2016. https://wiki.archlinux.org/index.php/PCI\_passthrough\_via\_OVMF
14. "Xen Definition." TechTarget. March, 2009. Accessed December 18, 2016. http://searchservervirtualization.techtarget.com/definition/Xen
15. "Nested Virtualization in Xen." Xen Project Wiki. November 2, 2017. Accessed December 22, 2017. https://wiki.xenproject.org/wiki/Nested\_Virtualization\_in\_Xen
16. "UEFI Kickstart failed to find a suitable stage1 device." Red Hat Discussions. October 1, 2015. Accessed October 18, 2018. https://access.redhat.com/discussions/1534853
17. "Providers VirtualBox Configuration." Vagrant Documentation. November 23, 2020. Accessed February 10, 2021. https://www.vagrantup.com/docs/virtualbox/configuration.html
18. "APIC Virtualization Performance Testing and Iozone." Intel Developer Zone Blog. December 17, 2013. Accessed September 6, 2018. https://software.intel.com/en-us/blogs/2013/12/17/apic-virtualization-performance-testing-and-iozone
19. "Intel x2APIC and APIC Virtualization (APICv or vAPIC)." Red Hat vfio-users Mailing list. June 14, 2016. Accessed September 6, 2018. https://www.redhat.com/archives/vfio-users/2016-June/msg00055.html
20. "QEMU Disk IO Which perfoms Better: Native or threads?" SlideShare. February, 2016. Accessed May 13, 2018. https://www.slideshare.net/pradeepkumarsuvce/qemu-disk-io-which-performs-better-native-or-threads
21. "Introduction to Vagrant." Vagrant Documentation. April 24, 2017. Accessed May 9, 2017. https://www.vagrantup.com/intro/getting-started/index.html
22. "Available Vagrant Plugins." mitchell/vagrant GitHub. November 9, 2016. Accessed May 8, 2017. https://github.com/mitchellh/vagrant/wiki/Available-Vagrant-Plugins
23. "[Vagrant] Boxes." Vagrant Documentation. April 24, 2017. Accessed May 9, 2017. https://www.vagrantup.com/docs/boxes.html
24. "[Vagrant] Networking." Vagrant Documentation. April 24, 2017. Accessed May 9, 2017. https://www.vagrantup.com/docs/networking/
25. "Vagrant Libvirt Provider [README]." vagrant-libvirt GitHub. May 8, 2017. Accessed October 2, 2018. https://github.com/vagrant-libvirt/vagrant-libvirt
26. "[Vagrant] Provisioning." Vagrant Documentation. April 24, 2017. Accessed May 9, 2017. https://www.vagrantup.com/docs/provisioning/
27. "[Vagrant] Multi-Machine." Vagrant Documentation. April 24, 2017. Accessed May 9, 2017. https://www.vagrantup.com/docs/multi-machine/
28. "Vagrantfile." Linux system administration and monitoring / Windows servers and CDN video. May 9, 2017. Accessed May 9, 2017. http://sysadm.pp.ua/linux/sistemy-virtualizacii/vagrantfile.html
29. "RHV 4 Upload Image tasks end in Paused by System state." Red Hat Customer Portal. April 11, 2017. Accessed March 26, 2018. https://access.redhat.com/solutions/2592941
30. "Testing oVirt 3.3 with Nested KVM." Red Hat Open Source Community. August 15, 2013. Accessed March 29, 2018. https://community.redhat.com/blog/2013/08/testing-ovirt-3-3-with-nested-kvm/
31. "Storage." oVirt Documentation. Accessed March 20, 2018. https://www.ovirt.org/documentation/admin-guide/chap-Storage/
32. "Install nightly snapshot." oVirt Documentation. Accessed March 21, 2018. https://www.ovirt.org/develop/dev-process/install-nightly-snapshot/
33. "Guide: How to Enable Huge Pages to improve VFIO KVM Performance in Fedora 25." Gaming on Linux with VFIO. August 20, 2017. Accessed March 23, 2018. http://vfiogaming.blogspot.com/2017/08/guide-how-to-enable-huge-pages-to.html
34. "PCI passthrough via OVMF." Arch Linux Wiki. February 13, 2018. Accessed February 26, 2018. https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF
35. "RYZEN GPU PASSTHROUGH SETUP GUIDE: FEDORA 26 + WINDOWS GAMING ON LINUX." Level One Techs. June 25, 2017. Accessed February 26, 2018. https://level1techs.com/article/ryzen-gpu-passthrough-setup-guide-fedora-26-windows-gaming-linux
36. "IOMMU Groups – What You Need to Consider." Heiko's Blog. July 25, 2017. Accessed March 3, 2018. https://heiko-sieger.info/iommu-groups-what-you-need-to-consider/
37. "Kickstart Documentation." Pykickstart. Accessed March 15, 2018. http://pykickstart.readthedocs.io/en/latest/kickstart-docs.html
38. "Creating an automated CentOS 7 Install via Kickstart file." Marc Lopez Personal Blog. December 1, 2014. Accessed March 15, 2018. https://marclop.svbtle.com/creating-an-automated-centos-7-install-via-kickstart-file
39. "oVirt Architecture." oVirt Documentation. Accessed March 20, 2018. https://www.ovirt.org/documentation/architecture/architecture/
40. "Deploying Self-Hosted Engine." oVirt Documentation. Accessed March 20, 2018. https://www.ovirt.org/documentation/self-hosted/chap-Deploying_Self-Hosted_Engine/
41. "[ovirt-users] Fresh install - unable to web gui login." oVirt Users Mailing List. January 11, 2018. Accessed March 26, 2018. http://lists.ovirt.org/pipermail/users/2018-January/086223.html
42. "Install Terraform." HashiCorp Learn. Accessed July 8, 2020.https://learn.hashicorp.com/terraform/getting-started/install
43. "Providers." Terraform CLI. Accessed July 8, 2020. https://www.terraform.io/docs/providers/index.html
44. "Create a Terraform Module." Linode Guides & Tutorials. May 1, 2020. Accessed July 8, 2020. https://www.linode.com/docs/applications/configuration-management/terraform/create-terraform-module/
45. "OpenStack Provider." Terraform Docs. Accessed July 18, 2020. https://www.terraform.io/docs/providers/openstack/index.html
46. "How to create a vagrant VM from a libvirt vm/image." openATTIC. January 11, 2018. Accessed October 19, 2020. https://www.openattic.org/posts/how-to-create-a-vagrant-vm-from-a-libvirt-vmimage/
47. "Qemu/KVM Virtual Machines." Proxmox VE Wiki. November 26, 2020. Accessed January 21, 2021. https://pve.proxmox.com/wiki/Qemu/KVM_Virtual_Machines
48. "Providers VMware Configuration." Vagrant Documentation. November 23, 2020. Accessed February 10, 2021. https://www.vagrantup.com/docs/providers/vmware/configuration
49. "VMware Integration." Vagrant by HashiCorp. Accessed February 10, 2021. https://www.vagrantup.com/vmware
50. "KVM Virtualization: Start VNC Remote Access For Guest Operating Systems." nixCraft. May 6, 2017. Accessed February 18, 2021. https://www.cyberciti.biz/faq/linux-kvm-vnc-for-guest-machine/
51. "CHAPTER 11. MANAGING STORAGE FOR VIRTUAL MACHINES." Red Hat Customer Portal. Accessed February 25, 2021. https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_virtualization/managing-storage-for-virtual-machines_configuring-and-managing-virtualization#understanding-virtual-machine-storage_managing-storage-for-virtual-machines
