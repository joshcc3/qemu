.. _Deprecated features:

Deprecated features
===================

In general features are intended to be supported indefinitely once
introduced into QEMU. In the event that a feature needs to be removed,
it will be listed in this section. The feature will remain functional for the
release in which it was deprecated and one further release. After these two
releases, the feature is liable to be removed. Deprecated features may also
generate warnings on the console when QEMU starts up, or if activated via a
monitor command, however, this is not a mandatory requirement.

Prior to the 2.10.0 release there was no official policy on how
long features would be deprecated prior to their removal, nor
any documented list of which features were deprecated. Thus
any features deprecated prior to 2.10.0 will be treated as if
they were first deprecated in the 2.10.0 release.

What follows is a list of all features currently marked as
deprecated.

System emulator command line arguments
--------------------------------------

Short-form boolean options (since 6.0)
''''''''''''''''''''''''''''''''''''''

Boolean options such as ``share=on``/``share=off`` could be written
in short form as ``share`` and ``noshare``.  This is now deprecated
and will cause a warning.

``delay`` option for socket character devices (since 6.0)
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''

The replacement for the ``nodelay`` short-form boolean option is ``nodelay=on``
rather than ``delay=off``.

``-smp`` ("parameter=0" SMP configurations) (since 6.2)
'''''''''''''''''''''''''''''''''''''''''''''''''''''''

Specified CPU topology parameters must be greater than zero.

In the SMP configuration, users should either provide a CPU topology
parameter with a reasonable value (greater than zero) or just omit it
and QEMU will compute the missing value.

However, historically it was implicitly allowed for users to provide
a parameter with zero value, which is meaningless and could also possibly
cause unexpected results in the -smp parsing. So support for this kind of
configurations (e.g. -smp 8,sockets=0) is deprecated since 6.2 and will
be removed in the near future, users have to ensure that all the topology
members described with -smp are greater than zero.

Plugin argument passing through ``arg=<string>`` (since 6.1)
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Passing TCG plugins arguments through ``arg=`` is redundant is makes the
command-line less readable, especially when the argument itself consist of a
name and a value, e.g. ``-plugin plugin_name,arg="arg_name=arg_value"``.
Therefore, the usage of ``arg`` is redundant. Single-word arguments are treated
as short-form boolean values, and passed to plugins as ``arg_name=on``.
However, short-form booleans are deprecated and full explicit ``arg_name=on``
form is preferred.

``-no-hpet`` (since 8.0)
''''''''''''''''''''''''

The HPET setting has been turned into a machine property.
Use ``-machine hpet=off`` instead.

``-no-acpi`` (since 8.0)
''''''''''''''''''''''''

The ``-no-acpi`` setting has been turned into a machine property.
Use ``-machine acpi=off`` instead.

``-async-teardown`` (since 8.1)
'''''''''''''''''''''''''''''''

Use ``-run-with async-teardown=on`` instead.

``-chroot`` (since 8.1)
'''''''''''''''''''''''

Use ``-run-with chroot=dir`` instead.

``-singlestep`` (since 8.1)
'''''''''''''''''''''''''''

The ``-singlestep`` option has been turned into an accelerator property,
and given a name that better reflects what it actually does.
Use ``-accel tcg,one-insn-per-tb=on`` instead.

User-mode emulator command line arguments
-----------------------------------------

``-singlestep`` (since 8.1)
'''''''''''''''''''''''''''

The ``-singlestep`` option has been given a name that better reflects
what it actually does. For both linux-user and bsd-user, use the
new ``-one-insn-per-tb`` option instead.

QEMU Machine Protocol (QMP) commands
------------------------------------

``blockdev-open-tray``, ``blockdev-close-tray`` argument ``device`` (since 2.8)
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Use argument ``id`` instead.

``eject`` argument ``device`` (since 2.8)
'''''''''''''''''''''''''''''''''''''''''

Use argument ``id`` instead.

``blockdev-change-medium`` argument ``device`` (since 2.8)
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Use argument ``id`` instead.

``block_set_io_throttle`` argument ``device`` (since 2.8)
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Use argument ``id`` instead.

``blockdev-add`` empty string argument ``backing`` (since 2.10)
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Use argument value ``null`` instead.

``block-commit`` arguments ``base`` and ``top`` (since 3.1)
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Use arguments ``base-node`` and ``top-node`` instead.

``nbd-server-add`` and ``nbd-server-remove`` (since 5.2)
''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Use the more generic commands ``block-export-add`` and ``block-export-del``
instead.  As part of this deprecation, where ``nbd-server-add`` used a
single ``bitmap``, the new ``block-export-add`` uses a list of ``bitmaps``.

``query-qmp-schema`` return value member ``values`` (since 6.2)
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Member ``values`` in return value elements with meta-type ``enum`` is
deprecated.  Use ``members`` instead.

``drive-backup`` (since 6.2)
''''''''''''''''''''''''''''

Use ``blockdev-backup`` in combination with ``blockdev-add`` instead.
This change primarily separates the creation/opening process of the backup
target with explicit, separate steps. ``blockdev-backup`` uses mostly the
same arguments as ``drive-backup``, except the ``format`` and ``mode``
options are removed in favor of using explicit ``blockdev-create`` and
``blockdev-add`` calls. See :doc:`/interop/live-block-operations` for
details.

Incorrectly typed ``device_add`` arguments (since 6.2)
''''''''''''''''''''''''''''''''''''''''''''''''''''''

Due to shortcomings in the internal implementation of ``device_add``, QEMU
incorrectly accepts certain invalid arguments: Any object or list arguments are
silently ignored. Other argument types are not checked, but an implicit
conversion happens, so that e.g. string values can be assigned to integer
device properties or vice versa.

This is a bug in QEMU that will be fixed in the future so that previously
accepted incorrect commands will return an error. Users should make sure that
all arguments passed to ``device_add`` are consistent with the documented
property types.

``StatusInfo`` member ``singlestep`` (since 8.1)
''''''''''''''''''''''''''''''''''''''''''''''''

The ``singlestep`` member of the ``StatusInfo`` returned from the
``query-status`` command is deprecated. This member has a confusing
name and it never did what the documentation claimed or what its name
suggests. We do not believe that anybody is actually using the
information provided in this member.

The information it reports is whether the TCG JIT is in "one
instruction per translated block" mode (which can be set on the
command line or via the HMP, but not via QMP). The information remains
available via the HMP 'info jit' command.

QEMU Machine Protocol (QMP) events
----------------------------------

``MEM_UNPLUG_ERROR`` (since 6.2)
''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Use the more generic event ``DEVICE_UNPLUG_GUEST_ERROR`` instead.

``vcpu`` trace events (since 8.1)
'''''''''''''''''''''''''''''''''

The ability to instrument QEMU helper functions with vCPU-aware trace
points was removed in 7.0. However QMP still exposed the vcpu
parameter. This argument has now been deprecated and the remaining
remaining trace points that used it are selected just by name.

Human Monitor Protocol (HMP) commands
-------------------------------------

``singlestep`` (since 8.1)
''''''''''''''''''''''''''

The ``singlestep`` command has been replaced by the ``one-insn-per-tb``
command, which has the same behaviour but a less misleading name.

Host Architectures
------------------

BE MIPS (since 7.2)
'''''''''''''''''''

As Debian 10 ("Buster") moved into LTS the big endian 32 bit version of
MIPS moved out of support making it hard to maintain our
cross-compilation CI tests of the architecture. As we no longer have
CI coverage support may bitrot away before the deprecation process
completes. The little endian variants of MIPS (both 32 and 64 bit) are
still a supported host architecture.

System emulation on 32-bit x86 hosts (since 8.0)
''''''''''''''''''''''''''''''''''''''''''''''''

Support for 32-bit x86 host deployments is increasingly uncommon in mainstream
OS distributions given the widespread availability of 64-bit x86 hardware.
The QEMU project no longer considers 32-bit x86 support for system emulation to
be an effective use of its limited resources, and thus intends to discontinue
it. Since all recent x86 hardware from the past >10 years is capable of the
64-bit x86 extensions, a corresponding 64-bit OS should be used instead.


System emulator machines
------------------------

Arm ``virt`` machine ``dtb-kaslr-seed`` property (since 7.1)
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

The ``dtb-kaslr-seed`` property on the ``virt`` board has been
deprecated; use the new name ``dtb-randomness`` instead. The new name
better reflects the way this property affects all random data within
the device tree blob, not just the ``kaslr-seed`` node.

``pc-i440fx-2.0`` up to ``pc-i440fx-2.3`` (since 8.2)
'''''''''''''''''''''''''''''''''''''''''''''''''''''

These old machine types are quite neglected nowadays and thus might have
various pitfalls with regards to live migration. Use a newer machine type
instead.


Backend options
---------------

Using non-persistent backing file with pmem=on (since 6.1)
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

This option is used when ``memory-backend-file`` is consumed by emulated NVDIMM
device. However enabling ``memory-backend-file.pmem`` option, when backing file
is (a) not DAX capable or (b) not on a filesystem that support direct mapping
of persistent memory, is not safe and may lead to data loss or corruption in case
of host crash.
Options are:

    - modify VM configuration to set ``pmem=off`` to continue using fake NVDIMM
      (without persistence guaranties) with backing file on non DAX storage
    - move backing file to NVDIMM storage and keep ``pmem=on``
      (to have NVDIMM with persistence guaranties).

Device options
--------------

Emulated device options
'''''''''''''''''''''''

``-device virtio-blk,scsi=on|off`` (since 5.0)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The virtio-blk SCSI passthrough feature is a legacy VIRTIO feature.  VIRTIO 1.0
and later do not support it because the virtio-scsi device was introduced for
full SCSI support.  Use virtio-scsi instead when SCSI passthrough is required.

Note this also applies to ``-device virtio-blk-pci,scsi=on|off``, which is an
alias.

``-device nvme-ns,eui64-default=on|off`` (since 7.1)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In QEMU versions 6.1, 6.2 and 7.0, the ``nvme-ns`` generates an EUI-64
identifier that is not globally unique. If an EUI-64 identifier is required, the
user must set it explicitly using the ``nvme-ns`` device parameter ``eui64``.

``-device nvme,use-intel-id=on|off`` (since 7.1)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``nvme`` device originally used a PCI Vendor/Device Identifier combination
from Intel that was not properly allocated. Since version 5.2, the controller
has used a properly allocated identifier. Deprecate the ``use-intel-id``
machine compatibility parameter.

``-device cxl-type3,memdev=xxxx`` (since 8.0)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``cxl-type3`` device initially only used a single memory backend.  With
the addition of volatile memory support, it is now necessary to distinguish
between persistent and volatile memory backends.  As such, memdev is deprecated
in favor of persistent-memdev.

``-fsdev proxy`` and ``-virtfs proxy`` (since 8.1)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The 9p ``proxy`` filesystem backend driver has been deprecated and will be
removed (along with its proxy helper daemon) in a future version of QEMU. Please
use ``-fsdev local`` or ``-virtfs local`` for using the 9p ``local`` filesystem
backend, or alternatively consider deploying virtiofsd instead.

The 9p ``proxy`` backend was originally developed as an alternative to the 9p
``local`` backend. The idea was to enhance security by dispatching actual low
level filesystem operations from 9p server (QEMU process) over to a separate
process (the virtfs-proxy-helper binary). However this alternative never gained
momentum. The proxy backend is much slower than the local backend, hasn't seen
any development in years, and showed to be less secure, especially due to the
fact that its helper daemon must be run as root, whereas with the local backend
QEMU is typically run as unprivileged user and allows to tighten behaviour by
mapping permissions et al by using its 'mapped' security model option.

Nowadays it would make sense to reimplement the ``proxy`` backend by using
QEMU's ``vhost`` feature, which would eliminate the high latency costs under
which the 9p ``proxy`` backend currently suffers. However as of to date nobody
has indicated plans for such kind of reimplementation unfortunately.

RISC-V 'any' CPU type ``-cpu any`` (since 8.2)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The 'any' CPU type was introduced back in 2018 and has been around since the
initial RISC-V QEMU port. Its usage has always been unclear: users don't know
what to expect from a CPU called 'any', and in fact the CPU does not do anything
special that isn't already done by the default CPUs rv32/rv64.

After the introduction of the 'max' CPU type, RISC-V now has a good coverage
of generic CPUs: rv32 and rv64 as default CPUs and 'max' as a feature complete
CPU for both 32 and 64 bit builds. Users are then discouraged to use the 'any'
CPU type starting in 8.2.

RISC-V CPU properties which start with capital 'Z' (since 8.2)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

All RISC-V CPU properties which start with capital 'Z' are being deprecated
starting in 8.2. The reason is that they were wrongly added with capital 'Z'
in the past. CPU properties were later added with lower-case names, which
is the format we want to use from now on.

Users which try to use these deprecated properties will receive a warning
recommending to switch to their stable counterparts:

- "Zifencei" should be replaced with "zifencei"
- "Zicsr" should be replaced with "zicsr"
- "Zihintntl" should be replaced with "zihintntl"
- "Zihintpause" should be replaced with "zihintpause"
- "Zawrs" should be replaced with "zawrs"
- "Zfa" should be replaced with "zfa"
- "Zfh" should be replaced with "zfh"
- "Zfhmin" should be replaced with "zfhmin"
- "Zve32f" should be replaced with "zve32f"
- "Zve64f" should be replaced with "zve64f"
- "Zve64d" should be replaced with "zve64d"

``-device pvrdma`` and the rdma subsystem (since 8.2)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The pvrdma device and the whole rdma subsystem are in a bad shape and
without active maintenance. The QEMU project intends to remove this
device and subsystem from the code base in a future release without
replacement unless somebody steps up and improves the situation.


Block device options
''''''''''''''''''''

``"backing": ""`` (since 2.12)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In order to prevent QEMU from automatically opening an image's backing
chain, use ``"backing": null`` instead.

``rbd`` keyvalue pair encoded filenames: ``""`` (since 3.1)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Options for ``rbd`` should be specified according to its runtime options,
like other block drivers.  Legacy parsing of keyvalue pair encoded
filenames is useful to open images with the old format for backing files;
These image files should be updated to use the current format.

Example of legacy encoding::

  json:{"file.driver":"rbd", "file.filename":"rbd:rbd/name"}

The above, converted to the current supported format::

  json:{"file.driver":"rbd", "file.pool":"rbd", "file.image":"name"}

``iscsi,password=xxx`` (since 8.0)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Specifying the iSCSI password in plain text on the command line using the
``password`` option is insecure. The ``password-secret`` option should be
used instead, to refer to a ``--object secret...`` instance that provides
a password via a file, or encrypted.

Backwards compatibility
-----------------------

Runnability guarantee of CPU models (since 4.1)
'''''''''''''''''''''''''''''''''''''''''''''''

Previous versions of QEMU never changed existing CPU models in
ways that introduced additional host software or hardware
requirements to the VM.  This allowed management software to
safely change the machine type of an existing VM without
introducing new requirements ("runnability guarantee").  This
prevented CPU models from being updated to include CPU
vulnerability mitigations, leaving guests vulnerable in the
default configuration.

The CPU model runnability guarantee won't apply anymore to
existing CPU models.  Management software that needs runnability
guarantees must resolve the CPU model aliases using the
``alias-of`` field returned by the ``query-cpu-definitions`` QMP
command.

While those guarantees are kept, the return value of
``query-cpu-definitions`` will have existing CPU model aliases
point to a version that doesn't break runnability guarantees
(specifically, version 1 of those CPU models).  In future QEMU
versions, aliases will point to newer CPU model versions
depending on the machine type, so management software must
resolve CPU model aliases before starting a virtual machine.

QEMU guest agent
----------------

``--blacklist`` command line option (since 7.2)
'''''''''''''''''''''''''''''''''''''''''''''''

``--blacklist`` has been replaced by ``--block-rpcs`` (which is a better
wording for what this option does). The short form ``-b`` still stays
the same and thus is the preferred way for scripts that should run with
both, older and future versions of QEMU.

``blacklist`` config file option (since 7.2)
''''''''''''''''''''''''''''''''''''''''''''

The ``blacklist`` config file option has been renamed to ``block-rpcs``
(to be in sync with the renaming of the corresponding command line
option).

Migration
---------

``skipped`` MigrationStats field (since 8.1)
''''''''''''''''''''''''''''''''''''''''''''

``skipped`` field in Migration stats has been deprecated.  It hasn't
been used for more than 10 years.

